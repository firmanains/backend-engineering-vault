---
title: Case - Choosing Between Kafka and a Database Outbox Table for One Consumer
type: case
level: intermediate
domain: architecture
status: unread
difficulty: 3
est_minutes: 14
source: synthesized
concepts: ["[[../30 APIs and Web/The Transactional Outbox Pattern|The Transactional Outbox Pattern]]", "[[../30 APIs and Web/Queue vs Log Semantics|Queue vs Log Semantics]]", "[[../90 Architecture and Design/Choosing Which Technical Battles to Fight|Choosing Which Technical Battles to Fight]]"]
prerequisites: []
next: []
tags: [backend, architecture, apis, gov-context]
created: 2026-08-02
---

## The Situation

Sebuah aplikasi perlu mengirim notifikasi email setiap kali status kasus berubah — kebutuhan sederhana, dengan **satu** consumer (service notifikasi email) yang mendengarkan perubahan itu. Seorang engineer, terinspirasi arsitektur event-driven yang dipelajari dari referensi industri, mengusulkan memasang Kafka untuk menangani ini — infrastruktur yang belum pernah dipakai tim ini sebelumnya, butuh setup cluster, monitoring, dan operasional baru yang harus dipelajari dari nol.

## What Was Actually Asked For

Permintaan awal: "kirim notifikasi setiap kali status berubah, secara andal." Kebutuhan sebenarnya bukan "pakai Kafka" — Kafka adalah salah satu **cara** mencapai keandalan itu, bukan tujuan itu sendiri. Untuk kebutuhan dengan satu consumer dan volume yang tidak terlalu tinggi, ada solusi yang jauh lebih sederhana yang sudah memenuhi kebutuhan andal ini tanpa infrastruktur baru.

## Options

### Option 1 — Kafka penuh

Pasang Kafka, publikasikan event perubahan status ke topic, service notifikasi berlangganan sebagai consumer. Solusi yang matang dan skalabel untuk kebutuhan messaging kompleks — tapi untuk **satu** consumer dengan volume rendah-menengah, ini adalah overhead operasional signifikan (cluster untuk dipelihara, dipantau, dan dipahami tim) dibanding manfaat langsung yang didapat sekarang.

### Option 2 — Outbox table di database yang sudah ada

Terapkan [[../30 APIs and Web/The Transactional Outbox Pattern|The Transactional Outbox Pattern]] — tulis event perubahan status ke tabel `outbox` dalam transaksi yang sama dengan perubahan data kasus itu sendiri (menjamin atomicity tanpa dual write, lihat [[../60 Distributed Systems/Dual Writes and Their Dangers|Dual Writes and Their Dangers]]), lalu proses terpisah sederhana (polling berkala) membaca tabel itu dan mengirim notifikasi. Memakai infrastruktur yang sudah ada (database yang sama), tanpa komponen baru untuk dipelajari dan dioperasikan.

### Option 3 — Panggilan langsung sinkron (tanpa outbox atau messaging)

Setelah status kasus berubah, langsung panggil fungsi/service pengirim email dalam alur yang sama, tanpa perantara apa pun. Paling sederhana secara implementasi, tapi rawan gagal parsial (persis masalah dual write) — kalau pengiriman email gagal setelah perubahan status berhasil disimpan, tidak ada mekanisme retry otomatis, notifikasi hilang begitu saja.

## The Decision

Option 2 dipilih. Constraint penentunya adalah prinsip yang relevan langsung dari [[../90 Architecture and Design/Choosing Which Technical Battles to Fight|Choosing Which Technical Battles to Fight]]: investasi infrastruktur baru (Kafka) hanya sepadan kalau kebutuhannya benar-benar menuntut kemampuan yang **tidak bisa** dipenuhi solusi lebih sederhana — dan untuk kasus satu consumer dengan volume rendah-menengah, outbox table sederhana sudah cukup andal (menjamin atomicity, retry otomatis lewat polling yang gagal dan dicoba ulang) tanpa biaya operasional Kafka. Kafka dicatat sebagai opsi yang layak dipertimbangkan ulang kalau kebutuhan berkembang jauh melampaui satu consumer sederhana.

## What Breaks at 10x and 100x

Pada volume 10x (lebih banyak jenis event berbeda, atau consumer bertambah jadi tiga-empat service berbeda yang masing-masing butuh event berbeda), outbox table sederhana dengan polling mulai terasa keterbatasannya — setiap consumer baru butuh logika polling dan penandaan "sudah diproses" sendiri, dan koordinasi antar banyak consumer atas satu tabel outbox jadi lebih rumit dikelola dibanding sistem messaging yang memang dirancang untuk itu. Pada 100x (volume event sangat tinggi, banyak consumer, kebutuhan replay dan retensi panjang), di titik inilah Kafka (atau sistem messaging serupa) benar-benar jadi investasi yang sepadan — bukan karena "lebih modern", tapi karena kebutuhan nyata (throughput, banyak consumer independen, replay) sudah melampaui apa yang bisa diberikan outbox table sederhana secara wajar.

## What You Would Do Differently

Menetapkan kriteria eksplisit sejak awal untuk kapan berpindah dari outbox sederhana ke messaging penuh (misalnya "kalau jumlah consumer independen lebih dari tiga, atau volume event melebihi angka tertentu per detik") — kriteria konkret yang membuat keputusan migrasi nanti berbasis data, bukan dorongan tren teknologi semata.

## Concepts Exercised

- [[../30 APIs and Web/The Transactional Outbox Pattern|The Transactional Outbox Pattern]] — solusi inti yang dipilih, menghindari infrastruktur baru untuk kebutuhan yang masih sederhana.
- [[../30 APIs and Web/Queue vs Log Semantics|Queue vs Log Semantics]] — kerangka memahami kapan semantik messaging penuh (seperti Kafka) benar-benar dibutuhkan dibanding solusi lebih sederhana.
- [[../90 Architecture and Design/Choosing Which Technical Battles to Fight|Choosing Which Technical Battles to Fight]] — prinsip yang mendasari keputusan menolak Kafka untuk kebutuhan yang belum menuntutnya.

## Your Turn

Enam bulan kemudian, tim menambahkan consumer kedua (service analitik yang ingin memproses event perubahan status yang sama untuk statistik). Apakah ini sudah cukup alasan bermigrasi ke Kafka, atau outbox table masih bisa diperluas?

> [!success]- Kunci jawaban
> Dua consumer masih realistis ditangani outbox table yang diperluas — masing-masing consumer bisa memiliki penanda progres terpisah (misalnya kolom `processed_by_notification` dan `processed_by_analytics` di tabel outbox, atau tabel penanda progres per-consumer terpisah), memungkinkan keduanya membaca event yang sama secara independen tanpa saling mengganggu. Ini bukan alasan kuat untuk bermigrasi ke Kafka — kriteria yang lebih relevan (mengikuti prinsip di "What You Would Do Differently") adalah jumlah consumer yang jauh lebih besar (bukan sekadar dua), atau kebutuhan yang benar-benar spesifik messaging penuh (replay dari titik waktu tertentu, throughput sangat tinggi) yang belum muncul di sini. Migrasi prematur ke Kafka untuk dua consumer tetap lebih mahal dibanding manfaat nyata yang didapat sekarang.

## Catatan Saya

*Kalau kamu punya kejadian nyata dari kerjaan yang mirip tema ini, tulis sebagai case study baru dengan `source: personal`.*
