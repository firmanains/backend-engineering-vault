---
title: Case - The Just Add An Index Request
type: case
level: intermediate
domain: architecture
status: unread
difficulty: 3
est_minutes: 13
source: synthesized
concepts: ["[[../40 Databases/Composite Indexes and the Leftmost-Prefix Rule|Composite Indexes and the Leftmost-Prefix Rule]]", "[[../40 Databases/Covering Indexes|Covering Indexes]]", "[[../40 Databases/Reading EXPLAIN|Reading EXPLAIN]]"]
prerequisites: []
next: []
tags: [backend, architecture, databases, performance, gov-context]
created: 2026-08-02
---

## The Situation

Sebuah query pencarian kasus berdasarkan kombinasi status, instansi, dan rentang tanggal terasa lambat — beberapa detik untuk kembali, mengganggu pengalaman petugas yang menggunakannya berkali-kali sehari. Seorang developer, tanpa banyak analisis, langsung mengusulkan "tambah index saja" pada ketiga kolom yang dipakai filter. Index ditambahkan, query pencarian jadi cepat — tapi seminggu kemudian, tim menyadari operasi tulis (insert dan update kasus, yang jauh lebih sering terjadi dibanding pencarian ini) menjadi terasa lebih lambat, dan ukuran database membengkak signifikan.

## What Was Actually Asked For

Permintaan awal: "percepat pencarian ini." Solusi "tambah index saja" menjawab permintaan literal itu, tapi tidak mempertimbangkan **biaya** yang datang bersamanya — index bukan perbaikan gratis, ia trade-off eksplisit: mempercepat baca dengan mengorbankan kecepatan tulis dan ruang penyimpanan. Kebutuhan sebenarnya adalah index yang **tepat** — cukup untuk mempercepat pencarian yang benar-benar sering dipakai, tanpa index berlebih yang membebani tulisan tanpa manfaat proporsional.

## Options

### Option 1 — Tambah index pada setiap kolom yang dipakai filter secara terpisah

Pendekatan naif yang sering jadi respons pertama: index individual pada `status`, `instansi_id`, dan `tanggal` masing-masing. Terdengar masuk akal secara intuitif, tapi database (tergantung optimizer-nya) sering hanya bisa memakai satu index secara efisien per query, atau butuh menggabungkan beberapa index dengan cara yang tidak seefisien satu composite index yang dirancang tepat — dan tiga index terpisah berarti tiga kali beban tulis tambahan, bukan satu.

### Option 2 — Composite index dengan urutan kolom yang tepat

Buat satu index gabungan pada `(instansi_id, status, tanggal)`, dengan urutan kolom dipilih berdasarkan [[../40 Databases/Composite Indexes and the Leftmost-Prefix Rule|Composite Indexes and the Leftmost-Prefix Rule]] — kolom dengan selektivitas tinggi (instansi_id, karena banyak instansi berbeda) di depan, memungkinkan database mempersempit pencarian secepat mungkin. Satu index yang melayani kombinasi filter yang benar-benar dipakai query ini, jauh lebih efisien dibanding tiga index terpisah — tapi hanya efektif kalau urutan kolom benar-benar cocok dengan pola query nyata.

### Option 3 — Partial index untuk subset data yang relevan

Kalau pencarian ini hanya pernah dilakukan untuk kasus dengan status tertentu (misalnya "aktif", bukan kasus yang sudah "selesai" dan jarang dicari lagi), buat partial index yang hanya mencakup baris dengan kondisi itu — index jauh lebih kecil (hanya mencakup subset data yang relevan), lebih murah dipelihara saat tulis, dan tetap sangat cepat untuk query yang memang hanya menyentuh subset itu.

## The Decision

