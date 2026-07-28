---
title: NULL Semantics and Three-Valued Logic
type: concept
level: junior
domain: databases
status: unread
difficulty: 3
est_minutes: 16
prerequisites: ["[[UNION vs UNION ALL]]"]
next: ["[[Set Operations in SQL]]"]
tags: [backend, databases]
created: 2026-07-28
---

## TL;DR

`NULL` di SQL berarti "nilai tidak diketahui", bukan "kosong" atau "nol". Konsekuensinya, setiap perbandingan yang melibatkan `NULL` (`= `, `<`, `>`, `<>`) tidak menghasilkan `TRUE` atau `FALSE`, melainkan hasil ketiga: `UNKNOWN`. `NULL = NULL` menghasilkan `UNKNOWN`, bukan `TRUE` — dua hal yang "tidak diketahui" tidak bisa dinyatakan sama, karena kamu tidak tahu apa keduanya. `WHERE` dan `HAVING` hanya meloloskan baris yang hasilnya persis `TRUE`; baris dengan hasil `UNKNOWN` **dibuang**, sama seperti `FALSE`. Ini melahirkan bug yang sangat umum: kode yang terlihat benar secara logika dua-nilai (benar/salah) diam-diam salah begitu ada `NULL` di datanya.

## The Problem

Sebuah query mencari "permohonan yang **belum** ditolak" — asumsinya, kolom `alasan_penolakan` kosong berarti belum ditolak:

```sql
SELECT * FROM permohonan WHERE alasan_penolakan <> 'ditolak_admin';
```

Query ini bermaksud menampilkan semua permohonan yang alasan penolakannya bukan `'ditolak_admin'` — termasuk yang belum pernah ditolak sama sekali (`alasan_penolakan` bernilai `NULL`, karena kolom itu memang hanya diisi kalau permohonan ditolak). Tapi hasilnya: permohonan dengan `alasan_penolakan = NULL` **tidak muncul** di hasil, padahal secara logis jelas-jelas bukan `'ditolak_admin'`. Penyebabnya: `NULL <> 'ditolak_admin'` tidak menghasilkan `TRUE` — ia menghasilkan `UNKNOWN`, karena membandingkan "nilai yang tidak diketahui" dengan string apa pun selalu menghasilkan "tidak diketahui juga". `WHERE` membuang baris `UNKNOWN` persis seperti `FALSE`, sehingga permohonan yang justru paling relevan — yang belum pernah ditolak — hilang diam-diam dari hasil. Tidak ada error, hanya laporan yang secara sistematis kehilangan sebagian data setiap kali dijalankan.

## Intuition

Bayangkan `NULL` seperti **kotak survei "Tidak Tahu / Tidak Menjawab"**, dikontraskan dengan kotak "Ya" dan "Tidak". Kalau kamu bertanya ke dua responden yang sama-sama mencentang "Tidak Tahu" — "apakah jawaban kalian berdua sama?" — jawaban yang jujur bukan "ya" atau "tidak", tapi "tidak bisa dipastikan, karena kita bahkan tidak tahu apa isi jawaban aslinya." Itulah kenapa `NULL = NULL` menghasilkan `UNKNOWN`, bukan `TRUE`: SQL menolak berpura-pura tahu bahwa dua hal yang sama-sama tidak diketahui pasti identik.

Analogi ini bocor pada satu titik penting: `IS NULL` dan `IS NOT NULL` **sengaja dirancang berbeda** dari operator perbandingan biasa — keduanya secara eksplisit menjawab `TRUE`/`FALSE` (bukan `UNKNOWN`) untuk pertanyaan "apakah kotak survei ini dicentang Tidak Tahu?". Jadi meskipun `NULL = NULL` adalah `UNKNOWN`, `NULL IS NULL` tegas `TRUE`. Aturan tiga-nilai berlaku untuk operator perbandingan nilai, bukan untuk pemeriksaan keberadaan `NULL` itu sendiri.

## How It Works

Tabel kebenaran tiga-nilai untuk `AND`, `OR`, `NOT` (`T` = TRUE, `F` = FALSE, `U` = UNKNOWN):

| A | B | A AND B | A OR B |
|---|---|---|---|
| T | U | U | T |
| F | U | F | U |
| U | U | U | U |

Poin paling penting untuk dihafal: `TRUE OR UNKNOWN` = `TRUE` (kalau salah satu sisi sudah pasti benar, hasilnya pasti benar terlepas sisi lain tidak diketahui), tapi `FALSE OR UNKNOWN` = `UNKNOWN` (belum tentu benar, karena sisi yang tidak diketahui *mungkin* benar). Dan hanya baris dengan hasil akhir `TRUE` yang lolos `WHERE`/`HAVING` — `UNKNOWN` maupun `FALSE` sama-sama dibuang.

Cara benar menangani kasus "The Problem":

```sql
-- Salah: NULL <> 'ditolak_admin' adalah UNKNOWN, baris NULL hilang
SELECT * FROM permohonan WHERE alasan_penolakan <> 'ditolak_admin';

-- Benar: menangani NULL secara eksplisit
SELECT * FROM permohonan
WHERE alasan_penolakan <> 'ditolak_admin' OR alasan_penolakan IS NULL;
```

