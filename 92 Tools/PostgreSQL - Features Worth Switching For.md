---
title: PostgreSQL - Features Worth Switching For
type: tool
level: intermediate
domain: tools
status: unread
difficulty: 3
est_minutes: 16
depth: deep
volatility: medium
implements: ["[[../40 Databases/Deliberate Denormalisation|Deliberate Denormalisation]]"]
prerequisites: ["[[PostgreSQL]]"]
next: ["[[PostgreSQL - Locking and SELECT FOR UPDATE]]"]
tags: [backend, tools, databases, go]
created: 2026-08-02
---

## What It Is, In One Paragraph

Sub-note ini bukan tentang PostgreSQL secara umum — ini daftar fitur spesifik yang, kalau kamu sedang membandingkan PostgreSQL dengan MariaDB/MySQL untuk proyek baru, sering jadi alasan konkret dan nyata (bukan preferensi abstrak) untuk memilih PostgreSQL: `RETURNING`, `JSONB` dengan index GIN, `CHECK` constraint yang sungguhan ditegakkan, array sebagai tipe kolom asli, `ON CONFLICT` untuk upsert, partial dan expression index, CTE dan window function yang matang, serta `LISTEN`/`NOTIFY`.

## The Concept It Implements

Sebagian besar fitur di sini adalah implementasi konkret dari [[../40 Databases/Deliberate Denormalisation|Deliberate Denormalisation]] yang disengaja — `JSONB` khususnya memungkinkan menyimpan data semi-terstruktur di dalam model relasional tanpa mengorbankan kemampuan query dan index, jembatan praktis antara dunia relasional ketat dan fleksibilitas dokumen.

## Mental Model

Pikirkan fitur-fitur ini sebagai empat kelompok: **kenyamanan menulis** (`RETURNING`, `ON CONFLICT`), **fleksibilitas skema tanpa kehilangan query power** (`JSONB` + GIN index, array), **penegakan aturan bisnis di level database** (`CHECK` constraint sungguhan, bukan sekadar validasi aplikasi), dan **query yang lebih ekspresif** (CTE, window function, partial/expression index).

## The 20% You Actually Use

```sql
-- ON CONFLICT: upsert dalam satu statement, tanpa race condition
-- SELECT-lalu-INSERT-atau-UPDATE terpisah
INSERT INTO kasus_counter (tanggal, jumlah) VALUES (CURRENT_DATE, 1)
ON CONFLICT (tanggal) DO UPDATE SET jumlah = kasus_counter.jumlah + 1;

-- JSONB dengan index GIN: query cepat pada data semi-terstruktur
CREATE INDEX idx_metadata ON kasus USING GIN (metadata);
SELECT * FROM kasus WHERE metadata @> '{"wilayah": "jawa-barat"}';

-- Partial index: hanya mengindeks baris yang relevan, jauh lebih kecil
CREATE INDEX idx_kasus_aktif ON kasus (created_at) WHERE status != 'selesai';

-- CHECK constraint yang SUNGGUHAN ditegakkan
ALTER TABLE kasus ADD CONSTRAINT status_valid
	CHECK (status IN ('diajukan', 'diverifikasi', 'disetujui', 'ditolak'));
```

```go
// RETURNING menghindari query terpisah untuk mendapat ID/timestamp
// yang baru dibuat database
var id int64
var createdAt time.Time
err := pool.QueryRow(ctx,
	`INSERT INTO kasus (nomor, status) VALUES ($1, $2) RETURNING id, created_at`,
	nomor, status,
).Scan(&id, &createdAt)
if err != nil {
	return fmt.Errorf("insert kasus: %w", err)
}
```

## Configuration That Bites

`JSONB` tanpa index GIN pada kolom yang sering di-query berdasarkan isinya akan melakukan full table scan setiap kali — kekuatan `JSONB` hanya terasa penuh kalau index yang tepat dipasang; menganggapnya otomatis cepat tanpa index adalah kesalahan umum pemula PostgreSQL yang baru pindah dari sistem NoSQL murni.

## Operating and Debugging It

Untuk memverifikasi index GIN benar-benar dipakai (bukan full scan), jalankan `EXPLAIN ANALYZE` pada query `JSONB` — cari `Bitmap Index Scan` menggunakan index GIN yang dimaksud di rencana eksekusi, bukan `Seq Scan`.

## Choosing It

Dibanding menyimpan data semi-terstruktur di kolom `TEXT` dan mem-parsing di aplikasi: `JSONB` dengan GIN index memberi query dan filtering di level database tanpa menarik seluruh baris ke aplikasi dulu. Dibanding memakai MongoDB murni untuk kebutuhan serupa: `JSONB` di PostgreSQL memberi fleksibilitas dokumen **plus** transaksi ACID dan join relasional penuh — pilih MongoDB murni hanya kalau seluruh model data memang dokumen-sentris dan tidak butuh relasi ketat sama sekali.

## Gotchas

> [!warning] Jebakan
> Memakai `JSONB` untuk data yang sebenarnya punya struktur tetap dan jelas — kolom relasional biasa dengan tipe data eksplisit tetap lebih baik untuk data yang strukturnya sudah diketahui, `JSONB` paling bernilai untuk data yang memang bervariasi strukturnya.

> [!warning] Jebakan
> Menambahkan `CHECK` constraint setelah tabel sudah berisi jutaan baris tanpa mempertimbangkan waktu validasi — PostgreSQL memvalidasi seluruh baris yang ada saat constraint ditambahkan, operasi yang bisa memakan waktu signifikan pada tabel besar.

## Version Caveat

Fitur `ON CONFLICT` (upsert) tersedia sejak PostgreSQL 9.5; `JSONB` sejak 9.4; detail performa GIN index bisa berbeda antar versi — dokumentasi resmi postgresql.org adalah sumber kebenaran untuk versi yang benar-benar dipakai.

## Connected Notes

- [[PostgreSQL]] — note utama yang dilengkapi sub-note ini dengan fitur spesifik yang jadi pembeda kuat.
- [[../40 Databases/Deliberate Denormalisation|Deliberate Denormalisation]] — `JSONB` adalah implementasi konkret trade-off denormalisasi yang disengaja.
- [[PostgreSQL vs MySQL - How To Choose]] — fitur-fitur di note ini sering jadi argumen konkret dalam perbandingan itu.
- [[../40 Databases/Upserts|Upserts]] — `ON CONFLICT` adalah implementasi PostgreSQL untuk pola upsert yang dibahas umum di note itu.

## Catatan Saya

*Kosong — diisi pembaca.*
