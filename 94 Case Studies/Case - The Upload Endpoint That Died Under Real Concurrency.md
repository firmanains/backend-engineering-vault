---
title: Case - The Upload Endpoint That Died Under Real Concurrency
type: case
level: intermediate
domain: architecture
status: unread
difficulty: 3
est_minutes: 14
source: synthesized
concepts: ["[[../30 APIs and Web/Streaming vs Buffering|Streaming vs Buffering]]", "[[../30 APIs and Web/Request Size Limits Along The Path|Request Size Limits Along The Path]]", "[[../30 APIs and Web/Load Shedding|Load Shedding]]"]
prerequisites: []
next: []
tags: [backend, architecture, apis, performance, gov-context]
created: 2026-08-02
---

## The Situation

Endpoint upload dokumen di sebuah aplikasi lolos seluruh pengujian fungsional — satu file, diunggah satu per satu, berhasil mulus. Begitu dirilis dan dipakai ratusan petugas serentak menjelang tenggat pengajuan (semua mengunggah dokumen pendukung dalam waktu bersamaan), aplikasi mulai crash berulang dengan error kehabisan memori. Investigasi menunjukkan kode upload membaca **seluruh** isi file ke memori sekaligus (`ioutil.ReadAll` pada body request) sebelum memprosesnya — untuk satu file 10MB, ini biasa saja; untuk ratusan file 10MB diproses bersamaan, total memori yang dibutuhkan jauh melebihi kapasitas Pod.

## What Was Actually Asked For

Permintaan awal, setelah insiden: "endpoint upload harus lebih kuat." Kebutuhan sebenarnya lebih spesifik: endpoint harus **tidak crash** di bawah concurrency nyata, dan idealnya **membatasi dampak** kalau beban memang melebihi kapasitas (menolak sebagian permintaan dengan jelas, bukan membuat seluruh aplikasi down untuk semua orang).

## Options

### Option 1 — Naikkan kapasitas memori Pod

Tambah `resources.limits.memory` di manifest Kubernetes, memberi Pod lebih banyak "ruang" menampung beban memori upload bersamaan. Solusi paling cepat diterapkan, tapi hanya menunda masalah — jumlah petugas yang mengunggah bersamaan bisa terus bertambah (terutama menjelang tenggat), dan menaikkan kapasitas memori tidak punya batas atas yang masuk akal untuk terus dinaikkan setiap kali beban bertambah.

### Option 2 — Streaming, bukan buffering penuh ke memori

Ubah kode upload untuk memproses file sebagai stream (lihat [[../30 APIs and Web/Streaming vs Buffering|Streaming vs Buffering]]) — data dibaca dan langsung diteruskan ke tujuan akhir (disk, object storage) secara bertahap, tanpa pernah menampung seluruh isi file di memori sekaligus. Menyelesaikan akar masalah penggunaan memori per-request, dan skalanya jauh lebih baik terhadap concurrency — tapi butuh perubahan kode yang lebih signifikan dibanding sekadar menaikkan resource.

### Option 3 — Load shedding: batasi jumlah upload konkuren

Tambahkan mekanisme di aplikasi yang membatasi jumlah upload yang diproses bersamaan (lihat [[../30 APIs and Web/Load Shedding|Load Shedding]]) — permintaan yang melebihi batas ditolak segera dengan error jelas (429 Too Many Requests), bukan diterima dan membuat seluruh sistem kewalahan. Mengurangi risiko crash total, tapi berarti sebagian pengguna mengalami kegagalan yang terlihat (harus mencoba lagi) saat beban puncak.

## The Decision

Kombinasi Option 2 dan 3 dipilih — streaming menyelesaikan akar masalah penggunaan memori per-request (solusi struktural), dan load shedding ditambahkan sebagai lapisan pertahanan kedua untuk skenario beban ekstrem yang bahkan streaming pun tidak sepenuhnya bisa menyerap (bandwidth jaringan atau I/O disk yang jadi bottleneck, bukan cuma memori). Constraint penentunya: menaikkan resource saja (Option 1) tidak menyelesaikan masalah yang bisa terus tumbuh proporsional dengan jumlah pengguna aktif — solusi harus mengubah karakteristik pemakaian resource per-request, bukan hanya menambah kapasitas total.

## What Breaks at 10x and 100x

Pada volume 10x, streaming saja mungkin belum cukup kalau bandwidth jaringan Pod jadi bottleneck baru (banyak file besar mengalir bersamaan meski masing-masing tidak menumpuk di memori) — load shedding (Option 3) jadi lapisan yang makin penting di titik ini. Pada 100x, arsitektur mungkin perlu bergeser lebih jauh — pre-signed URL (lihat [[../30 APIs and Web/Pre-signed URLs|Pre-signed URLs]]) yang membuat klien mengunggah **langsung** ke object storage, sepenuhnya melewati server aplikasi untuk transfer file besar, menghilangkan bottleneck di aplikasi itu sendiri secara struktural.

## What You Would Do Differently

Menguji endpoint dengan load testing yang mensimulasikan concurrency realistis (bukan hanya pengujian fungsional satu-per-satu) sebelum rilis — beban seperti "ratusan petugas mengunggah bersamaan menjelang tenggat" adalah pola yang bisa diprediksi untuk sistem pemerintah dengan siklus pengajuan musiman, dan seharusnya jadi skenario pengujian standar, bukan ditemukan lewat insiden production.

## Concepts Exercised

- [[../30 APIs and Web/Streaming vs Buffering|Streaming vs Buffering]] — solusi struktural yang mengubah karakteristik pemakaian memori per-request.
- [[../30 APIs and Web/Request Size Limits Along The Path|Request Size Limits Along The Path]] — batas ukuran yang relevan dipertimbangkan bersamaan dengan strategi streaming.
- [[../30 APIs and Web/Load Shedding|Load Shedding]] — lapisan pertahanan tambahan untuk beban yang melebihi kapasitas bahkan setelah streaming diterapkan.

## Your Turn

Setelah streaming diterapkan, tim menyadari bahwa validasi ukuran file (menolak file di atas 20MB) yang sebelumnya dilakukan **setelah** seluruh file terbaca ke memori sekarang tidak bisa dilakukan dengan cara yang sama, karena file tidak pernah sepenuhnya berada di memori. Bagaimana validasi ukuran ini seharusnya diterapkan dengan pendekatan streaming?

> [!success]- Kunci jawaban
> Validasi ukuran harus diterapkan **selama** streaming berlangsung, bukan setelah selesai — menghitung jumlah byte yang sudah diproses secara bertahap, dan menghentikan proses (menolak request) begitu jumlah itu melebihi batas, tanpa perlu menunggu seluruh file selesai diterima. Di Go, ini bisa dilakukan dengan membungkus reader request dengan `io.LimitReader` atau `http.MaxBytesReader`, yang secara otomatis menghentikan pembacaan begitu batas byte terlampaui — pendekatan yang bahkan lebih efisien dari validasi setelah-selesai yang lama, karena upload yang jelas akan ditolak (file terlalu besar) dihentikan lebih dini, tidak perlu menghabiskan bandwidth menerima seluruh file dulu baru menolaknya di akhir.

## Catatan Saya

*Kalau kamu punya kejadian nyata dari kerjaan yang mirip tema ini, tulis sebagai case study baru dengan `source: personal`.*
