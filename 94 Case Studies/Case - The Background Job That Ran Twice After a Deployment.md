---
title: Case - The Background Job That Ran Twice After a Deployment
type: case
level: intermediate
domain: architecture
status: unread
difficulty: 3
est_minutes: 13
source: synthesized
concepts: ["[[../30 APIs and Web/Graceful Shutdown|Graceful Shutdown]]", "[[../30 APIs and Web/Idempotency|Idempotency]]", "[[../70 Infrastructure and Delivery/Blue-Green and Canary Releases|Blue-Green and Canary Releases]]"]
prerequisites: []
next: []
tags: [backend, architecture, infrastructure, gov-context]
created: 2026-08-02
---

## The Situation

Sebuah job background yang mengirim email pengingat tenggat kasus berjalan setiap jam, dipicu cron di dalam aplikasi itu sendiri. Suatu hari, tim melakukan deploy versi baru tepat saat job ini sedang berjalan (memproses ratusan email) — Kubernetes menghentikan Pod lama (yang sedang menjalankan job) dan menjalankan Pod baru, yang juga memicu job pengingat yang sama karena jadwal cron internalnya mulai lagi dari nol. Hasilnya: sebagian petugas menerima email pengingat **dua kali** dalam waktu berdekatan — bukan bug fatal, tapi membingungkan dan terlihat tidak profesional untuk sistem pemerintah.

## What Was Actually Asked For

Permintaan awal: "jangan sampai email dikirim dua kali." Yang sebenarnya dibutuhkan lebih dalam dari itu: sistem harus **aman terhadap deployment yang terjadi kapan saja**, termasuk saat job sedang berjalan — bukan hanya menambal kasus spesifik email pengingat ini, karena pola yang sama (job terputus dan diulang penuh oleh instance baru) berpotensi terjadi pada job lain di masa depan.

## Options

### Option 1 — Jadwalkan deploy di luar jam job berjalan

Pastikan deploy tidak pernah dilakukan tepat saat job cron internal sedang berjalan (misalnya hindari deploy di menit-menit awal setiap jam). Solusi termudah, tapi rapuh — bergantung disiplin manual mengingat jadwal job, dan tidak menyelesaikan akar masalah kalau ada deploy darurat yang tidak bisa ditunda menunggu jadwal aman.

### Option 2 — Graceful shutdown yang menunggu job selesai

Terapkan graceful shutdown (lihat [[../30 APIs and Web/Graceful Shutdown|Graceful Shutdown]]) — Pod lama diberi waktu menyelesaikan job yang sedang berjalan sebelum benar-benar dimatikan, dan Pod baru menunggu sinyal bahwa job sebelumnya benar-benar selesai sebelum memicu jadwalnya sendiri. Mengurangi kemungkinan tumpang tindih, tapi tidak menghilangkannya sepenuhnya — job yang berjalan sangat lama tetap berisiko terputus paksa kalau grace period habis, dan koordinasi "Pod baru menunggu Pod lama selesai" butuh mekanisme tambahan yang tidak trivial diimplementasikan benar.

### Option 3 — Pindahkan job dari cron internal ke scheduler eksternal dengan lock terdistribusi

Ganti cron internal aplikasi dengan job scheduler eksternal (CronJob Kubernetes, atau scheduler terpisah) yang memicu job **sekali** per jadwal, dan job itu sendiri memakai lock terdistribusi (advisory lock database, atau `SELECT FOR UPDATE SKIP LOCKED`) untuk memastikan hanya satu instance yang benar-benar mengeksekusi job itu meski ada beberapa Pod yang menerima trigger yang sama. Solusi paling struktural — tidak lagi bergantung pada cron internal yang terikat siklus hidup Pod aplikasi itu sendiri.

## The Decision

