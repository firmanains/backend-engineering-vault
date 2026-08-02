---
title: Case - Splitting a Monolith Coupled at the Database
type: case
level: intermediate
domain: architecture
status: unread
difficulty: 4
est_minutes: 16
source: synthesized
concepts: ["[[../90 Architecture and Design/Modular Monolith vs Microservices|Modular Monolith vs Microservices]]", "[[../90 Architecture and Design/Defining Service Boundaries|Defining Service Boundaries]]", "[[../60 Distributed Systems/The Strangler Fig Pattern|The Strangler Fig Pattern]]"]
prerequisites: []
next: []
tags: [backend, architecture, distributed, gov-context]
created: 2026-08-02
---

## The Situation

Sebuah aplikasi monolitik yang menangani seluruh alur kasus hukum — pengajuan, verifikasi, persetujuan, arsip — sudah sulit dipelihara: satu tim developer harus menguji seluruh aplikasi untuk perubahan sekecil apa pun, dan deploy jadi acara yang menegangkan karena risiko efek samping tak terduga di bagian yang tidak berkaitan. Manajemen meminta pemisahan jadi beberapa service independen. Investigasi teknis menemukan masalah yang lebih dalam dari sekadar "kode yang terlalu besar": puluhan tabel di database yang sama saling di-`JOIN` lintas apa yang **seharusnya** jadi batas modul berbeda — modul verifikasi langsung query tabel yang "dimiliki" modul persetujuan, dan sebaliknya.

## What Was Actually Asked For

Permintaan manajemen: "pecah jadi microservices." Yang sebenarnya mereka butuhkan: tim yang lebih kecil bisa bekerja independen tanpa saling menunggu atau saling merusak — microservices adalah satu cara mencapai itu, tapi bukan satu-satunya, dan bukan solusi otomatis kalau database yang jadi sumber coupling sebenarnya tidak ikut dipisah dengan benar.

## Options

### Option 1 — Microservices penuh sekaligus (big-bang split)

Rancang batas service baru, bangun database terpisah untuk masing-masing, migrasikan seluruh data dan kode sekaligus dalam satu proyek besar. Secara teori mencapai tujuan akhir tercepat, tapi riwayat industri (dan [[../60 Distributed Systems/The Strangler Fig Pattern|The Strangler Fig Pattern]]) menunjukkan pendekatan big-bang untuk sistem sebesar ini punya risiko kegagalan tinggi — terutama dengan coupling database yang sudah dalam seperti di sini, memisahkan semuanya sekaligus berarti menyelesaikan seluruh masalah query lintas modul dalam satu waktu, risiko yang sangat besar.

### Option 2 — Modular monolith dulu, microservices belakangan (kalau memang perlu)

Sebelum memisah jadi service terpisah, perbaiki dulu batas modul **di dalam** monolit yang sama (lihat [[../90 Architecture and Design/Modular Monolith vs Microservices|Modular Monolith vs Microservices]] dan [[../90 Architecture and Design/Defining Service Boundaries|Defining Service Boundaries]]) — hilangkan query langsung lintas modul, ganti dengan pemanggilan fungsi/interface yang jelas batasnya, meski masih dalam satu proses dan satu database. Kalau batas modul di kode sudah bersih, migrasi ke service terpisah (kalau memang masih dibutuhkan) jadi jauh lebih mekanis dan lebih aman.

### Option 3 — Strangler fig: pisahkan satu modul dulu sebagai percobaan

Pilih satu modul yang paling terisolasi (misalnya modul arsip, yang paling jarang berinteraksi dengan modul lain) untuk dipisah lebih dulu sebagai service independen, mengikuti pola [[../60 Distributed Systems/The Strangler Fig Pattern|The Strangler Fig Pattern]] — membangun pengalaman nyata dan pola yang bisa diulang untuk modul berikutnya, sebelum berkomitmen memisah seluruh sistem.

## The Decision

