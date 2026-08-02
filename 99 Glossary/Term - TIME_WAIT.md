---
title: Term - TIME_WAIT
type: term
level: junior
domain: foundations
status: unread
difficulty: 1
est_minutes: 3
prerequisites: []
next: []
tags: [backend, foundations]
created: 2026-08-02
---

**TIME_WAIT** adalah state akhir sebuah koneksi TCP setelah kedua sisi setuju menutupnya — pihak yang menutup koneksi lebih dulu (biasanya klien) menahan pasangan alamat (IP dan port) koneksi itu selama periode tertentu, tidak langsung membebaskannya, sebagai jaga-jaga kalau ada paket "nyasar" dari koneksi lama itu yang masih dalam perjalanan di jaringan dan bisa disalahartikan sebagai bagian dari koneksi baru yang kebetulan memakai pasangan alamat sama.

Ini kenapa istilah ini penting dipahami: service yang membuka dan menutup banyak koneksi keluar berumur pendek bisa mengumpulkan ribuan socket dalam state TIME_WAIT, yang menahan [[Term - Ephemeral Port]] tidak bisa dipakai ulang sampai periode itu selesai — penyebab umum kehabisan port yang gejalanya membingungkan kalau tidak tahu mekanisme ini ada.

## Muncul Di

- [[../10 Foundations/TCP Handshake and Connection Lifecycle|TCP Handshake and Connection Lifecycle]] — penjelasan penuh siklus hidup koneksi TCP termasuk state TIME_WAIT.
- [[Term - Ephemeral Port]] — port yang ditahan TIME_WAIT sebelum bisa dipakai ulang.

## Catatan Saya

*Kosong — diisi pembaca.*
