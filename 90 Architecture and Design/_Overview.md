---
title: Architecture and Design Overview
type: concept
level: junior
domain: architecture
status: mastered
difficulty: 1
est_minutes: 8
prerequisites: ["[[../20 Go Language/_Overview|Go Language Overview]]"]
next: ["[[Handler-Service-Repository Layering]]"]
tags: [backend, architecture]
created: 2026-07-26
---

## Apa Ini

Domain ini adalah tempat teknis bertemu keputusan dan manusia: hygiene dasar (layering, dependency injection manual, git workflow, semantic versioning) di level junior; layering arsitektur yang lebih besar (hexagonal/clean architecture, DDD ringan, modular monolith vs microservices, service boundary, komunikasi sync vs async) di level intermediate; dan kepemimpinan teknis (API governance, standar lintas tim, RFC process, mentoring, memilih pertempuran teknis, mengelola technical debt) di level senior.

## Kenapa Ini Penting

Clean architecture bukan sekadar susunan folder — dan seorang koordinator teknis untuk 10+ developer akan lebih sering dinilai dari kemampuan membuat *keputusan yang bisa dijalankan tim*, bukan dari kode yang ditulis sendiri. Domain ini adalah tempat vault ini secara eksplisit melatih itu: bagaimana menyusun boundary yang masuk akal, bagaimana menulis ADR yang membuat keputusan bertahan, dan bagaimana memilih pertempuran teknis mana yang layak diperjuangkan.

## Reading Order

### Junior — hygiene

1. [[Handler-Service-Repository Layering]] — pemisahan tanggung jawab paling dasar yang membuat kode testable.
2. [[Manual Dependency Injection in Go]] — menyusun dependency tanpa framework DI, gaya idiomatic Go.
3. [[Git Workflow and Code Review]] — konvensi branching dan review yang membuat kerja tim tidak saling menabrak.
4. [[Semantic Versioning]] — mengomunikasikan breaking change lewat nomor versi, bukan lewat kejutan.

### Intermediate — arsitektur

5. [[Hexagonal and Clean Architecture in Go]] — menerapkan batas arsitektur secara idiomatic, bukan meniru Java.
6. [[Lightweight DDD]] — memakai bahasa domain di kode tanpa ritual DDD yang berat.
7. [[Modular Monolith vs Microservices]] — microservices tidak mengurangi kompleksitas, ia memindahkannya ke jaringan.
8. [[Defining Service Boundaries]] — garis batas yang membuat service benar-benar independen, bukan sekadar dipisah secara fisik.
9. [[Synchronous vs Asynchronous Communication]] — memilih model komunikasi antar service berdasarkan siapa yang butuh menunggu.

### Senior — kepemimpinan teknis

10. [[API Governance]] — menjaga konsistensi API lintas tim tanpa mematikan otonomi tim.
11. [[Cross-Team Code Standards]] — standar yang disepakati, bukan dipaksakan, dan tetap diikuti setelah orang yang membuatnya pindah tim.
12. [[The RFC Process]] — mengusulkan perubahan besar secara tertulis supaya bisa didiskusikan sebelum ditulis kodenya.
13. [[Mentoring]] — mengalihkan pengetahuan yang ada di kepala satu orang senior ke seluruh tim.
14. [[Choosing Which Technical Battles to Fight]] — tidak semua ketidaksepakatan teknis layak diperjuangkan sampai akhir.
15. [[Managing Technical Debt Explicitly]] — memperlakukan utang teknis sebagai keputusan yang dicatat dan dibayar sengaja, bukan diabaikan sampai meledak.

## Bagaimana Domain Ini Terhubung Dengan Domain Lain

- [[../20 Go Language/_Overview|Go Language Overview]] — interface dan functional options di domain itu adalah alat konkret untuk dependency injection dan clean architecture di sini.
- [[../30 APIs and Web/_Overview|APIs and Web Overview]] — API versioning di domain itu adalah salah satu wujud paling konkret dari API governance di sini.
- [[../60 Distributed Systems/_Overview|Distributed Systems Overview]] — ADR dan design review di domain itu adalah aplikasi langsung dari RFC process dan forming trade-offs di sini.
- [[../70 Infrastructure and Delivery/_Overview|Infrastructure and Delivery Overview]] — feature flag dan zero-downtime migration di domain itu adalah alat eksekusi untuk keputusan arsitektural yang dibuat di sini.

## Progress

```dataview
TABLE status, level, difficulty
FROM #architecture
WHERE type = "concept"
SORT level ASC, difficulty ASC
```

## Catatan Saya

*Tulis di sini keputusan arsitektural di kantor yang menurutmu, dengan hindsight, seharusnya diambil berbeda.*
