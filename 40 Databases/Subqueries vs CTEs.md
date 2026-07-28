---
title: Subqueries vs CTEs
type: concept
level: junior
domain: databases
status: unread
difficulty: 2
est_minutes: 15
prerequisites: ["[[Aggregation and GROUP BY Semantics]]"]
next: ["[[Window Functions]]"]
tags: [backend, databases]
created: 2026-07-28
---

## TL;DR

Subquery adalah query di dalam query — bisa muncul di `WHERE`, `FROM`, atau `SELECT`. CTE (`WITH nama AS (...)`) adalah cara menamai sebuah subquery dan menuliskannya **sekali di atas**, lalu memakainya seperti tabel biasa di query utama. Keduanya sering menghasilkan rencana eksekusi yang identik — bedanya murni keterbacaan untuk kasus sederhana. Tapi CTE punya satu kekuatan yang tidak dimiliki subquery biasa: ia bisa dipakai **lebih dari sekali** di query yang sama tanpa menulis ulang logikanya, dan (di database yang mendukung `WITH RECURSIVE`) bisa mengekspresikan query rekursif yang mustahil ditulis sebagai subquery biasa.

## The Problem

Sebuah query butuh menampilkan "permohonan yang jumlah dokumen lampirannya di atas rata-rata jumlah lampiran seluruh permohonan." Ditulis sebagai subquery bersarang:

```sql
SELECT p.id, p.judul
FROM permohonan p
WHERE (SELECT COUNT(*) FROM lampiran l WHERE l.permohonan_id = p.id) >
      (SELECT AVG(jml) FROM (
          SELECT COUNT(*) AS jml FROM lampiran GROUP BY permohonan_id
      ) sub);
```

Query ini benar, tapi tiga level bersarang membuatnya sulit dibaca ulang tiga bulan kemudian — dan kalau logika "jumlah lampiran per permohonan" perlu dipakai lagi di bagian lain query (misalnya juga ditampilkan sebagai kolom output), logikanya harus ditulis ulang persis sama di tempat kedua, dengan risiko kedua salinan diam-diam menjadi tidak sinkron kalau salah satu diubah saat maintenance. CTE memecahkan kedua masalah ini sekaligus: definisikan "jumlah lampiran per permohonan" satu kali, beri nama, lalu pakai namanya di mana pun dibutuhkan dalam query yang sama.

## Intuition

Bayangkan subquery bersarang seperti **kalimat dengan banyak anak kalimat di dalam anak kalimat** — gramatikal benar, tapi pembaca harus menahan konteks berlapis-lapis di kepala sebelum sampai ke maksud utama. CTE seperti **mendefinisikan istilah dulu di awal**, lalu memakai istilah itu di kalimat-kalimat berikutnya — "misalkan X adalah rata-rata jumlah lampiran; sekarang tunjukkan permohonan yang lampirannya lebih dari X."

Analogi ini bocor pada satu hal: mendefinisikan istilah dalam bahasa manusia tidak mengubah makna kalimat, tapi CTE **bisa** mengubah rencana eksekusi fisik dibanding subquery bersarang, tergantung database dan versinya — beberapa optimizer memperlakukan CTE sebagai "optimization fence" (dihitung sekali, hasilnya "dibekukan", baru dipakai query luar) sementara yang lain menyatukannya kembali (inline) ke query induk seolah-olah ditulis sebagai subquery biasa. Perilaku ini **berbeda antar versi dan antar mesin database**, jadi jangan berasumsi CTE selalu "gratis" secara performa dibanding subquery — untuk kasus yang sensitif performa, verifikasi lewat [[Reading EXPLAIN]], jangan lewat asumsi.

> [!question] Perlu diverifikasi
> Klaim: perilaku CTE sebagai "optimization fence" vs "inlined" bervariasi antar versi PostgreSQL dan MySQL/MariaDB.
> Kenapa ragu: perilaku ini memang pernah berubah signifikan di riwayat PostgreSQL (materialization behavior), dan detail versi persisnya mudah basi.
> Cara verifikasi: dokumentasi resmi PostgreSQL bagian "WITH Queries (Common Table Expressions)", cari kata kunci "MATERIALIZED".

## How It Works

Subquery di `WHERE` (dipakai sebagai nilai atau di dalam `IN`/perbandingan):

```sql
SELECT nama FROM instansi
WHERE id IN (SELECT instansi_id FROM permohonan WHERE tahun = 2026);
```

Subquery di `FROM` (diperlakukan seperti tabel turunan):

```sql
SELECT sub.instansi_id, sub.total
FROM (SELECT instansi_id, COUNT(*) AS total FROM permohonan GROUP BY instansi_id) sub
WHERE sub.total > 100;
```

CTE yang menyederhanakan contoh "The Problem":

