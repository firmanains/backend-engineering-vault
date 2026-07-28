---
title: Aggregation and GROUP BY Semantics
type: concept
level: junior
domain: databases
status: unread
difficulty: 2
est_minutes: 15
prerequisites: ["[[Join Types and Their Mental Models]]"]
next: ["[[Subqueries vs CTEs]]"]
tags: [backend, databases]
created: 2026-07-28
---

## TL;DR

`GROUP BY` mengelompokkan baris yang punya nilai sama di kolom yang disebutkan, lalu fungsi agregat (`COUNT()`, `SUM()`, `AVG()`, `MAX()`, `MIN()`) menghitung satu nilai ringkasan **per kelompok**, bukan per baris. Aturan yang sering dilanggar tapi jarang dipahami alasannya: setiap kolom di `SELECT` yang **bukan** hasil fungsi agregat harus muncul di `GROUP BY` — kalau tidak, database tidak tahu nilai kolom itu mana yang harus dipilih untuk mewakili satu kelompok yang berisi banyak baris berbeda. MySQL dalam mode tertentu membolehkan pelanggaran ini secara diam-diam (memilih nilai baris sembarang), yang jauh lebih berbahaya daripada error tegas.

## The Problem

Tim ingin laporan "instansi, beserta tanggal permohonan pertamanya, dan total permohonan yang pernah diajukan." Seseorang menulis:

```sql
SELECT instansi_id, tanggal_ajuan, COUNT(*) AS total
FROM permohonan
GROUP BY instansi_id;
```

Di PostgreSQL, ini langsung ditolak dengan error `column "permohonan.tanggal_ajuan" must appear in the GROUP BY clause or be used in an aggregate function`. Tapi kalau dijalankan di MariaDB dengan `ONLY_FULL_GROUP_BY` tidak aktif (kondisi yang di beberapa instalasi lama masih jadi default), query ini **jalan tanpa error** — dan mengembalikan `tanggal_ajuan` dari salah satu baris di kelompok itu, dipilih **secara tidak terdefinisi** oleh mesin database. Laporan tampak benar sampai suatu hari nilai yang keluar untuk instansi tertentu berubah tanpa ada perubahan data yang jelas — karena mesin database kebetulan memilih baris fisik yang berbeda kali ini. Ini bukan bug yang mudah dilacak: tidak ada error, tidak ada log, hanya angka yang diam-diam tidak konsisten antar-run.

## Intuition

Bayangkan `GROUP BY` seperti **menyortir surat ke dalam pigeonhole (kotak sortir) berdasarkan kode pos**. Setelah semua surat masuk kotaknya masing-masing, kamu hanya bisa menanyakan pertanyaan **tentang seluruh isi satu kotak** — "berapa banyak surat di kotak ini?" (`COUNT`), "berapa total berat surat di kotak ini?" (`SUM`) — kamu tidak bisa lagi menanyakan "apa isi surat yang pertama masuk kotak ini?", karena begitu surat masuk kotak, identitas baris individualnya hilang; yang tersisa hanyalah kotak sebagai satu kesatuan.

Analogi ini bocor pada satu titik: pigeonhole sungguhan tidak "kebingungan" kalau kamu memaksa bertanya tentang satu surat spesifik — ia hanya tidak punya jawaban. SQL, sayangnya, **kadang tetap memberi jawaban** untuk pertanyaan semacam itu (kolom non-agregat yang tidak ada di `GROUP BY`) alih-alih menolak tegas — itulah yang membuat mode longgar MySQL berbahaya: ia menjawab pertanyaan yang seharusnya tidak valid, dengan jawaban yang dipilih sembarangan.

## How It Works

Aturan intinya: **setiap kolom di `SELECT` harus salah satu dari dua kategori** — (a) muncul persis di `GROUP BY`, atau (b) dibungkus fungsi agregat. Tidak ada kategori ketiga.

```sql
-- BENAR: tanggal_ajuan_pertama dibungkus MIN(), instansi_id ada di GROUP BY
SELECT instansi_id, MIN(tanggal_ajuan) AS tanggal_ajuan_pertama, COUNT(*) AS total
FROM permohonan
GROUP BY instansi_id;
```

`WHERE` menyaring baris **sebelum** dikelompokkan; `HAVING` menyaring **kelompok** setelah agregasi selesai (lihat [[The Logical Order of Query Execution]] untuk urutan lengkapnya):

```sql
SELECT instansi_id, COUNT(*) AS total
FROM permohonan
WHERE tahun = 2026          -- saring BARIS: hanya permohonan tahun ini
GROUP BY instansi_id
HAVING COUNT(*) > 50;       -- saring KELOMPOK: hanya instansi dengan >50 permohonan
```

