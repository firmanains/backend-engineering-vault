---
title: Case - Slow Query vs Cache vs Read Replica
type: case
level: intermediate
domain: architecture
status: unread
difficulty: 3
est_minutes: 15
source: synthesized
concepts: ["[[../40 Databases/Reading EXPLAIN|Reading EXPLAIN]]", "[[../50 Concurrency and Performance/Cache-Aside, Write-Through, and Write-Behind|Cache-Aside, Write-Through, and Write-Behind]]", "[[../40 Databases/Read Replicas and Replication Lag|Read Replicas and Replication Lag]]"]
prerequisites: []
next: []
tags: [backend, architecture, databases, performance, gov-context]
created: 2026-08-02
---

## The Situation

Dashboard ringkasan kasus per instansi terasa lambat — 3-4 detik untuk memuat, mengganggu pengalaman supervisor yang membukanya berkali-kali sehari. Tim diminta memperbaikinya, dan tiga solusi berbeda diusulkan oleh tiga orang berbeda dalam rapat: perbaiki query-nya, tambahkan cache, atau tambahkan read replica. Ketiganya secara teknis valid — pertanyaannya bukan mana yang "benar", tapi mana yang paling sesuai dengan akar masalah dan biaya operasional yang realistis untuk tim ini.

## What Was Actually Asked For

Permintaan awal: "dashboard-nya lambat, perbaiki." Sebelum memilih solusi, tim perlu tahu **kenapa** lambat — ketiga solusi ini menjawab penyebab yang berbeda: query lambat menjawab masalah query itu sendiri tidak efisien; cache menjawab masalah data yang sama diminta berulang kali tanpa perlu dihitung ulang setiap saat; read replica menjawab masalah beban baca yang mengganggu beban tulis di database yang sama. Tanpa diagnosis dulu, memilih salah satu dari ketiganya secara acak berisiko menyelesaikan masalah yang salah.

## Options

### Option 1 — Optimasi query (index, restrukturisasi query)

Jalankan `EXPLAIN` (lihat [[../40 Databases/Reading EXPLAIN|Reading EXPLAIN]]) pada query dashboard — kalau ditemukan full table scan yang seharusnya bisa dihindari dengan index yang tepat, atau join yang bisa disederhanakan, perbaikan di sini adalah solusi termurah dan paling struktural: menyelesaikan akar masalah tanpa infrastruktur tambahan. Tapi kalau query sudah cukup optimal dan lambat semata-mata karena volume data yang memang besar, optimasi query saja tidak akan cukup.

### Option 2 — Cache hasil dashboard

Simpan hasil perhitungan dashboard di Redis dengan TTL beberapa menit (lihat [[../50 Concurrency and Performance/Cache-Aside, Write-Through, and Write-Behind|Cache-Aside, Write-Through, and Write-Behind]]) — permintaan berikutnya dalam jendela TTL itu dilayani dari cache, jauh lebih cepat dari menghitung ulang setiap kali. Solusi murah dan cepat diterapkan, tapi hanya menyembunyikan biaya komputasi (masih ada satu permintaan yang membayar biaya penuh setiap kali cache kedaluwarsa), dan menerima trade-off data yang sedikit tertinggal (eventual consistency).

### Option 3 — Read replica untuk beban baca dashboard

Arahkan query dashboard ke [[../40 Databases/Read Replicas and Replication Lag|Read Replicas and Replication Lag]], memisahkan beban baca berat ini dari database utama yang menangani transaksi tulis petugas. Menyelesaikan masalah kalau akar masalahnya adalah kontensi resource antara beban baca dan tulis (bukan query yang lambat itu sendiri) — tapi butuh infrastruktur tambahan yang harus dioperasikan dan dipantau.

## The Decision

