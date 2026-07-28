---
title: Databases Overview
type: concept
level: junior
domain: databases
status: mastered
difficulty: 1
est_minutes: 10
prerequisites: ["[[../10 Foundations/_Overview|Foundations Overview]]"]
next: ["[[The Logical Order of Query Execution]]"]
tags: [backend, databases]
created: 2026-07-26
---

## Apa Ini

Domain ini mencakup SQL sebagai craft (bukan sekadar sintaks — cara berpikir tentang query), modelling relasional, index, transaction dan isolation level, lalu naik ke B+Tree, MVCC, replication, sharding, storage engine (row vs columnar, LSM vs B-Tree), penyimpanan di luar relasional, dan search sebagai bentuk khusus pencarian data.

Ini adalah domain kedua terbesar di vault ini secara sengaja: stack harianmu adalah **MariaDB**, dan hampir setiap bug produksi yang sulit — data yang tidak konsisten, laporan yang lambat, counter yang salah — pada akhirnya adalah bug pemahaman tentang database, bukan bug bahasa pemrograman.

## Kenapa Ini Penting

Seorang engineer yang bisa menulis query yang *bekerja* tapi tidak paham `EXPLAIN`, isolation level, atau kenapa index tertentu tidak dipakai optimizer, akan menghabiskan waktu bertahun-tahun menambal gejala (menambah index sembarangan, menambah cache, menambah read replica) tanpa pernah tahu mana dari ketiganya yang benar-benar menjawab akar masalahnya. Domain ini membangun kerangka untuk membuat keputusan itu secara sadar.

## Reading Order

### Junior — SQL sebagai craft

1. [[The Logical Order of Query Execution]] — kenapa `WHERE` tidak bisa melihat alias dari `SELECT`.
2. [[Join Types and Their Mental Models]] — INNER, LEFT, RIGHT, FULL, dan cara membayangkan masing-masing.
3. [[Aggregation and GROUP BY Semantics]] — kenapa kolom yang tidak di-aggregate harus ada di `GROUP BY`.
4. [[Subqueries vs CTEs]] — kapan CTE membuat query lebih terbaca, kapan ia hanya kosmetik.
5. [[Window Functions]] — menghitung tanpa mengubah jumlah baris hasil.
6. [[UNION vs UNION ALL]] — biaya tersembunyi dari deduplikasi otomatis.
7. [[NULL Semantics and Three-Valued Logic]] — kenapa `NULL = NULL` bukan `true`.
8. [[Set Operations in SQL]] — `INTERSECT`, `EXCEPT`, dan kapan dipakai dibanding `JOIN`.
9. [[Upserts]] — `INSERT ... ON CONFLICT` / `ON DUPLICATE KEY UPDATE` dan race condition yang sering dilupakan.

### Junior — fondasi database

10. [[Relational Modelling]] — mengubah kebutuhan bisnis jadi tabel dan relasi.
11. [[Normalisation 1NF to 3NF]] — mengurangi duplikasi data, dan kapan itu prioritas yang salah.
12. [[Deliberate Denormalisation]] — denormalisasi sebagai keputusan sadar, bukan kemalasan.
13. [[Data Types and Constraints]] — memakai database untuk menegakkan invariant, bukan hanya menyimpan byte.
14. [[Index Basics]] — kenapa index mempercepat baca dengan mengorbankan tulis dan ruang.
15. [[Database Transactions]] — unit kerja yang harus sukses atau gagal seutuhnya.
16. [[ACID]] — empat jaminan yang mendasari kepercayaan pada database relasional.
17. [[Basic Isolation Levels]] — kenapa isolation level default MySQL dan PostgreSQL berbeda dan itu penting.
18. [[database-sql and sqlx]] — cara idiomatic memanggil SQL dari Go.
19. [[Prepared Statements]] — performa dan pertahanan dari SQL injection dalam satu mekanisme.
20. [[Connection Pooling]] — kenapa membuka koneksi baru tiap request adalah cara membunuh database-mu sendiri.
21. [[Database Migrations]] — mengubah skema tanpa kehilangan data atau uptime.