Jebakan yang lebih halus muncul saat `GROUP BY` digabung dengan `JOIN` satu-ke-banyak: kalau satu `instansi` punya banyak baris `pegawai`, dan kamu `JOIN` ke `permohonan` lalu `GROUP BY instansi_id` tanpa hati-hati, `COUNT()` bisa menghitung **kombinasi silang** pegawai × permohonan, bukan jumlah permohonan yang sebenarnya — karena baris hasil join sudah digandakan sebelum sempat dikelompokkan (lihat [[Join Types and Their Mental Models]]). Solusinya biasanya `COUNT(DISTINCT permohonan.id)`, atau memisahkan agregasi jadi subquery/CTE terpisah sebelum join.

## In Go

```go
package main

import (
	"context"
	"database/sql"
	"fmt"
)

type RingkasanInstansi struct {
	InstansiID        int
	TotalPermohonan   int
	TanggalPertama    string
}

// AmbilRingkasanPerInstansi menegakkan aturan GROUP BY secara eksplisit:
// setiap kolom non-agregat di SELECT (instansi_id) ada di GROUP BY,
// dan kolom lain semuanya dibungkus fungsi agregat.
func AmbilRingkasanPerInstansi(ctx context.Context, db *sql.DB, tahun int) ([]RingkasanInstansi, error) {
	rows, err := db.QueryContext(ctx, `
		SELECT instansi_id, COUNT(*) AS total, MIN(tanggal_ajuan) AS pertama
		FROM permohonan
		WHERE tahun = ?
		GROUP BY instansi_id
		HAVING COUNT(*) > 0
		ORDER BY total DESC
	`, tahun)
	if err != nil {
		return nil, fmt.Errorf("query ringkasan per instansi: %w", err)
	}
	defer rows.Close()

	var hasil []RingkasanInstansi
	for rows.Next() {
		var r RingkasanInstansi
		if err := rows.Scan(&r.InstansiID, &r.TotalPermohonan, &r.TanggalPertama); err != nil {
			return nil, fmt.Errorf("scan baris ringkasan instansi: %w", err)
		}
		hasil = append(hasil, r)
	}
	if err := rows.Err(); err != nil {
		return nil, fmt.Errorf("iterasi baris ringkasan instansi: %w", err)
	}
	return hasil, nil
}
```

## In His Stack

MariaDB mewarisi mode `sql_mode` dari MySQL, dan `ONLY_FULL_GROUP_BY` adalah salah satu flag di dalamnya — di versi/instalasi yang lebih baru biasanya sudah aktif secara default, tapi instalasi lama yang belum pernah diaudit `sql_mode`-nya (umum di sistem legacy Yii1) bisa saja masih longgar. Ini penting dicek secara eksplisit, bukan diasumsikan: `SELECT @@sql_mode;` di MariaDB akan menunjukkan apakah proteksi ini aktif. Kalau tim mengandalkan query lama yang secara diam-diam bergantung pada perilaku longgar ini, mengaktifkan `ONLY_FULL_GROUP_BY` bisa mematahkan query itu — tapi mematahkannya secara **tegas dan terlihat** jauh lebih baik daripada membiarkannya terus menghasilkan angka yang tidak terdefinisi.

> [!question] Perlu diverifikasi
> Klaim: `ONLY_FULL_GROUP_BY` aktif secara default di instalasi MariaDB yang lebih baru.
> Kenapa ragu: default `sql_mode` berbeda antara MySQL dan MariaDB (dua project yang bercabang terpisah) dan bisa berubah antar versi rilis masing-masing; jangan diasumsikan sama antara keduanya.
> Cara verifikasi: jalankan `SELECT @@sql_mode;` langsung di instance MariaDB yang relevan, atau periksa release notes MariaDB untuk versi yang dipakai.

## Trade-offs and When Not To Use It

Tidak ada opsi "tidak memakai" `GROUP BY` kalau kebutuhannya memang agregasi per kelompok — pertanyaan trade-off yang sebenarnya adalah **di lapisan mana** agregasi dilakukan. Mengagregasi di SQL (di database) hampir selalu lebih efisien daripada menarik semua baris mentah ke aplikasi Go lalu mengelompokkannya secara manual — database bisa memakai index dan mengagregasi tanpa pernah membentuk seluruh dataset di memori. Mengagregasi di aplikasi baru masuk akal kalau logika pengelompokannya terlalu kompleks untuk diekspresikan bersih dalam SQL, atau kalau hasilnya perlu digabung dengan data dari sumber lain yang tidak ada di database yang sama.

## Common Mistakes

> [!warning] Jebakan
> Menaruh kolom non-agregat di `SELECT` tanpa menaruhnya juga di `GROUP BY`. Di database yang longgar (MySQL/MariaDB tanpa `ONLY_FULL_GROUP_BY`) ini tidak error, hanya menghasilkan nilai yang dipilih secara tidak terdefinisi.

> [!warning] Jebakan
> Melakukan `COUNT(*)` setelah `JOIN` satu-ke-banyak tanpa `DISTINCT`, sehingga menghitung baris hasil join yang sudah digandakan, bukan jumlah entitas aslinya.

