---
title: PostgreSQL
type: tool
level: intermediate
domain: tools
status: unread
difficulty: 3
est_minutes: 16
depth: deep
volatility: medium
implements: ["[[../40 Databases/ACID|ACID]]", "[[../40 Databases/Basic Isolation Levels|Basic Isolation Levels]]"]
prerequisites: ["[[../40 Databases/_Overview|Databases Overview]]"]
next: ["[[PostgreSQL - Features Worth Switching For]]"]
tags: [backend, tools, databases, go]
created: 2026-08-02
---

## What It Is, In One Paragraph

PostgreSQL adalah database relasional open-source yang dikenal luas sebagai yang paling ketat menegakkan standar SQL dan paling kaya fitur di antara database relasional open-source utama — bukan sekadar "MySQL versi lain", tapi mesin dengan filosofi desain yang berbeda sejak awal: extensibility (tipe data kustom, index kustom), penegakan constraint yang sungguhan (bukan sekadar diterima secara sintaks), dan komunitas yang secara historis lebih konservatif menambah fitur baru sampai benar-benar matang, dibanding mengejar kecepatan adopsi fitur.

## The Concept It Implements

PostgreSQL adalah implementasi konkret dari [[../40 Databases/ACID|ACID]] dan [[../40 Databases/Basic Isolation Levels|Basic Isolation Levels]] — gagasan transaksi, isolation level, dan konsistensi yang dibahas abstrak di domain databases diwujudkan di sini lewat mesin MVCC-nya sendiri. Memahami PostgreSQL memberi contoh konkret yang sangat berguna untuk memahami trade-off yang sama di MariaDB, meski detail implementasinya berbeda.

## Mental Model

Empat bagian yang perlu dipegang untuk bernalar tentang PostgreSQL: **MVCC** (setiap baris menyimpan beberapa versi, transaksi melihat snapshot sesuai isolation level-nya, bukan mengunci untuk baca seperti sebagian mesin lain); **WAL (Write-Ahead Log)** (setiap perubahan ditulis ke log berurutan sebelum diterapkan ke data sesungguhnya, dasar dari durability dan replikasi); **extension system** (fungsionalitas tambahan seperti `pg_stat_statements`, `PostGIS`, atau `pgvector` dipasang sebagai extension, bukan built-in permanen — inilah yang membuat PostgreSQL fleksibel tanpa membengkakkan core-nya); dan **planner berbasis cost** (setiap query dievaluasi banyak kemungkinan rencana eksekusi, dipilih yang diperkirakan termurah berdasarkan statistik tabel).

```mermaid
flowchart LR
    Client["Klien"] --> Planner["Query Planner\n(pilih rencana eksekusi termurah)"]
    Planner --> Exec["Executor"]
    Exec --> MVCC["MVCC\n(baca versi baris sesuai snapshot)"]
    Exec --> WAL["WAL\n(setiap perubahan dicatat dulu)"]
```

## The 20% You Actually Use

```sql
-- EXPLAIN ANALYZE: satu perintah paling sering dipakai memahami kenapa query lambat
EXPLAIN ANALYZE SELECT * FROM kasus WHERE status = 'diajukan' ORDER BY created_at DESC LIMIT 20;

-- RETURNING: dapatkan baris yang baru di-insert/update tanpa query terpisah
INSERT INTO kasus (nomor, status) VALUES ('K-2026-001', 'diajukan') RETURNING id, created_at;

-- Melihat koneksi aktif dan query yang sedang berjalan
SELECT pid, state, query, now() - query_start AS duration
FROM pg_stat_activity WHERE state != 'idle' ORDER BY duration DESC;
```

```go
// go-idiomatic: memakai driver pgx (lebih disukai dibanding lib/pq
// yang sudah masuk mode maintenance-only)
import "github.com/jackc/pgx/v5/pgxpool"

pool, err := pgxpool.New(ctx, "postgres://user:pass@localhost:5432/dbname")
if err != nil {
	return fmt.Errorf("membuka pool koneksi: %w", err)
}
defer pool.Close()
```

## Configuration That Bites

