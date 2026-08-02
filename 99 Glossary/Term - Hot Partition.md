---
title: Term - Hot Partition
type: term
level: senior
domain: distributed
status: unread
difficulty: 3
est_minutes: 3
prerequisites: []
next: []
tags: [backend, distributed, databases]
created: 2026-08-02
---

**Hot partition** adalah satu atau beberapa partisi/shard dalam sistem yang di-sharding yang menerima beban jauh lebih besar dari partisi lain — biasanya karena shard key yang dipilih tidak mencerminkan pola akses nyata, membuat data (atau permintaan terhadap data) terkonsentrasi tidak merata meski secara volume data terlihat terbagi rata di atas kertas.

Ini kenapa istilah ini penting dipahami: sistem yang di-sharding untuk skalabilitas tetap dibatasi kapasitasnya oleh hot partition — menambah lebih banyak shard tidak membantu kalau beban tetap terkonsentrasi di satu atau sedikit shard yang sama, sesuatu yang baru terlihat jelas setelah sistem berjalan dengan traffic nyata, bukan dari desain di atas kertas.

## Muncul Di

- [[../60 Distributed Systems/Sharding Strategies and Hot Partitions|Sharding Strategies and Hot Partitions]] — pembahasan penuh penyebab dan strategi menghindari hot partition.
- [[../60 Distributed Systems/Consistent Hashing|Consistent Hashing]] — teknik yang membantu mengurangi risiko hot partition lewat virtual node.
- [[../60 Distributed Systems/Multi-Tenancy|Multi-Tenancy]] — hot partition akibat satu tenant yang jauh lebih besar dari tenant lain.

## Catatan Saya

*Kosong — diisi pembaca.*