Option 2 dipilih sebagai solusi utama, dengan Option 3 dipertimbangkan sebagai optimasi tambahan setelah dikonfirmasi lewat data nyata bahwa mayoritas pencarian memang untuk kasus berstatus aktif. Sebelum memutuskan urutan kolom composite index, tim menjalankan `EXPLAIN` (lihat [[../40 Databases/Reading EXPLAIN|Reading EXPLAIN]]) pada query nyata dengan data volume mendekati production untuk memverifikasi index yang dirancang benar-benar dipakai optimizer dan menghasilkan rencana eksekusi yang diharapkan — bukan asumsi tanpa verifikasi. Constraint penentunya: index individual (Option 1) yang sudah terlanjur dibuat sebelumnya adalah biaya yang tidak sepadan (tiga index yang masing-masing membebani tulis, tanpa manfaat penuh satu composite index yang efisien) — index lama itu dihapus setelah composite index terbukti bekerja.

## What Breaks at 10x and 100x

Pada volume 10x (data kasus bertambah 10 kali lipat), composite index yang dirancang tepat tetap efisien secara logaritmik (struktur B+Tree yang mendasarinya, lihat [[../40 Databases/B+Tree Structure|B+Tree Structure]]), tapi ukuran index itu sendiri ikut membesar proporsional — pemantauan ukuran index dan dampaknya ke waktu backup/restore jadi perhatian operasional baru. Pada 100x, kalau pola pencarian terus berkembang (filter kombinasi baru yang tidak tercakup index yang ada), pertimbangan bergeser ke apakah kebutuhan pencarian sudah cukup kompleks untuk dipindah ke mesin pencarian khusus (Elasticsearch), alih-alih terus menambah index baru di database transaksional untuk setiap pola pencarian baru.

## What You Would Do Differently

Selalu menjalankan `EXPLAIN` sebelum dan sesudah menambah index apa pun, bukan mengasumsikan index otomatis dipakai dan otomatis membantu — verifikasi ini seharusnya jadi kebiasaan standar tim, bukan langkah opsional yang sering dilewati saat terburu-buru mengejar perbaikan cepat.

## Concepts Exercised

- [[../40 Databases/Composite Indexes and the Leftmost-Prefix Rule|Composite Indexes and the Leftmost-Prefix Rule]] — prinsip inti yang menentukan desain solusi akhir.
- [[../40 Databases/Covering Indexes|Covering Indexes]] — pertimbangan lanjutan untuk optimasi index yang bahkan lebih jauh, mencakup seluruh kolom yang dibutuhkan query.
- [[../40 Databases/Reading EXPLAIN|Reading EXPLAIN]] — alat verifikasi wajib sebelum dan sesudah perubahan index diterapkan.

## Your Turn

Setelah composite index `(instansi_id, status, tanggal)` diterapkan, seorang developer lain ingin menambah query baru yang mencari kasus **hanya** berdasarkan `status` saja (tanpa `instansi_id`). Apakah index yang sudah ada bisa membantu query baru ini?

> [!success]- Kunci jawaban
> Tidak — sesuai leftmost-prefix rule, composite index `(instansi_id, status, tanggal)` hanya bisa dipakai efisien untuk query yang memfilter berdasarkan `instansi_id` di posisi terdepan (baik sendiri, atau bersama kolom setelahnya secara berurutan). Query yang melewati `instansi_id` dan langsung memfilter `status` saja tidak bisa memanfaatkan index ini secara efisien — database kemungkinan besar akan melakukan full table scan atau index scan yang tidak optimal. Kalau pola query baru ini memang akan sering dipakai, dibutuhkan index terpisah yang dimulai dari `status` (atau composite index baru dengan `status` di posisi depan) — tapi sebelum menambahkannya, kembali ke prinsip dasar kasus ini: verifikasi dulu dengan `EXPLAIN` seberapa sering query ini benar-benar dipakai, dan apakah biaya index tambahan (lagi-lagi memperlambat tulis) sepadan dengan manfaatnya.

## Catatan Saya

*Kalau kamu punya kejadian nyata dari kerjaan yang mirip tema ini, tulis sebagai case study baru dengan `source: personal`.*
