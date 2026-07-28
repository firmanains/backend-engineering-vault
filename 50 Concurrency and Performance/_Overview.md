---
title: Concurrency and Performance Overview
type: concept
level: intermediate
domain: concurrency
status: mastered
difficulty: 1
est_minutes: 10
prerequisites: ["[[../20 Go Language/_Overview|Go Language Overview]]"]
next: ["[[Goroutines]]"]
tags: [backend, concurrency, performance, go]
created: 2026-07-26
---

## Apa Ini

Domain ini adalah tempat Go berhenti menjadi sekadar bahasa dan mulai menjadi runtime: goroutine, channel, `select`, `sync`, `context`, sampai scheduler GMP, garbage collector, dan `pprof`. Di ujung yang lain, domain ini juga memuat performance secara umum (latency percentile, Little's Law, load testing, capacity planning) dan caching — dua topik yang menurut deskripsi folder aslinya memang satu keluarga dengan concurrency: semuanya soal bagaimana sistem berperilaku di bawah beban.

Seluruh isi domain ini adalah level **intermediate** — tidak ada versi juniornya, karena topik ini secara wajar membutuhkan bahasa Go (`20 Go Language`) sudah dikuasai lebih dulu.

## Kenapa Ini Penting

Ini adalah domain yang paling langsung melayani tujuan "mastery sungguhan dalam Go". Menulis `go func() {...}()` mudah; tahu kapan goroutine itu bocor, kenapa GC pause muncul di p99 latency, atau kenapa cache yang terlihat benar ternyata rentan stampede saat satu key populer kedaluwarsa — itu yang membedakan engineer yang "bisa Go" dari yang benar-benar menguasainya.

## Reading Order

### Intermediate — Go concurrency

1. [[Goroutines]] — unit concurrency Go, dan kenapa ia bukan "lightweight thread" begitu saja.
2. [[Buffered vs Unbuffered Channels]] — kapan channel memblokir dan kapan tidak.
3. [[The Select Statement]] — menunggu banyak channel sekaligus tanpa polling.
4. [[The Sync Package]] — `Mutex`, `WaitGroup`, `Once`, dan kapan masing-masing dipakai.
5. [[Context for Cancellation and Deadlines]] — membawa "stop sekarang" lintas goroutine dan lintas layer.
6. [[Worker Pools]] — membatasi concurrency yang berjalan bersamaan secara sengaja.
7. [[Fan-In Fan-Out]] — memecah kerja ke banyak goroutine lalu menyatukan hasilnya.
8. [[Pipelines]] — merangkai tahap pemrosesan lewat channel.
9. [[Race Conditions and the Race Detector]] — bug yang tidak selalu muncul, dan alat untuk menangkapnya sebelum production.
10. [[The Go Memory Model]] — jaminan (dan bukan jaminan) urutan operasi lintas goroutine.
11. [[errgroup]] — mengelola sekumpulan goroutine dan error-nya sebagai satu unit.
12. [[singleflight]] — mencegah request duplikat memicu kerja duplikat.
13. [[Goroutine Leaks]] — goroutine yang lupa berhenti, dan cara mendeteksinya sebelum memory habis.

### Intermediate — Go runtime

14. [[Goroutine Scheduler (GMP)]] — bagaimana runtime Go menjadwalkan jutaan goroutine di atas segelintir OS thread.
15. [[Preemption]] — kenapa goroutine yang tidak kooperatif tidak lagi bisa memblokir scheduler selamanya.
16. [[Garbage Collection in Go]] — model GC Go dan dampaknya pada latency, bukan hanya throughput.
17. [[Escape Analysis]] — kenapa compiler memutuskan sebuah value pergi ke heap, bukan stack.
18. [[pprof Profiling]] — melihat langsung di mana CPU dan memori sebenarnya dihabiskan.
19. [[Benchmarking in Go]] — mengukur performa dengan metodologi, bukan perasaan.
20. [[Reducing Allocations]] — teknik konkret menurunkan tekanan pada garbage collector.
21. [[sync.Pool]] — mendaur ulang objek untuk mengurangi alokasi di hot path.

### Intermediate — Performance

22. [[Latency Percentiles (p50, p95, p99)]] — kenapa rata-rata berbohong tentang pengalaman user.
23. [[Little's Law]] — hubungan matematis antara concurrency, throughput, dan latency.
24. [[Load Testing]] — mensimulasikan beban nyata sebelum user yang menemukan batasnya.
25. [[Capacity Planning]] — menghitung kebutuhan resource dari angka nyata, bukan tebakan.
26. [[Profiling a Real Application]] — menggabungkan pprof, benchmark, dan load test jadi satu latihan utuh.

### Intermediate — Caching

27. [[Cache-Aside, Write-Through, and Write-Behind]] — tiga pola menjaga cache tetap relevan, dengan trade-off berbeda.
28. [[Cache Invalidation Strategies]] — masalah yang terkenal sulit dalam computer science, dan cara mengelolanya.
29. [[TTL and Jitter]] — mencegah seluruh cache kedaluwarsa serentak.
30. [[Eviction Policies]] — LRU, LFU, dan bagaimana masing-masing memutuskan apa yang dibuang.
31. [[Cache Stampede]] — ketika satu key kedaluwarsa memicu badai request ke database.
32. [[Distributed Locks and Why They Are Dangerous]] — solusi yang terlihat sederhana, penuh jebakan di produksi.

> [!info] Kenapa caching bukan bagian dari 40 Databases?
> Caching mengubah bentuk *akses*, bukan bentuk *penyimpanan* data — dan deskripsi folder ini di §5 secara eksplisit menyebutnya berdampingan dengan goroutine dan profiling. Lihat `Curriculum Changelog.md` untuk alasan lengkap.

## Bagaimana Domain Ini Terhubung Dengan Domain Lain

- [[../20 Go Language/_Overview|Go Language Overview]] — prasyarat langsung; domain ini adalah lanjutan bahasa Go ke arah runtime dan concurrency.
- [[../40 Databases/_Overview|Databases Overview]] — connection pool tuning di domain itu adalah aplikasi langsung dari worker pool dan goroutine di sini.
- [[../30 APIs and Web/_Overview|APIs and Web Overview]] — resilience pattern di domain itu (circuit breaker, rate limiting) sering diimplementasikan memakai primitif concurrency di sini.
- [[../60 Distributed Systems/_Overview|Distributed Systems Overview]] — Little's Law dan capacity planning di sini adalah dasar kuantitatif untuk keputusan scalability di domain senior itu.
- [[../92 Tools/_Overview|Tools Overview]] — Redis mengimplementasikan sebagian besar concept caching di sini.

## Progress

```dataview
TABLE status, difficulty, est_minutes
FROM #concurrency
WHERE type = "concept"
SORT difficulty ASC
```

## Catatan Saya

*Tulis di sini goroutine leak atau race condition nyata yang pernah kamu temukan (atau curigai) di service Go di kantor.*
