---
title: Case - Log Volume That Costs More Than The Servers
type: case
level: intermediate
domain: architecture
status: unread
difficulty: 3
est_minutes: 14
source: synthesized
concepts: ["[[../70 Infrastructure and Delivery/Structured Logging and Log Levels|Structured Logging and Log Levels]]", "[[../80 Security/Audit Logging|Audit Logging]]", "[[../92 Tools/Loki|Loki]]"]
prerequisites: []
next: []
tags: [backend, architecture, infrastructure, gov-context]
created: 2026-08-02
---

## The Situation

Sebuah tim memasang logging terpusat (Elasticsearch) untuk seluruh 13 aplikasi, dengan niat baik memudahkan debugging lintas sistem. Setiap request, termasuk yang berhasil biasa saja, dicatat dengan level `info` yang cukup detail (header lengkap, isi payload, waktu tiap tahap pemrosesan). Enam bulan kemudian, tagihan bulanan untuk infrastruktur logging (storage dan compute Elasticsearch) ternyata **melebihi** biaya seluruh server yang menjalankan 13 aplikasi itu sendiri — angka yang mengejutkan manajemen dan memicu pertanyaan apakah logging ini benar-benar sepadan.

## What Was Actually Asked For

Permintaan awal tim: "kami butuh log yang lengkap untuk debugging." Setelah ditinjau ulang, kebutuhan sebenarnya jauh lebih sempit: mayoritas debugging nyata hanya butuh log **error** dan **warning**, plus kemampuan mengikuti satu request tertentu lewat correlation ID kalau memang perlu — bukan **setiap** request sukses dicatat detail penuh selamanya, "just in case" suatu hari dibutuhkan.

## Options

### Option 1 — Turunkan level logging default ke warning ke atas

Ubah konfigurasi supaya hanya `warning` dan `error` yang dicatat di production, `info` dan `debug` dimatikan. Solusi paling cepat mengurangi volume signifikan, tapi berisiko kehilangan konteks yang kadang berguna (urutan kejadian normal sebelum error terjadi) yang biasanya ada di level `info`.

### Option 2 — Retensi lebih pendek untuk log level rendah

Pertahankan level logging seperti sekarang, tapi terapkan retensi berbeda per level — `error`/`warning` disimpan lama (misalnya 90 hari), `info` disimpan singkat (misalnya 3 hari), `debug` nyaris tidak disimpan sama sekali di production. Mengurangi biaya penyimpanan jangka panjang tanpa mengurangi visibilitas jangka pendek yang sering dibutuhkan debugging aktif.

### Option 3 — Pindah ke sistem logging yang lebih murah untuk volume tinggi (Loki)

Selain menyesuaikan level dan retensi, migrasikan infrastruktur logging dari Elasticsearch (yang mengindeks penuh setiap baris) ke [[../92 Tools/Loki|Loki]] (yang hanya mengindeks label/metadata, jauh lebih murah untuk volume tinggi) — lihat pertimbangan trade-off di note tool Loki itu sendiri. Solusi paling struktural, tapi butuh migrasi infrastruktur dan penyesuaian cara tim mencari log (kueri berdasarkan label dulu, baru isi).

## The Decision

Ketiga opsi diterapkan bersamaan secara bertahap: (1) segera turunkan level default ke `warning` untuk mengurangi volume dalam hitungan hari (perbaikan cepat), (2) terapkan kebijakan retensi berjenjang berdasarkan level (perbaikan jangka menengah), dan (3) evaluasi migrasi ke Loki sebagai proyek terpisah dalam beberapa bulan ke depan (perbaikan struktural jangka panjang). Constraint penentunya: biaya yang sudah melebihi anggaran butuh perbaikan **segera** (tidak bisa menunggu proyek migrasi besar selesai), tapi perbaikan cepat saja (Option 1) tidak menyelesaikan akar masalah arsitektur logging yang mahal untuk skala 13 aplikasi — kombinasi bertahap menyeimbangkan urgensi dan solusi jangka panjang.

## What Breaks at 10x and 100x

Pada volume 10x (lebih banyak aplikasi ditambahkan ke sistem logging terpusat, atau traffic tumbuh signifikan), kebijakan level dan retensi yang sudah diperbaiki tetap membantu, tapi migrasi ke sistem yang lebih murah (Loki, atau sejenisnya) menjadi kebutuhan mendesak, bukan lagi opsional — perbedaan biaya antara indexing penuh dan label-only jadi jauh lebih signifikan di skala ini. Pada 100x, bahkan Loki pun butuh strategi sampling untuk log level rendah (mencatat hanya sebagian kecil request sukses secara acak, bukan semuanya) — prinsip yang sama seperti sampling trace di distributed tracing, diterapkan ke logging.

## What You Would Do Differently

Menetapkan kebijakan level dan retensi logging **sebelum** sistem logging terpusat dipasang untuk semua 13 aplikasi, bukan setelah tagihan tinggi memaksa evaluasi ulang — biaya logging yang proporsional dengan volume traffic seharusnya diperkirakan (lihat [[../60 Distributed Systems/Reading Requirements and Capacity Estimation|Reading Requirements and Capacity Estimation]]) sejak tahap desain, bukan ditemukan lewat kejutan tagihan bulanan.

## Concepts Exercised

- [[../70 Infrastructure and Delivery/Structured Logging and Log Levels|Structured Logging and Log Levels]] — disiplin level logging yang jadi inti solusi kasus ini.
- [[../80 Security/Audit Logging|Audit Logging]] — kontras penting: log audit (yang butuh retensi panjang untuk kepatuhan) tidak boleh ikut terpotong kebijakan retensi pendek yang diterapkan untuk log debugging biasa.
- [[../92 Tools/Loki|Loki]] — solusi infrastruktur alternatif yang jauh lebih murah untuk volume log tinggi.

## Your Turn

Setelah kebijakan level `warning` ke atas diterapkan, tim compliance mengeluh mereka butuh log lengkap (termasuk yang sebelumnya level `info`) untuk investigasi insiden yang terjadi sebelum kebijakan baru ini berlaku. Apa yang bisa dilakukan, dan apa yang tidak bisa diperbaiki lagi?

> [!success]- Kunci jawaban
> Log dari **sebelum** kebijakan baru diterapkan (kalau retensi lama sudah menghapusnya) tidak bisa dipulihkan — ini konsekuensi permanen dari keputusan retensi, dan perlu dikomunikasikan jujur ke tim compliance bahwa data itu memang sudah tidak ada. Untuk investigasi ke depan, penting membedakan **log debugging** (yang boleh dikurangi volumenya untuk hemat biaya) dari **audit trail** (lihat [[../80 Security/Audit Logging|Audit Logging]]) yang mencatat aksi penting seperti perubahan status kasus atau akses data sensitif — audit trail seharusnya **tidak** ikut terpotong oleh kebijakan retensi log debugging yang lebih pendek, karena keduanya punya tujuan dan kebutuhan retensi yang sama sekali berbeda. Kalau kedua jenis log ini bercampur dalam satu aliran yang sama (seperti kemungkinan terjadi di sistem ini), langkah perbaikan berikutnya adalah memisahkan audit trail jadi aliran tersendiri dengan kebijakan retensi sendiri, sesuai kebutuhan compliance yang berbeda dari kebutuhan debugging.

## Catatan Saya

*Kalau kamu punya kejadian nyata dari kerjaan yang mirip tema ini, tulis sebagai case study baru dengan `source: personal`.*