Option 3 dipilih sebagai solusi jangka panjang. Constraint penentunya: masalah aslinya adalah cron internal terikat siklus hidup Pod — setiap Pod baru yang naik (baik karena deploy, restart, atau scaling) secara tidak sengaja memicu ulang jadwalnya sendiri dari nol, masalah struktural yang tidak bisa diselesaikan penuh hanya dengan menunda waktu deploy (Option 1) atau menunggu lebih sabar (Option 2). Memindahkan penjadwalan keluar dari siklus hidup Pod aplikasi, dikombinasikan dengan lock yang mencegah eksekusi ganda meski trigger diterima lebih dari satu Pod, menghilangkan seluruh kelas masalah ini.

## What Breaks at 10x and 100x

Pada volume 10x (lebih banyak job berbeda yang perlu dijadwalkan), mengelola cron internal per-job di kode aplikasi jadi semakin tidak terkelola — titik ini memperkuat alasan memakai scheduler eksternal terpusat yang bisa mengelola banyak job dengan visibilitas yang jelas kapan masing-masing terakhir/berikutnya berjalan. Pada 100x dengan job yang durasinya sangat panjang (bukan lagi hitungan menit), lock terdistribusi sederhana (`SELECT FOR UPDATE`) mungkin perlu dilengkapi mekanisme heartbeat — memastikan lock yang dipegang worker yang mati mendadak (bukan selesai normal) bisa dideteksi dan dilepas, mencegah job itu terjebak "terkunci selamanya" oleh worker yang sudah tidak ada.

## What You Would Do Differently

Menerapkan idempotency pada level pengiriman email itu sendiri sejak awal (lihat [[../30 APIs and Web/Idempotency|Idempotency]]) — mencatat email pengingat mana yang sudah terkirim untuk kasus dan tenggat tertentu, dan memeriksa catatan itu sebelum mengirim — lapisan pertahanan tambahan yang tetap berguna bahkan kalau suatu saat job memang terpicu ganda karena sebab lain yang belum terpikirkan sekarang.

## Concepts Exercised

- [[../30 APIs and Web/Graceful Shutdown|Graceful Shutdown]] — mekanisme yang dipertimbangkan untuk mengurangi (meski tidak menghilangkan) risiko job terputus deploy.
- [[../30 APIs and Web/Idempotency|Idempotency]] — lapisan pertahanan tambahan yang direkomendasikan sebagai jaring pengaman.
- [[../70 Infrastructure and Delivery/Blue-Green and Canary Releases|Blue-Green and Canary Releases]] — strategi rilis yang relevan dipertimbangkan untuk mengurangi jendela waktu di mana dua versi aplikasi berjalan bersamaan.

## Your Turn

Setelah bermigrasi ke CronJob Kubernetes dengan lock terdistribusi, tim menemukan bahwa job kadang tidak berjalan sama sekali di jam yang dijadwalkan — investigasi menunjukkan CronJob sebelumnya masih berjalan (job lama belum selesai) saat jadwal berikutnya tiba, dan konfigurasi default menolak menjalankan job baru selama yang lama masih berjalan. Apakah ini masalah yang perlu diperbaiki?

> [!success]- Kunci jawaban
> Ini bukan bug — ini perilaku default yang sebenarnya **diinginkan** untuk mencegah job yang sama berjalan tumpang tindih (persis masalah yang ingin dihindari sejak awal kasus ini). Yang perlu diperbaiki bukan perilaku ini, melainkan **kenapa** job berjalan lebih lama dari jadwal berikutnya — kemungkinan job itu sendiri butuh dioptimalkan (diproses per-batch, bukan sekaligus semua email) atau interval jadwalnya perlu disesuaikan supaya realistis dengan waktu eksekusi job yang sebenarnya. Menonaktifkan proteksi tumpang tindih ini untuk "memaksa" job selalu berjalan sesuai jadwal justru mengembalikan risiko eksekusi ganda yang sudah susah payah dihindari.

## Catatan Saya

*Kalau kamu punya kejadian nyata dari kerjaan yang mirip tema ini, tulis sebagai case study baru dengan `source: personal`.*
