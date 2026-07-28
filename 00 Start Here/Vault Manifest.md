---
title: Vault Manifest
type: concept
level: junior
domain: foundations
status: mastered
difficulty: 1
est_minutes: 15
prerequisites: ["[[Read Me First]]"]
next: ["[[Backend Engineering Map]]"]
tags: [backend, meta]
created: 2026-07-26
---

Ini adalah daftar **setiap note yang direncanakan** di vault ini, dengan path lengkapnya, ditulis di Batch 0 sebelum satu pun concept note dibuat. Tujuannya supaya bentuk penuh vault terlihat dari hari pertama, bukan terungkap sedikit-sedikit batch demi batch.

Note yang belum punya file nyata adalah backlog implisit — lihat aturan di `Backlog.md`. Keputusan pemetaan domain yang tidak jelas dari kurikulum asli dijelaskan di `Curriculum Changelog.md`.

Total per kategori (akan berubah kalau ada penambahan topik lanjutan, dicatat di Curriculum Changelog):

- **Concept notes**: 89 junior, 117 intermediate, 52 senior = **258**
- **Tool notes**: 25 note produk + 3 sub-note database = **28**
- **Case studies**: **25**
- **Projects**: **3**
- **Glossary**: starter set of 20 terms (corrected in Batch 1 — see Curriculum Changelog), tumbuh organik dari first-appearance link

---

## 10 Foundations — junior only

1. `10 Foundations/Processes vs Threads.md`
2. `10 Foundations/Memory Layout - Stack vs Heap.md`
3. `10 Foundations/Blocking vs Non-Blocking IO.md`
4. `10 Foundations/Syscalls and File Descriptors.md`
5. `10 Foundations/How An OS Handles Network Connections.md`
6. `10 Foundations/The TCP-IP Model.md`
7. `10 Foundations/TCP Handshake and Connection Lifecycle.md`
8. `10 Foundations/TCP vs UDP.md`
9. `10 Foundations/DNS Resolution.md`
10. `10 Foundations/The TLS Handshake.md`
11. `10 Foundations/HTTP 1.1 In Depth.md`
12. `10 Foundations/Introduction to HTTP2.md`

## 20 Go Language

**Junior** (Go core + Go testing):
1. `20 Go Language/The Go Type System.md`
2. `20 Go Language/Structs and Methods.md`
3. `20 Go Language/Pointer vs Value Receivers.md`
4. `20 Go Language/Interfaces and Implicit Satisfaction.md`
5. `20 Go Language/Slice Internals.md`
6. `20 Go Language/Map Internals.md`
7. `20 Go Language/Defer, Panic, and Recover.md`
8. `20 Go Language/Errors as Values.md`
9. `20 Go Language/Error Wrapping.md`
10. `20 Go Language/Packages and Modules.md`
11. `20 Go Language/The Go Toolchain.md`
12. `20 Go Language/Struct Tags and JSON Marshalling.md`
13. `20 Go Language/Table-Driven Tests.md`
14. `20 Go Language/Stdlib Testing vs Testify.md`
15. `20 Go Language/Mocking Through Interfaces.md`
16. `20 Go Language/Testing HTTP Handlers with httptest.md`
17. `20 Go Language/Meaningful Test Coverage.md`

**Intermediate** (Advanced Go only — concurrency and runtime live in `50`):
18. `20 Go Language/Generics.md`
19. `20 Go Language/Reflection and Its Costs.md`
20. `20 Go Language/Embedding.md`
21. `20 Go Language/Functional Options.md`
22. `20 Go Language/Designing Stable Library APIs.md`
23. `20 Go Language/Sentinel Errors vs Error Types.md`

## 30 APIs and Web

