---
title: Set Operations in SQL
type: concept
level: junior
domain: databases
status: unread
difficulty: 2
est_minutes: 13
prerequisites: ["[[NULL Semantics and Three-Valued Logic]]"]
next: ["[[Upserts]]"]
tags: [backend, databases]
created: 2026-07-28
---

## TL;DR

`UNION`/`UNION ALL` bukan satu-satunya operasi himpunan di SQL. `INTERSECT` mengembalikan baris yang muncul di **kedua** hasil query, dan `EXCEPT` (disebut `MINUS` di beberapa dialek seperti Oracle) mengembalikan baris dari query pertama yang **tidak** muncul di query kedua. Ketiganya membandingkan **seluruh baris**, bukan satu kolom — dua baris dianggap sama hanya kalau semua kolomnya identik. Mereka sering bisa digantikan `JOIN` atau `NOT IN`/`NOT EXISTS`, tapi untuk pertanyaan yang secara alami berbentuk "bandingkan dua himpunan hasil", menuliskannya sebagai operasi himpunan langsung jauh lebih jelas maksudnya daripada memaksakannya jadi join.

## The Problem

Tim audit butuh menemukan "pegawai yang terdaftar di sistem HR tapi **tidak** terdaftar di sistem akses gedung" — indikasi orang yang sudah resign tapi kartu aksesnya belum dinonaktifkan, atau sebaliknya, kejanggalan yang harus ditelusuri. Ditulis naif dengan `NOT IN`:

```sql
SELECT nik FROM pegawai_hr
WHERE nik NOT IN (SELECT nik FROM pegawai_akses_gedung);
```

Query ini punya jebakan tersembunyi: kalau subquery `SELECT nik FROM pegawai_akses_gedung` mengembalikan **satu saja** baris dengan `nik = NULL` (misalnya data yang belum lengkap diisi), seluruh hasil `NOT IN` berubah jadi **kosong** — bukan error, hanya hasil yang salah secara diam-diam. Ini konsekuensi langsung dari [[NULL Semantics and Three-Valued Logic]]: `NOT IN` secara semantik setara dengan rangkaian `<>` yang di-`AND`-kan, dan begitu salah satu dari rangkaian itu membandingkan dengan `NULL`, hasil keseluruhan `AND`-nya jatuh ke `UNKNOWN` untuk baris tersebut, membuang baris yang seharusnya lolos. `EXCEPT` tidak punya jebakan ini — ia secara eksplisit dirancang untuk perbandingan himpunan dan menangani `NULL` secara konsisten sebagai "sama dengan `NULL` lain" untuk keperluan pembandingan baris (persis seperti `UNION` melakukan deduplikasi).

## Intuition

Bayangkan dua himpunan sebagai dua **daftar tamu undangan** untuk dua acara berbeda. `INTERSECT` menjawab "siapa yang diundang ke **keduanya**?" `EXCEPT` menjawab "siapa yang diundang ke acara pertama tapi **tidak** ke acara kedua?" Ini benar-benar operasi himpunan matematis — tidak peduli urutan baris, tidak peduli berapa kali sebuah baris "seharusnya" muncul kalau ada duplikat (`INTERSECT`/`EXCEPT` standar sudah membuang duplikat di hasilnya, mirip `UNION`).

Analogi ini bocor pada satu hal: daftar tamu di dunia nyata membandingkan berdasarkan identitas orangnya (satu kolom), sementara `INTERSECT`/`EXCEPT` di SQL membandingkan **seluruh baris** sebagai satu kesatuan — dua baris dengan `nik` yang sama tapi kolom lain berbeda (misalnya `nama` yang salah ketik di satu sistem) dianggap **baris yang berbeda**, sehingga tidak akan cocok di `INTERSECT` dan akan ikut muncul di `EXCEPT` kedua arah. Kalau maksudnya membandingkan berdasarkan satu kolom identitas saja, `SELECT` harus dibatasi hanya kolom itu sebelum operasi himpunan diterapkan.

## How It Works

```sql
-- INTERSECT: pegawai yang terdaftar di KEDUA sistem
SELECT nik FROM pegawai_hr
INTERSECT
SELECT nik FROM pegawai_akses_gedung;

-- EXCEPT: pegawai HR yang TIDAK punya akses gedung
SELECT nik FROM pegawai_hr
EXCEPT
SELECT nik FROM pegawai_akses_gedung;
```

Versi yang setara memakai `JOIN`/`NOT EXISTS`, untuk perbandingan:

```sql
-- setara INTERSECT, lewat JOIN
SELECT DISTINCT h.nik FROM pegawai_hr h
JOIN pegawai_akses_gedung a ON a.nik = h.nik;

-- setara EXCEPT, lewat NOT EXISTS (aman terhadap NULL, tidak seperti NOT IN)
SELECT h.nik FROM pegawai_hr h
WHERE NOT EXISTS (
    SELECT 1 FROM pegawai_akses_gedung a WHERE a.nik = h.nik
);
```

