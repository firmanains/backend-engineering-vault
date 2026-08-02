---
title: Projects Overview
type: concept
level: junior
domain: architecture
status: mastered
difficulty: 1
est_minutes: 8
prerequisites: []
next: ["[[Project 1 - Authenticated REST API]]"]
tags: [backend, architecture, go]
created: 2026-08-02
---

## Apa Ini

Tiga proyek capstone, satu per level (junior, intermediate, senior), masing-masing dirancang sebagai spesifikasi lengkap — bukan sekadar ide proyek satu kalimat. Setiap proyek mencantumkan tujuan belajar eksplisit, kebutuhan fungsional, milestone bertahap, definisi selesai yang konkret, dan checklist self-review. Semuanya ditulis dalam Go, dan masing-masing menuntut penerapan nyata konsep-konsep yang sudah dipelajari di level yang sesuai — proyek adalah tempat pengetahuan yang terpisah-pisah di banyak note akhirnya harus bekerja sama dalam satu sistem yang benar-benar berjalan.

## Kenapa Ini Penting

Membaca tentang connection pooling, circuit breaker, atau saga adalah satu hal — membangunnya sendiri dan melihatnya gagal dengan cara yang tidak terduga adalah hal yang sama sekali berbeda, dan jauh lebih berharga. Proyek-proyek ini sengaja dirancang untuk memaksa keputusan trade-off nyata (bukan cuma soal pilihan ganda di exercise), dan Project 3 secara eksplisit menuntut dokumen desain dengan ADR — praktik yang sama yang dibahas di [[../60 Distributed Systems/Writing Architecture Decision Records|Writing Architecture Decision Records]] — supaya kebiasaan mendokumentasikan keputusan terbentuk sejak sekarang, bukan nanti setelah jadi kebiasaan buruk yang harus diubah di tempat kerja.

## Reading Order

1. [[Project 1 - Authenticated REST API]] — level junior: REST API dengan autentikasi, database relasional, migration, layering bersih, upload/download file yang benar, dan test yang benar-benar berarti.
2. [[Project 2 - Worker Pool, Cache, and Streaming Service]] — level intermediate: service dengan worker pool, caching Redis, producer dan consumer Kafka, observability penuh, dan pola resiliensi.
3. [[Project 3 - A Small Distributed System]] — level senior: sistem terdistribusi kecil — beberapa service, saga, pola outbox, idempotency, dan skenario chaos — ditutup dengan dokumen desain berisi ADR.

## Bagaimana Domain Ini Terhubung Dengan Domain Lain

Ketiga proyek ini sengaja dirancang menarik konsep dari **banyak** domain sekaligus, bukan satu domain tunggal — Project 1 menarik dari `20 Go Language`, `30 APIs and Web`, `40 Databases`, dan `80 Security`; Project 2 menambahkan `50 Concurrency and Performance` dan `70 Infrastructure and Delivery`; Project 3 menambahkan `60 Distributed Systems` dan `90 Architecture and Design` secara penuh. Setiap spesifikasi proyek mencantumkan tautan eksplisit ke concept note yang relevan sebagai bacaan wajib sebelum memulai.

## Progress

```dataview
TABLE status, level
FROM #backend
WHERE type = "project"
SORT file.name ASC
```

## Catatan Saya

*Tulis di sini progresmu mengerjakan tiap proyek, keputusan desain yang kamu ambil dan kenapa, dan bagian mana yang paling sulit.*