```sql
WITH lampiran_per_permohonan AS (
    SELECT permohonan_id, COUNT(*) AS jumlah
    FROM lampiran
    GROUP BY permohonan_id
)
SELECT p.id, p.judul, lp.jumlah
FROM permohonan p
JOIN lampiran_per_permohonan lp ON lp.permohonan_id = p.id
WHERE lp.jumlah > (SELECT AVG(jumlah) FROM lampiran_per_permohonan);
```

Perhatikan `lampiran_per_permohonan` dipakai **dua kali** — sekali di `JOIN`, sekali di subquery `AVG()` — tanpa menulis ulang logika `COUNT() ... GROUP BY`-nya. Ini juga sekaligus contoh nyata bahwa urutan logis dari [[The Logical Order of Query Execution]] berlaku *di dalam* CTE secara independen sebelum hasilnya "disuntikkan" ke query luar.

CTE rekursif (`WITH RECURSIVE`) mengekspresikan sesuatu yang mustahil ditulis subquery biasa — misalnya menelusuri struktur hierarki:

```sql
WITH RECURSIVE atasan_dari(pegawai_id, nama, level) AS (
    SELECT id, nama, 0 FROM pegawai WHERE id = 42          -- anchor: titik awal
    UNION ALL
    SELECT p.id, p.nama, a.level + 1                        -- recursive: naik satu level
    FROM pegawai p
    JOIN atasan_dari a ON p.id = (SELECT atasan_id FROM pegawai WHERE id = a.pegawai_id)
)
SELECT * FROM atasan_dari ORDER BY level;
```

## In Go

```go
package main

import (
	"context"
	"database/sql"
	"fmt"
)

type PermohonanBanyakLampiran struct {
	ID            int
	Judul         string
	JumlahLampiran int
}

// AmbilPermohonanDiAtasRataRata memakai CTE, bukan subquery bersarang,
// karena "jumlah lampiran per permohonan" dipakai dua kali dalam query ini —
// sekali untuk JOIN utama, sekali untuk menghitung AVG() pembandingnya.
func AmbilPermohonanDiAtasRataRata(ctx context.Context, db *sql.DB) ([]PermohonanBanyakLampiran, error) {
	rows, err := db.QueryContext(ctx, `
		WITH lampiran_per_permohonan AS (
			SELECT permohonan_id, COUNT(*) AS jumlah
			FROM lampiran
			GROUP BY permohonan_id
		)
		SELECT p.id, p.judul, lp.jumlah
		FROM permohonan p
		JOIN lampiran_per_permohonan lp ON lp.permohonan_id = p.id
		WHERE lp.jumlah > (SELECT AVG(jumlah) FROM lampiran_per_permohonan)
		ORDER BY lp.jumlah DESC
	`)
	if err != nil {
		return nil, fmt.Errorf("query permohonan di atas rata-rata lampiran: %w", err)
	}
	defer rows.Close()

	var hasil []PermohonanBanyakLampiran
	for rows.Next() {
		var p PermohonanBanyakLampiran
		if err := rows.Scan(&p.ID, &p.Judul, &p.JumlahLampiran); err != nil {
			return nil, fmt.Errorf("scan baris permohonan: %w", err)
		}
		hasil = append(hasil, p)
	}
	if err := rows.Err(); err != nil {
		return nil, fmt.Errorf("iterasi baris permohonan: %w", err)
	}
	return hasil, nil
}
```

## In His Stack

Yii2 `ActiveQuery` tidak punya dukungan CTE bawaan yang idiomatic — kebanyakan proyek Yii2 yang butuh CTE menulisnya sebagai raw SQL lewat `Yii::$app->db->createCommand()`, keluar dari query builder sepenuhnya. Ini konsekuensi nyata yang layak dipertimbangkan saat memilih antara subquery bersarang (masih bisa diekspresikan lewat `ActiveQuery` biasa) dan CTE (biasanya butuh raw SQL) — trade-off keterbacaan CTE harus ditimbang lawan hilangnya proteksi query builder (escaping otomatis, dsb.) saat menulis raw SQL.

## Trade-offs and When Not To Use It

Untuk subquery sederhana yang dipakai sekali (misalnya satu `WHERE ... IN (SELECT ...)` singkat), membungkusnya jadi CTE hanya menambah baris tanpa manfaat nyata — subquery inline sudah cukup terbaca. CTE mulai bernilai begitu: (a) logikanya dipakai lebih dari sekali dalam query yang sama, (b) query-nya butuh beberapa tahap transformasi berurutan yang masing-masing punya nama yang bermakna (meningkatkan keterbacaan drastis dibanding bersarang tiga level), atau (c) butuh rekursi. Untuk query yang sangat sensitif performa, jangan berasumsi CTE otomatis "lebih efisien" atau "lebih lambat" dibanding subquery setara — selalu verifikasi lewat `EXPLAIN` di database dan versi yang sebenarnya dipakai produksi.

## Common Mistakes

> [!warning] Jebakan
> Menyalin-tempel logika subquery yang sama di beberapa tempat dalam satu query, alih-alih mengekstraknya jadi satu CTE — risiko kedua salinan diam-diam menjadi tidak sinkron saat salah satunya diubah tapi yang lain terlupa.