**Junior** (API design + payload/file handling + HTTP in Go):
1. `30 APIs and Web/REST Principles.md`
2. `30 APIs and Web/Resource Modelling.md`
3. `30 APIs and Web/Choosing Status Codes.md`
4. `30 APIs and Web/Idempotency.md`
5. `30 APIs and Web/Pagination - Offset vs Cursor.md`
6. `30 APIs and Web/Filtering and Sorting.md`
7. `30 APIs and Web/API Versioning.md`
8. `30 APIs and Web/Consistent Error Responses.md`
9. `30 APIs and Web/OpenAPI.md`
10. `30 APIs and Web/Content Types and multipart-form-data.md`
11. `30 APIs and Web/Binary in JSON and the Base64 Tax.md`
12. `30 APIs and Web/Streaming vs Buffering.md`
13. `30 APIs and Web/Upload and Download Patterns.md`
14. `30 APIs and Web/Pre-signed URLs.md`
15. `30 APIs and Web/Request Size Limits Along The Path.md`
16. `30 APIs and Web/net-http Handlers and Middleware.md`
17. `30 APIs and Web/Routing in Go.md`
18. `30 APIs and Web/Context Propagation in HTTP Servers.md`
19. `30 APIs and Web/Timeouts in HTTP Servers.md`
20. `30 APIs and Web/Graceful Shutdown.md`
21. `30 APIs and Web/Load Balancing and Reverse Proxies.md` — *(ditambahkan, lihat Curriculum Changelog)*

**Intermediate** (integration + other protocols + messaging/streaming + resilience):
22. `30 APIs and Web/Designing an API for a Partner You Do Not Control.md`
23. `30 APIs and Web/Contract Negotiation and Versioning.md`
24. `30 APIs and Web/Webhooks and How to Secure Them.md`
25. `30 APIs and Web/Polling vs Push.md`
26. `30 APIs and Web/File-Based Integration.md`
27. `30 APIs and Web/Batch vs Realtime Integration.md`
28. `30 APIs and Web/Handling an Unreliable Counterparty.md`
29. `30 APIs and Web/Sandbox Environments.md`
30. `30 APIs and Web/Integration Testing Across an Organisational Boundary.md`
31. `30 APIs and Web/gRPC and Protobuf.md`
32. `30 APIs and Web/Schema Evolution in Protobuf.md`
33. `30 APIs and Web/GraphQL and Its Trade-offs.md`
34. `30 APIs and Web/WebSocket.md`
35. `30 APIs and Web/Server-Sent Events.md`
36. `30 APIs and Web/Long Polling.md`
37. `30 APIs and Web/Queue vs Log Semantics.md`
38. `30 APIs and Web/Topics, Partitions, and Offsets.md`
39. `30 APIs and Web/Consumer Groups and Rebalancing.md`
40. `30 APIs and Web/Delivery Semantics.md`
41. `30 APIs and Web/Idempotent Consumers.md`
42. `30 APIs and Web/The Transactional Outbox Pattern.md`
43. `30 APIs and Web/Dead Letter Queues.md`
44. `30 APIs and Web/Ordering Guarantees in Streaming Systems.md`
45. `30 APIs and Web/Backpressure.md`
46. `30 APIs and Web/Timeout Budgets.md`
47. `30 APIs and Web/Retries with Exponential Backoff and Jitter.md`
48. `30 APIs and Web/Circuit Breakers.md`
49. `30 APIs and Web/Bulkheads.md`
50. `30 APIs and Web/Rate Limiting Algorithms.md`
51. `30 APIs and Web/Load Shedding.md`
52. `30 APIs and Web/Graceful Degradation.md`

## 40 Databases

**Junior** (SQL as craft + databases):
1. `40 Databases/The Logical Order of Query Execution.md`
2. `40 Databases/Join Types and Their Mental Models.md`
3. `40 Databases/Aggregation and GROUP BY Semantics.md`
4. `40 Databases/Subqueries vs CTEs.md`
5. `40 Databases/Window Functions.md`
6. `40 Databases/UNION vs UNION ALL.md`
7. `40 Databases/NULL Semantics and Three-Valued Logic.md`
8. `40 Databases/Set Operations in SQL.md`
9. `40 Databases/Upserts.md`
10. `40 Databases/Relational Modelling.md`
11. `40 Databases/Normalisation 1NF to 3NF.md`
12. `40 Databases/Deliberate Denormalisation.md`
13. `40 Databases/Data Types and Constraints.md`
14. `40 Databases/Index Basics.md`
15. `40 Databases/Database Transactions.md`
16. `40 Databases/ACID.md`
17. `40 Databases/Basic Isolation Levels.md`
18. `40 Databases/database-sql and sqlx.md`
19. `40 Databases/Prepared Statements.md`
20. `40 Databases/Connection Pooling.md`
21. `40 Databases/Database Migrations.md`

