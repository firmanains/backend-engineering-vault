---
title: Tools Overview
type: concept
level: junior
domain: tools
status: mastered
difficulty: 1
est_minutes: 10
prerequisites: []
next: []
tags: [backend, tools]
created: 2026-07-26
---

## Aturan Concept vs Tool

**Kalau sesuatu punya vendor dan nomor versi, ia masuk di sini.** Semua yang durable — ide yang bertahan lepas dari produk mana pun yang mengimplementasikannya — tinggal di folder domain bernomor (`10` sampai `90`). Aturan ini yang menjaga vault ini tidak lapuk: Kafka bisa saja digantikan produk lain sepuluh tahun lagi, tapi [[../30 APIs and Web/Queue vs Log Semantics|Queue vs Log Semantics]] tetap relevan.

Setiap tool note di folder ini **wajib** menautkan concept note yang diimplementasikannya (field `implements:` di frontmatter), dan sebaliknya, concept note yang punya implementasi nyata wajib menautkan minimal satu tool note. Ini yang membuat vault berguna baik saat belajar maupun saat kerja nyata.

## Depth Tier

Tidak semua tool layak investasi penulisan yang sama. Tier menentukan **berapa dalam** dan **berapa banyak sub-note**:

| Tier | Definisi | Bentuk note |
|---|---|---|
| **Deep** | Dipakai, atau akan dipakai, langsung. | Note penuh + sub-note (config nyata, debugging operasional, performance tuning). |
| **Working** | Kemungkinan besar ditemui, dipilih, atau dievaluasi. | Note penuh, tanpa sub-note, fokus ke mental model dan kriteria memilih. |
| **Orientation** | Perlu tahu masalah apa yang ia selesaikan dan kapan orang akan memakainya — tidak perlu lebih dalam dari itu sekarang. | Satu note, ~600–900 kata, berat di "kapan memilih ini". |

## Reading Order — dikelompokkan per tier

### Deep (9 tool + 3 sub-note database)

1. [[PostgreSQL]] — tetap deep meski bukan daily driver: paling terdokumentasi untuk *belajar dari*, dan perbandingannya dengan MySQL mengajarkan lebih banyak tentang database daripada keduanya sendiri-sendiri.
2. [[PostgreSQL - Features Worth Switching For]] — `RETURNING`, `JSONB`, `CHECK` constraint sungguhan, array, `ON CONFLICT`, partial/expression index, CTE, window function, `LISTEN`/`NOTIFY`.
3. [[PostgreSQL - Locking and SELECT FOR UPDATE]] — row lock, `FOR UPDATE` vs `FOR SHARE`, `SKIP LOCKED`, `NOWAIT`, advisory lock.
4. [[PostgreSQL vs MySQL - How To Choose]] — perbandingan jujur, termasuk kasus MySQL yang benar-benar menang.
5. [[MySQL and MariaDB - Dialect and Behaviour]] — InnoDB, `INSERT IGNORE`, `ON DUPLICATE KEY UPDATE`, gap lock, `utf8mb4`, online DDL — daily driver-mu.
6. [[Redis]] — struktur data di luar `GET`/`SET`, persistence, eviction, `SETNX`, streams, pipelining.
7. [[Kafka]] — implementasi utama [[../30 APIs and Web/Queue vs Log Semantics|Queue vs Log Semantics]] dan [[../30 APIs and Web/Consumer Groups and Rebalancing|Consumer Groups and Rebalancing]].
8. [[Elasticsearch]] — implementasi utama [[../40 Databases/Inverted Indexes and How Search Engines Work|Inverted Indexes and How Search Engines Work]].
9. [[Docker]] — implementasi praktis [[../70 Infrastructure and Delivery/Immutable Infrastructure vs Configuration Management|Immutable Infrastructure vs Configuration Management]].
10. [[Kubernetes]] — implementasi utama container orchestration di `70 Infrastructure and Delivery`.
11. [[Jenkins]] — implementasi utama [[../70 Infrastructure and Delivery/CI-CD Pipelines|CI-CD Pipelines]].
12. [[Nginx]] — implementasi utama [[../30 APIs and Web/Load Balancing and Reverse Proxies|Load Balancing and Reverse Proxies]].

