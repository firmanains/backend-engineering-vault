---
title: Level 1 - Junior Path
type: concept
level: junior
domain: foundations
status: mastered
difficulty: 1
est_minutes: 10
prerequisites: ["[[Backend Engineering Map]]"]
next: ["[[Level 2 - Intermediate Path]]"]
tags: [backend, meta]
created: 2026-07-26
---

Ini adalah urutan baca konkret untuk level junior, lintas semua domain, 89 note. Ikuti urutan ini dari atas ke bawah kalau kamu membaca secara *linear* (lihat [[How To Read This Vault]]). Setiap `_Overview.md` domain punya urutan yang sama, dipecah per folder — peta ini menyatukannya jadi satu jalur tunggal supaya kamu tidak perlu berpindah folder untuk tahu "selanjutnya apa".

Urutan domainnya sengaja bukan `10 → 20 → 30 ...` murni: `10 Foundations` dan `20 Go Language` datang dulu karena keduanya adalah alat berpikir dan alat menulis kode yang dipakai di semua bagian lain. `40 Databases` (bagian SQL) datang sebelum `30 APIs and Web` karena API yang kamu bangun di jalur ini akan langsung menyentuh database.

## 1. 10 Foundations — cara mesin dan jaringan bekerja (12 note)

1. Processes vs Threads — beda antara **wadah** (process, yang memiliki memori) dan **pelaksana** (thread, yang dijadwalkan CPU), dan di kategori mana goroutine berada.
2. Memory Layout - Stack vs Heap — kenapa Go memutuskan sebuah value hidup di stack atau heap, dan kenapa itu penting untuk performa.
3. Blocking vs Non-Blocking IO — kenapa satu model I/O bisa membuat satu request menahan seluruh server.
4. Syscalls and File Descriptors — apa yang sebenarnya terjadi saat kode memanggil `read()` atau membuka koneksi TCP.
5. How An OS Handles Network Connections — dari packet masuk NIC sampai byte muncul di buffer aplikasi.
6. The TCP-IP Model — kerangka berpikir empat/lima layer yang dipakai untuk mendiagnosis semua masalah jaringan setelah ini.
7. TCP Handshake and Connection Lifecycle — three-way handshake, `TIME_WAIT`, dan kenapa itu penting untuk connection pooling nanti.
8. TCP vs UDP — trade-off reliability vs latency, dan kapan kehilangan packet lebih baik daripada menunggu.
9. DNS Resolution — kenapa nama domain butuh beberapa round-trip sebelum permintaan HTTP pertama dikirim.
10. The TLS Handshake — bagaimana kerahasiaan dan autentikasi dinegosiasikan sebelum satu byte data dikirim.
11. HTTP 1.1 In Depth — method, status code, header, cookie, keep-alive — protokol yang dipakai hampir semua API di pekerjaan sehari-hari.
12. Introduction to HTTP2 — multiplexing dan kenapa ia memecahkan head-of-line blocking di level aplikasi.

## 2. 20 Go Language — bahasa dan idiomnya (17 note)

13. The Go Type System — dasar untuk semua yang berikutnya.
14. Structs and Methods — unit dasar pemodelan data di Go.
15. Pointer vs Value Receivers — keputusan yang salah di sini menyebabkan bug aliasing yang sulit dilacak.
16. Interfaces and Implicit Satisfaction — kenapa Go tidak butuh kata kunci `implements`, dan apa konsekuensinya untuk desain API.
17. Slice Internals — length, capacity, dan aliasing bug klasik saat slice dioper sebagai argumen.
18. Map Internals — bagaimana map bekerja di bawah, dan kenapa iterasi map tidak pernah berurutan.
19. Defer, Panic, and Recover — mekanisme cleanup dan error handling terakhir sebelum goroutine mati.
20. Errors as Values — filosofi error handling Go yang paling sering disalahpahami oleh pendatang dari bahasa lain.
21. Error Wrapping — `fmt.Errorf("...: %w", err)` dan kenapa itu penting untuk debugging di production.
22. Packages and Modules — unit organisasi kode dan dependency management.
23. The Go Toolchain — `go build`, `go test`, `go vet`, dan alat lain yang dipakai setiap hari.
24. Struct Tags and JSON Marshalling — bagaimana data Go berubah jadi payload API dan sebaliknya.
25. Table-Driven Tests — pola testing paling idiomatic di Go.
26. Stdlib Testing vs Testify — kapan assertion library membantu, kapan ia menyembunyikan informasi.
27. Mocking Through Interfaces — cara Go melakukan dependency injection untuk testing tanpa framework mocking berat.
28. Testing HTTP Handlers with httptest — menguji handler tanpa server sungguhan berjalan.
29. Meaningful Test Coverage — kenapa angka coverage 100% bisa berbahaya, bukan aman.

