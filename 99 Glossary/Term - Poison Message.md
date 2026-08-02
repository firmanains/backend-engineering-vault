---
title: Term - Poison Message
type: term
level: intermediate
domain: apis
status: unread
difficulty: 2
est_minutes: 3
prerequisites: []
next: []
tags: [backend, apis]
created: 2026-08-02
---

**Poison message** adalah pesan di sebuah antrean atau topic yang **selalu** gagal diproses consumer, tidak peduli berapa kali dicoba ulang — biasanya karena isinya rusak, formatnya tidak sesuai yang diharapkan, atau memicu bug spesifik di logika pemrosesan. Berbeda dari kegagalan sesaat (yang biasanya berhasil di percobaan berikutnya), poison message akan terus gagal selamanya sampai ditangani secara eksplisit.

Ini kenapa istilah ini penting dipahami: consumer yang retry tanpa batas terhadap poison message bisa terjebak loop tak berkesudahan — memproses ulang pesan yang sama berulang kali tanpa pernah maju ke pesan berikutnya, memblokir seluruh antrean di belakangnya (mirip [[Term - Head-of-Line Blocking]]). Dead letter queue (lihat [[../30 APIs and Web/Dead Letter Queues|Dead Letter Queues]]) adalah mekanisme standar menangani ini — pesan yang gagal melebihi batas percobaan dipindah keluar dari jalur normal, bukan terus diulang.

## Muncul Di

- [[../30 APIs and Web/Dead Letter Queues|Dead Letter Queues]] — mekanisme utama menangani poison message.
- [[../94 Case Studies/Case - The Webhook That Was Down For Six Hours|Case - The Webhook That Was Down For Six Hours]] — konteks nyata di mana dead letter queue relevan untuk pesan yang terus gagal.

## Catatan Saya

*Kosong — diisi pembaca.*
