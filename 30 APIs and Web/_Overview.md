---
title: APIs and Web Overview
type: concept
level: junior
domain: apis
status: mastered
difficulty: 1
est_minutes: 10
prerequisites: ["[[../10 Foundations/_Overview|Foundations Overview]]"]
next: ["[[REST Principles]]"]
tags: [backend, apis, integration]
created: 2026-07-26
---

## Apa Ini

Domain ini mencakup HTTP dan REST, desain payload dan file handling, gRPC, GraphQL, protokol realtime (WebSocket, SSE, long polling), messaging dan streaming, dan yang paling berat bobotnya di vault ini: **pola integrasi dengan sistem pihak lain yang tidak bisa kamu ubah**, dan resilience (timeout, retry, circuit breaker) yang menjaga integrasi itu tetap hidup saat pihak lain gagal.

Ini adalah domain paling besar di vault ini secara sengaja. Pekerjaan sehari-harimu adalah koordinasi teknis lintas 13+ aplikasi legal-services milik pemerintah — yang berarti sebagian besar masalah nyata yang kamu hadapi bukan "bagaimana menulis satu service", tapi "bagaimana dua sistem yang tidak saling percaya penuh bisa bekerja sama dengan andal".

## Kenapa Ini Penting

Sebuah API yang dirancang tanpa mempertimbangkan idempotency, tanpa versi yang jelas, atau tanpa strategi retry yang aman, akan bekerja sempurna di demo dan gagal secara spektakuler di production — dua kali dipanggil karena timeout partner terlalu pendek, retry storm yang mengubah gangguan kecil jadi outage total, atau webhook yang diam-diam berhenti terkirim selama berjam-jam tanpa ada yang sadar. Untuk engineer yang berhadapan dengan partner pemerintah dan enterprise yang punya fleksibilitas teknis terbatas, domain ini adalah tempat sebagian besar kegagalan produksi sebenarnya lahir.

## Reading Order

### Junior — API design, payload, dan HTTP di Go

1. [[REST Principles]] — apa yang sebenarnya dijanjikan REST, bukan sekadar "API pakai JSON lewat HTTP".
2. [[Resource Modelling]] — memetakan domain bisnis jadi resource dan URL yang masuk akal.
3. [[Choosing Status Codes]] — kenapa `200` untuk semuanya adalah tanda API yang belum dipikirkan matang.
4. [[Idempotency]] — kenapa endpoint yang aman diretry adalah keharusan, bukan bonus.
5. [[Pagination - Offset vs Cursor]] — trade-off yang muncul begitu tabel tumbuh besar.
6. [[Filtering and Sorting]] — konvensi query parameter yang konsisten lintas endpoint.
7. [[API Versioning]] — bagaimana mengubah kontrak tanpa mematahkan klien lama.
8. [[Consistent Error Responses]] — format error yang bisa diandalkan klien untuk parsing otomatis.
9. [[OpenAPI]] — kontrak API sebagai dokumen yang bisa diperiksa mesin, bukan hanya dibaca manusia.
10. [[Content Types and multipart-form-data]] — bagaimana HTTP membedakan JSON, form, dan file upload.
11. [[Binary in JSON and the Base64 Tax]] — biaya nyata mengirim file lewat JSON.
12. [[Streaming vs Buffering]] — kapan seluruh body harus masuk memori, kapan tidak boleh.
13. [[Upload and Download Patterns]] — pola production untuk file besar tanpa menghabiskan memori server.
14. [[Pre-signed URLs]] — melepas beban transfer file dari server aplikasi ke object storage langsung.
15. [[Request Size Limits Along The Path]] — kenapa satu limit di aplikasi saja tidak cukup.
16. [[net-http Handlers and Middleware]] — fondasi HTTP server idiomatic di Go.
17. [[Routing in Go]] — dari `http.ServeMux` sampai router pihak ketiga, dan kapan masing-masing cukup.
18. [[Context Propagation in HTTP Servers]] — membawa deadline dan nilai request lintas layer.
19. [[Timeouts in HTTP Servers]] — mencegah satu request lambat menghabiskan seluruh server.
20. [[Graceful Shutdown]] — mematikan server tanpa memutus request yang sedang berjalan.
21. [[Load Balancing and Reverse Proxies]] — bagaimana traffic dibagi ke banyak instance sebelum sampai ke handler-mu.

### Intermediate — integrasi, protokol lain, messaging, resilience

