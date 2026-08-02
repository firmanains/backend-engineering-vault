---
title: Case - The Webhook That Was Down For Six Hours
type: case
level: intermediate
domain: architecture
status: unread
difficulty: 3
est_minutes: 15
source: synthesized
concepts: ["[[../30 APIs and Web/Retries with Exponential Backoff and Jitter|Retries with Exponential Backoff and Jitter]]", "[[../30 APIs and Web/Dead Letter Queues|Dead Letter Queues]]", "[[../60 Distributed Systems/Exactly-Once Delivery as an Illusion|Exactly-Once Delivery as an Illusion]]"]
prerequisites: []
next: []
tags: [backend, architecture, apis, gov-context, integration, resilience]
created: 2026-08-02
---

## The Situation

Sistem sebuah instansi mengirim notifikasi webhook ke partner setiap kali status kasus berubah — puluhan notifikasi per menit di jam sibuk. Suatu malam, server partner mengalami gangguan infrastruktur dan tidak bisa menerima webhook selama enam jam penuh. Sistem pengirim, yang menerapkan retry sederhana (mencoba ulang setiap webhook gagal setiap 30 detik tanpa batas), terus mengirim ulang **setiap** webhook yang gagal berkali-kali sepanjang enam jam itu — menciptakan volume permintaan retry yang terus menumpuk, karena webhook baru terus dihasilkan dari perubahan status kasus yang terjadi selama periode gangguan itu juga masuk antrean retry yang sama.

## What Was Actually Asked For

Permintaan awal dari tim operasional: "webhook harus selalu sampai, apa pun yang terjadi." Kebutuhan sebenarnya, setelah dipikirkan lebih matang: bukan **jaminan mutlak** semua webhook sampai (itu mustahil dijamin sepenuhnya, lihat [[../60 Distributed Systems/Exactly-Once Delivery as an Illusion|Exactly-Once Delivery as an Illusion]]), tapi **jaminan bahwa retry yang gagal terus-menerus tidak membebani sistem pengirim sendiri sampai mengganggu operasi lain**, dan **kejelasan tentang webhook mana saja yang akhirnya gagal terkirim**, supaya bisa ditindaklanjuti manual kalau perlu.

## Options

### Option 1 — Retry tanpa batas dengan interval tetap (kondisi saat ini)

Retry setiap 30 detik selamanya sampai berhasil. Sederhana, tapi seperti terlihat di "The Situation" — retry yang menumpuk selama gangguan panjang bisa membebani sistem pengirim sendiri (resource terpakai untuk retry yang hampir pasti gagal lagi), dan begitu partner pulih, lonjakan retry yang menumpuk bisa membanjiri mereka sekaligus (thundering herd), berpotensi memperlambat pemulihan mereka sendiri.

### Option 2 — Exponential backoff dengan jitter, plus batas percobaan maksimum

Interval retry membesar secara eksponensial (30 detik, 1 menit, 2 menit, 4 menit, dst., lihat [[../30 APIs and Web/Retries with Exponential Backoff and Jitter|Retries with Exponential Backoff and Jitter]]) dengan jitter acak untuk menghindari retry dari banyak webhook menumpuk di waktu yang persis sama, sampai batas percobaan tertentu (misalnya 10 kali dalam 24 jam) sebelum menyerah dan menandai webhook itu gagal permanen. Mengurangi beban pada sistem pengirim dan mencegah thundering herd ke partner saat mereka pulih, tapi butuh keputusan eksplisit tentang apa yang terjadi setelah batas percobaan habis.

### Option 3 — Dead letter queue dengan notifikasi manual

Sama seperti Option 2, tapi webhook yang gagal setelah batas percobaan dipindah ke dead letter queue (lihat [[../30 APIs and Web/Dead Letter Queues|Dead Letter Queues]]) alih-alih dibuang begitu saja — tim operasional mendapat notifikasi dan bisa meninjau, memutuskan mengirim ulang manual atau menghubungi partner langsung untuk kasus yang benar-benar kritis.

