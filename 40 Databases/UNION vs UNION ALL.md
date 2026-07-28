---
title: UNION vs UNION ALL
type: concept
level: junior
domain: databases
status: unread
difficulty: 2
est_minutes: 12
prerequisites: ["[[Window Functions]]"]
next: ["[[NULL Semantics and Three-Valued Logic]]"]
tags: [backend, databases, performance]
created: 2026-07-28
---

## TL;DR

`UNION` menggabungkan hasil dua (atau lebih) query jadi satu hasil, **lalu membuang baris duplikat** — persis seperti `SELECT DISTINCT` diterapkan ke gabungan keduanya. `UNION ALL` menggabungkan tanpa membuang apa pun. Perbedaan ini terdengar kosmetik sampai kamu sadar deduplikasi butuh membandingkan **setiap baris dengan setiap baris lain** di seluruh hasil gabungan — pada dataset besar, ini bisa jadi operasi yang jauh lebih mahal daripada query itu sendiri. Memakai `UNION` padahal tahu hasilnya tidak mungkin punya duplikat (atau duplikatnya memang tidak masalah) adalah salah satu "pemborosan performa gratis" yang paling mudah dihindari.

## The Problem

Sebuah laporan gabungan menyatukan data dari dua sumber: `permohonan_online` dan `permohonan_manual` (dua tabel terpisah karena berasal dari sistem lama yang belum sepenuhnya bermigrasi). Query awal:

```sql
SELECT id, judul, 'online' AS sumber FROM permohonan_online
UNION
SELECT id, judul, 'manual' AS sumber FROM permohonan_manual;
```

Query ini benar secara hasil, tapi lambat — jauh lebih lambat daripada dua query terpisah dijalankan berurutan. Penyebabnya: `id` di kedua tabel berasal dari sequence/auto-increment yang **berbeda** (tidak pernah bertabrakan secara desain), dan kolom `sumber` sudah membuat setiap baris otomatis unik di antara kedua sisi. Tidak ada satu pun baris yang benar-benar bisa jadi duplikat di sini — tapi `UNION` tetap memaksa database membandingkan seluruh baris satu sama lain untuk memastikan itu, pekerjaan sia-sia pada dataset yang bisa berisi jutaan baris permohonan gabungan. Mengganti `UNION` menjadi `UNION ALL` menghilangkan langkah deduplikasi yang memang tidak pernah dibutuhkan, dan laporan yang sama jadi jauh lebih cepat.

## Intuition

Bayangkan `UNION ALL` seperti **menuangkan isi dua ember ke dalam satu ember besar** — apa pun yang ada di kedua ember, semuanya masuk, termasuk kalau kebetulan ada dua kelereng identik dari ember berbeda. `UNION` seperti melakukan hal yang sama, lalu **memeriksa setiap kelereng di ember besar satu per satu**, membuang yang persis sama dengan kelereng lain yang sudah ada.

Analogi ini bocor pada satu hal: memeriksa "kelereng identik" terdengar sepele untuk sedikit kelereng, tapi database harus melakukannya dengan cara yang menjamin kebenaran pada **jutaan baris** — biasanya lewat sorting atau hashing seluruh hasil gabungan terlebih dulu. Biaya ini bertumbuh bersama ukuran data, sementara `UNION ALL` tidak pernah membayar biaya itu sama sekali karena ia tidak pernah berjanji hasilnya bebas duplikat.

## How It Works

Syarat wajib kedua bentuk: jumlah kolom di tiap `SELECT` harus sama, dan tipe data di posisi kolom yang sama harus kompatibel (nama kolom di hasil akhir mengikuti `SELECT` yang pertama).

```sql
-- UNION ALL: semua baris digabung, termasuk duplikat kalau ada
SELECT nama, kota FROM pelanggan_lama
UNION ALL
SELECT nama, kota FROM pelanggan_baru;

-- UNION: hasil gabungan disaring, baris yang persis sama di semua kolom dihapus
SELECT nama, kota FROM pelanggan_lama
UNION
SELECT nama, kota FROM pelanggan_baru;
```

