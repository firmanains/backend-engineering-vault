---
title: Backlog
type: concept
level: junior
domain: foundations
status: mastered
difficulty: 1
est_minutes: 3
prerequisites: ["[[Vault Manifest]]"]
next: []
tags: [backend, meta]
created: 2026-07-26
---

## Aturan

Sebuah note boleh ditautkan (`[[...]]`) sebelum ia ditulis, tapi tautan yang menggantung selamanya tidak boleh ada. Selama vault ini masih di Batch 0, **hampir seluruh isi [[Vault Manifest]] adalah backlog** — itu memang tujuan Batch 0: menunjukkan bentuk penuh vault sebelum satu pun concept note ditulis.

Karena itu, file ini **tidak mendaftar ulang** seluruh manifest. Sumber kebenaran untuk "note apa saja yang direncanakan tapi belum ditulis" adalah [[Vault Manifest]] itu sendiri — setiap baris di sana yang belum punya file nyata di folder yang bersangkutan adalah item backlog secara implisit.

File ini hanya mendaftar dua kategori:

1. **Note yang ditautkan dari sebuah note yang sudah ditulis, tapi ternyata belum ada di [[Vault Manifest]] sama sekali** — ini adalah gap kurikulum yang ditemukan sambil menulis, dan begitu ditambahkan ke sini, judulnya juga harus ditambahkan ke manifest dan ke `Curriculum Changelog.md` kalau memang topik baru.
2. **Note yang sudah ada di manifest, tapi rusak tautannya** karena salah ketik judul — dicatat di sini sampai diperbaiki.

## Item Backlog Saat Ini

| Note yang ditautkan | Ditautkan dari | Alasan ditambahkan | Sudah ada di manifest? |
|---|---|---|---|
| [[Service Discovery]] | `92 Tools/_Overview.md` (Consul) | Consul butuh concept note yang diimplementasikannya; tidak ada di kurikulum asli §11. | Ya — ditambahkan, lihat `Curriculum Changelog.md`. |

`[[Load Balancing and Reverse Proxies]]` sudah selesai ditulis (`30 APIs and Web/`) — dihapus dari tabel di atas. `Service Discovery` masih tertulis di tabel karena filenya belum ada; hapus baris ini begitu `70 Infrastructure and Delivery/Service Discovery.md` dibuat.

## Titik Lanjutan (Resume Point)

Ditulis setiap kali sesi penulisan berhenti di tengah jalan, supaya sesi berikutnya tahu persis harus mulai dari mana tanpa perlu menebak dari isi folder.

**Status per 2026-08-02:**

- **`80 Security` selesai penuh sampai kedalaman maksimalnya (17/17 note — 10 junior + 7 senior)**: senior (Threat Modelling with STRIDE, Zero Trust, mTLS, Key Management and Rotation, Encryption at Rest vs In Transit, Audit Logging, Compliance Trails for Government Systems). Domain ini **tidak** punya note intermediate di manifest — junior langsung ke senior.
- **`30 APIs and Web` selesai penuh sampai kedalaman maksimalnya (52/52 note — 21 junior + 31 intermediate)**: seluruh note intermediate sudah ditulis, dari `Designing an API for a Partner You Do Not Control` sampai `Graceful Degradation`, mencakup klaster integration patterns (9: partner integration, contract, webhook, polling/push, file-based, batch/realtime, unreliable counterparty, sandbox, integration testing), protokol lain (6: gRPC and Protobuf, Schema Evolution in Protobuf, GraphQL and Its Trade-offs, WebSocket, Server-Sent Events, Long Polling), messaging/streaming (8: Queue vs Log Semantics sampai Ordering Guarantees in Streaming Systems), dan resilience (8: Backpressure sampai Graceful Degradation). Domain ini **tidak** punya tingkat senior sendiri di manifest.

