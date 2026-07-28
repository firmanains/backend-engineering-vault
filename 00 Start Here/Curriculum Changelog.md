---
title: Curriculum Changelog
type: concept
level: junior
domain: foundations
status: mastered
difficulty: 1
est_minutes: 4
prerequisites: ["[[Vault Manifest]]"]
next: []
tags: [backend, meta]
created: 2026-07-26
---

Kurikulum di `CLAUDE.md` §11 adalah **cakupan minimum**, bukan batas atas. File ini mencatat setiap topik yang ditambahkan di luar kurikulum asli, dan alasannya — supaya enam bulan dari sekarang jelas mana yang berasal dari spec awal dan mana yang ditambahkan sambil menulis.

## Batch 0

### Ditambahkan: Load Balancing and Reverse Proxies

- **Domain**: `30 APIs and Web/` (junior)
- **Kenapa**: `92 Tools/_Overview.md` menugaskan Nginx sebagai tool `deep`, dan aturan §9 mewajibkan setiap tool note menautkan concept yang diimplementasikannya. Kurikulum asli tidak punya concept note untuk load balancing atau reverse proxy sama sekali — topik itu tersirat di "how an OS handles network connections" tapi tidak pernah eksplisit. Tanpa note ini, Nginx tidak punya concept note untuk ditautkan.

### Ditambahkan: Service Discovery

- **Domain**: `70 Infrastructure and Delivery/` (intermediate)
- **Kenapa**: Consul ditugaskan sebagai tool tier `orientation`, dan fungsi utamanya — service discovery — tidak muncul sebagai concept note mana pun di kurikulum asli. Ditempatkan di `70` alih-alih `60 Distributed Systems` karena di level intermediate ini dibahas sebagai kebutuhan operasional container orchestration, bukan teori distributed systems; teorinya (failure detectors, quorum) tetap tinggal di `60`.

## Batch 1

### Koreksi: daftar starter glossary di Batch 0 salah

Saat menulis batch pertama concept note (`10 Foundations`), ditemukan bahwa daftar starter glossary di [[Vault Manifest]] keliru: sebagian besar istilah yang didaftar di sana (Idempotency, ACID, CAP Theorem, MVCC, Sharding, Goroutine, JWT, Saga, dan lainnya) **sudah punya concept note penuh sendiri** di salah satu domain `10`–`90`. Menulis "Term - Idempotency.md" di `99 Glossary/` di samping "30 APIs and Web/Idempotency.md" akan menghasilkan dua note untuk satu ide yang sama — membingungkan tautan mana yang benar dituju, dan melanggar prinsip "one note, one concept" secara tidak langsung (dua note, satu concept).

**Perbaikan**: aturan glossary sekarang eksplisit — sebuah istilah hanya masuk `99 Glossary/` kalau ia **tidak** punya (dan tidak direncanakan punya) concept note sendiri. Daftar starter di [[Vault Manifest]] sudah diganti dengan istilah pendukung yang genuinely tidak cukup besar untuk template concept note penuh (misalnya `Ephemeral Port`, `TIME_WAIT`, `Optimistic Locking`, `Thundering Herd`), bukan istilah yang sudah jadi topik utama sebuah folder domain.

## Keputusan Pemetaan Domain (bukan penambahan topik, tapi keputusan struktural)

Kurikulum §11 mengelompokkan topik per level (Level 1/2/3) tanpa menugaskan folder domain secara eksplisit, sementara §5 menugaskan folder berdasarkan nama singkat ("50 Concurrency and Performance ← goroutines, profiling, latency, caching"). Beberapa pemetaan yang tidak jelas secara langsung, diputuskan sebagai berikut:

- **Go concurrency + Go runtime** (goroutine, channel, GMP scheduler, GC, pprof, dll.) ditempatkan di `50 Concurrency and Performance`, bukan `20 Go Language`, mengikuti deskripsi folder di §5 yang eksplisit menyebut "goroutines, profiling". `20 Go Language` menyimpan mekanik bahasa murni: type system, struct, interface, error handling, testing, dan "Advanced Go" (generics, reflection, embedding, functional options).
- **Caching** ditempatkan di `50 Concurrency and Performance` mengikuti deskripsi folder yang sama ("...caching"), bukan di `40 Databases`.
- **Resilience** (timeout, retry, circuit breaker, rate limiting) ditempatkan di `30 APIs and Web`, karena pola ini diterapkan tepat di titik memanggil layanan lain — satu paket dengan integration patterns yang sudah ada di domain itu.
- **Messaging and streaming** ditempatkan di `30 APIs and Web`, karena deskripsi folder itu sendiri menyebut "realtime, integration", dan messaging pada level intermediate adalah pola integrasi antar sistem, bukan teori distributed systems senior.
- **Search** ditempatkan di `40 Databases`, karena inverted index adalah struktur data penyimpanan dan pencarian — satu keluarga dengan "beyond relational".
- **Observability** ditempatkan di `70 Infrastructure and Delivery`, karena secara operasional selalu dibahas berdampingan dengan CI/CD dan container orchestration di kurikulum, meski tidak punya folder sendiri di §5.
- **Engineering hygiene** dipecah dua: layering/DI/git-workflow/semver masuk `90 Architecture and Design` (karena ini soal bagaimana tim menyusun dan menyepakati kode), sedangkan structured logging dan konfigurasi 12-factor masuk `70 Infrastructure and Delivery` (karena ini soal bagaimana service berjalan di production).

Detail lengkap pemetaan setiap note ke folder ada di [[Vault Manifest]].

## Connected Notes

- [[Vault Manifest]] — daftar lengkap note, termasuk dua topik tambahan di atas.
- [[Backlog]] — tempat kedua topik tambahan tercatat sampai file-nya benar-benar ditulis.

## Catatan Saya

*Kosong — file ini murni catatan sistem, bukan tempat refleksi pribadi.*
