---
title: Case - The Rate-Limited External API and the Batch Job
type: case
level: intermediate
domain: architecture
status: unread
difficulty: 3
est_minutes: 13
source: synthesized
concepts: ["[[../30 APIs and Web/Rate Limiting Algorithms|Rate Limiting Algorithms]]", "[[../30 APIs and Web/Batch vs Realtime Integration|Batch vs Realtime Integration]]", "[[../30 APIs and Web/Backpressure|Backpressure]]"]
prerequisites: []
next: []
tags: [backend, architecture, apis, gov-context, integration]
created: 2026-08-02
---

## The Situation

Sebuah job batch malam hari perlu memvalidasi 50.000 nomor identitas kependudukan lewat API pusat data kependudukan milik kementerian lain — API itu membatasi 10 permintaan per detik per klien, kebijakan yang tidak bisa dinegosiasikan karena berlaku sama untuk semua instansi pengguna. Dengan batas itu, memproses 50.000 nomor butuh setidaknya 5.000 detik (lebih dari 80 menit) kalau dijalankan persis di batas rate limit — dan job ini harus selesai sebelum jam kerja dimulai, jendela waktu yang cukup ketat kalau ditambah waktu pemrosesan lain.

## What Was Actually Asked For

Permintaan awal: "job ini harus selesai lebih cepat." Setelah didiskusikan, ternyata rate limit API partner adalah batasan keras yang tidak bisa diubah — kebutuhan sebenarnya bukan "mempercepat panggilan API" (tidak mungkin melampaui rate limit tanpa dilarang atau diblokir), tapi **memastikan job selesai dalam jendela waktu yang tersedia**, dengan cara yang tidak melanggar batasan partner dan tidak rapuh terhadap kegagalan sebagian panggilan.

## Options

### Option 1 — Jalankan job lebih awal, beri jendela waktu lebih longgar

Mulai job lebih awal (misalnya jam 8 malam, bukan tengah malam) untuk memberi ruang waktu yang cukup meski dibatasi rate limit. Solusi paling sederhana, tapi bergantung asumsi data yang divalidasi sudah "final" cukup awal — kalau ada data yang berubah setelah jam 8 malam, hasil validasi bisa jadi usang saat dipakai di pagi hari.

### Option 2 — Paralelkan lewat banyak API key/klien terdaftar

Kalau partner mengizinkan lebih dari satu API key per instansi, jalankan beberapa "klien" paralel masing-masing dengan rate limit sendiri, secara efektif melipatgandakan throughput total. Bisa signifikan mempercepat, tapi bergantung kebijakan partner (apakah ini diizinkan atau dianggap penyalahgunaan), dan menambah kompleksitas mengelola banyak kredensial serta rate limiter terpisah untuk masing-masing.

### Option 3 — Rate limiter yang presisi (token bucket) plus pemrosesan yang bisa dilanjutkan

Implementasikan rate limiter yang benar-benar presisi di batas 10/detik (lihat [[../30 APIs and Web/Rate Limiting Algorithms|Rate Limiting Algorithms]]) — bukan asumsi kasar, tapi algoritma token bucket yang memaksimalkan throughput persis di batas tanpa melanggarnya sedikit pun (safety margin nol butuh implementasi yang benar-benar akurat). Kombinasikan dengan mekanisme yang menyimpan progres (mirip backfill per-batch) sehingga kalau job terganggu di tengah jalan, ia bisa dilanjutkan dari titik terakhir, bukan mengulang dari awal dan membuang waktu yang sudah terpakai.

## The Decision

