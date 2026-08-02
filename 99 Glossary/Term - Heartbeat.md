---
title: Term - Heartbeat
type: term
level: senior
domain: distributed
status: unread
difficulty: 1
est_minutes: 3
prerequisites: []
next: []
tags: [backend, distributed]
created: 2026-08-02
---

**Heartbeat** adalah sinyal kecil yang dikirim secara berkala oleh sebuah node atau proses untuk memberi tahu pihak lain bahwa ia masih hidup dan berfungsi normal — ketiadaan heartbeat dalam jangka waktu tertentu jadi sinyal (meski tidak pasti) bahwa node itu mungkin sudah mati atau tidak bisa dihubungi.

Ini kenapa istilah ini penting dipahami: heartbeat adalah mekanisme dasar di balik hampir semua failure detector (lihat [[../60 Distributed Systems/Failure Detectors|Failure Detectors]]), leader election (leader mengirim heartbeat ke follower; ketiadaannya memicu election baru di Raft), dan health check aplikasi (liveness probe Kubernetes pada dasarnya adalah heartbeat yang diperiksa dari luar, bukan dikirim aktif oleh Pod).

## Muncul Di

- [[../60 Distributed Systems/Failure Detectors|Failure Detectors]] — heartbeat sebagai mekanisme dasar mendeteksi node yang mungkin mati.
- [[../60 Distributed Systems/Consensus - Raft|Consensus - Raft]] — heartbeat leader ke follower yang memicu election baru kalau berhenti.
- [[../70 Infrastructure and Delivery/Kubernetes Config, Secrets, Probes, and Autoscaling|Kubernetes Config, Secrets, Probes, and Autoscaling]] — liveness/readiness probe sebagai bentuk heartbeat praktis.

## Catatan Saya

*Kosong — diisi pembaca.*
