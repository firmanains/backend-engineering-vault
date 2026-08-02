---
title: Term - Keep-Alive
type: term
level: junior
domain: foundations
status: unread
difficulty: 1
est_minutes: 3
prerequisites: []
next: []
tags: [backend, foundations, apis]
created: 2026-08-02
---

**Keep-alive** merujuk pada dua mekanisme berbeda yang sering tertukar. Di level HTTP, keep-alive berarti koneksi TCP yang sudah dibuka **dipakai ulang** untuk beberapa request berturut-turut, alih-alih membuka koneksi baru setiap kali (menghindari biaya handshake TCP dan TLS berulang). Di level TCP itu sendiri, keep-alive adalah paket kecil periodik yang dikirim untuk memeriksa apakah koneksi yang tampak idle masih benar-benar hidup di kedua ujungnya, mendeteksi koneksi mati yang tidak pernah ditutup secara eksplisit (misalnya karena salah satu pihak crash tanpa sempat mengirim sinyal penutupan).

Ini kenapa istilah ini penting dipahami: keep-alive HTTP adalah salah satu optimasi performa paling murah untuk komunikasi berulang antar service, dan salah konfigurasi (timeout keep-alive yang tidak selaras antara klien dan server) adalah sumber umum error koneksi yang terlihat acak.

## Muncul Di

- [[../10 Foundations/HTTP 1.1 In Depth|HTTP 1.1 In Depth]] — penjelasan keep-alive HTTP dan connection reuse.
- [[../40 Databases/Connection Pooling|Connection Pooling]] — prinsip yang sama (memakai ulang koneksi) diterapkan untuk koneksi database.

## Catatan Saya

*Kosong — diisi pembaca.*