`max_connections` default sering terlalu rendah untuk aplikasi production dengan banyak instance yang masing-masing punya connection pool sendiri — total koneksi dari semua instance aplikasi bisa melebihi batas ini, menyebabkan error "too many connections" saat traffic naik. Solusinya bukan sekadar menaikkan angka ini tanpa batas (setiap koneksi memakan memori signifikan di sisi server) — pertimbangkan PgBouncer sebagai connection pooler di depan PostgreSQL untuk menyerap banyak koneksi aplikasi jadi kolam koneksi database yang lebih kecil dan efisien.

`shared_buffers` default konservatif (sering hanya sebagian kecil dari RAM yang tersedia) — untuk server dedicated PostgreSQL, angka ini biasanya perlu dinaikkan signifikan (aturan umum industri sekitar 25% RAM, meski ini bervariasi tergantung beban kerja) dari default yang dirancang aman untuk berbagai ukuran mesin, bukan dioptimalkan untuk mesin produksi tertentu.

## Operating and Debugging It

Saat query lambat, `EXPLAIN ANALYZE` adalah langkah pertama — perhatikan selisih antara `rows estimated` (perkiraan planner) dan `rows actual` (kenyataan); selisih besar biasanya menandakan statistik tabel usang (`ANALYZE` perlu dijalankan) yang membuat planner memilih rencana eksekusi yang salah. `pg_stat_activity` menunjukkan query yang sedang berjalan lama atau terkunci — kombinasikan dengan `pg_locks` untuk mendiagnosis deadlock atau query yang saling menunggu.

## Choosing It

Dibanding **MySQL/MariaDB**: PostgreSQL lebih ketat menegakkan standar SQL dan constraint, punya tipe data lebih kaya (JSONB, array, range types), tapi historisnya sedikit lebih berat secara resource untuk beban tulis sangat tinggi — lihat [[PostgreSQL vs MySQL - How To Choose]] untuk perbandingan lengkap. Dibanding **database NoSQL** (MongoDB, dsb): PostgreSQL (dengan JSONB) sering memberi fleksibilitas skema yang cukup tanpa mengorbankan transaksi ACID dan join yang kuat — pilih NoSQL murni hanya kalau kebutuhan skalabilitas horizontal atau pola akses benar-benar tidak cocok dengan model relasional sama sekali.

## Gotchas

> [!warning] Jebakan
> Lupa menjalankan `VACUUM` (atau mengandalkan autovacuum yang salah konfigurasi) pada tabel dengan banyak `UPDATE`/`DELETE` — baris lama yang tidak dibersihkan (dead tuples, konsekuensi langsung dari MVCC) menumpuk dan memperlambat query serta membengkakkan ukuran tabel di disk.

> [!warning] Jebakan
> Menjalankan migrasi `ALTER TABLE ADD COLUMN ... NOT NULL DEFAULT ...` pada tabel besar tanpa memeriksa versi PostgreSQL yang dipakai — versi lama mengunci tabel penuh untuk menulis ulang seluruh baris, versi yang lebih baru mengoptimalkan operasi ini jadi jauh lebih cepat; perilaku ini penting diverifikasi untuk versi yang benar-benar dipakai.

## Version Caveat

Note ini menulis perilaku umum yang berlaku luas di seri PostgreSQL 14 ke atas; detail spesifik (nama parameter, perilaku optimasi tertentu) bisa berbeda antar versi — dokumentasi resmi di postgresql.org adalah sumber kebenaran untuk versi yang benar-benar dipakai.

## Connected Notes

- [[../40 Databases/ACID|ACID]] — PostgreSQL adalah implementasi konkret jaminan ACID yang dibahas abstrak di note itu.
- [[PostgreSQL - Features Worth Switching For]] — kelanjutan langsung, fitur PostgreSQL yang jadi alasan kuat memilihnya dibanding alternatif.
- [[PostgreSQL - Locking and SELECT FOR UPDATE]] — pendalaman mekanisme locking PostgreSQL untuk kebutuhan konkurensi.
- [[PostgreSQL vs MySQL - How To Choose]] — perbandingan jujur dengan alternatif open-source terdekatnya.
- [[../40 Databases/Basic Isolation Levels|Basic Isolation Levels]] — isolation level yang diimplementasikan lewat mekanisme MVCC PostgreSQL.

## Catatan Saya

*Kosong — diisi pembaca.*
