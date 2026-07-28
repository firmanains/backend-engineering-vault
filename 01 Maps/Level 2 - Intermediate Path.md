---
title: Level 2 - Intermediate Path
type: concept
level: intermediate
domain: foundations
status: mastered
difficulty: 1
est_minutes: 10
prerequisites: ["[[Level 1 - Junior Path]]"]
next: ["[[Level 3 - Senior Path]]"]
tags: [backend, meta]
created: 2026-07-26
---

Urutan baca konkret untuk level intermediate, 117 note. Prasyaratnya adalah seluruh [[Level 1 - Junior Path]] — banyak note di sini secara eksplisit membangun di atas note junior dengan nama yang mirip (misalnya "Isolation Levels and Their Anomalies" membangun di atas "Basic Isolation Levels").

Urutannya: memperdalam Go (bahasa lalu concurrency/runtime) dulu, karena semua kode contoh di note-note berikutnya memakainya; lalu database mendalam; lalu API/integrasi/messaging/resilience yang memakai keduanya; lalu delivery dan observability untuk menjalankan dan mengamati apa yang sudah dibangun; ditutup dengan arsitektur untuk melangkah mundur dan menata semuanya.

## 1. 20 Go Language — Advanced Go (6 note)

1. Generics — abstraksi tipe tanpa `interface{}` dan type assertion di mana-mana.
2. Reflection and Its Costs — kekuatan yang datang dengan biaya performa dan keamanan tipe.
3. Embedding — komposisi ala Go, dan bedanya dari inheritance.
4. Functional Options — pola konfigurasi API yang bisa berkembang tanpa breaking change.
5. Designing Stable Library APIs — menulis package yang aman dipakai orang lain bertahun-tahun.
6. Sentinel Errors vs Error Types — dua cara membedakan jenis error, dan kapan masing-masing tepat.

## 2. 50 Concurrency and Performance — Go concurrency (13 note)

7. Goroutines — unit concurrency Go, dan kenapa ia bukan "lightweight thread" begitu saja.
8. Buffered vs Unbuffered Channels — kapan channel memblokir dan kapan tidak.
9. The Select Statement — menunggu banyak channel sekaligus tanpa polling.
10. The Sync Package — `Mutex`, `WaitGroup`, `Once`, dan kapan masing-masing dipakai.
11. Context for Cancellation and Deadlines — membawa "stop sekarang" lintas goroutine dan lintas layer.
12. Worker Pools — membatasi concurrency yang berjalan bersamaan secara sengaja.
13. Fan-In Fan-Out — memecah kerja ke banyak goroutine lalu menyatukan hasilnya.
14. Pipelines — merangkai tahap pemrosesan lewat channel.
15. Race Conditions and the Race Detector — bug yang tidak selalu muncul, dan alat untuk menangkapnya sebelum production.
16. The Go Memory Model — jaminan (dan bukan jaminan) urutan operasi lintas goroutine.
17. errgroup — mengelola sekumpulan goroutine dan error-nya sebagai satu unit.
18. singleflight — mencegah request duplikat memicu kerja duplikat.
19. Goroutine Leaks — goroutine yang lupa berhenti, dan cara mendeteksinya sebelum memory habis.

## 3. 50 Concurrency and Performance — Go runtime (8 note)

20. Goroutine Scheduler (GMP) — bagaimana runtime Go menjadwalkan jutaan goroutine di atas segelintir OS thread.
21. Preemption — kenapa goroutine yang tidak kooperatif tidak lagi bisa memblokir scheduler selamanya.
22. Garbage Collection in Go — model GC Go dan dampaknya pada latency, bukan hanya throughput.
23. Escape Analysis — kenapa compiler memutuskan sebuah value pergi ke heap, bukan stack.
24. pprof Profiling — melihat langsung di mana CPU dan memori sebenarnya dihabiskan.
25. Benchmarking in Go — mengukur performa dengan metodologi, bukan perasaan.
26. Reducing Allocations — teknik konkret menurunkan tekanan pada garbage collector.
27. sync.Pool — mendaur ulang objek untuk mengurangi alokasi di hot path.

## 4. 50 Concurrency and Performance — Performance (5 note)

28. Latency Percentiles (p50, p95, p99) — kenapa rata-rata berbohong tentang pengalaman user.
29. Little's Law — hubungan matematis antara concurrency, throughput, dan latency.
30. Load Testing — mensimulasikan beban nyata sebelum user yang menemukan batasnya.
31. Capacity Planning — menghitung kebutuhan resource dari angka nyata, bukan tebakan.
32. Profiling a Real Application — menggabungkan pprof, benchmark, dan load test jadi satu latihan utuh.

## 5. 50 Concurrency and Performance — Caching (6 note)

