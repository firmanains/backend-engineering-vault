---
title: Term - Nonce
type: term
level: intermediate
domain: security
status: unread
difficulty: 2
est_minutes: 3
prerequisites: []
next: []
tags: [backend, security]
created: 2026-08-02
---

**Nonce** ("number used once") adalah nilai acak atau unik yang dipakai **hanya sekali** dalam konteks kriptografi atau protokol keamanan — mencegah serangan replay (pihak lain merekam dan mengirim ulang pesan yang sah untuk menipu sistem seolah-olah pesan itu baru) karena nonce yang sama tidak boleh diterima dua kali. Muncul di enkripsi (AES-GCM butuh nonce unik per operasi enkripsi dengan key yang sama, lihat [[../80 Security/Encryption at Rest vs In Transit|Encryption at Rest vs In Transit]]) dan protokol autentikasi (mencegah replay token/signature).

Ini kenapa istilah ini penting dipahami: memakai ulang nonce yang seharusnya unik (terutama pada enkripsi AES-GCM) bisa merusak jaminan keamanan enkripsi itu secara serius — kesalahan implementasi yang halus dan mudah terlewat kalau tidak paham kenapa nonce harus benar-benar unik setiap kali.

## Muncul Di

- [[../80 Security/Encryption at Rest vs In Transit|Encryption at Rest vs In Transit]] — nonce sebagai bagian wajib skema enkripsi AES-GCM.

## Catatan Saya

*Kosong — diisi pembaca.*