Fungsi agregat juga punya aturan `NULL` sendiri yang sering disalahpahami: `COUNT(*)` menghitung **semua baris** termasuk yang kolomnya `NULL`, tapi `COUNT(kolom)` mengabaikan baris di mana `kolom` bernilai `NULL`. `SUM()`, `AVG()`, `MAX()`, `MIN()` semuanya **mengabaikan** `NULL` secara diam-diam — `AVG()` dari `[10, NULL, 20]` adalah `15` (rata-rata dari 2 nilai, bukan 3), bukan `10` seperti kalau `NULL` dihitung sebagai `0`.

## In Go

```go
package main

import (
	"context"
	"database/sql"
	"fmt"
)

// AlasanPenolakan bertipe sql.NullString, bukan string, karena kolomnya
// bisa NULL di database — memaksa kode Go menangani "tidak diketahui"
// secara eksplisit, bukan diam-diam memperlakukannya sebagai string kosong.
type Permohonan struct {
	ID              int
	AlasanPenolakan sql.NullString
}

func AmbilPermohonanBelumDitolakAdmin(ctx context.Context, db *sql.DB) ([]Permohonan, error) {
	rows, err := db.QueryContext(ctx, `
		SELECT id, alasan_penolakan
		FROM permohonan
		WHERE alasan_penolakan <> 'ditolak_admin' OR alasan_penolakan IS NULL
	`)
	if err != nil {
		return nil, fmt.Errorf("query permohonan belum ditolak admin: %w", err)
	}
	defer rows.Close()

	var hasil []Permohonan
	for rows.Next() {
		var p Permohonan
		if err := rows.Scan(&p.ID, &p.AlasanPenolakan); err != nil {
			return nil, fmt.Errorf("scan baris permohonan: %w", err)
		}
		hasil = append(hasil, p)
	}
	if err := rows.Err(); err != nil {
		return nil, fmt.Errorf("iterasi baris permohonan: %w", err)
	}
	return hasil, nil
}

func cetakAlasan(p Permohonan) string {
	if !p.AlasanPenolakan.Valid {
		return "(belum pernah ditolak)"
	}
	return p.AlasanPenolakan.String
}
```

`sql.NullString` memaksa developer secara eksplisit memeriksa `.Valid` sebelum membaca `.String` — kalau kolom di-scan langsung ke `string` biasa padahal isinya `NULL`, `database/sql` akan mengembalikan error saat `Scan()`, bukan diam-diam mengisi string kosong. Ini kontras yang penting: Go memaksa penanganan eksplisit di titik scan, sementara SQL sendiri (lewat aturan tiga-nilai) bisa dengan mudah membuang baris tanpa error sama sekali kalau `WHERE`-nya tidak menangani `NULL` secara sadar.

## In His Stack

Kolom yang "boleh kosong" di MariaDB (tanpa `NOT NULL`) sering dipakai sebagai penanda status implisit — seperti `alasan_penolakan` di contoh ini, atau `tanggal_dihapus` untuk soft-delete. Yii2 `ActiveRecord` yang dikonfigurasi ceroboh kadang memperlakukan kolom `NULL` sama seperti string kosong `''` di validasi form, padahal keduanya adalah nilai yang **berbeda secara semantik** di database — `WHERE kolom = ''` dan `WHERE kolom IS NULL` **tidak** menjaring baris yang sama. Kalau kontrak API dengan partner instansi mendefinisikan sebuah field sebagai "opsional", penting menyepakati eksplisit apakah "tidak diisi" direpresentasikan sebagai `NULL`, string kosong, atau field yang sama sekali tidak dikirim — ketiganya butuh penanganan `WHERE` yang berbeda di sisi query.

## Trade-offs and When Not To Use It

`NULL` bukan pilihan yang bisa "dihindari" begitu saja — ia bagian mendasar dari model relasional untuk merepresentasikan ketiadaan nilai. Tapi ada trade-off desain nyata: kolom yang **bisa** `NULL` menambah beban kognitif ke setiap query yang menyentuhnya (setiap perbandingan harus dipikirkan ulang untuk kasus `NULL`), sementara kolom `NOT NULL` dengan nilai default eksplisit (misalnya `0` untuk jumlah, atau string sentinel yang jelas maknanya) menghilangkan seluruh kelas bug ini — dengan konsekuensi kamu kehilangan kemampuan membedakan "sengaja bernilai 0" dari "belum pernah diisi". Pilih `NOT NULL` dengan default yang bermakna kapan pun ketiadaan informasi itu sendiri **bukan** fakta bisnis yang perlu direkam terpisah dari "nol"/"kosong".

## Common Mistakes

> [!warning] Jebakan
> Memakai `<>` atau `!=` untuk "mengecualikan" sebuah nilai, tanpa sadar baris dengan `NULL` di kolom itu ikut hilang dari hasil karena `NULL <> apa pun` adalah `UNKNOWN`, bukan `TRUE`.