`ORDER BY` untuk keseluruhan hasil gabungan hanya boleh ditulis **sekali**, di akhir, setelah bagian `UNION`/`UNION ALL` terakhir — bukan di tiap `SELECT` individual (kecuali dibungkus subquery terpisah dengan `LIMIT` per bagian).

```sql
SELECT id, judul FROM permohonan_online
UNION ALL
SELECT id, judul FROM permohonan_manual
ORDER BY judul;
```

## In Go

```go
package main

import (
	"context"
	"database/sql"
	"fmt"
)

type PermohonanGabungan struct {
	ID     int
	Judul  string
	Sumber string
}

// AmbilPermohonanGabungan memakai UNION ALL secara sengaja, karena kolom
// "sumber" sudah membuat baris dari kedua tabel selalu berbeda — tidak
// pernah ada duplikat asli yang perlu dibuang UNION.
func AmbilPermohonanGabungan(ctx context.Context, db *sql.DB) ([]PermohonanGabungan, error) {
	rows, err := db.QueryContext(ctx, `
		SELECT id, judul, 'online' AS sumber FROM permohonan_online
		UNION ALL
		SELECT id, judul, 'manual' AS sumber FROM permohonan_manual
		ORDER BY judul
	`)
	if err != nil {
		return nil, fmt.Errorf("query permohonan gabungan: %w", err)
	}
	defer rows.Close()

	var hasil []PermohonanGabungan
	for rows.Next() {
		var p PermohonanGabungan
		if err := rows.Scan(&p.ID, &p.Judul, &p.Sumber); err != nil {
			return nil, fmt.Errorf("scan baris permohonan gabungan: %w", err)
		}
		hasil = append(hasil, p)
	}
	if err := rows.Err(); err != nil {
		return nil, fmt.Errorf("iterasi baris permohonan gabungan: %w", err)
	}
	return hasil, nil
}
```

## In His Stack

Skenario "dua tabel dari sistem lama dan sistem baru yang belum sepenuhnya bermigrasi" sangat umum di lingkungan dengan 13+ aplikasi yang berkembang bertahun-tahun — `UNION`/`UNION ALL` sering jadi cara cepat menyatukan tampilan data lintas skema tanpa menyentuh migrasi sungguhan. Yii2 `ActiveQuery` tidak punya method bawaan untuk `UNION` yang idiomatic; biasanya ditulis lewat `Query::union()` yang tersedia di query builder murni (bukan `ActiveRecord`), atau langsung sebagai raw SQL — konsekuensinya query jenis ini cenderung lebih jarang lewat lapisan proteksi `ActiveRecord` dan perlu diperiksa lebih hati-hati untuk SQL injection kalau ada bagian dinamis.

## Trade-offs and When Not To Use It

`UNION ALL` hampir selalu pilihan default yang tepat kecuali kamu **benar-benar tahu** hasil gabungannya bisa mengandung duplikat asli yang harus dihapus untuk kebenaran bisnis (misalnya menggabungkan dua sumber yang bisa saja mencatat entitas yang sama). Kalau ragu, tanyakan dulu "apakah kedua sisi query ini bisa secara desain menghasilkan baris yang identik di semua kolom?" — kalau jawabannya tidak (seperti kolom `sumber` yang selalu berbeda di atas), `UNION` hanya membayar biaya pemeriksaan yang hasilnya sudah pasti "tidak ada yang dibuang."

## Common Mistakes

> [!warning] Jebakan
> Memakai `UNION` sebagai kebiasaan default tanpa mempertimbangkan apakah deduplikasi memang dibutuhkan — pada dataset besar, biaya sorting/hashing untuk deduplikasi bisa mendominasi total waktu eksekusi query.

> [!warning] Jebakan
> Menulis `ORDER BY` di tiap `SELECT` individual sebelum `UNION`/`UNION ALL`, dengan asumsi urutan itu akan dipertahankan di hasil akhir — urutan hasil gabungan hanya dijamin oleh `ORDER BY` di akhir seluruh statement.

> [!warning] Jebakan
> Menggabungkan `SELECT` dengan jumlah kolom atau tipe data yang tidak benar-benar sepadan di posisi yang sama — beberapa mesin database melakukan konversi tipe implisit yang bisa menghasilkan data yang salah tanpa error, alih-alih menolak query.

