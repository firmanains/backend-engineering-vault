---
title: Term - Sequence Number (TCP)
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

**Sequence number** adalah angka yang disertakan TCP di setiap segmen data yang dikirim, menandai posisi byte pertama segmen itu dalam keseluruhan aliran data koneksi. Penerima memakai nomor ini untuk menyusun ulang segmen yang mungkin tiba tidak berurutan (karena jaringan bisa mengirim paket lewat jalur berbeda-beda) dan mendeteksi segmen yang hilang (celah di urutan nomor berarti ada data yang belum sampai dan perlu dikirim ulang).

Ini kenapa istilah ini penting dipahami: sequence number adalah mekanisme dasar yang membuat TCP bisa menjamin data sampai **berurutan dan lengkap** meski jaringan di bawahnya (IP) sama sekali tidak menjamin urutan pengiriman paket — memahami ini menjelaskan kenapa TCP disebut protokol "reliable" sementara UDP (yang tidak punya sequence number) tidak.

## Muncul Di

- [[../10 Foundations/TCP Handshake and Connection Lifecycle|TCP Handshake and Connection Lifecycle]] — sequence number disepakati dan disinkronkan sejak awal handshake.
- [[../10 Foundations/TCP vs UDP|TCP vs UDP]] — kontras langsung dengan UDP yang tidak menyediakan jaminan urutan ini.

## Catatan Saya

*Kosong — diisi pembaca.*