> [!warning] Jebakan
> Menaruh syarat yang seharusnya di `HAVING` (bergantung pada hasil agregat) ke `WHERE`. `WHERE` dievaluasi sebelum `GROUP BY`, jadi ia tidak pernah bisa melihat nilai `COUNT()`/`SUM()` — lihat [[The Logical Order of Query Execution]].

## Exercises

1. Tulis query untuk menghitung rata-rata jumlah permohonan per instansi per bulan, dan jelaskan kolom mana yang wajib ada di `GROUP BY`.
2. Kenapa `SELECT nama, COUNT(*) FROM pegawai GROUP BY departemen_id` bisa error atau menghasilkan `nama` yang tidak terduga, tergantung `sql_mode`?
3. Jelaskan skenario konkret di mana `JOIN` satu-ke-banyak sebelum `GROUP BY` menghasilkan `COUNT()` yang salah, dan tunjukkan dua cara memperbaikinya.
4. Desain terbuka: sebuah dashboard menampilkan "rata-rata waktu pemrosesan permohonan per instansi", dihitung dari `JOIN` tabel `permohonan` ke tabel `log_status` (yang punya banyak baris per permohonan, satu untuk tiap perubahan status). Setelah `GROUP BY instansi_id`, angka rata-ratanya jauh lebih rendah dari yang diharapkan tim operasional. Diagnosis kemungkinan penyebabnya, dan rancang query yang benar.

> [!success]- Kunci jawaban
> **2.** Query ini melanggar aturan `GROUP BY` karena `nama` bukan kolom agregat dan tidak ada di `GROUP BY departemen_id` — kalau satu departemen berisi banyak pegawai dengan `nama` berbeda-beda, database tidak punya cara valid menentukan `nama` siapa yang mewakili baris hasil itu. Di `ONLY_FULL_GROUP_BY` aktif, ini error tegas; di mode longgar, `nama` yang keluar dipilih dari salah satu baris secara tidak terdefinisi dan bisa berubah antar-eksekusi.
> **4.** Kemungkinan besar `JOIN` ke `log_status` menggandakan setiap baris `permohonan` sebanyak jumlah perubahan statusnya sebelum `GROUP BY instansi_id` sempat berjalan — akibatnya rata-rata dihitung dari dataset yang bobotnya sudah bias ke permohonan dengan banyak perubahan status. Perbaikannya: hitung dulu waktu pemrosesan **per permohonan** (misalnya `MAX(waktu) - MIN(waktu)` dikelompokkan per `permohonan_id`) di dalam CTE atau subquery terpisah, baru `JOIN` hasilnya ke `instansi` dan `GROUP BY instansi_id` di lapisan luar — memastikan setiap permohonan hanya berkontribusi satu nilai ke rata-rata akhir, bukan satu nilai per baris log statusnya.

## Self-Check

- Aturan apa yang menentukan kolom mana boleh ada di `SELECT` bersama `GROUP BY`?
- Kenapa mode `sql_mode` yang longgar di MySQL/MariaDB lebih berbahaya daripada error tegas?
- Kenapa `JOIN` satu-ke-banyak sebelum `GROUP BY` bisa merusak hasil `COUNT()`/`SUM()`?
- Apa perbedaan tugas `WHERE` dan `HAVING` dalam sebuah query yang memakai `GROUP BY`?

## Connected Notes

- [[The Logical Order of Query Execution]] — menjelaskan kenapa `WHERE` beroperasi sebelum pengelompokan dan `HAVING` sesudahnya.
- [[Join Types and Their Mental Models]] — sumber paling umum dari `COUNT()`/`SUM()` yang salah adalah baris yang sudah digandakan oleh join satu-ke-banyak sebelum diagregasi.
- [[Window Functions]] — alternatif untuk kasus di mana kamu butuh nilai agregat **tanpa** mengurangi jumlah baris hasil, sesuatu yang `GROUP BY` tidak bisa lakukan.
- [[NULL Semantics and Three-Valued Logic]] — fungsi agregat seperti `COUNT(kolom)` dan `AVG()` punya perlakuan khusus terhadap `NULL` yang mudah salah diasumsikan.
- [[Subqueries vs CTEs]] — memisahkan tahap agregasi dari tahap join adalah pola umum yang butuh subquery atau CTE untuk tetap terbaca.

## Further Reading

- Dokumentasi resmi MySQL, bagian `sql_mode` dan `ONLY_FULL_GROUP_BY` — menjelaskan perilaku persis di tiap mode.
- Dokumentasi resmi PostgreSQL, bagian "Aggregate Functions" dan "GROUP BY" di manual `SELECT`.

## Catatan Saya

*Tulis di sini kalau kamu pernah menemukan laporan dengan angka yang "goyang" antar-run — kemungkinan besar ini penyebabnya.*
