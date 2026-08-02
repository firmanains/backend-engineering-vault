---
title: Term - Accept Queue (Backlog)
type: term
level: junior
domain: foundations
status: unread
difficulty: 2
est_minutes: 3
prerequisites: []
next: []
tags: [backend, foundations]
created: 2026-08-02
---

**Accept queue** (sering disebut backlog) adalah antrean di level kernel yang menampung koneksi TCP yang sudah menyelesaikan three-way handshake tapi belum sempat diambil (`accept()`) oleh proses aplikasi yang mendengarkan port itu. Ukurannya dibatasi (dikonfigurasi lewat parameter seperti `net.core.somaxconn` di Linux, dan argumen backlog saat memanggil `listen()`) — kalau antrean ini penuh, koneksi baru yang masuk bisa ditolak atau di-drop kernel sebelum aplikasi sempat tahu ada permintaan itu sama sekali.

Ini kenapa istilah ini penting dipahami: aplikasi yang lambat memanggil `accept()` (karena sibuk memproses request lain, atau salah konfigurasi) bisa membuat koneksi baru gagal diterima meski secara teknis server "hidup" dan port terbuka — gejala yang membingungkan karena terlihat seperti masalah jaringan, padahal sebenarnya antrean di level kernel yang penuh.

## Muncul Di

- [[../10 Foundations/How An OS Handles Network Connections|How An OS Handles Network Connections]] — penjelasan lengkap bagaimana kernel menangani koneksi masuk lewat accept queue.

## Catatan Saya

*Kosong — diisi pembaca.*