`NOT EXISTS` aman terhadap `NULL` karena ia hanya bertanya "apakah ada baris yang cocok?" — pertanyaan `EXISTS`/`NOT EXISTS` selalu menghasilkan `TRUE`/`FALSE` tegas, tidak pernah `UNKNOWN`, berbeda dari `NOT IN` yang secara internal melakukan perbandingan `<>` per nilai yang rentan jatuh ke `UNKNOWN` begitu bertemu `NULL`.

## In Go

```go
package main

import (
	"context"
	"database/sql"
	"fmt"
)

// AmbilPegawaiTanpaAksesGedung memakai NOT EXISTS, bukan NOT IN — aman
// terhadap kemungkinan NULL di tabel pegawai_akses_gedung.nik yang datanya
// belum lengkap diisi.
func AmbilPegawaiTanpaAksesGedung(ctx context.Context, db *sql.DB) ([]string, error) {
	rows, err := db.QueryContext(ctx, `
		SELECT h.nik
		FROM pegawai_hr h
		WHERE NOT EXISTS (
			SELECT 1 FROM pegawai_akses_gedung a WHERE a.nik = h.nik
		)
		ORDER BY h.nik
	`)
	if err != nil {
		return nil, fmt.Errorf("query pegawai tanpa akses gedung: %w", err)
	}
	defer rows.Close()

	var nikList []string
	for rows.Next() {
		var nik string
		if err := rows.Scan(&nik); err != nil {
			return nil, fmt.Errorf("scan nik pegawai: %w", err)
		}
		nikList = append(nikList, nik)
	}
	if err := rows.Err(); err != nil {
		return nil, fmt.Errorf("iterasi baris pegawai: %w", err)
	}
	return nikList, nil
}
```

## In His Stack

`INTERSECT` dan `EXCEPT` relatif baru didukung MariaDB dibanding fitur SQL dasar lainnya — sistem legacy yang belum dimutakhirkan mungkin belum punya akses ke keduanya, dan pola `JOIN`/`NOT EXISTS` yang setara (seperti dicontohkan di atas) tetap jadi cara paling portabel untuk skenario audit lintas sistem semacam ini, terutama kalau kamu berkoordinasi di antara 13+ aplikasi dengan versi database yang mungkin tidak seragam.

> [!question] Perlu diverifikasi
> Klaim: versi MariaDB pertama yang mendukung `INTERSECT` dan `EXCEPT`.
> Kenapa ragu: fitur ini termasuk penambahan yang relatif baru di riwayat MariaDB dan nomor versi persisnya mudah salah diingat.
> Cara verifikasi: changelog resmi MariaDB untuk fitur "INTERSECT" dan "EXCEPT", atau jalankan `SELECT VERSION();` lalu cek dokumentasi versi tersebut.

## Trade-offs and When Not To Use It

`INTERSECT`/`EXCEPT` paling bernilai justru sebagai **niat yang eksplisit** — pembaca query langsung tahu ini soal membandingkan dua himpunan hasil, tanpa harus menguraikan `JOIN`/`NOT EXISTS` untuk menyimpulkan maksud yang sama. Tapi kalau database target belum mendukungnya (versi lama, atau dialek tertentu), atau kalau perbandingannya perlu diperluas menyertakan kolom tambahan di hasil (`INTERSECT`/`EXCEPT` hanya bisa membandingkan baris utuh, tidak bisa "bandingkan berdasarkan `nik` saja, tapi tampilkan kolom lain juga"), `JOIN`/`NOT EXISTS` tetap lebih fleksibel dan lebih portabel.

## Common Mistakes

> [!warning] Jebakan
> Memakai `NOT IN` dengan subquery yang berpotensi mengembalikan `NULL`, sehingga seluruh hasil diam-diam menjadi kosong tanpa error — ganti dengan `NOT EXISTS` atau `EXCEPT` yang aman terhadap `NULL`.

> [!warning] Jebakan
> Mengharapkan `INTERSECT`/`EXCEPT` membandingkan berdasarkan satu kolom identitas, padahal keduanya membandingkan **seluruh baris** — dua baris dengan identitas sama tapi kolom lain berbeda tidak akan cocok.

> [!warning] Jebakan
> Memakai `INTERSECT`/`EXCEPT` di database yang belum mendukungnya (versi lama), tanpa menyiapkan fallback `JOIN`/`NOT EXISTS` yang setara.

## Exercises