**Intermediate** (databases in depth + storage engines + beyond relational + search):
22. `40 Databases/B+Tree Structure.md`
23. `40 Databases/Composite Indexes and the Leftmost-Prefix Rule.md`
24. `40 Databases/Covering Indexes.md`
25. `40 Databases/Reading EXPLAIN.md`
26. `40 Databases/Isolation Levels and Their Anomalies.md`
27. `40 Databases/MVCC.md`
28. `40 Databases/Locking and Row Locks.md`
29. `40 Databases/Deadlocks.md`
30. `40 Databases/The N+1 Query Problem.md`
31. `40 Databases/Tuning the Connection Pool.md`
32. `40 Databases/Read Replicas and Replication Lag.md`
33. `40 Databases/Partitioning.md`
34. `40 Databases/Introduction to Sharding.md`
35. `40 Databases/Materialised Views.md`
36. `40 Databases/Row-Oriented vs Columnar Storage.md`
37. `40 Databases/OLTP vs OLAP vs HTAP.md`
38. `40 Databases/LSM-Trees vs B-Trees.md`
39. `40 Databases/Write Amplification and Compression.md`
40. `40 Databases/Beyond Relational - Document, Key-Value, Wide-Column, Graph, and Time-Series Stores.md`
41. `40 Databases/Inverted Indexes and How Search Engines Work.md`
42. `40 Databases/Relevance Scoring.md`
43. `40 Databases/Keeping Search in Sync with the Source of Truth.md`

## 50 Concurrency and Performance — intermediate only

**Go concurrency**:
1. `50 Concurrency and Performance/Goroutines.md`
2. `50 Concurrency and Performance/Buffered vs Unbuffered Channels.md`
3. `50 Concurrency and Performance/The Select Statement.md`
4. `50 Concurrency and Performance/The Sync Package.md`
5. `50 Concurrency and Performance/Context for Cancellation and Deadlines.md`
6. `50 Concurrency and Performance/Worker Pools.md`
7. `50 Concurrency and Performance/Fan-In Fan-Out.md`
8. `50 Concurrency and Performance/Pipelines.md`
9. `50 Concurrency and Performance/Race Conditions and the Race Detector.md`
10. `50 Concurrency and Performance/The Go Memory Model.md`
11. `50 Concurrency and Performance/errgroup.md`
12. `50 Concurrency and Performance/singleflight.md`
13. `50 Concurrency and Performance/Goroutine Leaks.md`

**Go runtime**:
14. `50 Concurrency and Performance/Goroutine Scheduler (GMP).md`
15. `50 Concurrency and Performance/Preemption.md`
16. `50 Concurrency and Performance/Garbage Collection in Go.md`
17. `50 Concurrency and Performance/Escape Analysis.md`
18. `50 Concurrency and Performance/pprof Profiling.md`
19. `50 Concurrency and Performance/Benchmarking in Go.md`
20. `50 Concurrency and Performance/Reducing Allocations.md`
21. `50 Concurrency and Performance/sync.Pool.md`

**Performance**:
22. `50 Concurrency and Performance/Latency Percentiles (p50, p95, p99).md`
23. `50 Concurrency and Performance/Little's Law.md`
24. `50 Concurrency and Performance/Load Testing.md`
25. `50 Concurrency and Performance/Capacity Planning.md`
26. `50 Concurrency and Performance/Profiling a Real Application.md`

**Caching**:
27. `50 Concurrency and Performance/Cache-Aside, Write-Through, and Write-Behind.md`
28. `50 Concurrency and Performance/Cache Invalidation Strategies.md`
29. `50 Concurrency and Performance/TTL and Jitter.md`
30. `50 Concurrency and Performance/Eviction Policies.md`
31. `50 Concurrency and Performance/Cache Stampede.md`
32. `50 Concurrency and Performance/Distributed Locks and Why They Are Dangerous.md`

## 60 Distributed Systems — senior only

**Distributed systems theory**:
1. `60 Distributed Systems/The Fallacies of Distributed Computing.md`
2. `60 Distributed Systems/CAP Theorem and PACELC.md`
3. `60 Distributed Systems/Consistency Models.md`
4. `60 Distributed Systems/Time and Ordering - Lamport and Vector Clocks.md`
5. `60 Distributed Systems/Failure Detectors.md`
6. `60 Distributed Systems/Quorums.md`
7. `60 Distributed Systems/Consensus - Raft.md`
8. `60 Distributed Systems/Consensus - Paxos in Overview.md`
9. `60 Distributed Systems/Leader Election and Split Brain.md`