- **Seluruh level junior selesai (89/89 note)**: `10 Foundations` (12/12), `20 Go Language` (17/17 junior), `30 APIs and Web` (21/21 junior), `40 Databases` (21/21 junior), `70 Infrastructure and Delivery` (4/4 junior), `80 Security` (10/10 junior), `90 Architecture and Design` (4/4 junior).
- **`40 Databases` selesai penuh sampai kedalaman maksimalnya (43/43 note — 21 junior + 22 intermediate)**: seluruh note intermediate sudah ditulis, dari `B+Tree Structure` sampai `Keeping Search in Sync with the Source of Truth`. Domain ini **tidak** punya tingkat senior sendiri di manifest (sengaja, lihat `Curriculum Changelog.md` §"Keputusan Pemetaan Domain") — kelanjutan level senior untuk topik sharding, replication skala besar, dan migrasi berbasis CDC ada di `60 Distributed Systems` (klaster "Scalability" dan "Migration and evolution"), yang akan ditulis penuh saat gilirannya tiba di sequence domain, bukan dipecah sekarang.
- **`20 Go Language` selesai penuh sampai kedalaman maksimalnya (23/23 note — 17 junior + 6 intermediate "Advanced Go")**: `Generics`, `Reflection and Its Costs`, `Embedding`, `Functional Options`, `Designing Stable Library APIs`, `Sentinel Errors vs Error Types`. Domain ini juga **tidak** punya tingkat senior sendiri (concurrency dan runtime Go sengaja dipetakan ke `50 Concurrency and Performance`, bukan ke sini — lihat `Curriculum Changelog.md`).
- **`50 Concurrency and Performance` selesai penuh (32/32 note — domain ini memang intermediate-only, tidak ada junior/senior)**: klaster Go concurrency (13: Goroutines sampai Goroutine Leaks), Go runtime (8: Goroutine Scheduler (GMP) sampai sync.Pool), Performance (5: Latency Percentiles sampai Profiling a Real Application), Caching (6: Cache-Aside/Write-Through/Write-Behind sampai Distributed Locks and Why They Are Dangerous).
- **`90 Architecture and Design` selesai penuh sampai kedalaman maksimalnya (15/15 note — 4 junior + 5 intermediate + 6 senior)**: intermediate (Hexagonal and Clean Architecture in Go, Lightweight DDD, Modular Monolith vs Microservices, Defining Service Boundaries, Synchronous vs Asynchronous Communication) dan senior/kepemimpinan teknis (API Governance, Cross-Team Code Standards, The RFC Process, Mentoring, Choosing Which Technical Battles to Fight, Managing Technical Debt Explicitly) — domain pertama sejak `40 Databases` yang benar-benar punya ketiga tingkat lengkap.
- **Ganti strategi build: dari level-breadth ke domain-depth.** Sejak `CLAUDE.md` §14 diperbarui, sesi berikutnya **tidak lagi** mengikuti `01 Maps/Level 2 - Intermediate Path.md` secara lintas-domain — satu domain dituntaskan penuh (intermediate lalu senior, sejauh manifest domain itu punya keduanya) sebelum pindah ke domain lain. Urutan pilot: `40 Databases` → `20 Go Language` → `50 Concurrency and Performance` → `90 Architecture and Design`, sesuai urutan domain di `CLAUDE.md` §14.
- **`70 Infrastructure and Delivery` selesai penuh sampai kedalaman maksimalnya (25/25 note — 4 junior + 21 intermediate)**: klaster delivery/IaC (11: CI-CD Pipelines sampai Service Discovery) dan klaster observability (10: The Three Pillars of Observability sampai SLIs and SLOs). Domain ini **tidak** punya tingkat senior sendiri di manifest — junior dan intermediate adalah kedalaman maksimalnya.
- **Belum disentuh sama sekali**: seluruh 39 note senior `60 Distributed Systems`, seluruh tool note (28), seluruh case study (25), semua project (3), dan glossary (1/20 — baru `Term - Ephemeral Port.md`).

**Langkah berikutnya**: urutan domain sudah ditetapkan eksplisit di `CLAUDE.md` §14 ("Domain sequence after `40 Databases`"), disusun berdasarkan tujuan pembaca di §2, bukan urutan folder `10`–`92` secara linear. `40 Databases`, `20 Go Language`, `50 Concurrency and Performance`, `90 Architecture and Design`, `30 APIs and Web`, `80 Security`, dan `70 Infrastructure and Delivery` sudah selesai; langkah terakhir dalam sequence ini adalah:

1. `60 Distributed Systems` — 39 note senior, dikerjakan terakhir supaya bertumpu pada fondasi yang sudah selesai (`50 Concurrency and Performance`, `30 APIs and Web`, `70 Infrastructure and Delivery`) — di sinilah kelanjutan senior `40 Databases` (sharding, replication, migrasi berbasis CDC) akhirnya ditulis. **← berikutnya.**

Setelah `60 Distributed Systems` selesai, seluruh domain concept note (`10`–`90`) akan lengkap sampai kedalaman maksimal masing-masing. Yang tersisa setelah itu murni `92 Tools` (28 note), `94 Case Studies` (25), `95 Projects` (3), dan sisa `99 Glossary` (19) — sejauh ini terus di-interleave per aturan §14, tapi kalau tersisa sendirian di akhir, dikerjakan sebagai penyisiran domain masing-masing.

Beberapa link forward yang ditulis di batch-batch sebelumnya menunjuk ke note yang belum ada filenya (`Sharding Strategies and Hot Partitions`, `Consistent Hashing`, `Change Data Capture`, seluruh tool note dan case study yang ditautkan dari `70 Infrastructure and Delivery` dan `80 Security`, dll.) — seluruhnya **sudah terdaftar di [[Vault Manifest]]**, jadi tidak perlu entri baru di tabel backlog di atas; ini murni "belum ditulis", bukan "gap kurikulum".

Setiap note yang sudah ditulis mengikuti struktur lengkap `02 Templates/Concept Note Template.md` (TL;DR, Problem, Intuition dengan catatan kebocoran analogi, How It Works, In Go, In His Stack, Trade-offs, 3 Common Mistakes, Exercises dengan soal desain terbuka, Self-Check, Connected Notes, Further Reading, Catatan Saya kosong) — pertahankan standar yang sama persis untuk note-note berikutnya, jangan diringkas.

## Connected Notes

- [[Vault Manifest]] — daftar lengkap seluruh note yang direncanakan; sumber kebenaran untuk backlog implisit.
- [[Curriculum Changelog]] — tempat mencatat topik yang ditambahkan di luar kurikulum asli, termasuk dua topik di atas.
- [[Progress Tracker]] — status note yang *sudah* ditulis; Backlog adalah kebalikannya (yang belum).

## Catatan Saya

*Tulis di sini kalau kamu menemukan konsep yang menurutmu perlu ditambahkan tapi belum ada di manifest.*