Diagnosis dengan `EXPLAIN` dijalankan lebih dulu (langkah wajib sebelum memilih solusi apa pun) — hasilnya menunjukkan query dashboard sudah cukup efisien secara struktur (index yang tepat sudah terpakai), tapi memang menghitung agregat dari data yang cukup besar setiap kali dipanggil, dan sering dipanggil berulang oleh supervisor yang sama dalam waktu berdekatan. Berdasarkan diagnosis ini, Option 2 (cache) dipilih sebagai solusi utama — akar masalahnya adalah **permintaan berulang untuk data yang sama**, bukan query yang secara inheren tidak efisien (menyingkirkan Option 1 sebagai solusi utama) atau kontensi resource read-write (menyingkirkan Option 3, karena beban tulis di database utama masih dalam batas wajar). Constraint penentunya: solusi yang dipilih harus menjawab **akar masalah yang benar-benar teridentifikasi**, bukan solusi yang "terdengar benar" tanpa diagnosis dulu.

## What Breaks at 10x and 100x

Pada volume 10x (lebih banyak instansi, lebih banyak supervisor mengakses dashboard berbeda), cache per-dashboard tetap bekerja baik selama masing-masing dashboard punya key cache yang terpisah jelas — tapi total memori Redis yang dibutuhkan tumbuh proporsional, perlu dipantau. Pada 100x, kalau kebutuhan dashboard makin kompleks (agregasi lintas periode panjang, bukan sekadar ringkasan harian), cache saja tidak lagi cukup menutupi biaya komputasi — di titik ini, memisahkan beban analitik ke database khusus (ClickHouse, lihat [[../40 Databases/OLTP vs OLAP vs HTAP|OLTP vs OLAP vs HTAP]]) jadi investasi yang lebih sepadan dibanding terus mengandalkan cache di atas query yang makin berat.

## What You Would Do Differently

Menjadikan `EXPLAIN` sebagai langkah wajib pertama sebelum rapat mengusulkan solusi — rapat yang dimulai dengan tiga solusi berbeda tanpa diagnosis bersama sering berakhir sebagai perdebatan preferensi, bukan keputusan berbasis data; diagnosis yang dilakukan lebih dulu akan langsung mengarahkan diskusi ke solusi yang paling relevan tanpa perlu memperdebatkan ketiganya secara paralel.

## Concepts Exercised

- [[../40 Databases/Reading EXPLAIN|Reading EXPLAIN]] — langkah diagnosis wajib yang menentukan solusi mana yang benar-benar relevan.
- [[../50 Concurrency and Performance/Cache-Aside, Write-Through, and Write-Behind|Cache-Aside, Write-Through, and Write-Behind]] — solusi yang dipilih berdasarkan diagnosis akar masalah.
- [[../40 Databases/Read Replicas and Replication Lag|Read Replicas and Replication Lag]] — alternatif yang dipertimbangkan tapi tidak dipilih karena tidak menjawab akar masalah yang teridentifikasi.

## Your Turn

Enam bulan setelah cache diterapkan, dashboard kembali terasa lambat — kali ini bukan setiap kali dipanggil, tapi khusus di jam-jam tertentu (awal bulan, saat banyak laporan bulanan dibuat bersamaan). Apakah ini berarti cache gagal, dan solusi mana yang paling relevan sekarang?

> [!success]- Kunci jawaban
> Cache tidak "gagal" — gejala baru ini menunjukkan pola beban yang berbeda dari yang dulu diselesaikan. Lonjakan spesifik di awal bulan kemungkinan besar adalah **cache stampede**: banyak supervisor mengakses dashboard bersamaan tepat saat cache kedaluwarsa (atau untuk kombinasi filter yang jarang di-cache sebelumnya), menyebabkan banyak permintaan menghitung ulang secara bersamaan alih-alih satu permintaan menghitung dan yang lain menunggu hasilnya. Solusi yang relevan sekarang: menambahkan mekanisme mencegah stampede (lock sederhana yang memastikan hanya satu permintaan yang benar-benar menghitung ulang saat cache miss, permintaan lain menunggu hasil itu selesai, bukan semuanya menghitung ulang paralel) — bukan kembali ke Option 1 atau 3 dari awal, karena akar masalah baru ini spesifik pada pola konkurensi cache miss, bukan query atau kontensi database.

## Catatan Saya

*Kalau kamu punya kejadian nyata dari kerjaan yang mirip tema ini, tulis sebagai case study baru dengan `source: personal`.*
