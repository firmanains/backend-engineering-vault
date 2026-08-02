---
title: Term - Fan-Out (Messaging)
type: term
level: intermediate
domain: apis
status: unread
difficulty: 1
est_minutes: 3
prerequisites: []
next: []
tags: [backend, apis]
created: 2026-08-02
---

**Fan-out** adalah pola di mana satu event atau pesan disebarkan ke **banyak** consumer independen, masing-masing memprosesnya untuk tujuan berbeda-beda tanpa saling tahu atau bergantung satu sama lain — misalnya satu event "kasus disetujui" yang memicu service notifikasi, service pencatatan, dan service analitik sekaligus, masing-masing membaca event yang sama dari topic atau exchange yang sama.

Ini kenapa istilah ini penting dipahami: fan-out adalah salah satu alasan utama messaging berbasis publish-subscribe (Kafka consumer group berbeda, atau exchange type `fanout` di RabbitMQ) lebih fleksibel dibanding pemanggilan langsung satu-ke-satu — menambah consumer baru tidak butuh mengubah kode pengirim sama sekali.

## Muncul Di

- [[../30 APIs and Web/Consumer Groups and Rebalancing|Consumer Groups and Rebalancing]] — consumer group berbeda yang masing-masing membaca topic yang sama adalah bentuk konkret fan-out di Kafka.
- [[../92 Tools/RabbitMQ|RabbitMQ]] — exchange type `fanout` yang secara literal mengimplementasikan pola ini.

## Catatan Saya

*Kosong — diisi pembaca.*