**Transactions across services**:
10. `60 Distributed Systems/Sagas - Orchestration vs Choreography.md`
11. `60 Distributed Systems/Two-Phase Commit and Why It Is Avoided.md`
12. `60 Distributed Systems/Idempotency Keys.md`
13. `60 Distributed Systems/Exactly-Once Delivery as an Illusion.md`
14. `60 Distributed Systems/Compensating Transactions.md`

**Event-driven architecture**:
15. `60 Distributed Systems/Event Sourcing.md`
16. `60 Distributed Systems/CQRS.md`
17. `60 Distributed Systems/Event Schema Evolution.md`
18. `60 Distributed Systems/Change Data Capture.md`
19. `60 Distributed Systems/Defensible Eventual Consistency.md`

**Scalability**:
20. `60 Distributed Systems/Sharding Strategies and Hot Partitions.md`
21. `60 Distributed Systems/Consistent Hashing.md`
22. `60 Distributed Systems/Multi-Region Architecture and Geo-Replication.md`
23. `60 Distributed Systems/CDNs and Edge Compute.md`
24. `60 Distributed Systems/Multi-Tenancy.md`

**Reliability engineering**:
25. `60 Distributed Systems/Error Budgets.md`
26. `60 Distributed Systems/Incident Command and Blameless Postmortems.md`
27. `60 Distributed Systems/Chaos Engineering.md`
28. `60 Distributed Systems/Planned Degradation.md`
29. `60 Distributed Systems/Disaster Recovery - RTO and RPO.md`

**Migration and evolution**:
30. `60 Distributed Systems/Expand-Contract Schema Migration.md`
31. `60 Distributed Systems/The Strangler Fig Pattern.md`
32. `60 Distributed Systems/Zero-Downtime Database Migration Using CDC.md`
33. `60 Distributed Systems/Dual Writes and Their Dangers.md`
34. `60 Distributed Systems/Backfilling Large Datasets Safely.md`

**System design methodology**:
35. `60 Distributed Systems/Reading Requirements and Capacity Estimation.md`
36. `60 Distributed Systems/Forming and Defending Trade-offs.md`
37. `60 Distributed Systems/Writing Architecture Decision Records.md`
38. `60 Distributed Systems/Running Design Reviews.md`
39. `60 Distributed Systems/Cost Engineering.md`

## 70 Infrastructure and Delivery

**Junior** (ops basics + one hygiene item):
1. `70 Infrastructure and Delivery/Linux for Backend Engineers.md`
2. `70 Infrastructure and Delivery/Docker - Images, Layers, and Multi-Stage Builds for Go.md`
3. `70 Infrastructure and Delivery/Docker Compose for Local Development.md`
4. `70 Infrastructure and Delivery/Configuration via Environment (12-Factor App).md`

**Intermediate** (delivery/IaC + observability):
5. `70 Infrastructure and Delivery/CI-CD Pipelines.md`
6. `70 Infrastructure and Delivery/Kubernetes Core Concepts - Pods, Deployments, Services, Ingress.md`
7. `70 Infrastructure and Delivery/Kubernetes Config, Secrets, Probes, and Autoscaling.md`
8. `70 Infrastructure and Delivery/Declarative vs Imperative Infrastructure.md`
9. `70 Infrastructure and Delivery/Desired-State Reconciliation.md`
10. `70 Infrastructure and Delivery/State Files and Drift.md`
11. `70 Infrastructure and Delivery/Immutable Infrastructure vs Configuration Management.md`
12. `70 Infrastructure and Delivery/Blue-Green and Canary Releases.md`
13. `70 Infrastructure and Delivery/Feature Flags.md`
14. `70 Infrastructure and Delivery/Zero-Downtime Database Migrations.md`
15. `70 Infrastructure and Delivery/Service Discovery.md` — *(ditambahkan, lihat Curriculum Changelog)*
16. `70 Infrastructure and Delivery/The Three Pillars of Observability.md`
17. `70 Infrastructure and Delivery/Structured Logging and Log Levels.md`
18. `70 Infrastructure and Delivery/Metrics - The RED and USE Methods.md`
19. `70 Infrastructure and Delivery/Pull vs Push Metrics Collection.md`
20. `70 Infrastructure and Delivery/Query Languages for Metrics.md`
21. `70 Infrastructure and Delivery/Dashboard Design.md`
22. `70 Infrastructure and Delivery/Distributed Tracing.md`
23. `70 Infrastructure and Delivery/Correlation IDs.md`
24. `70 Infrastructure and Delivery/Alerts That Do Not Cause Fatigue.md`
25. `70 Infrastructure and Delivery/SLIs and SLOs.md`