33. Cache-Aside, Write-Through, and Write-Behind — tiga pola menjaga cache tetap relevan, dengan trade-off berbeda.
34. Cache Invalidation Strategies — masalah yang terkenal sulit dalam computer science, dan cara mengelolanya.
35. TTL and Jitter — mencegah seluruh cache kedaluwarsa serentak.
36. Eviction Policies — LRU, LFU, dan bagaimana masing-masing memutuskan apa yang dibuang.
37. Cache Stampede — ketika satu key kedaluwarsa memicu badai request ke database.
38. Distributed Locks and Why They Are Dangerous — solusi yang terlihat sederhana, penuh jebakan di produksi.

## 6. 40 Databases — mendalam (14 note)

39. B+Tree Structure — struktur data di balik hampir semua index relasional.
40. Composite Indexes and the Leftmost-Prefix Rule — kenapa urutan kolom di index penting.
41. Covering Indexes — index yang membuat database tidak perlu menyentuh tabel sama sekali.
42. Reading EXPLAIN — membaca rencana eksekusi query seperti seorang DBA.
43. Isolation Levels and Their Anomalies — dirty read, non-repeatable read, phantom read, write skew.
44. MVCC — bagaimana database membaca tanpa memblokir penulisan yang sedang berjalan.
45. Locking and Row Locks — mengendalikan akses konkuren di level baris, bukan tabel.
46. Deadlocks — dua transaksi saling menunggu, dan cara mendiagnosisnya.
47. The N+1 Query Problem — jebakan performa paling umum di aplikasi berbasis ORM.
48. Tuning the Connection Pool — ukuran pool yang tepat untuk beban nyata.
49. Read Replicas and Replication Lag — membagi beban baca, dan konsekuensi data yang sedikit tertinggal.
50. Partitioning — memecah tabel besar tanpa memecah aplikasi.
51. Introduction to Sharding — memecah data lintas instance database.
52. Materialised Views — menyimpan hasil query mahal sebagai tabel yang di-refresh.

## 7. 40 Databases — storage engine dan beyond relational (8 note)

53. Row-Oriented vs Columnar Storage — dua cara menyusun data di disk untuk beban kerja berbeda.
54. OLTP vs OLAP vs HTAP — kenapa query analitik di database transaksional adalah bau arsitektur.
55. LSM-Trees vs B-Trees — dua struktur penyimpanan dengan trade-off write vs read yang berlawanan.
56. Write Amplification and Compression — biaya tersembunyi dari storage engine berbasis LSM.
57. Beyond Relational - Document, Key-Value, Wide-Column, Graph, and Time-Series Stores — memilih model data berdasarkan pola akses, bukan tren.
58. Inverted Indexes and How Search Engines Work — struktur data di balik pencarian full-text.
59. Relevance Scoring — kenapa hasil pencarian punya urutan, bukan sekadar cocok atau tidak.
60. Keeping Search in Sync with the Source of Truth — masalah nyata di balik setiap fitur search yang "kadang hasilnya aneh".

## 8. 30 APIs and Web — integration patterns (9 note)

61. Designing an API for a Partner You Do Not Control — constraint desain saat pihak lain tidak bisa berubah.
62. Contract Negotiation and Versioning — menyepakati kontrak dengan pihak yang tidak selalu punya tim teknis fleksibel.
63. Webhooks and How to Secure Them — mengamankan endpoint yang menerima panggilan dari luar.
64. Polling vs Push — trade-off kesederhanaan vs latency dan beban.
65. File-Based Integration — ketika partner masih mengandalkan file, bukan API.
66. Batch vs Realtime Integration — memilih ritme pertukaran data yang sesuai kebutuhan bisnis.
67. Handling an Unreliable Counterparty — desain yang tetap benar walau pihak lain lambat atau gagal.
68. Sandbox Environments — menguji integrasi tanpa menyentuh data produksi pihak lain.
69. Integration Testing Across an Organisational Boundary — menguji sesuatu yang separuh kendalinya bukan milikmu.

## 9. 30 APIs and Web — protokol lain (6 note)

70. gRPC and Protobuf — RPC berbasis schema dengan performa yang jauh lebih baik dari JSON over HTTP.
71. Schema Evolution in Protobuf — mengubah kontrak binary tanpa mematahkan klien lama.
72. GraphQL and Its Trade-offs — fleksibilitas query di sisi klien, dengan biaya kompleksitas di sisi server.
73. WebSocket — komunikasi dua arah yang tetap terbuka.
74. Server-Sent Events — push satu arah yang lebih sederhana dari WebSocket.
75. Long Polling — realtime semu di atas infrastruktur HTTP biasa.

## 10. 30 APIs and Web — messaging dan streaming (9 note)

76. Queue vs Log Semantics — dua model perpesanan dengan jaminan yang sangat berbeda.
77. Topics, Partitions, and Offsets — unit dasar sistem berbasis log seperti Kafka.
78. Consumer Groups and Rebalancing — bagaimana banyak consumer membagi beban satu topic.
79. Delivery Semantics — at-most-once, at-least-once, dan kenapa exactly-once selalu punya syarat tersembunyi.
80. Idempotent Consumers — bertahan dari pesan yang diterima dua kali.
81. The Transactional Outbox Pattern — menjaga database dan message broker tetap konsisten tanpa distributed transaction.
82. Dead Letter Queues — tempat pesan yang gagal diproses berulang kali, bukan hilang begitu saja.
83. Ordering Guarantees in Streaming Systems — kapan urutan pesan dijamin, dan kapan tidak.
84. Backpressure — apa yang terjadi ketika consumer lebih lambat dari producer.

