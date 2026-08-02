---
title: RabbitMQ
type: tool
level: intermediate
domain: tools
status: unread
difficulty: 3
est_minutes: 15
depth: working
volatility: medium
implements: ["[[../30 APIs and Web/Queue vs Log Semantics|Queue vs Log Semantics]]"]
prerequisites: ["[[../30 APIs and Web/Queue vs Log Semantics|Queue vs Log Semantics]]"]
next: ["[[Debezium]]"]
tags: [backend, tools, apis, go]
created: 2026-08-02
---

## What It Is, In One Paragraph

RabbitMQ adalah message broker yang mengimplementasikan semantik queue tradisional (pesan dihapus setelah dikonsumsi dan diakui), dengan sistem routing pesan yang fleksibel lewat exchange — kontras langsung dengan Kafka yang mengimplementasikan semantik log.

## The Concept It Implements

RabbitMQ adalah kontras langsung untuk [[../30 APIs and Web/Queue vs Log Semantics|Queue vs Log Semantics]] — semantik queue (bukan log) yang dibahas di note itu adalah pilihan desain inti RabbitMQ.

## Mental Model

Tiga bagian: **exchange** (menerima pesan dari producer, memutuskan queue mana yang harus menerimanya berdasarkan tipe exchange — direct, topic, fanout, headers); **queue** (tempat pesan menunggu diambil consumer, dihapus setelah di-acknowledge); **binding** (aturan yang menghubungkan exchange ke queue tertentu, menentukan routing).

```mermaid
flowchart LR
    P["Producer"] --> Ex["Exchange\n(direct/topic/fanout)"]
    Ex -->|"binding rule"| Q1["Queue 1"]
    Ex -->|"binding rule"| Q2["Queue 2"]
    Q1 --> C1["Consumer"]
```

## The 20% You Actually Use

```go
import amqp "github.com/rabbitmq/amqp091-go"

conn, _ := amqp.Dial("amqp://guest:guest@localhost:5672/")
ch, _ := conn.Channel()

ch.QueueDeclare("kasus-notifications", true, false, false, false, nil)
ch.Publish("", "kasus-notifications", false, false, amqp.Publishing{
	ContentType: "application/json",
	Body:        []byte(`{"event":"diajukan"}`),
})

msgs, _ := ch.Consume("kasus-notifications", "", false, false, false, false, nil)
for msg := range msgs {
	process(msg.Body)
	msg.Ack(false) // acknowledge SETELAH proses berhasil, bukan sebelum
}
```

## Configuration That Bites

Meng-acknowledge pesan **sebelum** memastikan pemrosesan benar-benar berhasil (auto-ack, atau ack terlalu dini di kode) berisiko kehilangan pesan kalau proses gagal setelah ack terkirim — pesan yang sudah di-ack dianggap RabbitMQ sudah selesai ditangani, tidak akan dikirim ulang meski pemrosesannya sebenarnya gagal.

## Operating and Debugging It

RabbitMQ Management Plugin (UI web bawaan) menunjukkan jumlah pesan per queue, consumer yang terhubung, dan laju pesan masuk-keluar — titik pertama diperiksa saat pesan menumpuk (consumer tidak mengimbangi) atau hilang tak terduga.

## Choosing It

Dibanding [[Kafka]]: RabbitMQ lebih cocok untuk routing pesan kompleks dan kasus di mana semantik "pesan hilang setelah dikonsumsi" memang diinginkan; Kafka unggul untuk throughput sangat tinggi dan kebutuhan replay/retensi panjang. Dibanding [[NATS]]: RabbitMQ punya fitur routing dan persistence yang lebih matang; NATS lebih ringan dan sederhana untuk kebutuhan messaging dasar.

## Gotchas

> [!warning] Jebakan
> Meng-acknowledge pesan sebelum pemrosesan benar-benar selesai dan berhasil — pesan yang gagal diproses setelah ack terkirim hilang permanen, tidak akan dikirim ulang.

## Version Caveat

Dokumentasi resmi rabbitmq.com adalah sumber kebenaran untuk fitur exchange type dan plugin yang benar-benar dipakai untuk versi tertentu.

## Connected Notes

- [[../30 APIs and Web/Queue vs Log Semantics|Queue vs Log Semantics]] — semantik queue yang diimplementasikan konkret oleh RabbitMQ, kontras dengan Kafka.
- [[Kafka]] — kontras langsung sebagai implementasi semantik log.
- [[../30 APIs and Web/Idempotent Consumers|Idempotent Consumers]] — kebutuhan yang sama relevan untuk consumer RabbitMQ, terutama saat redelivery terjadi setelah nack atau timeout.

## Catatan Saya

*Kosong — diisi pembaca.*
