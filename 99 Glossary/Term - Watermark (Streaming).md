---
title: Term - Watermark (Streaming)
type: term
level: intermediate
domain: apis
status: unread
difficulty: 3
est_minutes: 3
prerequisites: []
next: []
tags: [backend, apis]
created: 2026-08-02
---

**Watermark** dalam pemrosesan stream adalah penanda yang menyatakan "semua event dengan timestamp sebelum titik ini sudah pasti diterima" — dipakai sistem streaming untuk memutuskan kapan aman menutup sebuah jendela waktu (window) agregasi dan menghasilkan hasilnya, meski event bisa tiba tidak berurutan atau terlambat karena kondisi jaringan. Tanpa watermark, sistem tidak pernah benar-benar tahu kapan "cukup yakin" semua event untuk periode tertentu sudah datang.

Ini kenapa istilah ini penting dipahami: watermark adalah trade-off eksplisit antara latency (menunggu lebih lama demi yakin semua event terlambat sudah masuk) dan akurasi (menutup jendela lebih cepat, tapi berisiko melewatkan event yang datang setelahnya) — pilihan yang harus disengaja, bukan default yang diabaikan.

## Muncul Di

- [[../30 APIs and Web/Ordering Guarantees in Streaming Systems|Ordering Guarantees in Streaming Systems]] — konteks luas tentang urutan dan ketepatan waktu event dalam sistem streaming.

## Catatan Saya

*Kosong — diisi pembaca.*
