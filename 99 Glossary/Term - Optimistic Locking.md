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

**Optimistic locking** adalah strategi menangani konflik konkurensi tanpa benar-benar mengunci data selama dibaca — setiap baris punya penanda versi (kolom `version`, atau timestamp `updated_at`), dan tulisan hanya berhasil kalau versi yang dibaca masih sama dengan versi saat ini di database; kalau sudah berubah (pihak lain menulis lebih dulu), tulisan ditolak dan aplikasi harus membaca ulang dan mencoba lagi. Namanya "optimistic" karena mengasumsikan konflik jarang terjadi — berbeda dari [[Term - Pessimistic Locking]] yang mengunci data sejak awal, mengasumsikan konflik mungkin sering terjadi.

Ini kenapa istilah ini penting dipahami: optimistic locking cocok untuk kasus konflik yang jarang (menghindari overhead mengunci setiap baca), tapi butuh logika retry eksplisit di aplikasi untuk kasus tulisan yang ditolak — tanpa itu, pengguna hanya melihat error tanpa penyelesaian otomatis.

## Muncul Di

- [[../92 Tools/PostgreSQL - Locking and SELECT FOR UPDATE|PostgreSQL - Locking and SELECT FOR UPDATE]] — kontras langsung dengan pessimistic locking (`SELECT FOR UPDATE`).
- [[Term - Pessimistic Locking]] — strategi berlawanan yang mengunci data sejak awal.

## Catatan Saya

*Kosong — diisi pembaca.*
