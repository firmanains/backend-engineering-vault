---
title: Case - The Search Index That Drifted Out of Sync
type: case
level: intermediate
domain: architecture
status: unread
difficulty: 3
est_minutes: 14
source: synthesized
concepts: ["[[../40 Databases/Keeping Search in Sync with the Source of Truth|Keeping Search in Sync with the Source of Truth]]", "[[../60 Distributed Systems/Change Data Capture|Change Data Capture]]", "[[../60 Distributed Systems/Dual Writes and Their Dangers|Dual Writes and Their Dangers]]"]
prerequisites: []
next: []
tags: [backend, architecture, databases, distributed, gov-context]
created: 2026-08-02
---

## The Situation

Sebuah fitur pencarian kasus di sistem legal-services dibangun dengan Elasticsearch — aplikasi menulis ke database utama, lalu memanggil Elasticsearch untuk mengindeks kasus yang sama, di kode yang sama, secara berurutan. Beberapa bulan setelah rilis, petugas mulai melaporkan kasus yang jelas ada di sistem (bisa diakses langsung lewat ID) tapi tidak muncul di hasil pencarian. Investigasi menunjukkan sekitar 3% kasus tidak pernah ter-index — penyebabnya bervariasi: panggilan Elasticsearch yang gagal karena gangguan sesaat, timeout, atau proses batch internal yang mengubah data langsung lewat query SQL tanpa lewat kode aplikasi yang biasa mengirim ke Elasticsearch.

## What Was Actually Asked For

Permintaan awal: "perbaiki index yang hilang." Tapi menambahkan kasus yang hilang secara manual hanya menyelesaikan gejala saat ini, bukan akar masalahnya — kebutuhan sebenarnya adalah **mekanisme yang menjamin index selalu mencerminkan database**, tidak peduli jalur apa yang mengubah data (endpoint API, batch job, atau perbaikan darurat manual).

## Options

### Option 1 — Perbaiki manual dan tambah retry di kode yang sudah ada

Cari kasus yang hilang, index ulang manual, dan tambahkan retry logic di kode yang memanggil Elasticsearch supaya kegagalan sesaat tidak langsung membuat kasus gagal ter-index. Solusi tercepat untuk gejala saat ini, tapi tidak menutup celah proses batch internal yang mengubah data tanpa lewat kode aplikasi yang sama — drift akan terus terjadi dari jalur yang tidak tercakup retry ini.

### Option 2 — Job sinkronisasi berkala (reconciliation job)

Jalankan job terjadwal (misalnya setiap malam) yang membandingkan database dan index Elasticsearch, menemukan dan memperbaiki selisihnya. Menutup celah dari jalur mana pun yang menyebabkan drift, tapi kasus yang hilang baru diperbaiki di siklus berikutnya (bisa sampai 24 jam tertinggal), dan job perbandingan pada dataset besar bisa mahal secara komputasi kalau dijalankan naif (membandingkan seluruh dataset setiap kali).

### Option 3 — Change Data Capture (CDC)

Terapkan CDC (lihat [[../60 Distributed Systems/Change Data Capture|Change Data Capture]]) yang membaca transaction log database dan mengindeks perubahan ke Elasticsearch secara otomatis — menangkap **semua** perubahan tanpa kecuali, apa pun jalurnya (endpoint, batch job, atau perbaikan darurat manual), karena CDC bekerja di level database, bukan kode aplikasi tertentu. Solusi paling struktural, tapi butuh infrastruktur tambahan (Debezium, Kafka) yang harus dipasang dan dioperasikan.

## The Decision

Option 3 dipilih sebagai solusi jangka panjang, dengan Option 2 (reconciliation job sederhana) diterapkan sebagai jaring pengaman sementara sambil infrastruktur CDC disiapkan — mengingat pemasangan CDC butuh waktu (mengaktifkan binlog, memasang Debezium dan Kafka), reconciliation job memberi perbaikan cepat untuk masalah yang sudah terjadi sekarang. Constraint penentunya: masalah ini berulang dari jalur yang tidak terduga (batch internal) — solusi yang hanya menutup jalur yang sudah diketahui (Option 1) tidak cukup, harus solusi yang secara struktural menangkap perubahan dari **mana pun** asalnya.

## What Breaks at 10x and 100x

Pada volume 10x, reconciliation job (Option 2) yang membandingkan seluruh dataset setiap malam mulai memakan waktu yang tidak proporsional dan membebani database — titik ini mempercepat prioritas menyelesaikan migrasi penuh ke CDC. Pada 100x, bahkan CDC sendiri butuh perhatian pada lag pemrosesan — volume perubahan yang sangat tinggi bisa membuat consumer CDC (yang menulis ke Elasticsearch) tertinggal, butuh scaling consumer atau partisi topic Kafka yang lebih baik untuk mengimbangi laju perubahan.

## What You Would Do Differently

Memasang CDC sejak awal proyek, bukan menambahkan sinkronisasi manual di kode aplikasi dulu — investasi CDC di awal (saat volume data masih kecil dan risiko rendah) jauh lebih murah dibanding migrasi setelah drift sudah jadi masalah nyata di production dengan data besar.

## Concepts Exercised

- [[../40 Databases/Keeping Search in Sync with the Source of Truth|Keeping Search in Sync with the Source of Truth]] — masalah inti yang dibahas kasus ini.
- [[../60 Distributed Systems/Change Data Capture|Change Data Capture]] — solusi struktural yang dipilih sebagai perbaikan jangka panjang.
- [[../60 Distributed Systems/Dual Writes and Their Dangers|Dual Writes and Their Dangers]] — akar masalah teknis (menulis ke dua tempat secara langsung) yang menyebabkan drift ini terjadi sejak awal.

## Your Turn

Setelah CDC diterapkan, tim menyadari bahwa event CDC untuk penghapusan kasus (`DELETE`) tidak menghapus dokumen yang sesuai di Elasticsearch — index terus menyimpan kasus yang sudah dihapus dari database. Kenapa ini bisa terjadi, dan bagaimana memperbaikinya?

> [!success]- Kunci jawaban
> Penyebab paling mungkin: consumer CDC yang menulis ke Elasticsearch hanya menangani event `insert` dan `update`, lupa menangani event `delete` secara eksplisit — CDC menangkap ketiga jenis operasi ini sebagai event terpisah, dan consumer harus secara eksplisit menerjemahkan event `delete` jadi operasi hapus dokumen di Elasticsearch, bukan diasumsikan otomatis tertangani. Perbaikannya: tambahkan penanganan eksplisit untuk event `delete` di kode consumer CDC, dan jalankan reconciliation job satu kali untuk membersihkan dokumen "hantu" (kasus yang sudah dihapus di database tapi masih ada di index) yang terlanjur menumpuk sejak CDC diaktifkan sampai bug ini ditemukan.

## Catatan Saya

*Kalau kamu punya kejadian nyata dari kerjaan yang mirip tema ini, tulis sebagai case study baru dengan `source: personal`.*