## Exercises

1. Jelaskan kenapa `UNION ALL` pada dua tabel yang skema primary key-nya dijamin tidak pernah tumpang tindih tidak pernah butuh deduplikasi.
2. Tulis query yang menggabungkan hasil dari tiga tabel sekaligus dengan `UNION ALL`, lalu urutkan hasil akhirnya berdasarkan satu kolom.
3. Kenapa `ORDER BY` di dalam salah satu `SELECT` sebelum `UNION` biasanya tidak berpengaruh terhadap urutan hasil akhir?
4. Desain terbuka: sistem menyatukan data pelanggan dari dua database berbeda (hasil akuisisi dua sistem lama yang belum digabung), dan ada kemungkinan **nyata** beberapa pelanggan terdaftar di kedua sistem dengan data yang sedikit berbeda (typo nama, format nomor telepon berbeda). Rancang strategi query untuk laporan "daftar pelanggan unik gabungan" — jelaskan kenapa `UNION` biasa (yang hanya membuang baris identik persis) tidak cukup untuk kasus ini, dan pendekatan apa yang lebih tepat.

> [!success]- Kunci jawaban
> **4.** `UNION` biasa hanya membuang baris yang **identik persis di semua kolom** — kalau nama pelanggan yang sama tertulis "Budi Santoso" di satu sistem dan "budi santoso" (kapitalisasi berbeda) atau dengan nomor telepon berformat berbeda di sistem lain, `UNION` akan tetap memperlakukan keduanya sebagai baris berbeda dan tidak membuang satu pun. Ini bukan lagi masalah `UNION` vs `UNION ALL` — ini masalah *entity resolution* atau *deduplication* di lapisan bisnis, yang butuh normalisasi data terlebih dulu (menyeragamkan kapitalisasi, format nomor telepon) dan kemungkinan pencocokan fuzzy (misalnya berdasarkan kombinasi nama + tanggal lahir + sebagian nomor identitas), bukan sesuatu yang bisa diselesaikan `UNION`/`UNION ALL` sendirian.

## Self-Check

- Apa perbedaan tepat antara `UNION` dan `UNION ALL` dalam hal hasil dan biaya eksekusi?
- Kapan `UNION ALL` aman dipakai tanpa risiko menghasilkan duplikat yang tidak diinginkan?
- Di mana `ORDER BY` harus ditulis untuk mengurutkan keseluruhan hasil gabungan `UNION`/`UNION ALL`?
- Kenapa `UNION` tidak bisa dipakai untuk membersihkan data pelanggan duplikat yang formatnya sedikit berbeda antar sumber?

## Connected Notes

- [[Window Functions]] — sama-sama beroperasi lintas baris, tapi window function menambah informasi ke baris yang ada, sementara `UNION`/`UNION ALL` menggabungkan baris dari sumber berbeda.
- [[Join Types and Their Mental Models]] — kontras penting: `JOIN` menggabungkan **kolom** dari dua tabel berdasarkan syarat kecocokan, `UNION`/`UNION ALL` menggabungkan **baris** dari dua hasil dengan struktur kolom yang sama.
- [[Set Operations in SQL]] — `UNION` adalah satu dari beberapa operasi himpunan SQL; `INTERSECT` dan `EXCEPT` melengkapi keluarga operasi ini.
- [[NULL Semantics and Three-Valued Logic]] — deduplikasi `UNION` memperlakukan dua `NULL` sebagai "sama" untuk keperluan pembuangan duplikat, berbeda dari perilaku `NULL = NULL` di `WHERE`.
- [[Reading EXPLAIN]] — cara memverifikasi langsung apakah `UNION` menambahkan langkah sort/dedup yang mahal di rencana eksekusi, dibanding `UNION ALL`.

## Further Reading

- Dokumentasi resmi PostgreSQL dan MySQL/MariaDB, bagian "UNION Clause" — keduanya menjelaskan syarat kompatibilitas kolom dan tipe secara referensi lengkap.

## Catatan Saya

*Tulis di sini query gabungan di kerjaanmu yang memakai UNION — periksa apakah deduplikasinya benar-benar dibutuhkan atau bisa diganti UNION ALL.*
