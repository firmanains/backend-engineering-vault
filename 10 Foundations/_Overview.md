---
title: Foundations Overview
type: concept
level: junior
domain: foundations
status: mastered
difficulty: 1
est_minutes: 8
prerequisites: []
next: ["[[Processes vs Threads]]"]
tags: [backend, foundations]
created: 2026-07-26
---

## Apa Ini

Domain ini adalah lapisan paling dekat dengan mesin: bagaimana proses dan thread benar-benar berjalan, bagaimana memori dibagi antara stack dan heap, bagaimana sistem operasi menangani sebuah koneksi jaringan dari packet mentah sampai byte yang dibaca aplikasi, dan bagaimana TCP, DNS, TLS, serta HTTP membangun lapisan demi lapisan di atas semua itu.

Semua folder lain di vault ini — bahasa Go, desain API, database, concurrency — diam-diam mengasumsikan kamu paham apa yang terjadi di lapisan ini. Ketika sebuah koneksi tiba-tiba macet, ketika `TIME_WAIT` membanjiri server, atau ketika TLS handshake gagal di partner tertentu tapi tidak di partner lain, jawabannya hampir selalu ada di sini.

## Kenapa Ini Penting

Seorang engineer yang melewati domain ini biasanya bisa menulis kode yang bekerja di happy path, tapi tidak punya kerangka untuk mendiagnosis kegagalan jaringan di production — kenapa satu koneksi menggantung, kenapa connection pool kehabisan slot, kenapa latency naik drastis saat traffic naik sedikit saja. Untuk pekerjaan integrasi sistem-ke-sistem dengan partner pemerintah/enterprise, domain ini bukan teori — ini yang membedakan "server-nya lambat" dari diagnosis yang benar-benar bisa dijelaskan ke partner.

## Reading Order

### Junior

1. [[Processes vs Threads]] — beda antara **wadah** (process, yang memiliki memori) dan **pelaksana** (thread, yang dijadwalkan CPU), dan di kategori mana goroutine berada.
2. [[Memory Layout - Stack vs Heap]] — kenapa Go memutuskan sebuah value hidup di stack atau heap, dan kenapa itu penting untuk performa.
3. [[Blocking vs Non-Blocking IO]] — kenapa satu model I/O bisa membuat satu request menahan seluruh server.
4. [[Syscalls and File Descriptors]] — apa yang sebenarnya terjadi saat kode memanggil `read()` atau membuka koneksi TCP.
5. [[How An OS Handles Network Connections]] — dari packet masuk NIC sampai byte muncul di buffer aplikasi.
6. [[The TCP-IP Model]] — kerangka berpikir empat/lima layer yang dipakai untuk mendiagnosis semua masalah jaringan setelah ini.
7. [[TCP Handshake and Connection Lifecycle]] — three-way handshake, `TIME_WAIT`, dan kenapa itu penting untuk connection pooling nanti.
8. [[TCP vs UDP]] — trade-off reliability vs latency, dan kapan kehilangan packet lebih baik daripada menunggu.
9. [[DNS Resolution]] — kenapa nama domain butuh beberapa round-trip sebelum permintaan HTTP pertama dikirim.
10. [[The TLS Handshake]] — bagaimana kerahasiaan dan autentikasi dinegosiasikan sebelum satu byte data dikirim.
11. [[HTTP 1.1 In Depth]] — method, status code, header, cookie, keep-alive — protokol yang dipakai hampir semua API di pekerjaan sehari-hari.
12. [[Introduction to HTTP2]] — multiplexing dan kenapa ia memecahkan head-of-line blocking di level aplikasi.

Domain ini tidak punya note intermediate atau senior tersendiri — kedalamannya di level itu diteruskan lewat domain lain (`50 Concurrency and Performance` untuk performa, `60 Distributed Systems` untuk banyak mesin sekaligus).

## Bagaimana Domain Ini Terhubung Dengan Domain Lain

- [[../20 Go Language/_Overview|Go Language Overview]] — Go membuat keputusan (goroutine di atas OS thread, GC yang mengelola heap) yang hanya masuk akal setelah paham domain ini.
- [[../30 APIs and Web/_Overview|APIs and Web Overview]] — HTTP di sini adalah dasar untuk semua desain API dan pola integrasi di domain itu.
- [[../50 Concurrency and Performance/_Overview|Concurrency and Performance Overview]] — scheduler Go dan profiling hanya masuk akal kalau sudah paham proses, thread, dan memori di domain ini.
- [[../80 Security/_Overview|Security Overview]] — TLS handshake di domain ini adalah dasar teknis untuk semua pembahasan enkripsi in-transit.

## Progress

```dataview
TABLE status, difficulty, est_minutes
FROM #foundations
WHERE type = "concept"
SORT difficulty ASC
```

## Catatan Saya

*Tulis di sini bagian mana dari domain ini yang paling langsung menjelaskan insiden nyata yang pernah kamu tangani.*