## 11. 30 APIs and Web — resilience (7 note)

85. Timeout Budgets — membagi total waktu yang boleh dipakai lintas beberapa panggilan berantai.
86. Retries with Exponential Backoff and Jitter — retry yang membantu, bukan memperparah insiden.
87. Circuit Breakers — berhenti memanggil layanan yang sedang jelas rusak.
88. Bulkheads — mengisolasi kegagalan supaya tidak menular ke bagian sistem lain.
89. Rate Limiting Algorithms — token bucket, leaky bucket, sliding window.
90. Load Shedding — menolak sebagian request secara sengaja untuk menyelamatkan sisanya.
91. Graceful Degradation — tetap berfungsi sebagian saat komponen pendukung mati.

## 12. 70 Infrastructure and Delivery — delivery dan IaC (10 note)

92. CI-CD Pipelines — otomatisasi dari commit sampai deploy.
93. Kubernetes Core Concepts - Pods, Deployments, Services, Ingress — kosakata dasar yang wajib sebelum menyentuh manifest apa pun.
94. Kubernetes Config, Secrets, Probes, and Autoscaling — mengoperasikan workload, bukan sekadar menjalankannya.
95. Declarative vs Imperative Infrastructure — menyatakan hasil akhir yang diinginkan, bukan langkah demi langkah.
96. Desired-State Reconciliation — bagaimana sistem seperti Kubernetes terus menutup gap ke keadaan yang diinginkan.
97. State Files and Drift — risiko saat kenyataan infrastruktur menyimpang dari definisinya.
98. Immutable Infrastructure vs Configuration Management — dua filosofi mengelola server, dan kenapa yang pertama makin dominan.
99. Blue-Green and Canary Releases — rilis yang bisa dibatalkan sebelum semua user terdampak.
100. Feature Flags — memisahkan deploy dari release.
101. Zero-Downtime Database Migrations — mengubah skema sambil aplikasi tetap melayani traffic.
102. Service Discovery — bagaimana satu service menemukan alamat service lain yang terus berpindah.

## 13. 70 Infrastructure and Delivery — observability (10 note)

103. The Three Pillars of Observability — log, metric, trace, dan kenapa ketiganya saling melengkapi bukan saling menggantikan.
104. Structured Logging and Log Levels — log yang bisa di-query, bukan sekadar dibaca.
105. Metrics - The RED and USE Methods — dua kerangka memilih metrik yang benar-benar berguna.
106. Pull vs Push Metrics Collection — dua model pengumpulan metrik dan implikasi operasionalnya.
107. Query Languages for Metrics — membaca dan menulis query untuk menjawab pertanyaan operasional nyata.
108. Dashboard Design — dashboard yang benar-benar dilihat orang saat insiden, bukan dekorasi.
109. Distributed Tracing — mengikuti satu request melintasi banyak service.
110. Correlation IDs — benang merah yang menyatukan log dari service yang berbeda-beda.
111. Alerts That Do Not Cause Fatigue — alert yang dipercaya, bukan yang diabaikan karena terlalu sering salah.
112. SLIs and SLOs — mengubah "sistem harus reliable" jadi angka yang bisa diukur dan disepakati.

## 14. 90 Architecture and Design — arsitektur (5 note)

113. Hexagonal and Clean Architecture in Go — menerapkan batas arsitektur secara idiomatic, bukan meniru Java.
114. Lightweight DDD — memakai bahasa domain di kode tanpa ritual DDD yang berat.
115. Modular Monolith vs Microservices — microservices tidak mengurangi kompleksitas, ia memindahkannya ke jaringan.
116. Defining Service Boundaries — garis batas yang membuat service benar-benar independen, bukan sekadar dipisah secara fisik.
117. Synchronous vs Asynchronous Communication — memilih model komunikasi antar service berdasarkan siapa yang butuh menunggu.

## Setelah Ini

Selesaikan Project 2 (`95 Projects/Project 2 - Worker Pool, Cache, and Streaming Service.md`), lalu lanjut ke [[Level 3 - Senior Path]].

## Connected Notes

- [[Level 1 - Junior Path]] — prasyarat penuh sebelum memulai jalur ini.
- [[Backend Engineering Map]] — posisi setiap domain di atas dalam peta besar vault.
- [[Vault Manifest]] — path lengkap tiap note yang disebut di sini.
- [[Level 3 - Senior Path]] — kelanjutan setelah 117 note ini selesai.

## Catatan Saya

*Tulis di sini bagian mana dari jalur intermediate yang paling terasa berbeda dari cara kerja harianmu di PHP/Yii2.*
