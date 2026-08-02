---
title: Term - Split Brain
type: term
level: senior
domain: distributed
status: unread
difficulty: 3
est_minutes: 3
prerequisites: []
next: []
tags: [backend, distributed]
created: 2026-08-02
---

**Split brain** adalah kondisi ketika sebuah cluster terpecah oleh partition jaringan menjadi dua (atau lebih) kelompok yang masing-masing, karena tidak bisa saling menghubungi, secara independen menganggap dirinya bagian yang "sah" — paling berbahaya ketika masing-masing kelompok memilih leader-nya sendiri, menghasilkan dua leader yang beroperasi bersamaan dan berpotensi membuat keputusan yang saling bertentangan.

Ini kenapa istilah ini penting dipahami: mekanisme consensus formal (lihat [[../60 Distributed Systems/Consensus - Raft|Consensus - Raft]]) secara eksplisit dirancang mencegah split brain lewat syarat mayoritas — kelompok yang jumlahnya minoritas tidak akan pernah berhasil memilih leader baru atau meng-commit tulisan baru, mencegah dua leader sama-sama "sah" beroperasi bersamaan.

## Muncul Di

- [[../60 Distributed Systems/Leader Election and Split Brain|Leader Election and Split Brain]] — pembahasan penuh mekanisme dan pencegahan split brain.
- [[../60 Distributed Systems/Quorums|Quorums]] — mekanisme mayoritas yang jadi pertahanan utama mencegah split brain.

## Catatan Saya

*Kosong — diisi pembaca.*
