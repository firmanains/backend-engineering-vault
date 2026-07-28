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

**Status per 2026-07-29:**

- **Seluruh level junior selesai (89/89 note)**: `10 Foundations` (12/12), `20 Go Language` (17/17 junior — domain ini memang hanya junior+intermediate, 6 note intermediate/"Advanced Go" masih belum ditulis), `30 APIs and Web` (21/21 junior), `40 Databases` (21/21 junior), `70 Infrastructure and Delivery` (4/4 junior: Linux for Backend Engineers, Docker - Images, Layers, and Multi-Stage Builds for Go, Docker Compose for Local Development, Configuration via Environment (12-Factor App)), `80 Security` (10/10 junior: Password Hashing, Sessions vs Tokens, JWT, OAuth2 Overview, RBAC, The OWASP Top 10, SQL Injection, XSS, CSRF, Secret Management), `90 Architecture and Design` (4/4 junior: Handler-Service-Repository Layering, Manual Dependency Injection in Go, Git Workflow and Code Review, Semantic Versioning).
- **Belum disentuh sama sekali**: seluruh note intermediate (117, termasuk 6 note "Advanced Go" di `20 Go Language` dan 14 note intermediate `40 Databases`: B+Tree Structure sampai Keeping Search in Sync with the Source of Truth) dan senior (52) di semua domain, seluruh tool note (28), seluruh case study (25), semua project (3), dan glossary (1/20 — baru `Term - Ephemeral Port.md`).

**Langkah berikutnya**: seluruh **89 note junior** sudah tuntas — mulai `01 Maps/Level 2 - Intermediate Path.md` (117 note), dimulai dari `20 Go Language` — 6 note Advanced Go yang belum ditulis (generics, reflection dan biayanya, embedding, functional options, designing stable library API, sentinel errors vs error types), lalu lanjut ke concurrency (`50`), database intermediate (`40`), dst. sesuai urutan di map itu.

Beberapa link forward yang ditulis di batch note junior kali ini menunjuk ke note intermediate/senior yang belum ada filenya (`The Three Pillars of Observability`, `Feature Flags`, `Kubernetes Core Concepts...`, `Key Management and Rotation`, `Threat Modelling with STRIDE`, `Audit Logging`, `Hexagonal and Clean Architecture in Go`, `Mentoring`, `Cross-Team Code Standards`, `Designing an API for a Partner You Do Not Control`, `Managing Technical Debt Explicitly`, dll.) — seluruhnya **sudah terdaftar di [[Vault Manifest]]**, jadi tidak perlu entri baru di tabel backlog di atas; ini murni "belum ditulis", bukan "gap kurikulum".

Setiap note yang sudah ditulis mengikuti struktur lengkap `02 Templates/Concept Note Template.md` (TL;DR, Problem, Intuition dengan catatan kebocoran analogi, How It Works, In Go, In His Stack, Trade-offs, 3 Common Mistakes, Exercises dengan soal desain terbuka, Self-Check, Connected Notes, Further Reading, Catatan Saya kosong) — pertahankan standar yang sama persis untuk note-note berikutnya, jangan diringkas.

## Connected Notes

- [[Vault Manifest]] — daftar lengkap seluruh note yang direncanakan; sumber kebenaran untuk backlog implisit.
- [[Curriculum Changelog]] — tempat mencatat topik yang ditambahkan di luar kurikulum asli, termasuk dua topik di atas.
- [[Progress Tracker]] — status note yang *sudah* ditulis; Backlog adalah kebalikannya (yang belum).

## Catatan Saya

*Tulis di sini kalau kamu menemukan konsep yang menurutmu perlu ditambahkan tapi belum ada di manifest.*