1. Tulis versi `INTERSECT` dari contoh "pegawai yang terdaftar di kedua sistem" memakai `JOIN` sebagai gantinya, dan jelaskan kenapa perlu `DISTINCT`.
2. Kenapa `NOT IN` dengan subquery yang mengandung `NULL` bisa mengembalikan hasil kosong secara keseluruhan, sementara `NOT EXISTS` tidak punya masalah ini?
3. Kapan `EXCEPT` tidak cukup, dan kamu perlu `JOIN`/`NOT EXISTS` yang membandingkan hanya sebagian kolom, bukan seluruh baris?
4. Desain terbuka: kamu perlu membangun laporan rekonsiliasi bulanan antara data transaksi di sistem internal dan data transaksi yang dikonfirmasi oleh partner bank (dikirim lewat file), untuk menemukan transaksi yang ada di satu sisi tapi tidak di sisi lain (dua arah). Rancang pendekatan query-nya, termasuk bagaimana menangani kemungkinan `NULL` atau data yang belum lengkap di salah satu sisi, dan jelaskan kenapa pendekatanmu aman terhadap jebakan `NOT IN`.

> [!success]- Kunci jawaban
> **1.** `SELECT DISTINCT h.nik FROM pegawai_hr h JOIN pegawai_akses_gedung a ON a.nik = h.nik`. `DISTINCT` dibutuhkan karena kalau salah satu sisi punya baris duplikat untuk `nik` yang sama (misalnya data akses gedung mencatat beberapa entri log per pegawai), `JOIN` akan menggandakan hasilnya, sementara `INTERSECT` standar secara definisi selalu mengembalikan himpunan hasil tanpa duplikat.
> **4.** Rekonsiliasi dua arah paling aman ditulis sebagai dua query `EXCEPT` (atau `NOT EXISTS` yang setara, kalau `EXCEPT` tidak tersedia): satu untuk "transaksi internal yang tidak ada di konfirmasi bank" (`SELECT id_transaksi FROM transaksi_internal EXCEPT SELECT id_transaksi FROM konfirmasi_bank`), satu lagi untuk arah sebaliknya. Sebelum operasi himpunan, kolom pembanding (`id_transaksi`) harus dipastikan `NOT NULL` di kedua sumber — kalau data konfirmasi bank punya kemungkinan baris dengan `id_transaksi` kosong/`NULL` karena parsing file yang gagal sebagian, baris itu perlu disaring lebih dulu (`WHERE id_transaksi IS NOT NULL`) sebelum masuk ke `EXCEPT`, supaya tidak mengacaukan interpretasi hasil rekonsiliasi (walau `EXCEPT` sendiri, tidak seperti `NOT IN`, tidak akan membuat seluruh hasil kosong hanya karena satu `NULL`).

## Self-Check

- Apa perbedaan hasil `INTERSECT` dan `EXCEPT`?
- Kenapa `NOT IN` berbahaya dipakai dengan subquery yang bisa mengembalikan `NULL`, sementara `EXCEPT`/`NOT EXISTS` tidak?
- Apakah `INTERSECT`/`EXCEPT` bisa membandingkan berdasarkan sebagian kolom saja? Kenapa atau kenapa tidak?
- Kapan lebih baik memakai `JOIN`/`NOT EXISTS` dibanding `INTERSECT`/`EXCEPT` langsung?

## Connected Notes

- [[NULL Semantics and Three-Valued Logic]] — sumber langsung dari jebakan `NOT IN` yang dibahas di note ini; `EXCEPT`/`NOT EXISTS` adalah cara amannya.
- [[UNION vs UNION ALL]] — operasi himpunan lain dalam keluarga yang sama; `UNION` menggabungkan, `INTERSECT`/`EXCEPT` membandingkan.
- [[Join Types and Their Mental Models]] — `INTERSECT` dan `EXCEPT` masing-masing punya padanan lewat `JOIN` dan `NOT EXISTS`/`LEFT JOIN ... WHERE ... IS NULL`.
- [[Upserts]] — sama-sama operasi yang membandingkan/mencocokkan baris antar sumber data, meski untuk tujuan berbeda (menulis, bukan membaca).
- [[Subqueries vs CTEs]] — `EXISTS`/`NOT EXISTS` adalah bentuk subquery khusus yang sering dipakai berdampingan dengan operasi himpunan.

## Further Reading

- Dokumentasi resmi PostgreSQL, bagian "Combining Queries (UNION, INTERSECT, EXCEPT)" di manual `SELECT`.
- Dokumentasi resmi MariaDB, bagian "INTERSECT" dan "EXCEPT".

## Catatan Saya

*Tulis di sini kebutuhan rekonsiliasi data lintas sistem di kerjaanmu yang bisa memakai INTERSECT/EXCEPT, dan bagaimana itu diselesaikan sebelumnya.*