## 3. 40 Databases — SQL sebagai craft (9 note)

30. The Logical Order of Query Execution — kenapa `WHERE` tidak bisa melihat alias dari `SELECT`.
31. Join Types and Their Mental Models — INNER, LEFT, RIGHT, FULL, dan cara membayangkan masing-masing.
32. Aggregation and GROUP BY Semantics — kenapa kolom yang tidak di-aggregate harus ada di `GROUP BY`.
33. Subqueries vs CTEs — kapan CTE membuat query lebih terbaca, kapan ia hanya kosmetik.
34. Window Functions — menghitung tanpa mengubah jumlah baris hasil.
35. UNION vs UNION ALL — biaya tersembunyi dari deduplikasi otomatis.
36. NULL Semantics and Three-Valued Logic — kenapa `NULL = NULL` bukan `true`.
37. Set Operations in SQL — `INTERSECT`, `EXCEPT`, dan kapan dipakai dibanding `JOIN`.
38. Upserts — `INSERT ... ON CONFLICT` / `ON DUPLICATE KEY UPDATE` dan race condition yang sering dilupakan.

## 4. 40 Databases — fondasi database (12 note)

39. Relational Modelling — mengubah kebutuhan bisnis jadi tabel dan relasi.
40. Normalisation 1NF to 3NF — mengurangi duplikasi data, dan kapan itu prioritas yang salah.
41. Deliberate Denormalisation — denormalisasi sebagai keputusan sadar, bukan kemalasan.
42. Data Types and Constraints — memakai database untuk menegakkan invariant, bukan hanya menyimpan byte.
43. Index Basics — kenapa index mempercepat baca dengan mengorbankan tulis dan ruang.
44. Database Transactions — unit kerja yang harus sukses atau gagal seutuhnya.
45. ACID — empat jaminan yang mendasari kepercayaan pada database relasional.
46. Basic Isolation Levels — kenapa isolation level default MySQL dan PostgreSQL berbeda dan itu penting.
47. database-sql and sqlx — cara idiomatic memanggil SQL dari Go.
48. Prepared Statements — performa dan pertahanan dari SQL injection dalam satu mekanisme.
49. Connection Pooling — kenapa membuka koneksi baru tiap request adalah cara membunuh database-mu sendiri.
50. Database Migrations — mengubah skema tanpa kehilangan data atau uptime.

## 5. 30 APIs and Web — desain API, payload, dan HTTP di Go (21 note)