> [!warning] Jebakan
> Menulis `WHERE kolom = NULL` (memakai `=`) alih-alih `WHERE kolom IS NULL`. `kolom = NULL` selalu menghasilkan `UNKNOWN`, tidak pernah `TRUE`, jadi query semacam ini tidak pernah mengembalikan baris apa pun, dan tidak ada error yang memberi tahu itu keliru.

> [!warning] Jebakan
> Berasumsi `AVG()` memperlakukan `NULL` sebagai `0` dalam perhitungan rata-rata — padahal `AVG()`, `SUM()`, `MAX()`, `MIN()` semuanya **mengabaikan** baris `NULL`, bukan menghitungnya sebagai nol, sehingga hasilnya bisa jauh berbeda dari yang diasumsikan.

## Exercises

1. Apa hasil dari `NULL = NULL`, dan kenapa itu bukan `TRUE`?
2. Tulis ulang `WHERE status <> 'selesai'` supaya baris dengan `status IS NULL` tetap ikut lolos, dan jelaskan kenapa versi aslinya membuang baris itu.
3. Jelaskan kenapa `AVG(kolom)` dari data `[10, NULL, 30]` menghasilkan `20`, bukan `13.33`.
4. Desain terbuka: sebuah tabel `permohonan` punya kolom `tanggal_diverifikasi` yang `NULL` berarti "belum diverifikasi". Tim ingin laporan "persentase permohonan yang **belum** diverifikasi, dari total permohonan yang dibuat lebih dari 7 hari lalu." Rancang query yang benar, dan jelaskan poin-poin di mana logika tiga-nilai `NULL` berperan dalam query itu (baik yang perlu diwaspadai maupun yang justru dimanfaatkan secara sengaja).

> [!success]- Kunci jawaban
> **1.** `NULL` berarti "nilai tidak diketahui" — dua hal yang sama-sama tidak diketahui tidak bisa dinyatakan setara, karena kamu tidak benar-benar tahu apakah keduanya sama. SQL karena itu menghasilkan `UNKNOWN`, bukan `TRUE` atau `FALSE`.
> **4.** `SELECT COUNT(CASE WHEN tanggal_diverifikasi IS NULL THEN 1 END) * 100.0 / COUNT(*) AS persentase_belum_verifikasi FROM permohonan WHERE tanggal_dibuat < NOW() - INTERVAL 7 DAY`. Di sini `IS NULL` dipakai secara sengaja (bukan `=`) untuk menjaring baris yang belum diverifikasi — ini bagian yang **memanfaatkan** semantik `NULL` dengan benar. Kewaspadaan yang perlu diingat: `COUNT(CASE WHEN ... THEN 1 END)` (tanpa `ELSE`) mengembalikan `NULL` untuk baris yang tidak cocok kondisi, dan `COUNT()` mengabaikan `NULL` — jadi hanya baris `tanggal_diverifikasi IS NULL` yang terhitung, persis yang diinginkan; kalau ditulis `SUM(CASE WHEN tanggal_diverifikasi IS NULL THEN 1 ELSE 0 END)` hasilnya identik tapi lewat jalur logika berbeda (`SUM` dari 0/1 eksplisit, bukan mengandalkan `NULL` diabaikan `COUNT`) — keduanya benar, tapi baik untuk sadar bahwa keduanya bergantung pada aturan `NULL` yang berbeda.

## Self-Check

- Kenapa `NULL = NULL` menghasilkan `UNKNOWN`, bukan `TRUE` atau `FALSE`?
- Apa beda `WHERE kolom = NULL` dan `WHERE kolom IS NULL`?
- Bagaimana `AVG()`, `SUM()`, `COUNT(*)`, dan `COUNT(kolom)` masing-masing memperlakukan baris `NULL`?
- Kenapa `WHERE kolom <> 'x'` bisa diam-diam membuang baris yang seharusnya relevan?

## Connected Notes

- [[The Logical Order of Query Execution]] — `WHERE`/`HAVING` hanya meloloskan baris `TRUE`; memahami ini plus aturan tiga-nilai menjelaskan kenapa baris `NULL` sering hilang tanpa error.
- [[Join Types and Their Mental Models]] — kolom dari sisi opsional `LEFT JOIN` selalu berpotensi `NULL`, membuat note ini prasyarat langsung untuk menulis `WHERE`/`ON` yang benar setelah `LEFT JOIN`.
- [[Aggregation and GROUP BY Semantics]] — fungsi agregat punya aturan `NULL` sendiri yang berbeda dari perbandingan biasa di `WHERE`.
- [[Data Types and Constraints]] — keputusan `NOT NULL` vs nullable adalah keputusan desain skema yang langsung menentukan seberapa sering aturan tiga-nilai ini harus dihadapi.
- [[Set Operations in SQL]] — `INTERSECT`/`EXCEPT` juga punya aturan tersendiri soal bagaimana `NULL` diperlakukan saat membandingkan baris.

## Further Reading

- Dokumentasi resmi PostgreSQL, bagian "Comparison Functions and Operators" — penjelasan referensi tentang `NULL` dan logika tiga-nilai.

## Catatan Saya

*Tulis di sini query di kerjaanmu yang pernah kehilangan baris secara diam-diam karena NULL — dan apakah sudah diperbaiki.*