### Intermediate — mendalam

22. [[B+Tree Structure]] — struktur data di balik hampir semua index relasional.
23. [[Composite Indexes and the Leftmost-Prefix Rule]] — kenapa urutan kolom di index penting.
24. [[Covering Indexes]] — index yang membuat database tidak perlu menyentuh tabel sama sekali.
25. [[Reading EXPLAIN]] — membaca rencana eksekusi query seperti seorang DBA.
26. [[Isolation Levels and Their Anomalies]] — dirty read, non-repeatable read, phantom read, write skew.
27. [[MVCC]] — bagaimana database membaca tanpa memblokir penulisan yang sedang berjalan.
28. [[Locking and Row Locks]] — mengendalikan akses konkuren di level baris, bukan tabel.
29. [[Deadlocks]] — dua transaksi saling menunggu, dan cara mendiagnosisnya.
30. [[The N+1 Query Problem]] — jebakan performa paling umum di aplikasi berbasis ORM.
31. [[Tuning the Connection Pool]] — ukuran pool yang tepat untuk beban nyata.
32. [[Read Replicas and Replication Lag]] — membagi beban baca, dan konsekuensi data yang sedikit tertinggal.
33. [[Partitioning]] — memecah tabel besar tanpa memecah aplikasi.
34. [[Introduction to Sharding]] — memecah data lintas instance database.
35. [[Materialised Views]] — menyimpan hasil query mahal sebagai tabel yang di-refresh.

### Intermediate — storage engine dan beyond relational

36. [[Row-Oriented vs Columnar Storage]] — dua cara menyusun data di disk untuk beban kerja berbeda.
37. [[OLTP vs OLAP vs HTAP]] — kenapa query analitik di database transaksional adalah bau arsitektur.
38. [[LSM-Trees vs B-Trees]] — dua struktur penyimpanan dengan trade-off write vs read yang berlawanan.
39. [[Write Amplification and Compression]] — biaya tersembunyi dari storage engine berbasis LSM.
40. [[Beyond Relational - Document, Key-Value, Wide-Column, Graph, and Time-Series Stores]] — memilih model data berdasarkan pola akses, bukan tren.
41. [[Inverted Indexes and How Search Engines Work]] — struktur data di balik pencarian full-text.
42. [[Relevance Scoring]] — kenapa hasil pencarian punya urutan, bukan sekadar cocok atau tidak.
43. [[Keeping Search in Sync with the Source of Truth]] — masalah nyata di balik setiap fitur search yang "kadang hasilnya aneh".

## Bagaimana Domain Ini Terhubung Dengan Domain Lain

- [[../20 Go Language/_Overview|Go Language Overview]] — `database/sql` dan `sqlx` di sini adalah titik temu langsung antara kedua domain.
- [[../30 APIs and Web/_Overview|APIs and Web Overview]] — pagination, idempotency, dan transactional outbox di domain itu semuanya bersandar pada transaction dan index di sini.
- [[../50 Concurrency and Performance/_Overview|Concurrency and Performance Overview]] — connection pool tuning di sini berpasangan langsung dengan goroutine dan worker pool di domain itu.
- [[../60 Distributed Systems/_Overview|Distributed Systems Overview]] — sharding dan replication di sini adalah pengantar untuk consistent hashing dan CDC di domain senior itu.
- [[../92 Tools/_Overview|Tools Overview]] — PostgreSQL, MySQL/MariaDB, ClickHouse, Redis, dan Elasticsearch semuanya mengimplementasikan concept di domain ini.

## Progress

```dataview
TABLE status, level, difficulty
FROM #databases
WHERE type = "concept"
SORT level ASC, difficulty ASC
```

## Catatan Saya

*Tulis di sini query MariaDB paling membingungkan yang pernah kamu temui di kerjaan, dan concept mana di atas yang menjelaskannya.*