## The Decision

Kombinasi Option 2 dan 3 dipilih — exponential backoff dengan jitter untuk mengurangi beban selama gangguan berlangsung, dan dead letter queue untuk webhook yang tetap gagal setelah batas percobaan, memastikan tidak ada notifikasi yang hilang diam-diam tanpa jejak. Constraint penentunya: sistem legal-services butuh akuntabilitas penuh (setiap notifikasi status kasus punya konsekuensi hukum bagi pihak yang menunggunya) — dead letter queue memberi jaring pengaman yang membuat kegagalan tetap terlihat dan bisa ditindaklanjuti, bukan hilang begitu saja setelah retry maksimum habis.

## What Breaks at 10x and 100x

Pada volume 10x, dead letter queue yang tidak dipantau aktif (notifikasi masuk tapi tidak ditinjau) bisa menumpuk tanpa penanganan — butuh proses operasional eksplisit (bukan cuma teknis) untuk meninjau dead letter queue secara rutin. Pada 100x, dengan banyak partner berbeda yang masing-masing bisa mengalami gangguan independen, sistem butuh circuit breaker per-partner (menghentikan sementara percobaan pengiriman ke partner yang jelas sedang down, alih-alih terus mencoba dan membuang resource) — evolusi alami dari retry sederhana menuju pola resiliensi yang lebih matang.

## What You Would Do Differently

Menyediakan endpoint bagi partner untuk secara manual meminta ulang webhook yang mereka lewatkan (berdasarkan rentang waktu) sejak awal desain — memberi partner kontrol memulihkan diri sendiri tanpa harus menunggu proses manual dari tim internal, mengurangi beban operasional kedua belah pihak.

## Concepts Exercised

- [[../30 APIs and Web/Retries with Exponential Backoff and Jitter|Retries with Exponential Backoff and Jitter]] — strategi inti yang diterapkan untuk mengurangi beban retry.
- [[../30 APIs and Web/Dead Letter Queues|Dead Letter Queues]] — mekanisme jaring pengaman untuk kegagalan yang tetap terjadi setelah retry maksimum.
- [[../60 Distributed Systems/Exactly-Once Delivery as an Illusion|Exactly-Once Delivery as an Illusion]] — kerangka berpikir yang membuat tim menerima bahwa "selalu sampai" bukan jaminan yang realistis, dan merancang berdasarkan kenyataan itu.

## Your Turn

Setelah dead letter queue diterapkan, ditemukan bahwa 5% webhook yang masuk dead letter queue ternyata sebenarnya **sudah berhasil** diproses partner (partner menerima dan memproses webhook itu, tapi respons konfirmasi mereka yang gagal terkirim balik karena gangguan jaringan sesaat). Apa implikasinya untuk cara dead letter queue ini ditangani?

> [!success]- Kunci jawaban
> Implikasinya: mengirim ulang webhook dari dead letter queue secara membabi buta berisiko memproses ulang notifikasi yang sebenarnya sudah diterima dan ditindaklanjuti partner — partner sendiri harus menerapkan idempotency (lihat [[../30 APIs and Web/Idempotent Consumers|Idempotent Consumers]]) di sisi mereka untuk aman menerima webhook duplikat, karena pengirim tidak pernah bisa 100% yakin apakah kegagalan itu di sisi pengiriman atau di sisi konfirmasi (persis dilema [[../60 Distributed Systems/Exactly-Once Delivery as an Illusion|Exactly-Once Delivery as an Illusion]]). Tim harus mengomunikasikan eksplisit ke semua partner bahwa idempotency di sisi mereka adalah syarat wajib menerima webhook ini, bukan asumsi implisit yang baru terungkap setelah masalah ini ditemukan.

## Catatan Saya

*Kalau kamu punya kejadian nyata dari kerjaan yang mirip tema ini, tulis sebagai case study baru dengan `source: personal`.*
