---
title: Distributed Systems Overview
type: concept
level: senior
domain: distributed
status: mastered
difficulty: 1
est_minutes: 10
prerequisites: ["[[../50 Concurrency and Performance/_Overview|Concurrency and Performance Overview]]", "[[../30 APIs and Web/_Overview|APIs and Web Overview]]"]
next: ["[[The Fallacies of Distributed Computing]]"]
tags: [backend, distributed]
created: 2026-07-26
---

## Apa Ini

Domain ini adalah tentang banyak mesin yang harus bekerja sama padahal jaringan di antara mereka bisa lambat, bisa gagal sebagian, dan tidak pernah bisa dipercaya sepenuhnya. Isinya: teori inti (CAP, consistency model, consensus, quorum), transaksi lintas service (saga, 2PC, idempotency key), event-driven architecture (event sourcing, CQRS, CDC), scalability (sharding, consistent hashing, multi-region), reliability engineering (error budget, chaos engineering, disaster recovery), migrasi skala besar, dan metodologi system design.

## Kenapa Ini Penting — dan Kenapa Domain Ini Diperlakukan Serius

Domain ini secara langsung melayani ambisi jangka panjang gelar master di distributed systems — ia **bukan** bab penutup kurikulum yang ditulis untuk melengkapi daftar topik. Perlakukan setiap note di sini sebagai persiapan akademik yang sama seriusnya dengan persiapan praktis. Di sisi lain, ini juga domain yang paling langsung menjelaskan kenapa sistem terdistribusi nyata — termasuk sistem multi-aplikasi milik pemerintah yang kamu koordinasikan — gagal dengan cara yang aneh: dua service yang "sama-sama benar" tapi hasil akhirnya salah, data yang berbeda antar region, migrasi yang macet di tengah jalan.

## Reading Order

### Senior — teori distributed systems

1. [[The Fallacies of Distributed Computing]] — asumsi yang secara diam-diam salah dan menghancurkan desain sistem terdistribusi.
2. [[CAP Theorem and PACELC]] — trade-off yang tidak bisa dihindari, hanya bisa dipilih sadar.
3. [[Consistency Models]] — linearizable, sequential, causal, eventual — dan apa yang sebenarnya dijanjikan masing-masing.
4. [[Time and Ordering - Lamport and Vector Clocks]] — mengurutkan kejadian tanpa jam yang sama persis di semua mesin.
5. [[Failure Detectors]] — bagaimana satu node memutuskan node lain sudah mati, padahal tidak pernah benar-benar yakin.
6. [[Quorums]] — mayoritas sebagai jaminan konsistensi di sistem terdistribusi.
7. [[Consensus - Raft]] — algoritma consensus yang dirancang supaya bisa dipahami, dijelaskan sampai benar-benar klik.
8. [[Consensus - Paxos in Overview]] — pendahulu Raft, dan kenapa reputasinya sulit dipahami tetap layak diketahui.
9. [[Leader Election and Split Brain]] — apa yang terjadi ketika dua node sama-sama yakin mereka pemimpin.

### Senior — transaksi lintas service

10. [[Sagas - Orchestration vs Choreography]] — mengganti satu transaksi besar dengan rangkaian transaksi lokal yang bisa dibatalkan.
11. [[Two-Phase Commit and Why It Is Avoided]] — solusi tekstual yang gagal di dunia nyata karena satu titik gagal bisa mengunci semua orang.
12. [[Idempotency Keys]] — memastikan efek sebuah operasi hanya terjadi sekali walau permintaannya diulang.
13. [[Exactly-Once Delivery as an Illusion]] — kenapa jaminan ini selalu bersyarat, bukan absolut.
14. [[Compensating Transactions]] — membatalkan efek yang sudah terjadi karena tidak ada rollback lintas service.

### Senior — event-driven architecture

15. [[Event Sourcing]] — menyimpan rangkaian kejadian sebagai sumber kebenaran, bukan hanya state akhir.
16. [[CQRS]] — memisahkan jalur baca dan tulis karena keduanya punya kebutuhan yang berbeda.
17. [[Event Schema Evolution]] — mengubah bentuk event tanpa mematahkan consumer lama.
18. [[Change Data Capture]] — menangkap perubahan database sebagai aliran event tanpa mengubah aplikasi penulisnya.
19. [[Defensible Eventual Consistency]] — kapan "konsisten nanti" adalah keputusan yang bisa dipertanggungjawabkan, bukan sekadar jalan pintas.

