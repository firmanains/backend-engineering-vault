---
title: Basic Isolation Levels
type: concept
level: junior
domain: databases
status: unread
difficulty: 3
est_minutes: 17
prerequisites: ["[[ACID]]"]
next: ["[[database-sql and sqlx]]"]
tags: [backend, databases]
created: 2026-07-28
---

## TL;DR

Isolation level menentukan **seberapa banyak** sebuah transaction bisa "melihat" perubahan dari transaction lain yang berjalan bersamaan tapi belum `COMMIT`. Semakin ketat isolasinya, semakin sedikit anomali yang mungkin terjadi — tapi semakin besar juga biaya locking dan semakin rendah concurrency yang bisa dicapai database. Poin yang sering tidak disadari: **MySQL/MariaDB dan PostgreSQL memakai isolation level default yang berbeda** (`REPEATABLE READ` vs `READ COMMITTED`), dan kode yang "bekerja dengan benar" di satu mesin database bisa punya bug konkurensi yang tidak pernah muncul di testing tapi muncul di produksi begitu dipindah ke mesin lain dengan default isolasi yang berbeda.

## The Problem

Sebuah laporan "total saldo seluruh rekening instansi" dijalankan dengan membaca setiap baris `rekening` satu per satu dalam satu transaction panjang (bukan `SUM()` tunggal — anggap laporan ini juga memproses tiap baris untuk keperluan lain). Di tengah proses membaca, sebuah transaction lain memindahkan saldo dari rekening yang **belum** dibaca laporan ke rekening yang **sudah** dibaca laporan, lalu `COMMIT`.

Kalau isolation level terlalu longgar, laporan bisa membaca saldo rekening tujuan **sebelum** perubahan itu (karena sudah lewat saat perubahan terjadi) dan saldo rekening asal **setelah** perubahan itu (karena belum sempat dibaca saat perubahan terjadi) — akibatnya laporan menghitung uang yang sama **dua kali**, atau kehilangannya sepenuhnya, tergantung urutan baca. Ini bukan bug di logika laporan — logikanya benar secara sekuensial. Bug ini murni soal isolation level yang tidak cukup ketat untuk menjamin laporan melihat "snapshot" data yang konsisten sepanjang seluruh prosesnya, meski data itu terus berubah dari transaction lain di latar belakang.

## Intuition

Bayangkan isolation level seperti **aturan seberapa banyak kamu boleh mengintip pekerjaan orang lain yang belum selesai**, di ruang kerja bersama. Level paling longgar seperti "boleh membaca coretan siapa pun di meja mereka, bahkan yang masih dicoret-coret dan belum final" (`READ UNCOMMITTED`) — cepat, tapi kamu bisa membaca sesuatu yang kemudian dihapus/diubah sebelum orang itu selesai. Level yang lebih ketat seperti "hanya boleh membaca dokumen yang sudah ditandatangani final" (`READ COMMITTED`) — lebih aman, tapi dokumen yang sama bisa saja **berubah** kalau kamu membacanya dua kali dalam sesi kerjamu, karena orang lain sempat menandatangani revisi baru di antaranya. Level paling ketat (`SERIALIZABLE`) lebih mirip **diberi satu salinan cetak dari seluruh ruang kerja, tepat pada saat kamu mulai bekerja** — orang lain terus mengubah papan aslinya, tapi salinanmu tidak ikut berubah, jadi apa pun yang kamu baca berkali-kali selalu sama. Saat kamu menyerahkan hasil kerjamu, ada petugas yang memeriksa apakah perubahan orang lain sementara itu membuat pekerjaanmu jadi tidak konsisten; kalau ya, kamu diminta mengulang dengan salinan yang baru.

Analogi ini bocor pada **biaya salinannya**: database tidak benar-benar menyalin seluruh tabel untuk setiap transaction. Ia hanya menyimpan versi lama dari baris yang memang berubah, dan menyusun "salinan"-mu dari situ saat dibutuhkan — mekanismenya dibahas di [[MVCC]].

## How It Works

Empat isolation level standar SQL, dari paling longgar ke paling ketat:

| Level | Mengizinkan | Biaya relatif | Default di mesin |
|---|---|---|---|
| `READ UNCOMMITTED` | Bisa membaca data yang belum di-`COMMIT` transaction lain ("dirty read") | Paling murah, paling jarang dipakai serius | — |
| `READ COMMITTED` | Hanya membaca data yang sudah di-`COMMIT`, tapi nilai bisa berubah antar baca dalam transaction yang sama | Lebih murah dari `REPEATABLE READ` | PostgreSQL |
| `REPEATABLE READ` | Nilai yang sudah dibaca tetap sama sepanjang transaction, tapi baris **baru** yang cocok kondisi bisa muncul ("phantom read") | Lebih mahal dari `READ COMMITTED` | MySQL/MariaDB (InnoDB) |
| `SERIALIZABLE` | Transaction berjalan seolah-olah dieksekusi satu per satu secara berurutan, tanpa tumpang tindih sama sekali | Paling ketat, paling mahal | — |