> [!warning] Jebakan
> Mengasumsikan CTE selalu lebih cepat (atau selalu lebih lambat) daripada subquery setara tanpa memverifikasi lewat `EXPLAIN` — perilaku materialization CTE bervariasi antar mesin database dan versi.

> [!warning] Jebakan
> Menulis `WITH RECURSIVE` tanpa syarat penghenti yang benar (anchor query yang tidak cukup membatasi, atau recursive query yang tidak pernah berhenti menghasilkan baris baru) — berisiko infinite loop yang membebani database sampai timeout atau kehabisan resource.

## Exercises

1. Tulis ulang sebuah subquery bersarang tiga level (buat contohmu sendiri dari domain permohonan) menjadi CTE, dan jelaskan bagian mana yang jadi lebih terbaca.
2. Kenapa CTE tidak otomatis lebih cepat dibanding subquery setara — apa yang menentukan mesin database memilih materialize atau inline sebuah CTE?
3. Kapan CTE biasa (non-rekursif) tidak cukup, dan `WITH RECURSIVE` benar-benar dibutuhkan? Beri satu contoh konkret dari domain kerja pemerintahan (misalnya struktur organisasi berjenjang).
4. Desain terbuka: kamu perlu membuat laporan "rantai persetujuan" sebuah permohonan — dari petugas pemroses awal sampai ke pejabat penandatangan akhir, di mana setiap level punya field `disetujui_oleh` yang menunjuk ke level sebelumnya, dan jumlah level tidak tetap (bisa 2, bisa 5, tergantung jenis permohonan). Rancang query untuk menampilkan seluruh rantai ini sebagai satu hasil terurut, dan jelaskan kenapa pendekatanmu tepat untuk kasus jumlah level yang tidak tetap ini.

> [!success]- Kunci jawaban
> **3.** `WITH RECURSIVE` dibutuhkan begitu kedalaman hubungan tidak diketahui di awal query — misalnya menelusuri seluruh rantai atasan seorang pegawai di struktur organisasi yang jenjangnya bisa berbeda-beda per unit kerja; subquery biasa (non-rekursif) harus tahu persis berapa level `JOIN` yang dibutuhkan saat query ditulis, jadi tidak bisa menangani kedalaman yang bervariasi.
> **4.** Karena jumlah level persetujuan tidak tetap, ini persis kasus yang butuh `WITH RECURSIVE`: anchor query memilih baris `log_persetujuan` paling awal untuk permohonan tersebut (level 0), recursive query terus mengambil baris berikutnya yang `disetujui_oleh`-nya merujuk ke baris level sebelumnya, sambil menambah counter `level`, sampai tidak ada baris berikutnya yang cocok (kondisi berhenti otomatis karena `JOIN` tidak lagi menemukan pasangan). Hasil akhirnya diurutkan berdasarkan `level` untuk menampilkan rantai dari awal sampai akhir dalam satu hasil, tanpa perlu tahu di awal berapa banyak level yang akan ditemukan.

## Self-Check

- Apa perbedaan mendasar subquery di `WHERE` dan subquery di `FROM`?
- Kapan CTE benar-benar memberi manfaat dibanding subquery bersarang biasa?
- Kenapa CTE tidak boleh diasumsikan selalu lebih cepat daripada subquery setara?
- Kapan `WITH RECURSIVE` dibutuhkan, dan kenapa subquery biasa tidak cukup untuk kasus itu?

## Connected Notes

- [[The Logical Order of Query Execution]] — urutan logis berlaku secara independen di dalam tiap subquery/CTE sebelum hasilnya dipakai query luar.
- [[Aggregation and GROUP BY Semantics]] — CTE sering dipakai justru untuk menampung hasil agregasi yang perlu dipakai ulang di beberapa tempat query utama.
- [[Window Functions]] — alternatif untuk sebagian kasus yang tadinya butuh subquery/CTE, terutama saat kebutuhannya adalah nilai agregat tanpa mengurangi jumlah baris.
- [[Reading EXPLAIN]] — satu-satunya cara benar memverifikasi apakah CTE dimaterialize atau di-inline oleh optimizer, alih-alih menebak.
- [[Relational Modelling]] — struktur hierarki (seperti rantai persetujuan atau struktur organisasi) yang butuh CTE rekursif berakar dari cara relasi itu dimodelkan.

## Further Reading

- Dokumentasi resmi PostgreSQL, bagian "WITH Queries (Common Table Expressions)" — termasuk penjelasan `RECURSIVE` dan `MATERIALIZED`/`NOT MATERIALIZED`.
- Dokumentasi resmi MySQL/MariaDB, bagian "WITH (Common Table Expressions)".

## Catatan Saya

*Tulis di sini query panjang di kerjaanmu yang menurutmu akan jauh lebih terbaca kalau ditulis ulang pakai CTE.*
