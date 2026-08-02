---
title: Term - Head-of-Line Blocking
type: term
level: intermediate
domain: apis
status: unread
difficulty: 2
est_minutes: 3
prerequisites: []
next: []
tags: [backend, apis, foundations]
created: 2026-08-02
---

**Head-of-line blocking** terjadi ketika satu permintaan yang lambat di depan antrean memblokir permintaan-permintaan lain di belakangnya, meski permintaan di belakang itu sendiri sebenarnya bisa selesai cepat — namanya berasal dari analogi antrean fisik, di mana satu orang yang transaksinya lama membuat semua orang di belakangnya menunggu meski urusan mereka sendiri singkat. Muncul di beberapa lapisan: pada HTTP/1.1 dengan koneksi yang dipakai berurutan (bukan paralel), pada TCP itu sendiri (segmen yang hilang memblokir segmen berikutnya yang sudah tiba, sampai segmen yang hilang dikirim ulang), dan pada antrean pesan yang diproses berurutan.

Ini kenapa istilah ini penting dipahami: memahami head-of-line blocking menjelaskan sebagian alasan HTTP/2 (multiplexing di satu koneksi) dan HTTP/3 (berbasis QUIC, menghindari head-of-line blocking di level transport) dirancang seperti sekarang.

## Muncul Di

- [[../10 Foundations/Introduction to HTTP2|Introduction to HTTP2]] — multiplexing HTTP/2 yang dirancang mengurangi head-of-line blocking di level aplikasi.
- [[../10 Foundations/HTTP 1.1 In Depth|HTTP 1.1 In Depth]] — keterbatasan HTTP/1.1 yang rentan head-of-line blocking tanpa multiplexing.

## Catatan Saya

*Kosong — diisi pembaca.*