Pembahasan detail anomali (dirty read, non-repeatable read, phantom read, write skew) dan trade-off performa masing-masing level ada di [[Isolation Levels and Their Anomalies]], level intermediate — note ini fokus pada gambaran besar yang wajib dipahami lebih dulu.

```sql
-- PostgreSQL
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
-- ... operasi transaction ...
COMMIT;

-- MySQL / MariaDB — isolation level diset SEBELUM transaction dimulai,
-- tidak bisa digabung ke dalam satu statement START TRANSACTION.
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
START TRANSACTION;
-- ... operasi transaction ...
COMMIT;
```

Perbedaan sintaks sekecil ini adalah contoh konkret kenapa menyetel isolation level lewat `sql.TxOptions` di Go lebih aman daripada menuliskannya sebagai SQL mentah — driver yang menerjemahkannya ke dialek yang benar.

Poin paling praktis untuk diingat sekarang: **default berbeda antar mesin database**. Kode yang ditulis dan diuji terhadap MariaDB (default `REPEATABLE READ`) mengandalkan jaminan yang tidak otomatis berlaku kalau dijalankan terhadap PostgreSQL tanpa konfigurasi eksplisit (default `READ COMMITTED`, lebih longgar) — dan sebaliknya.

> [!question] Perlu diverifikasi
> Klaim: MySQL/MariaDB (InnoDB) memakai `REPEATABLE READ` sebagai default, PostgreSQL memakai `READ COMMITTED` sebagai default.
> Kenapa ragu: default isolation level bisa dikonfigurasi berbeda di level server, dan meski ini adalah default pabrik yang cukup dikenal luas, sebaiknya tidak diasumsikan tanpa verifikasi untuk instance produksi tertentu.
> Cara verifikasi: `SELECT @@transaction_isolation;` di MariaDB, atau `SHOW default_transaction_isolation;` di PostgreSQL, dijalankan langsung di instance yang relevan.

## In Go

```go
package main

import (
	"context"
	"database/sql"
	"fmt"
)

// AmbilLaporanSaldoKonsisten secara eksplisit meminta REPEATABLE READ,
// tidak bergantung pada default mesin database yang sedang dipakai —
// penting karena kode ini harus berjalan benar di MariaDB maupun PostgreSQL.
func AmbilLaporanSaldoKonsisten(ctx context.Context, db *sql.DB) (int64, error) {
	tx, err := db.BeginTx(ctx, &sql.TxOptions{Isolation: sql.LevelRepeatableRead})
	if err != nil {
		return 0, fmt.Errorf("mulai transaction laporan saldo: %w", err)
	}
	defer tx.Rollback()

	var totalSaldo int64
	if err := tx.QueryRowContext(ctx, "SELECT SUM(saldo) FROM rekening").Scan(&totalSaldo); err != nil {
		return 0, fmt.Errorf("hitung total saldo: %w", err)
	}

	if err := tx.Commit(); err != nil {
		return 0, fmt.Errorf("commit laporan saldo: %w", err)
	}
	return totalSaldo, nil
}
```

Menetapkan `Isolation` secara eksplisit lewat `sql.TxOptions` — bukan mengandalkan default koneksi — adalah praktik yang lebih aman untuk kode yang portabilitasnya penting, atau untuk operasi yang secara sengaja butuh jaminan lebih ketat dari default yang sedang berlaku.

## In His Stack

Yii2 `Connection::beginTransaction($isolationLevel)` menerima parameter isolation level eksplisit, tapi kebanyakan kode Yii2 yang pernah kamu temui kemungkinan memanggilnya tanpa parameter itu sama sekali — artinya diam-diam mengandalkan default MariaDB (`REPEATABLE READ`). Ini penting diperiksa secara eksplisit kalau ada rencana migrasi sebagian beban kerja dari MariaDB ke PostgreSQL (skenario yang cukup umum saat mengevaluasi opsi database baru) — kode yang bergantung diam-diam pada jaminan `REPEATABLE READ` bisa punya bug konkurensi baru begitu berjalan di atas `READ COMMITTED` default PostgreSQL, tanpa satu baris kode pun terlihat berubah.

## Trade-offs and When Not To Use It

Isolation level yang lebih ketat (`SERIALIZABLE`) memberi jaminan paling kuat, tapi juga paling membatasi concurrency — transaction yang bertabrakan lebih mudah dipaksa gagal (butuh retry di sisi aplikasi) atau saling menunggu lebih lama. Untuk operasi baca yang tidak kritis terhadap anomali ringan (dashboard yang boleh sedikit tidak konsisten sesaat), memilih level default yang lebih longgar memberi throughput lebih baik tanpa risiko bisnis yang berarti. Aturan praktis: pilih level paling longgar yang **masih aman** untuk kebutuhan spesifik operasi itu — bukan otomatis memilih paling ketat "untuk jaga-jaga", karena itu membayar biaya performa untuk jaminan yang mungkin tidak pernah benar-benar dibutuhkan.

## Common Mistakes

