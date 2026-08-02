---
title: Case - Migrating a 200-Million-Row Table Without Downtime
type: case
level: intermediate
domain: architecture
status: unread
difficulty: 4
est_minutes: 16
source: synthesized
concepts: ["[[../70 Infrastructure and Delivery/Zero-Downtime Database Migrations|Zero-Downtime Database Migrations]]", "[[../60 Distributed Systems/Backfilling Large Datasets Safely|Backfilling Large Datasets Safely]]", "[[../60 Distributed Systems/Expand-Contract Schema Migration|Expand-Contract Schema Migration]]"]
prerequisites: []
next: []
tags: [backend, architecture, databases, gov-context]
created: 2026-08-02
---

## The Situation

Sebuah sistem legal-services yang sudah berjalan bertahun-tahun punya tabel riwayat kasus dengan lebih dari 200 juta baris — tabel ini harus diubah untuk menambah kolom baru (`wilayah_hukum`, dihitung dari data yang sudah ada) yang dibutuhkan fitur baru dengan tenggat rilis dua minggu. Sistem ini melayani petugas 24/7 di seluruh provinsi, tanpa jendela maintenance yang bisa diterima — downtime bahkan lima menit di jam kerja berarti ratusan petugas tidak bisa bekerja.

## What Was Actually Asked For

Tim produk meminta "tambahkan kolom ini secepat mungkin." Yang sebenarnya dibutuhkan: kolom baru terisi benar untuk seluruh 200 juta baris **tanpa** downtime yang terasa pengguna, dan proses migrasi ini sendiri tidak boleh memperlambat operasi normal sistem selama berjalan.

## Options

### Option 1 — Migration langsung, satu statement `ALTER TABLE ADD COLUMN`

Cara paling sederhana secara implementasi — satu perintah SQL. Tapi pada tabel 200 juta baris, tergantung mesin database dan versinya, operasi ini bisa mengunci tabel dalam waktu yang sangat lama (bisa berjam-jam) atau menulis ulang seluruh tabel, keduanya tidak bisa diterima untuk sistem yang harus tetap melayani 24/7.

### Option 2 — Migration dengan tool online schema change (gh-ost, pt-online-schema-change, atau `CREATE INDEX CONCURRENTLY`/setara)

Memakai tool yang dirancang khusus melakukan perubahan skema tanpa mengunci tabel penuh — bekerja dengan membuat tabel bayangan (shadow table), menyalin data secara bertahap, dan melakukan swap cepat di akhir. Solusi yang matang dan teruji luas di industri untuk kasus persis seperti ini, tapi butuh waktu setup dan pemahaman tool yang tepat, serta pengujian menyeluruh di lingkungan yang mirip production sebelum dijalankan di data sungguhan.

### Option 3 — Expand-contract manual dengan backfill per-batch

Tambah kolom baru sebagai nullable (operasi cepat, tidak menulis ulang tabel di database modern), lalu jalankan backfill terpisah per-batch (lihat [[../60 Distributed Systems/Backfilling Large Datasets Safely|Backfilling Large Datasets Safely]]) untuk mengisi nilai kolom itu secara bertahap tanpa membebani database sekaligus, mengikuti pola [[../60 Distributed Systems/Expand-Contract Schema Migration|Expand-Contract Schema Migration]]. Lebih banyak kode yang harus ditulis sendiri dibanding memakai tool jadi (Option 2), tapi memberi kontrol penuh atas kecepatan dan jeda proses, dan tidak bergantung pada tool eksternal yang mungkin belum pernah dipakai tim ini.

## The Decision

Kombinasi Option 2 dan 3 dipilih: menambah kolom sebagai nullable dulu (langkah cepat dan aman, bagian dari Option 3), lalu backfill dijalankan memakai skrip kustom per-batch (bukan tool eksternal, karena tim belum punya pengalaman dengan gh-ost/pt-online-schema-change dan waktu dua minggu tidak cukup untuk belajar dan menguji tool baru dengan aman) dengan ukuran batch dan jeda yang diuji dulu di staging. Constraint penentunya: familiaritas tim dengan pendekatan yang bisa mereka kontrol dan pahami sepenuhnya lebih penting daripada memakai tool yang secara teori lebih matang tapi belum pernah dicoba tim ini dalam tenggat sesempit ini.

## What Breaks at 10x and 100x

Pada volume 10x (2 miliar baris, skenario yang mungkin realistis dalam beberapa tahun ke depan untuk sistem nasional), backfill kustom per-batch mulai memakan waktu yang sangat lama (berhari-hari) meski aman — di titik ini, investasi mempelajari dan memakai tool online schema change yang lebih dioptimalkan (Option 2 penuh) menjadi lebih sepadan dibanding terus mengandalkan skrip kustom. Pada 100x, migrasi skema pada tabel sebesar itu mendekati batas yang realistis untuk satu database monolitik — pertanyaan yang relevan bukan lagi "bagaimana migrasi aman", tapi "apakah tabel ini seharusnya sudah di-partition atau di-shard" (lihat [[../60 Distributed Systems/Sharding Strategies and Hot Partitions|Sharding Strategies and Hot Partitions]]) jauh sebelum ukurannya mencapai titik ini.

## What You Would Do Differently

Menguji proses backfill di lingkungan staging dengan volume data yang benar-benar mendekati 200 juta baris (bukan dataset kecil yang tidak merepresentasikan beban nyata) sebelum menjalankan di production — perkiraan waktu dan dampak dari pengujian dataset kecil sering meleset jauh untuk skala data sebesar ini.

## Concepts Exercised

- [[../70 Infrastructure and Delivery/Zero-Downtime Database Migrations|Zero-Downtime Database Migrations]] — prinsip dasar expand-contract yang diterapkan langsung di kasus ini.
- [[../60 Distributed Systems/Backfilling Large Datasets Safely|Backfilling Large Datasets Safely]] — strategi backfill per-batch yang jadi inti solusi.
- [[../60 Distributed Systems/Expand-Contract Schema Migration|Expand-Contract Schema Migration]] — kerangka umum tahapan migrasi yang diikuti.

## Your Turn

Setelah backfill selesai untuk 200 juta baris, tim menyadari bahwa selama proses backfill berlangsung (yang memakan waktu tiga hari), ada sekitar 500 ribu baris baru yang ditambahkan pengguna aktif — apakah baris-baris ini otomatis punya nilai kolom `wilayah_hukum` yang benar?

> [!success]- Kunci jawaban
> Tergantung urutan langkah yang diambil — kalau kode aplikasi yang menulis baris baru sudah di-deploy untuk mengisi kolom `wilayah_hukum` **sebelum** backfill dimulai (mengikuti prinsip expand-contract: kode baru dulu, baru backfill data lama), maka 500 ribu baris baru itu otomatis terisi benar oleh kode aplikasi itu sendiri, tidak perlu ikut di-backfill. Kalau urutannya terbalik (backfill dijalankan sebelum kode aplikasi diperbarui menulis kolom baru), baris-baris baru selama tiga hari itu akan kosong dan butuh backfill tambahan setelah backfill utama selesai — kesalahan urutan yang seharusnya dicegah dengan mengikuti urutan expand-contract yang benar sejak awal: skema dulu, kode aplikasi berikutnya, baru backfill data historis.

## Catatan Saya

*Kalau kamu punya kejadian nyata dari kerjaan yang mirip tema ini, tulis sebagai case study baru dengan `source: personal`.*