Option 2 dipilih sebagai langkah pertama wajib, diikuti Option 3 untuk modul yang benar-benar butuh dipisah setelah batasnya jelas. Constraint penentunya: coupling di level database (query lintas modul) berarti pemisahan ke service terpisah **tidak mungkin dilakukan aman** tanpa lebih dulu menyelesaikan masalah itu di level kode — mencoba memisah service sebelum batas modul bersih hanya memindahkan masalah yang sama (coupling erat) ke bentuk yang lebih mahal diperbaiki (coupling lintas jaringan, bukan lagi lintas fungsi dalam satu proses).

## What Breaks at 10x and 100x

Pada volume 10x (lebih banyak developer bergabung, lebih banyak fitur ditambahkan), modular monolith yang batas modulnya sudah bersih tetap bisa menjadi bottleneck kalau **seluruh** tim harus deploy bersamaan setiap kali ada perubahan di modul mana pun — di titik ini, alasan memisah jadi service independen (deploy independen per tim) menjadi konkret dan sepadan, bukan lagi sekadar ikut tren. Pada 100x (skala yang jauh lebih besar), pertanyaan bergeser dari "haruskah dipisah" menjadi "bagaimana data yang dulunya satu database sekarang dikelola konsisten lintas banyak database independen" — kembali ke pertimbangan saga dan eventual consistency yang dibahas di [[../60 Distributed Systems/_Overview|Distributed Systems Overview]].

## What You Would Do Differently

Menetapkan aturan arsitektur eksplisit sejak monolit pertama kali dibangun — "modul A tidak boleh query langsung tabel milik modul B, harus lewat interface/fungsi publik modul B" — jauh lebih murah dijaga sebagai disiplin sejak awal dibanding dibongkar dan diperbaiki setelah coupling sudah dalam dan tersebar di banyak tempat kode.

## Concepts Exercised

- [[../90 Architecture and Design/Modular Monolith vs Microservices|Modular Monolith vs Microservices]] — trade-off inti yang jadi dasar keputusan tidak langsung lompat ke microservices.
- [[../90 Architecture and Design/Defining Service Boundaries|Defining Service Boundaries]] — kerangka menentukan batas modul yang benar sebelum pemisahan fisik dilakukan.
- [[../60 Distributed Systems/The Strangler Fig Pattern|The Strangler Fig Pattern]] — pendekatan bertahap yang diikuti untuk memisah modul satu per satu, bukan sekaligus.

## Your Turn

Setelah batas modul dibersihkan di dalam monolit (Option 2 selesai), tim mengevaluasi modul arsip untuk dipisah jadi service independen (Option 3) — tapi menemukan modul arsip masih butuh membaca beberapa field dari tabel milik modul verifikasi untuk menampilkan riwayat lengkap. Apakah ini berarti pemisahan harus dibatalkan?

> [!success]- Kunci jawaban
> Tidak — ini adalah situasi normal yang butuh keputusan desain eksplisit, bukan alasan membatalkan pemisahan. Pilihannya: (1) modul arsip memanggil API modul verifikasi setiap kali butuh data itu (sinkron, sederhana tapi menambah dependency runtime antar service); atau (2) modul arsip menyimpan **salinan** field yang dibutuhkan itu sendiri, disinkronkan lewat event saat data di modul verifikasi berubah (asinkron, lebih kompleks tapi modul arsip tetap bisa berfungsi independen meski modul verifikasi sedang down). Untuk data riwayat yang sifatnya "sekali dicatat, jarang berubah" seperti ini, opsi 2 (salinan lewat event) sering lebih sesuai — modul arsip yang tujuannya justru menyimpan riwayat historis seharusnya tidak bergantung real-time pada modul lain untuk menampilkan data yang secara alami sudah "final" dan tidak berubah lagi.

## Catatan Saya

*Kalau kamu punya kejadian nyata dari kerjaan yang mirip tema ini, tulis sebagai case study baru dengan `source: personal`.*