### Senior — scalability

20. [[Sharding Strategies and Hot Partitions]] — memecah data secara adil supaya satu partition tidak menanggung semua beban.
21. [[Consistent Hashing]] — memecah dan mendistribusikan ulang data tanpa mengocok semuanya saat node berubah.
22. [[Multi-Region Architecture and Geo-Replication]] — melayani pengguna dari lokasi terdekat sambil menjaga data tetap konsisten secukupnya.
23. [[CDNs and Edge Compute]] — memindahkan komputasi dan konten sedekat mungkin ke pengguna.
24. [[Multi-Tenancy]] — melayani banyak pelanggan dari satu sistem tanpa satu sama lain saling mengganggu.

### Senior — reliability engineering

25. [[Error Budgets]] — mengizinkan kegagalan dalam jumlah yang disepakati, supaya tim berani bergerak cepat.
26. [[Incident Command and Blameless Postmortems]] — menangani insiden secara terstruktur, lalu belajar darinya tanpa mencari kambing hitam.
27. [[Chaos Engineering]] — merusak sistem secara sengaja saat semua orang siap, supaya tidak rusak sendiri saat tidak ada yang siap.
28. [[Planned Degradation]] — memutuskan di muka fitur mana yang boleh mati duluan saat sistem tertekan.
29. [[Disaster Recovery - RTO and RPO]] — seberapa cepat harus pulih, dan seberapa banyak data boleh hilang.

### Senior — migrasi dan evolusi

30. [[Expand-Contract Schema Migration]] — mengubah skema dalam tahap yang masing-masing aman untuk di-rollback.
31. [[The Strangler Fig Pattern]] — mengganti sistem lama sedikit demi sedikit tanpa big-bang rewrite.
32. [[Zero-Downtime Database Migration Using CDC]] — memindahkan data ke sistem baru sambil sistem lama tetap melayani traffic.
33. [[Dual Writes and Their Dangers]] — kenapa menulis ke dua sistem sekaligus terlihat sederhana tapi hampir selalu berakhir tidak konsisten.
34. [[Backfilling Large Datasets Safely]] — mengisi data historis tanpa membanting database produksi.

### Senior — metodologi system design

35. [[Reading Requirements and Capacity Estimation]] — menerjemahkan kebutuhan samar jadi angka kapasitas yang konkret.
36. [[Forming and Defending Trade-offs]] — argumen yang membuat keputusan arsitektural bisa bertahan saat ditantang.
37. [[Writing Architecture Decision Records]] — mencatat keputusan dan alasannya supaya tidak didebat ulang dari nol setahun kemudian.
38. [[Running Design Reviews]] — memandu diskusi desain supaya menghasilkan keputusan, bukan sekadar opini yang bertabrakan.
39. [[Cost Engineering]] — menjadikan biaya infrastruktur sebagai bagian sadar dari keputusan desain, bukan kejutan di akhir bulan.

## Bagaimana Domain Ini Terhubung Dengan Domain Lain

- [[../30 APIs and Web/_Overview|APIs and Web Overview]] — outbox pattern dan messaging di domain itu adalah pengantar praktis untuk saga dan CDC di sini.
- [[../40 Databases/_Overview|Databases Overview]] — sharding dan replication di domain itu diperdalam di sini menjadi consistent hashing dan multi-region architecture.
- [[../50 Concurrency and Performance/_Overview|Concurrency and Performance Overview]] — capacity planning dan Little's Law di domain itu adalah dasar kuantitatif untuk estimasi kapasitas di sini.
- [[../90 Architecture and Design/_Overview|Architecture and Design Overview]] — ADR dan design review di sini berpasangan langsung dengan RFC process dan API governance di domain itu.
- [[../92 Tools/_Overview|Tools Overview]] — Kafka, Debezium, dan Temporal semuanya mengimplementasikan concept di domain ini.

## Progress

```dataview
TABLE status, difficulty, est_minutes
FROM #distributed
WHERE type = "concept"
SORT difficulty ASC
```

## Catatan Saya

*Tulis di sini pertanyaan yang muncul saat menghubungkan domain ini dengan rencana studi master distributed systems-mu.*