### Working (9)

13. [[Prometheus]] — implementasi utama [[../70 Infrastructure and Delivery/Pull vs Push Metrics Collection|Pull vs Push Metrics Collection]].
14. [[Grafana]] — implementasi utama [[../70 Infrastructure and Delivery/Dashboard Design|Dashboard Design]].
15. [[ClickHouse]] — implementasi utama [[../40 Databases/OLTP vs OLAP vs HTAP|OLTP vs OLAP vs HTAP]] dan [[../40 Databases/Row-Oriented vs Columnar Storage|Row-Oriented vs Columnar Storage]].
16. [[Terraform]] — implementasi utama [[../70 Infrastructure and Delivery/Declarative vs Imperative Infrastructure|Declarative vs Imperative Infrastructure]].
17. [[RabbitMQ]] — kontras langsung dengan Kafka untuk [[../30 APIs and Web/Queue vs Log Semantics|Queue vs Log Semantics]].
18. [[Debezium]] — implementasi utama [[../60 Distributed Systems/Change Data Capture|Change Data Capture]].
19. [[OpenTelemetry]] — implementasi utama [[../70 Infrastructure and Delivery/Distributed Tracing|Distributed Tracing]] dan [[../70 Infrastructure and Delivery/Correlation IDs|Correlation IDs]].
20. [[GitHub Actions]] — alternatif Jenkins untuk [[../70 Infrastructure and Delivery/CI-CD Pipelines|CI-CD Pipelines]].
21. [[ArgoCD]] — implementasi utama [[../70 Infrastructure and Delivery/Desired-State Reconciliation|Desired-State Reconciliation]] gaya GitOps.

### Orientation (7)

22. [[Ansible]] — implementasi utama sisi "configuration management" dari [[../70 Infrastructure and Delivery/Immutable Infrastructure vs Configuration Management|Immutable Infrastructure vs Configuration Management]]; jujur soal peran perannya yang menyusut dibanding image container immutable, relevan terutama untuk VM fleet lama.
23. [[NATS]] — alternatif ringan untuk [[../30 APIs and Web/Queue vs Log Semantics|Queue vs Log Semantics]].
24. [[Consul]] — implementasi utama [[../70 Infrastructure and Delivery/Service Discovery|Service Discovery]].
25. [[Vault]] — implementasi utama [[../80 Security/Secret Management|Secret Management]].
26. [[Temporal]] — implementasi durable-execution untuk [[../60 Distributed Systems/Sagas - Orchestration vs Choreography|Sagas - Orchestration vs Choreography]].
27. [[Loki]] — implementasi log-aggregation untuk [[../70 Infrastructure and Delivery/Structured Logging and Log Levels|Structured Logging and Log Levels]].
28. [[Jaeger]] — implementasi utama [[../70 Infrastructure and Delivery/Distributed Tracing|Distributed Tracing]], dari sisi backend tracing-nya.

## Bagaimana Domain Ini Terhubung Dengan Domain Lain

Setiap folder domain (`10` sampai `90`) punya sisi lain dari koin yang sama dengan folder ini — lihat bagian "Bagaimana Domain Ini Terhubung" di masing-masing `_Overview.md` domain. Yang paling padat terhubung ke sini: [[../40 Databases/_Overview|Databases Overview]] (PostgreSQL, MySQL, ClickHouse, Redis, Elasticsearch), [[../70 Infrastructure and Delivery/_Overview|Infrastructure and Delivery Overview]] (Docker, Kubernetes, Jenkins, Prometheus, Grafana, Terraform, ArgoCD, OpenTelemetry, Ansible, Consul), dan [[../30 APIs and Web/_Overview|APIs and Web Overview]] (Kafka, RabbitMQ, NATS, Nginx).

## Progress

```dataview
TABLE depth, volatility, status
FROM #tools
WHERE type = "tool"
SORT depth ASC
```

## Catatan Saya

*Tulis di sini tool di kantor yang tidak ada di daftar ini tapi menurutmu layak ditambahkan.*
