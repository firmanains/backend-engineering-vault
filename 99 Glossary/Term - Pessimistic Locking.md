---
title: Term - Pessimistic Locking
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

**Pessimistic locking** adalah strategi menangani konflik konkurensi dengan mengunci data **sejak awal dibaca** — transaksi lain yang mencoba membaca (dengan niat mengubah) atau menulis baris yang sama harus menunggu sampai lock dilepas. Diimplementasikan konkret lewat `SELECT ... FOR UPDATE` (lihat [[../92 Tools/PostgreSQL - Locking and SELECT FOR UPDATE|PostgreSQL - Locking and SELECT FOR UPDATE]]). Namanya "pessimistic" karena mengasumsikan konflik cukup mungkin terjadi sehingga lebih baik dicegah sejak awal, kontras dengan [[Term - Optimistic Locking]] yang menangani konflik setelah terjadi.

Ini kenapa istilah ini penting dipahami: pessimistic locking menghindari kebutuhan retry di aplikasi (transaksi yang menunggu lock akan otomatis dilanjutkan begitu lock dilepas), tapi menahan resource lebih lama dan berisiko deadlock kalau tidak dikelola hati-hati urutan penguncian antar transaksi.

## Muncul Di

- [[../92 Tools/PostgreSQL - Locking and SELECT FOR UPDATE|PostgreSQL - Locking and SELECT FOR UPDATE]] — implementasi konkret pessimistic locking lewat `SELECT FOR UPDATE`.
- [[Term - Optimistic Locking]] — strategi berlawanan yang tidak mengunci data sejak awal.
- [[../94 Case Studies/Case - The Counter That Undercounts|Case - The Counter That Undercounts]] — kasus nyata yang mempertimbangkan pessimistic locking sebagai salah satu solusi.

## Catatan Saya

*Kosong — diisi pembaca.*