Option 3 dipilih sebagai fondasi (rate limiter presisi memastikan throughput maksimal yang diizinkan benar-benar tercapai, bukan tersia-sia karena implementasi yang terlalu konservatif), dikombinasikan dengan Option 1 (mulai lebih awal) sebagai margin keamanan tambahan. Option 2 ditolak karena kebijakan partner secara eksplisit tidak mengizinkan multiple API key per instansi (dikonfirmasi lewat dokumentasi kontrak) — opsi yang secara teori menarik tapi tidak realistis dalam batasan kontrak nyata. Constraint penentunya: kepatuhan terhadap kebijakan partner adalah batasan keras yang tidak bisa dilanggar demi kecepatan, jadi optimasi harus dilakukan dalam batas itu, bukan mencoba melampauinya.

## What Breaks at 10x and 100x

Pada volume 10x (500.000 nomor), bahkan dengan rate limiter presisi, waktu total tetap terikat linear pada rate limit tetap partner (sekitar 14 jam pada batas yang sama) — jendela waktu semalam tidak lagi cukup, dan opsi realistisnya adalah mengajukan permintaan resmi ke partner untuk rate limit lebih tinggi (dengan justifikasi volume yang jelas), atau memproses secara bertahap lintas beberapa malam untuk data yang tidak semuanya butuh validasi ulang setiap hari. Pada 100x, ketergantungan pada satu API sinkron jadi bottleneck struktural yang butuh dipertanyakan ulang — mungkin ada kesepakatan integrasi berbeda (file batch harian dari partner, bukan panggilan API satu-satu) yang lebih sesuai untuk volume sebesar ini.

## What You Would Do Differently

Menanyakan ke partner sejak awal apakah mereka menyediakan endpoint validasi **batch** (mengirim banyak nomor sekaligus dalam satu panggilan, bukan satu nomor per panggilan) — kalau tersedia, ini bisa mengubah keseluruhan perhitungan rate limit secara drastis (rate limit per panggilan, bukan per nomor), sesuatu yang seharusnya ditanyakan sebelum merancang solusi di sekitar batasan per-nomor yang mungkin sebenarnya bisa dihindari.

## Concepts Exercised

- [[../30 APIs and Web/Rate Limiting Algorithms|Rate Limiting Algorithms]] — algoritma token bucket yang jadi fondasi solusi memaksimalkan throughput dalam batas yang diizinkan.
- [[../30 APIs and Web/Batch vs Realtime Integration|Batch vs Realtime Integration]] — pertimbangan trade-off pemrosesan batch semalam versus alternatif lain.
- [[../30 APIs and Web/Backpressure|Backpressure]] — prinsip menyesuaikan laju permintaan dengan kapasitas penerima, diterapkan langsung di kasus ini.

## Your Turn

Job batch ini, dengan rate limiter presisi, berjalan mulus selama berbulan-bulan — sampai suatu malam partner API mengalami gangguan dan setiap panggilan mengembalikan error selama satu jam penuh di tengah proses. Apa yang harus dilakukan job ini, dan apakah desain "bisa dilanjutkan dari titik terakhir" yang sudah ada cukup menangani skenario ini?

> [!success]- Kunci jawaban
> Desain "bisa dilanjutkan dari titik terakhir" memang membantu, tapi hanya kalau job tahu **kapan** harus berhenti mencoba dan menyerah sementara, bukan terus mencoba tanpa henti dan membuang seluruh sisa rate limit budget yang tersedia untuk permintaan yang hampir pasti gagal lagi. Solusi yang tepat: tambahkan deteksi kegagalan beruntun (circuit breaker sederhana) — kalau beberapa permintaan berturut-turut gagal dengan pola yang sama (bukan kegagalan acak sesekali), job berhenti sementara (misalnya 10 menit) sebelum mencoba lagi, alih-alih terus memukul API yang jelas sedang bermasalah. Progres yang sudah tersimpan (dari desain "bisa dilanjutkan") memastikan job tidak perlu mengulang dari awal begitu partner pulih — hanya melanjutkan dari nomor identitas terakhir yang berhasil diproses sebelum gangguan terjadi.

## Catatan Saya

*Kalau kamu punya kejadian nyata dari kerjaan yang mirip tema ini, tulis sebagai case study baru dengan `source: personal`.*