## 80 Security

**Junior**:
1. `80 Security/Password Hashing - bcrypt and argon2.md`
2. `80 Security/Sessions vs Tokens.md`
3. `80 Security/JWT - Structure, Signature, and When It Is The Wrong Tool.md`
4. `80 Security/OAuth2 Overview.md`
5. `80 Security/RBAC.md`
6. `80 Security/The OWASP Top 10.md`
7. `80 Security/SQL Injection.md`
8. `80 Security/XSS.md`
9. `80 Security/CSRF.md`
10. `80 Security/Secret Management.md`

**Senior**:
11. `80 Security/Threat Modelling with STRIDE.md`
12. `80 Security/Zero Trust.md`
13. `80 Security/mTLS.md`
14. `80 Security/Key Management and Rotation.md`
15. `80 Security/Encryption at Rest vs In Transit.md`
16. `80 Security/Audit Logging.md`
17. `80 Security/Compliance Trails for Government Systems.md`

## 90 Architecture and Design

**Junior** (engineering hygiene):
1. `90 Architecture and Design/Handler-Service-Repository Layering.md`
2. `90 Architecture and Design/Manual Dependency Injection in Go.md`
3. `90 Architecture and Design/Git Workflow and Code Review.md`
4. `90 Architecture and Design/Semantic Versioning.md`

**Intermediate**:
5. `90 Architecture and Design/Hexagonal and Clean Architecture in Go.md`
6. `90 Architecture and Design/Lightweight DDD.md`
7. `90 Architecture and Design/Modular Monolith vs Microservices.md`
8. `90 Architecture and Design/Defining Service Boundaries.md`
9. `90 Architecture and Design/Synchronous vs Asynchronous Communication.md`

**Senior** (technical leadership):
10. `90 Architecture and Design/API Governance.md`
11. `90 Architecture and Design/Cross-Team Code Standards.md`
12. `90 Architecture and Design/The RFC Process.md`
13. `90 Architecture and Design/Mentoring.md`
14. `90 Architecture and Design/Choosing Which Technical Battles to Fight.md`
15. `90 Architecture and Design/Managing Technical Debt Explicitly.md`

## 92 Tools

**Deep** (9): `92 Tools/Kafka.md` · `92 Tools/Elasticsearch.md` · `92 Tools/Redis.md` · `92 Tools/MySQL and MariaDB - Dialect and Behaviour.md` · `92 Tools/PostgreSQL.md` · `92 Tools/Docker.md` · `92 Tools/Kubernetes.md` · `92 Tools/Jenkins.md` · `92 Tools/Nginx.md`

**Working** (9): `92 Tools/Prometheus.md` · `92 Tools/Grafana.md` · `92 Tools/ClickHouse.md` · `92 Tools/Terraform.md` · `92 Tools/RabbitMQ.md` · `92 Tools/Debezium.md` · `92 Tools/OpenTelemetry.md` · `92 Tools/GitHub Actions.md` · `92 Tools/ArgoCD.md`

**Orientation** (7): `92 Tools/Ansible.md` · `92 Tools/NATS.md` · `92 Tools/Consul.md` · `92 Tools/Vault.md` · `92 Tools/Temporal.md` · `92 Tools/Loki.md` · `92 Tools/Jaeger.md`

**Required database sub-notes** (4, deep-tier only):
- `92 Tools/PostgreSQL - Features Worth Switching For.md`
- `92 Tools/PostgreSQL - Locking and SELECT FOR UPDATE.md`
- `92 Tools/PostgreSQL vs MySQL - How To Choose.md`
- (MySQL/MariaDB dialect note is listed above as the main MySQL tool note itself)

## 94 Case Studies