22. [[Designing an API for a Partner You Do Not Control]] — constraint desain saat pihak lain tidak bisa berubah.
23. [[Contract Negotiation and Versioning]] — menyepakati kontrak dengan pihak yang tidak selalu punya tim teknis fleksibel.
24. [[Webhooks and How to Secure Them]] — mengamankan endpoint yang menerima panggilan dari luar.
25. [[Polling vs Push]] — trade-off kesederhanaan vs latency dan beban.
26. [[File-Based Integration]] — ketika partner masih mengandalkan file, bukan API.
27. [[Batch vs Realtime Integration]] — memilih ritme pertukaran data yang sesuai kebutuhan bisnis.
28. [[Handling an Unreliable Counterparty]] — desain yang tetap benar walau pihak lain lambat atau gagal.
29. [[Sandbox Environments]] — menguji integrasi tanpa menyentuh data produksi pihak lain.
30. [[Integration Testing Across an Organisational Boundary]] — menguji sesuatu yang separuh kendalinya bukan milikmu.
31. [[gRPC and Protobuf]] — RPC berbasis schema dengan performa yang jauh lebih baik dari JSON over HTTP.
32. [[Schema Evolution in Protobuf]] — mengubah kontrak binary tanpa mematahkan klien lama.
33. [[GraphQL and Its Trade-offs]] — fleksibilitas query di sisi klien, dengan biaya kompleksitas di sisi server.
34. [[WebSocket]] — komunikasi dua arah yang tetap terbuka.
35. [[Server-Sent Events]] — push satu arah yang lebih sederhana dari WebSocket.
36. [[Long Polling]] — realtime semu di atas infrastruktur HTTP biasa.
37. [[Queue vs Log Semantics]] — dua model perpesanan dengan jaminan yang sangat berbeda.
38. [[Topics, Partitions, and Offsets]] — unit dasar sistem berbasis log seperti Kafka.
39. [[Consumer Groups and Rebalancing]] — bagaimana banyak consumer membagi beban satu topic.
40. [[Delivery Semantics]] — at-most-once, at-least-once, dan kenapa exactly-once selalu punya syarat tersembunyi.
41. [[Idempotent Consumers]] — bertahan dari pesan yang diterima dua kali.
42. [[The Transactional Outbox Pattern]] — menjaga database dan message broker tetap konsisten tanpa distributed transaction.
43. [[Dead Letter Queues]] — tempat pesan yang gagal diproses berulang kali, bukan hilang begitu saja.
44. [[Ordering Guarantees in Streaming Systems]] — kapan urutan pesan dijamin, dan kapan tidak.
45. [[Backpressure]] — apa yang terjadi ketika consumer lebih lambat dari producer.
46. [[Timeout Budgets]] — membagi total waktu yang boleh dipakai lintas beberapa panggilan berantai.
47. [[Retries with Exponential Backoff and Jitter]] — retry yang membantu, bukan memperparah insiden.
48. [[Circuit Breakers]] — berhenti memanggil layanan yang sedang jelas rusak.
49. [[Bulkheads]] — mengisolasi kegagalan supaya tidak menular ke bagian sistem lain.
50. [[Rate Limiting Algorithms]] — token bucket, leaky bucket, sliding window.
51. [[Load Shedding]] — menolak sebagian request secara sengaja untuk menyelamatkan sisanya.
52. [[Graceful Degradation]] — tetap berfungsi sebagian saat komponen pendukung mati.

## Bagaimana Domain Ini Terhubung Dengan Domain Lain

- [[../10 Foundations/_Overview|Foundations Overview]] — HTTP, TCP, dan TLS di domain itu adalah fondasi teknis semua yang ada di sini.
- [[../40 Databases/_Overview|Databases Overview]] — transactional outbox pattern dan idempotency di sini bersandar langsung pada transaction dan constraint di domain database.
- [[../50 Concurrency and Performance/_Overview|Concurrency and Performance Overview]] — worker pool dan caching dari domain itu adalah alat implementasi untuk resilience dan messaging di sini.
- [[../60 Distributed Systems/_Overview|Distributed Systems Overview]] — saga dan transactional outbox di sini adalah pengantar praktis untuk teori saga/2PC yang dibahas penuh di domain senior itu.
- [[../92 Tools/_Overview|Tools Overview]] — Kafka, RabbitMQ, NATS, dan Nginx semuanya mengimplementasikan concept di domain ini.

## Progress

```dataview
TABLE status, level, difficulty
FROM #apis
WHERE type = "concept"
SORT level ASC, difficulty ASC
```

## Catatan Saya

*Tulis di sini partner integrasi paling menyulitkan yang pernah kamu hadapi, dan concept mana di atas yang seharusnya dipakai lebih awal.*
