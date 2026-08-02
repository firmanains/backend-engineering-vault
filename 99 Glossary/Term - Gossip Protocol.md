---
title: Term - Gossip Protocol
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

**Gossip protocol** adalah cara menyebarkan informasi (status keanggotaan cluster, metadata, atau nilai yang berubah) ke seluruh node dalam sistem terdistribusi lewat komunikasi peer-to-peer acak — setiap node secara berkala memilih beberapa node lain secara acak dan bertukar informasi terkini dengan mereka, mirip cara gosip menyebar dari mulut ke mulut di masyarakat. Setelah beberapa putaran, informasi itu menyebar ke seluruh cluster tanpa perlu satu titik koordinasi pusat yang tahu segalanya.

Ini kenapa istilah ini penting dipahami: gossip protocol memberi cara yang sangat tahan gangguan (tidak ada single point of failure, karena tidak ada koordinator pusat) untuk menyebarkan informasi, dengan trade-off waktu propagasi yang tidak instan (butuh beberapa putaran sebelum informasi benar-benar sampai ke semua node) — kontras dengan consensus formal (Raft) yang lebih cepat dan konsisten tapi butuh koordinasi eksplisit lewat leader.

## Muncul Di

- [[../60 Distributed Systems/Failure Detectors|Failure Detectors]] — gossip protocol adalah salah satu cara mendeteksi dan menyebarkan informasi node yang dicurigai mati.
- [[../60 Distributed Systems/Consensus - Raft|Consensus - Raft]] — kontras dengan pendekatan consensus formal yang berbeda filosofi dari gossip.

## Catatan Saya

*Kosong — diisi pembaca.*