51. REST Principles — apa yang sebenarnya dijanjikan REST, bukan sekadar "API pakai JSON lewat HTTP".
52. Resource Modelling — memetakan domain bisnis jadi resource dan URL yang masuk akal.
53. Choosing Status Codes — kenapa `200` untuk semuanya adalah tanda API yang belum dipikirkan matang.
54. Idempotency — kenapa endpoint yang aman diretry adalah keharusan, bukan bonus.
55. Pagination - Offset vs Cursor — trade-off yang muncul begitu tabel tumbuh besar.
56. Filtering and Sorting — konvensi query parameter yang konsisten lintas endpoint.
57. API Versioning — bagaimana mengubah kontrak tanpa mematahkan klien lama.
58. Consistent Error Responses — format error yang bisa diandalkan klien untuk parsing otomatis.
59. OpenAPI — kontrak API sebagai dokumen yang bisa diperiksa mesin, bukan hanya dibaca manusia.
60. Content Types and multipart-form-data — bagaimana HTTP membedakan JSON, form, dan file upload.
61. Binary in JSON and the Base64 Tax — biaya nyata mengirim file lewat JSON.
62. Streaming vs Buffering — kapan seluruh body harus masuk memori, kapan tidak boleh.
63. Upload and Download Patterns — pola production untuk file besar tanpa menghabiskan memori server.
64. Pre-signed URLs — melepas beban transfer file dari server aplikasi ke object storage langsung.
65. Request Size Limits Along The Path — kenapa satu limit di aplikasi saja tidak cukup.
66. net-http Handlers and Middleware — fondasi HTTP server idiomatic di Go.
67. Routing in Go — dari `http.ServeMux` sampai router pihak ketiga, dan kapan masing-masing cukup.
68. Context Propagation in HTTP Servers — membawa deadline dan nilai request lintas layer.
69. Timeouts in HTTP Servers — mencegah satu request lambat menghabiskan seluruh server.
70. Graceful Shutdown — mematikan server tanpa memutus request yang sedang berjalan.
71. Load Balancing and Reverse Proxies — bagaimana traffic dibagi ke banyak instance sebelum sampai ke handler-mu.

## 6. 80 Security — dasar (10 note)

72. Password Hashing - bcrypt and argon2 — kenapa password tidak pernah disimpan, hanya hash-nya.
73. Sessions vs Tokens — dua model autentikasi dan implikasi statefulness-nya.
74. JWT - Structure, Signature, and When It Is The Wrong Tool — token yang sering dipakai secara salah.
75. OAuth2 Overview — delegasi akses tanpa berbagi password.
76. RBAC — mengelola izin lewat peran, bukan lewat pengecekan `if user.id == ...` yang tersebar.
77. The OWASP Top 10 — peta kerentanan yang paling sering muncul di web application.
78. SQL Injection — kenapa string concatenation di query adalah pintu masuk paling klasik.
79. XSS — kenapa output yang tidak di-escape bisa menjalankan kode di browser orang lain.
80. CSRF — kenapa session cookie saja tidak cukup untuk membuktikan permintaan itu disengaja user.
81. Secret Management — kenapa credential tidak pernah boleh ada di kode atau di Git.

## 7. 70 Infrastructure and Delivery — ops dasar (4 note)

82. Linux for Backend Engineers — perintah dan konsep yang dipakai setiap hari untuk debugging production.
83. Docker - Images, Layers, and Multi-Stage Builds for Go — mengemas binary Go jadi image kecil dan cepat dibuild.
84. Docker Compose for Local Development — menjalankan seluruh dependency lokal dengan satu perintah.
85. Configuration via Environment (12-Factor App) — kenapa konfigurasi tidak boleh hardcoded atau tercampur dengan kode.

## 8. 90 Architecture and Design — hygiene (4 note)

86. Handler-Service-Repository Layering — pemisahan tanggung jawab paling dasar yang membuat kode testable.
87. Manual Dependency Injection in Go — menyusun dependency tanpa framework DI, gaya idiomatic Go.
88. Git Workflow and Code Review — konvensi branching dan review yang membuat kerja tim tidak saling menabrak.
89. Semantic Versioning — mengomunikasikan breaking change lewat nomor versi, bukan lewat kejutan.

## Setelah Ini

Selesaikan Project 1 (`95 Projects/Project 1 - Authenticated REST API.md`) sebagai ujian praktis seluruh jalur ini, lalu lanjut ke [[Level 2 - Intermediate Path]].

## Connected Notes

- [[Backend Engineering Map]] — peta domain lengkap yang menjadi dasar urutan di atas.
- [[Vault Manifest]] — path lengkap tiap note yang disebut di sini.
- [[How To Read This Vault]] — pace yang disarankan (2–3 note baru per hari) untuk menjalani jalur ini.
- [[Level 2 - Intermediate Path]] — kelanjutan setelah 89 note ini selesai.

## Catatan Saya

*Tulis di sini urutan mana yang terasa salah setelah kamu benar-benar menjalaninya, dan kenapa.*
