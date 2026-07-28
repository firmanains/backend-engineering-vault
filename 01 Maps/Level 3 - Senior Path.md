---
title: Level 3 - Senior Path
type: concept
level: senior
domain: foundations
status: mastered
difficulty: 1
est_minutes: 10
prerequisites: ["[[Level 2 - Intermediate Path]]"]
next: []
tags: [backend, meta]
created: 2026-07-26
---

Urutan baca konkret untuk level senior, 52 note. Ini adalah jalur paling teoretis di vault ini, dan juga yang paling langsung melayani tujuan jangka panjangmu menuju **gelar master di distributed systems** — perlakukan `60 Distributed Systems` sebagai jalur serius, bukan bab penutup.

Urutan: teori distributed systems dulu (semua yang lain di folder ini dibangun di atasnya), lalu transaksi lintas service, event-driven architecture, scalability, reliability engineering, migrasi skala besar, dan metodologi system design sebagai penutup teori. Security senior dan arsitektur/kepemimpinan senior datang setelahnya karena keduanya adalah lapisan judgement yang menerapkan semua teori di atas pada keputusan nyata.

## 1. 60 Distributed Systems — teori (9 note)

1. The Fallacies of Distributed Computing — asumsi yang secara diam-diam salah dan menghancurkan desain sistem terdistribusi.
2. CAP Theorem and PACELC — trade-off yang tidak bisa dihindari, hanya bisa dipilih sadar.
3. Consistency Models — linearizable, sequential, causal, eventual — dan apa yang sebenarnya dijanjikan masing-masing.
4. Time and Ordering - Lamport and Vector Clocks — mengurutkan kejadian tanpa jam yang sama persis di semua mesin.
5. Failure Detectors — bagaimana satu node memutuskan node lain sudah mati, padahal tidak pernah benar-benar yakin.
6. Quorums — mayoritas sebagai jaminan konsistensi di sistem terdistribusi.
7. Consensus - Raft — algoritma consensus yang dirancang supaya bisa dipahami, dijelaskan sampai benar-benar klik.
8. Consensus - Paxos in Overview — pendahulu Raft, dan kenapa reputasinya sulit dipahami tetap layak diketahui.
9. Leader Election and Split Brain — apa yang terjadi ketika dua node sama-sama yakin mereka pemimpin.

## 2. 60 Distributed Systems — transaksi lintas service (5 note)

10. Sagas - Orchestration vs Choreography — mengganti satu transaksi besar dengan rangkaian transaksi lokal yang bisa dibatalkan.
11. Two-Phase Commit and Why It Is Avoided — solusi tekstual yang gagal di dunia nyata karena satu titik gagal bisa mengunci semua orang.
12. Idempotency Keys — memastikan efek sebuah operasi hanya terjadi sekali walau permintaannya diulang.
13. Exactly-Once Delivery as an Illusion — kenapa jaminan ini selalu bersyarat, bukan absolut.
14. Compensating Transactions — membatalkan efek yang sudah terjadi karena tidak ada rollback lintas service.

## 3. 60 Distributed Systems — event-driven architecture (5 note)

15. Event Sourcing — menyimpan rangkaian kejadian sebagai sumber kebenaran, bukan hanya state akhir.
16. CQRS — memisahkan jalur baca dan tulis karena keduanya punya kebutuhan yang berbeda.
17. Event Schema Evolution — mengubah bentuk event tanpa mematahkan consumer lama.
18. Change Data Capture — menangkap perubahan database sebagai aliran event tanpa mengubah aplikasi penulisnya.
19. Defensible Eventual Consistency — kapan "konsisten nanti" adalah keputusan yang bisa dipertanggungjawabkan, bukan sekadar jalan pintas.

## 4. 60 Distributed Systems — scalability (5 note)

20. Sharding Strategies and Hot Partitions — memecah data secara adil supaya satu partition tidak menanggung semua beban.
21. Consistent Hashing — memecah dan mendistribusikan ulang data tanpa mengocok semuanya saat node berubah.
22. Multi-Region Architecture and Geo-Replication — melayani pengguna dari lokasi terdekat sambil menjaga data tetap konsisten secukupnya.
23. CDNs and Edge Compute — memindahkan komputasi dan konten sedekat mungkin ke pengguna.
24. Multi-Tenancy — melayani banyak pelanggan dari satu sistem tanpa satu sama lain saling mengganggu.

## 5. 60 Distributed Systems — reliability engineering (5 note)

