---
title: Security Overview
type: concept
level: junior
domain: security
status: mastered
difficulty: 1
est_minutes: 8
prerequisites: ["[[../10 Foundations/_Overview|Foundations Overview]]"]
next: ["[[Password Hashing - bcrypt and argon2]]"]
tags: [backend, security]
created: 2026-07-26
---

## Apa Ini

Domain ini mencakup keamanan aplikasi dari dua ujung: dasar yang harus otomatis di level junior (password hashing, sessions vs tokens, JWT, OAuth2, RBAC, OWASP Top 10, secret management) dan lapisan senior yang lebih strategis (threat modelling, zero trust, mTLS, key management, audit logging, compliance).

## Kenapa Ini Penting

HTTPS tidak membuat API aman — itu hanya melindungi data selama transit, bukan dari otentikasi yang lemah, otorisasi yang salah, atau input yang tidak divalidasi. Untuk sistem legal-services milik pemerintah, kesalahan di domain ini bukan hanya bug, tapi berpotensi jadi insiden hukum dan kepercayaan publik. Bagian senior domain ini — terutama compliance trail — juga langsung relevan karena sistem pemerintah punya kebutuhan audit yang tidak selalu ada di sistem enterprise biasa.

## Reading Order

### Junior

1. [[Password Hashing - bcrypt and argon2]] — kenapa password tidak pernah disimpan, hanya hash-nya.
2. [[Sessions vs Tokens]] — dua model autentikasi dan implikasi statefulness-nya.
3. [[JWT - Structure, Signature, and When It Is The Wrong Tool]] — token yang sering dipakai secara salah.
4. [[OAuth2 Overview]] — delegasi akses tanpa berbagi password.
5. [[RBAC]] — mengelola izin lewat peran, bukan lewat pengecekan `if user.id == ...` yang tersebar.
6. [[The OWASP Top 10]] — peta kerentanan yang paling sering muncul di web application.
7. [[SQL Injection]] — kenapa string concatenation di query adalah pintu masuk paling klasik.
8. [[XSS]] — kenapa output yang tidak di-escape bisa menjalankan kode di browser orang lain.
9. [[CSRF]] — kenapa session cookie saja tidak cukup untuk membuktikan permintaan itu disengaja user.
10. [[Secret Management]] — kenapa credential tidak pernah boleh ada di kode atau di Git.

### Senior

11. [[Threat Modelling with STRIDE]] — kerangka sistematis menemukan cara sistem bisa diserang sebelum diserang sungguhan.
12. [[Zero Trust]] — tidak ada jaringan yang "aman secara default" hanya karena berada di dalam firewall.
13. [[mTLS]] — kedua pihak saling membuktikan identitas, bukan hanya server ke klien.
14. [[Key Management and Rotation]] — kunci kriptografi yang dikelola sebagai siklus hidup, bukan sekali dibuat lalu dilupakan.
15. [[Encryption at Rest vs In Transit]] — dua lapisan perlindungan data yang melindungi dari ancaman berbeda.
16. [[Audit Logging]] — mencatat siapa melakukan apa, secara tidak bisa diubah, untuk investigasi setelah kejadian.
17. [[Compliance Trails for Government Systems]] — kebutuhan audit yang lahir dari regulasi, bukan dari pilihan teknis semata.

## Bagaimana Domain Ini Terhubung Dengan Domain Lain

- [[../10 Foundations/_Overview|Foundations Overview]] — TLS handshake di domain itu adalah dasar teknis untuk mTLS dan encryption in-transit di sini.
- [[../30 APIs and Web/_Overview|APIs and Web Overview]] — webhook security dan idempotency key di domain itu bersinggungan langsung dengan JWT dan RBAC di sini.
- [[../60 Distributed Systems/_Overview|Distributed Systems Overview]] — audit logging dan compliance trail di sini adalah prasyarat operasional untuk disaster recovery dan incident command di domain senior itu.
- [[../92 Tools/_Overview|Tools Overview]] — Vault (secret management) dan Consul (mTLS di service mesh) mengimplementasikan concept di domain ini.

## Progress

```dataview
TABLE status, level, difficulty
FROM #security
WHERE type = "concept"
SORT level ASC, difficulty ASC
```

## Catatan Saya

*Tulis di sini kebijakan security di sistem pemerintah tempatmu bekerja yang paling sering menimbulkan friksi dengan tim developer.*
