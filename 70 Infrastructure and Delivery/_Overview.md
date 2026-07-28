---
title: Infrastructure and Delivery Overview
type: concept
level: junior
domain: infrastructure
status: mastered
difficulty: 1
est_minutes: 10
prerequisites: ["[[../10 Foundations/_Overview|Foundations Overview]]"]
next: ["[[Linux for Backend Engineers]]"]
tags: [backend, infrastructure]
created: 2026-07-26
---

## Apa Ini

Domain ini mencakup dua hal yang selalu berjalan berdampingan di dunia nyata: **bagaimana kode sampai ke production dan tetap berjalan** (Linux dasar, Docker, CI/CD, konsep Kubernetes, infrastructure as code, blue-green/canary release) dan **bagaimana kamu tahu apa yang sedang terjadi di dalamnya** (logging terstruktur, metrics, tracing, dashboard, alerting, SLI/SLO).

Observability sengaja ditempatkan di sini, bukan di folder tersendiri, karena secara operasional keduanya selalu dibahas berdampingan — deployment yang baik tanpa observability yang baik hanya berarti kamu akan tahu sesuatu rusak lewat laporan user, bukan lewat dashboard.

## Kenapa Ini Penting

Kubernetes tidak memberi high availability hanya karena diinstal, dan container tidak membuat kode buruk jadi baik — domain ini mengajarkan bagaimana infrastruktur benar-benar bekerja di baliknya, supaya keputusan seperti "tambah replica" atau "pakai canary release" dibuat dengan paham konsekuensinya, bukan sekadar mengikuti tutorial. Untuk koordinator teknis lintas banyak aplikasi, observability yang baik juga adalah alat komunikasi — dashboard dan alert yang jelas adalah cara termudah membuat 10+ developer melihat masalah yang sama tanpa harus dijelaskan berulang-ulang.

## Reading Order

### Junior — ops dasar

1. [[Linux for Backend Engineers]] — perintah dan konsep yang dipakai setiap hari untuk debugging production.
2. [[Docker - Images, Layers, and Multi-Stage Builds for Go]] — mengemas binary Go jadi image kecil dan cepat dibuild.
3. [[Docker Compose for Local Development]] — menjalankan seluruh dependency lokal dengan satu perintah.
4. [[Configuration via Environment (12-Factor App)]] — kenapa konfigurasi tidak boleh hardcoded atau tercampur dengan kode.

### Intermediate — delivery dan infrastructure as code

5. [[CI-CD Pipelines]] — otomatisasi dari commit sampai deploy.
6. [[Kubernetes Core Concepts - Pods, Deployments, Services, Ingress]] — kosakata dasar yang wajib sebelum menyentuh manifest apa pun.
7. [[Kubernetes Config, Secrets, Probes, and Autoscaling]] — mengoperasikan workload, bukan sekadar menjalankannya.
8. [[Declarative vs Imperative Infrastructure]] — menyatakan hasil akhir yang diinginkan, bukan langkah demi langkah.
9. [[Desired-State Reconciliation]] — bagaimana sistem seperti Kubernetes terus menutup gap ke keadaan yang diinginkan.
10. [[State Files and Drift]] — risiko saat kenyataan infrastruktur menyimpang dari definisinya.
11. [[Immutable Infrastructure vs Configuration Management]] — dua filosofi mengelola server, dan kenapa yang pertama makin dominan.
12. [[Blue-Green and Canary Releases]] — rilis yang bisa dibatalkan sebelum semua user terdampak.
13. [[Feature Flags]] — memisahkan deploy dari release.
14. [[Zero-Downtime Database Migrations]] — mengubah skema sambil aplikasi tetap melayani traffic.
15. [[Service Discovery]] — bagaimana satu service menemukan alamat service lain yang terus berpindah.

### Intermediate — observability

16. [[The Three Pillars of Observability]] — log, metric, trace, dan kenapa ketiganya saling melengkapi bukan saling menggantikan.
17. [[Structured Logging and Log Levels]] — log yang bisa di-query, bukan sekadar dibaca.
18. [[Metrics - The RED and USE Methods]] — dua kerangka memilih metrik yang benar-benar berguna.
19. [[Pull vs Push Metrics Collection]] — dua model pengumpulan metrik dan implikasi operasionalnya.
20. [[Query Languages for Metrics]] — membaca dan menulis query untuk menjawab pertanyaan operasional nyata.
21. [[Dashboard Design]] — dashboard yang benar-benar dilihat orang saat insiden, bukan dekorasi.
22. [[Distributed Tracing]] — mengikuti satu request melintasi banyak service.
23. [[Correlation IDs]] — benang merah yang menyatukan log dari service yang berbeda-beda.
24. [[Alerts That Do Not Cause Fatigue]] — alert yang dipercaya, bukan yang diabaikan karena terlalu sering salah.
25. [[SLIs and SLOs]] — mengubah "sistem harus reliable" jadi angka yang bisa diukur dan disepakati.

## Bagaimana Domain Ini Terhubung Dengan Domain Lain

- [[../10 Foundations/_Overview|Foundations Overview]] — proses, thread, dan file descriptor di domain itu adalah dasar untuk memahami container.
- [[../30 APIs and Web/_Overview|APIs and Web Overview]] — correlation ID dan distributed tracing di sini adalah alat wajib untuk mendiagnosis masalah integrasi di domain itu.
- [[../60 Distributed Systems/_Overview|Distributed Systems Overview]] — SLI/SLO dan disaster recovery di sini adalah pengantar praktis untuk error budget dan reliability engineering senior.
- [[../90 Architecture and Design/_Overview|Architecture and Design Overview]] — feature flag dan zero-downtime migration di sini adalah alat yang dipakai keputusan arsitektural di domain itu.
- [[../92 Tools/_Overview|Tools Overview]] — Docker, Kubernetes, Jenkins, Prometheus, Grafana, Terraform, ArgoCD, dan OpenTelemetry semuanya mengimplementasikan concept di domain ini.

## Progress

```dataview
TABLE status, level, difficulty
FROM #infrastructure
WHERE type = "concept"
SORT level ASC, difficulty ASC
```

## Catatan Saya

*Tulis di sini dashboard atau alert di kantor yang menurutmu paling berguna (atau paling sering diabaikan), dan kenapa.*