25. Error Budgets — mengizinkan kegagalan dalam jumlah yang disepakati, supaya tim berani bergerak cepat.
26. Incident Command and Blameless Postmortems — menangani insiden secara terstruktur, lalu belajar darinya tanpa mencari kambing hitam.
27. Chaos Engineering — merusak sistem secara sengaja saat semua orang siap, supaya tidak rusak sendiri saat tidak ada yang siap.
28. Planned Degradation — memutuskan di muka fitur mana yang boleh mati duluan saat sistem tertekan.
29. Disaster Recovery - RTO and RPO — seberapa cepat harus pulih, dan seberapa banyak data boleh hilang.

## 6. 60 Distributed Systems — migrasi dan evolusi (5 note)

30. Expand-Contract Schema Migration — mengubah skema dalam tahap yang masing-masing aman untuk di-rollback.
31. The Strangler Fig Pattern — mengganti sistem lama sedikit demi sedikit tanpa big-bang rewrite.
32. Zero-Downtime Database Migration Using CDC — memindahkan data ke sistem baru sambil sistem lama tetap melayani traffic.
33. Dual Writes and Their Dangers — kenapa menulis ke dua sistem sekaligus terlihat sederhana tapi hampir selalu berakhir tidak konsisten.
34. Backfilling Large Datasets Safely — mengisi data historis tanpa membanting database produksi.

## 7. 60 Distributed Systems — metodologi system design (5 note)

35. Reading Requirements and Capacity Estimation — menerjemahkan kebutuhan samar jadi angka kapasitas yang konkret.
36. Forming and Defending Trade-offs — argumen yang membuat keputusan arsitektural bisa bertahan saat ditantang.
37. Writing Architecture Decision Records — mencatat keputusan dan alasannya supaya tidak didebat ulang dari nol setahun kemudian.
38. Running Design Reviews — memandu diskusi desain supaya menghasilkan keputusan, bukan sekadar opini yang bertabrakan.
39. Cost Engineering — menjadikan biaya infrastruktur sebagai bagian sadar dari keputusan desain, bukan kejutan di akhir bulan.

## 8. 80 Security — lanjutan (7 note)

40. Threat Modelling with STRIDE — kerangka sistematis menemukan cara sistem bisa diserang sebelum diserang sungguhan.
41. Zero Trust — tidak ada jaringan yang "aman secara default" hanya karena berada di dalam firewall.
42. mTLS — kedua pihak saling membuktikan identitas, bukan hanya server ke klien.
43. Key Management and Rotation — kunci kriptografi yang dikelola sebagai siklus hidup, bukan sekali dibuat lalu dilupakan.
44. Encryption at Rest vs In Transit — dua lapisan perlindungan data yang melindungi dari ancaman berbeda.
45. Audit Logging — mencatat siapa melakukan apa, secara tidak bisa diubah, untuk investigasi setelah kejadian.
46. Compliance Trails for Government Systems — kebutuhan audit yang lahir dari regulasi, bukan dari pilihan teknis semata.

## 9. 90 Architecture and Design — kepemimpinan teknis (6 note)

47. API Governance — menjaga konsistensi API lintas tim tanpa mematikan otonomi tim.
48. Cross-Team Code Standards — standar yang disepakati, bukan dipaksakan, dan tetap diikuti setelah orang yang membuatnya pindah tim.
49. The RFC Process — mengusulkan perubahan besar secara tertulis supaya bisa didiskusikan sebelum ditulis kodenya.
50. Mentoring — mengalihkan pengetahuan yang ada di kepala satu orang senior ke seluruh tim.
51. Choosing Which Technical Battles to Fight — tidak semua ketidaksepakatan teknis layak diperjuangkan sampai akhir.
52. Managing Technical Debt Explicitly — memperlakukan utang teknis sebagai keputusan yang dicatat dan dibayar sengaja, bukan diabaikan sampai meledak.

## Setelah Ini

Selesaikan Project 3 (`95 Projects/Project 3 - A Small Distributed System.md`) — proyek ini secara sengaja menggabungkan saga, outbox, idempotency, dan skenario chaos, ditutup dengan ADR. Ini adalah bukti praktis bahwa jalur senior benar-benar dikuasai, bukan hanya dibaca.

## Connected Notes

- [[Level 2 - Intermediate Path]] — prasyarat penuh sebelum memulai jalur ini.
- [[Backend Engineering Map]] — posisi setiap domain di atas dalam peta besar vault.
- [[Vault Manifest]] — path lengkap tiap note yang disebut di sini.
- [[How To Read This Vault]] — ritual review yang sama pentingnya di level ini karena materinya lebih abstrak, lebih mudah terasa "paham" padahal belum.

## Catatan Saya

*Tulis di sini bagaimana teori distributed systems di jalur ini mulai terhubung dengan rencana studi masternya.*
