---
title: NATS
type: tool
level: intermediate
domain: tools
status: unread
difficulty: 2
est_minutes: 9
depth: orientation
volatility: low
implements: ["[[../30 APIs and Web/Queue vs Log Semantics|Queue vs Log Semantics]]"]
prerequisites: ["[[../30 APIs and Web/Queue vs Log Semantics|Queue vs Log Semantics]]"]
next: ["[[Consul]]"]
tags: [backend, tools, apis]
created: 2026-08-02
---

## What It Is, In One Paragraph

NATS adalah sistem messaging ringan yang dirancang untuk kesederhanaan dan performa tinggi — binary kecil, latency sangat rendah, dan model pemakaian yang jauh lebih sederhana dibanding Kafka atau RabbitMQ, dengan biaya fitur yang lebih sedikit (secara default tidak punya persistence sekuat Kafka, meski ekstensi JetStream menambah kemampuan itu).

## The Concept It Implements

NATS adalah alternatif ringan untuk [[../30 APIs and Web/Queue vs Log Semantics|Queue vs Log Semantics]] — mendukung pola publish-subscribe sederhana secara default, dan (lewat JetStream) semantik log yang lebih mendekati Kafka untuk kebutuhan persistence dan replay.

## Kapan Ini Dipakai

NATS paling masuk akal untuk kebutuhan messaging yang benar-benar sederhana dan butuh latency serendah mungkin — komunikasi antar service internal yang tidak butuh retensi pesan panjang atau garansi delivery seketat Kafka. Untuk kebutuhan yang butuh audit trail, replay panjang, atau ekosistem tooling yang sudah matang (Debezium, Kafka Connect), Kafka tetap pilihan yang lebih matang. NATS bernilai justru saat kesederhanaan operasional (satu binary kecil, konfigurasi minimal) lebih penting dibanding fitur lengkap.

## Mental Model Singkat

Model dasar: **publisher** mengirim pesan ke **subject** (nama topik, mendukung wildcard hierarkis seperti `kasus.*.diajukan`), **subscriber** mendengarkan subject tertentu. Mode dasar NATS bersifat fire-and-forget (tidak ada persistence) — JetStream menambah lapisan persistence opsional di atasnya untuk kebutuhan yang butuh jaminan pengiriman lebih kuat.

## Contoh Konkret

```go
import "github.com/nats-io/nats.go"

nc, _ := nats.Connect(nats.DefaultURL)
nc.Publish("kasus.diajukan", []byte(`{"id":"123"}`))

nc.Subscribe("kasus.*", func(m *nats.Msg) {
	process(m.Data)
})
```

## Kapan Memilih Ini vs Alternatif

Pilih NATS untuk komunikasi internal yang butuh latency sangat rendah dan tidak butuh persistence kompleks. Pilih [[Kafka]] untuk kebutuhan audit trail, replay panjang, atau throughput sangat tinggi dengan ekosistem tooling matang. Pilih [[RabbitMQ]] untuk kebutuhan routing pesan kompleks dengan semantik queue tradisional.

> [!warning] Jebakan
> Mengasumsikan NATS mode dasar (tanpa JetStream) menyimpan pesan seperti Kafka — mode dasar bersifat fire-and-forget, pesan hilang kalau tidak ada subscriber aktif saat dikirim.

## Version Caveat

JetStream (fitur persistence NATS) relatif baru dibanding NATS core — verifikasi dokumentasi resmi nats.io untuk kematangan fitur di versi yang benar-benar dipakai.

## Connected Notes

- [[../30 APIs and Web/Queue vs Log Semantics|Queue vs Log Semantics]] — pola messaging yang diimplementasikan NATS, baik mode dasar maupun JetStream.
- [[Kafka]] — alternatif lebih matang untuk kebutuhan persistence dan replay.
- [[RabbitMQ]] — alternatif dengan semantik queue tradisional dan routing lebih kompleks.

## Catatan Saya

*Kosong — diisi pembaca.*