1. `94 Case Studies/Case - Sending PDFs Through a JSON API.md`
2. `94 Case Studies/Case - The Partner Who Calls Twice.md`
3. `94 Case Studies/Case - The Webhook That Was Down For Six Hours.md`
4. `94 Case Studies/Case - The Report Query That Locks a Table During Business Hours.md`
5. `94 Case Studies/Case - Migrating a 200-Million-Row Table Without Downtime.md`
6. `94 Case Studies/Case - The Counter That Undercounts.md`
7. `94 Case Studies/Case - The Search Index That Drifted Out of Sync.md`
8. `94 Case Studies/Case - The Cache That Returned Stale Data After a Write.md`
9. `94 Case Studies/Case - The Background Job That Ran Twice After a Deployment.md`
10. `94 Case Studies/Case - The Upload Endpoint That Died Under Real Concurrency.md`
11. `94 Case Studies/Case - The Rate-Limited External API and the Batch Job.md`
12. `94 Case Studies/Case - Splitting a Monolith Coupled at the Database.md`
13. `94 Case Studies/Case - The Just Add An Index Request.md`
14. `94 Case Studies/Case - Slow Query vs Cache vs Read Replica.md`
15. `94 Case Studies/Case - Daily CSV Over SFTP in 2026.md`
16. `94 Case Studies/Case - Log Volume That Costs More Than The Servers.md`
17. `94 Case Studies/Case - The JWT That Cannot Be Revoked.md`
18. `94 Case Studies/Case - The Audit Requirement That Arrived Late.md`
19. `94 Case Studies/Case - The Timezone Bug in a Nationwide Report.md`
20. `94 Case Studies/Case - The Retry Storm That Became a Total Outage.md`
21. `94 Case Studies/Case - A Government Partner's API That Only Accepts SOAP.md`
22. `94 Case Studies/Case - Choosing Between Kafka and a Database Outbox Table for One Consumer.md`
23. `94 Case Studies/Case - A Multi-Tenant Batch Job That Starves The Rest of the Tenants.md`
24. `94 Case Studies/Case - An OAuth2 Integration With a Partner Who Has No Refresh Tokens.md`
25. `94 Case Studies/Case - A Schema Change That Broke a Partner Who Scraped Undocumented Fields.md`

## 95 Projects

1. `95 Projects/Project 1 - Authenticated REST API.md`
2. `95 Projects/Project 2 - Worker Pool, Cache, and Streaming Service.md`
3. `95 Projects/Project 3 - A Small Distributed System.md`

## 99 Glossary — starter set

Grows organically from first-appearance links in concept notes. **Rule**: a term only gets a glossary note if it does **not** already have (or is not planned to have) a full concept note elsewhere — glossary is for supporting vocabulary that recurs across many notes but is too small to justify the full template (TL;DR, Problem, Intuition, exercises, etc.), not a shorter duplicate of a concept note that already exists. See the correction logged in `Curriculum Changelog.md` (Batch 1): the original starter list here wrongly included terms like Idempotency, ACID, CAP Theorem, MVCC, Sharding, Goroutine, JWT, and others that already have (or will have) their own full concept note in a domain folder — those terms are **not** glossary notes; the concept note itself is the destination for `[[...]]` links to them.

Corrected starter set — terms that genuinely have no dedicated concept note planned:

`Term - Ephemeral Port.md` · `Term - TIME_WAIT.md` · `Term - Accept Queue (Backlog).md` · `Term - Sequence Number (TCP).md` · `Term - Keep-Alive.md` · `Term - SLA.md` · `Term - Optimistic Locking.md` · `Term - Pessimistic Locking.md` · `Term - Head-of-Line Blocking.md` · `Term - Fan-Out (Messaging).md` · `Term - Watermark (Streaming).md` · `Term - Poison Message.md` · `Term - Split Brain.md` · `Term - Thundering Herd.md` · `Term - Hot Partition.md` · `Term - Gossip Protocol.md` · `Term - Heartbeat.md` · `Term - Checksum.md` · `Term - Nonce.md` · `Term - CIDR.md`

## Connected Notes

- [[Backlog]] — aturan tentang note yang ditautkan tapi belum ditulis; manifest ini adalah sumber kebenarannya.
- [[Curriculum Changelog]] — alasan di balik dua topik tambahan dan keputusan pemetaan domain yang tidak eksplisit di kurikulum asli.
- [[Backend Engineering Map]] — versi ringkas manifest ini, disusun sebagai peta baca lintas domain alih-alih daftar path.
- [[Progress Tracker]] — status baca untuk note yang sudah ditulis dari manifest ini.

## Catatan Saya

*Kosong — file ini murni referensi struktural, bukan tempat refleksi pribadi.*
