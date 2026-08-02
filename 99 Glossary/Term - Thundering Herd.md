---
title: Term - Thundering Herd
type: term
level: intermediate
domain: concurrency
status: unread
difficulty: 2
est_minutes: 3
prerequisites: []
next: []
tags: [backend, concurrency, performance]
created: 2026-08-02
---

**Thundering herd** adalah situasi ketika banyak proses atau permintaan yang tadinya menunggu (misalnya menunggu cache yang kedaluwarsa, atau menunggu lock yang dilepas) semuanya "bangun" dan bertindak **bersamaan** tepat pada momen yang sama, membebani resource yang mereka rebutkan secara sekaligus — kontras dengan skenario ideal di mana permintaan yang menunggu itu tersebar merata dari waktu ke waktu.

Ini kenapa istilah ini penting dipahami: thundering herd muncul di banyak konteks berbeda — cache stampede (banyak request menghitung ulang cache yang sama begitu kedaluwarsa), retry storm (banyak retry menumpuk dan menyerang sistem yang baru pulih sekaligus, lihat [[../94 Case Studies/Case - The Retry Storm That Became a Total Outage|Case - The Retry Storm That Became a Total Outage]]), atau notifikasi massal yang memicu banyak proses bangun bersamaan — semuanya menuntut mitigasi yang sama: menyebarkan waktu reaksi (jitter), bukan membiarkan semua bereaksi persis di waktu yang sama.

## Muncul Di

- [[../94 Case Studies/Case - The Retry Storm That Became a Total Outage|Case - The Retry Storm That Became a Total Outage]] — kasus nyata thundering herd dalam bentuk retry storm.
- [[../30 APIs and Web/Retries with Exponential Backoff and Jitter|Retries with Exponential Backoff and Jitter]] — jitter sebagai mitigasi langsung terhadap thundering herd.
- [[../94 Case Studies/Case - Slow Query vs Cache vs Read Replica|Case - Slow Query vs Cache vs Read Replica]] — cache stampede sebagai bentuk lain thundering herd.

## Catatan Saya

*Kosong — diisi pembaca.*