> [!warning] Jebakan
> Mengasumsikan isolation level default sama di semua mesin database — kode yang benar di MariaDB bisa punya bug konkurensi baru begitu dijalankan di PostgreSQL tanpa isolation level eksplisit, karena defaultnya berbeda.

> [!warning] Jebakan
> Memilih `SERIALIZABLE` untuk semua transaction "demi keamanan", tanpa mempertimbangkan biaya performa dan peningkatan kegagalan transaction yang butuh retry — isolasi ketat bukan pilihan gratis.

> [!warning] Jebakan
> Menjalankan laporan atau proses baca panjang tanpa isolation level yang cukup ketat untuk menjamin snapshot konsisten, menghasilkan angka yang salah karena data berubah di tengah proses pembacaan — persis kasus di "The Problem".

## Exercises

1. Kenapa default isolation level MySQL/MariaDB dan PostgreSQL yang berbeda bisa jadi sumber bug tersembunyi saat migrasi antar database?
2. Jelaskan dengan kata-katamu sendiri kenapa level paling ketat (`SERIALIZABLE`) tidak selalu jadi pilihan terbaik meski memberi jaminan paling kuat.
3. Kenapa laporan yang membaca banyak baris dalam satu transaction panjang butuh isolation level yang lebih dipikirkan dibanding query tunggal sederhana?
4. Desain terbuka: sebuah proses batch malam hari menghitung ulang seluruh saldo agregat per instansi dari tabel transaksi mentah, sambil sistem tetap menerima transaksi baru sepanjang malam (bukan sistem yang benar-benar sepi). Rancang pendekatan isolation level untuk proses ini, dan diskusikan trade-off antara akurasi snapshot dan dampaknya terhadap transaksi baru yang datang selama proses berjalan.

> [!success]- Kunci jawaban
> **1.** Kode yang ditulis dan diuji dengan asumsi implisit (tidak eksplisit di kode) terhadap jaminan `REPEATABLE READ` (default MariaDB) bisa saja lolos testing dan berjalan benar bertahun-tahun di MariaDB — tapi begitu dipindah ke PostgreSQL yang defaultnya `READ COMMITTED` (lebih longgar), jaminan yang sebelumnya "gratis" dari default database sekarang tidak ada lagi, dan anomali konkurensi yang sebelumnya tidak mungkin terjadi jadi mungkin terjadi, tanpa satu baris kode aplikasi berubah.
> **4.** Proses batch ini sebaiknya secara eksplisit meminta `REPEATABLE READ` (atau lebih ketat kalau memang dibutuhkan) untuk seluruh durasi pembacaannya, memastikan ia melihat satu snapshot konsisten dari tabel transaksi meski transaksi baru terus masuk selama proses berjalan — transaksi baru itu tidak akan terlihat oleh proses batch (karena snapshot sudah "dibekukan" sejak transaction dimulai), dan itu memang perilaku yang benar: proses batch menghitung "saldo per titik waktu X", bukan "saldo yang terus bergerak". Trade-off yang harus diterima: transaksi baru yang masuk selama proses batch berjalan tidak akan tercermin di hasil perhitungan malam itu — itu bukan bug, itu definisi dari snapshot; kalau bisnis butuh transaksi baru tetap tercermin, prosesnya harus dijalankan ulang atau didesain sebagai proses inkremental, bukan sekadar menaikkan isolation level lebih tinggi lagi.

## Self-Check

- Apa perbedaan mendasar `READ COMMITTED` dan `REPEATABLE READ`?
- Isolation level apa yang jadi default di MySQL/MariaDB, dan apa yang jadi default di PostgreSQL?
- Kenapa `SERIALIZABLE` bukan pilihan default yang aman digunakan di mana-mana tanpa pertimbangan?
- Kenapa isolation level penting dipikirkan secara eksplisit untuk laporan yang membaca banyak baris dalam satu transaction panjang?

## Connected Notes

- [[ACID]] — isolation level adalah mekanisme konkret yang menegakkan huruf "I" (Isolation) yang dibahas secara konseptual di note itu.
- [[Database Transactions]] — isolation level hanya relevan dalam konteks transaction; prasyarat langsung untuk memahami note ini.
- [[Isolation Levels and Their Anomalies]] — kelanjutan langsung di level intermediate: pembahasan detail dirty read, non-repeatable read, phantom read, dan write skew.
- [[MVCC]] — mekanisme internal yang memungkinkan database memberi snapshot konsisten tanpa benar-benar memblokir seluruh akses baca, dibahas di level intermediate.
- [[Deadlocks]] — risiko yang meningkat seiring isolation level yang lebih ketat dan lock yang ditahan lebih lama.

## Further Reading

- Dokumentasi resmi MySQL/MariaDB, bagian "InnoDB Transaction Isolation Levels".
- Dokumentasi resmi PostgreSQL, bagian "Transaction Isolation".

## Catatan Saya

*Tulis di sini query laporan panjang di kerjaanmu yang menurutmu rentan membaca data tidak konsisten — dan isolation level apa yang sebenarnya sedang dipakai (cek, jangan tebak).*
