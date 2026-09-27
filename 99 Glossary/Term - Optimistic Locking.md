---
title: Term - Optimistic Locking
type: term
level: intermediate
domain: databases
status: unread
difficulty: 2
est_minutes: 3
prerequisites: []
next: []
tags: [backend, databases, concurrency]
created: 2026-08-02
---

**Optimistic locking** adalah strategi menangani konflik konkurensi tanpa benar-benar mengunci data selama dibaca — setiap baris punya penanda versi (biasanya kolom `version` berupa angka yang naik setiap kali baris diubah), dan tulisan hanya berhasil kalau versi yang dibaca masih sama dengan versi saat ini di database; kalau sudah berubah (pihak lain menulis lebih dulu), tulisan ditolak dan aplikasi harus membaca ulang dan mencoba lagi. Namanya "optimistic" karena mengasumsikan konflik jarang terjadi — berbeda dari [[Term - Pessimistic Locking]] yang mengunci data sejak awal, mengasumsikan konflik mungkin sering terjadi.

Ini kenapa istilah ini penting dipahami: optimistic locking cocok untuk kasus konflik yang jarang (menghindari overhead mengunci setiap baca), tapi butuh logika retry eksplisit di aplikasi untuk kasus tulisan yang ditolak — tanpa itu, pengguna hanya melihat error tanpa penyelesaian otomatis. Hindari memakai `updated_at` sebagai penanda versi: resolusi `DATETIME` default di MySQL/MariaDB adalah satu detik, sehingga dua perubahan dalam detik yang sama tidak terdeteksi sebagai konflik. Optimistic locking juga satu-satunya pilihan realistis ketika baca dan tulis terjadi di dua HTTP request terpisah, karena lock database tidak bisa ditahan melintasi request.

## Muncul Di

- [[../40 Databases/Locking and Row Locks|Locking and Row Locks]] — contoh `UPDATE ... WHERE versi = ?` dan kapan memilih optimistic dibanding pessimistic.

- [[../92 Tools/PostgreSQL - Locking and SELECT FOR UPDATE|PostgreSQL - Locking and SELECT FOR UPDATE]] — kontras langsung dengan pessimistic locking (`SELECT FOR UPDATE`).
- [[Term - Pessimistic Locking]] — strategi berlawanan yang mengunci data sejak awal.

## Catatan Saya

*Kosong — diisi pembaca.*
