---
title: ACID
type: concept
level: junior
domain: databases
status: unread
difficulty: 2
est_minutes: 15
prerequisites: ["[[Database Transactions]]"]
next: ["[[Basic Isolation Levels]]"]
tags: [backend, databases]
created: 2026-07-28
---

## TL;DR

ACID adalah empat jaminan yang membuat transaction database bisa dipercaya: **Atomicity** (semua-atau-tidak-sama-sekali), **Consistency** (transaction hanya berpindah dari satu keadaan valid ke keadaan valid lain, tidak pernah melanggar aturan/constraint), **Isolation** (transaction yang berjalan bersamaan tidak saling mengganggu, sejauh level isolasi yang dipilih menjaminnya), dan **Durability** (begitu `COMMIT` berhasil, perubahan itu bertahan meski server crash detik berikutnya). Keempatnya sering disebut bersamaan seolah satu paket monolitik, padahal masing-masing menjawab kegagalan yang berbeda — dan memahami keempatnya secara terpisah penting karena beberapa sistem penyimpanan modern sengaja melonggarkan satu atau dua di antaranya demi trade-off lain (skalabilitas, latensi).

## The Problem

Sebuah tim baru memindahkan sebagian data dari MariaDB ke sebuah key-value store yang lebih cepat untuk kasus penggunaan tertentu, dengan asumsi "toh sama-sama database, ACID-nya pasti ada." Beberapa bulan kemudian muncul bug aneh: dua proses yang menulis ke key yang sama nyaris bersamaan menghasilkan data yang tercampur sebagian dari kedua penulisan (bukan salah satu menang penuh), dan sebuah proses lain membaca nilai yang jelas melanggar aturan bisnis yang seharusnya selalu dijaga (misalnya total across dua field yang seharusnya selalu sama dengan konstanta tertentu, tapi sesaat terbaca tidak sama).

Investigasi menemukan penyebabnya: key-value store yang dipilih memang **tidak** menjanjikan isolation dan consistency setara ACID relasional — ia dirancang untuk kecepatan dan skalabilitas horizontal, dengan trade-off eksplisit yang didokumentasikan (tapi tidak dibaca) di halaman pertama dokumentasinya. Ini bukan bug di produk itu — ia bekerja persis sesuai spesifikasinya. Bug-nya ada di asumsi tim bahwa "database" secara otomatis berarti "ACID penuh", sebuah generalisasi yang berbahaya justru karena benar untuk MariaDB/PostgreSQL tapi salah untuk banyak sistem penyimpanan non-relasional lainnya (dibahas lebih dalam di [[Beyond Relational - Document, Key-Value, Wide-Column, Graph, and Time-Series Stores]], level intermediate).

## Intuition

Pecah ACID jadi analogi masing-masing:

**Atomicity** — seperti [[Database Transactions|draft email dengan tombol kirim]] yang sudah dibahas: semua atau tidak sama sekali.

**Consistency** — seperti **petugas pemeriksa di pintu keluar pabrik**, memastikan setiap produk yang keluar memenuhi seluruh standar kualitas (constraint, foreign key, aturan bisnis) sebelum diizinkan lewat — sebuah transaction tidak boleh "commit" kalau hasil akhirnya melanggar aturan yang sudah ditetapkan skema.

**Isolation** — seperti **bilik ujian terpisah**: peserta ujian tidak bisa melihat jawaban peserta lain sampai lembar jawaban keduanya dikumpulkan, sejauh mana "tidak bisa melihat" ini dijamin bergantung aturan spesifik yang dipilih (isolation level).

**Durability** — seperti **menyimpan dokumen penting ke brankas tahan api**, bukan sekadar meja kerja — begitu tersimpan, ia bertahan bahkan kalau ruangan itu terbakar (server crash) detik berikutnya.

Analogi ini bocor terutama pada **Consistency**: istilah "consistency" di ACID **bukan** hal yang sama dengan "consistency" di teori distributed systems (linearizability, eventual consistency, dsb. — dibahas di [[../60 Distributed Systems/_Overview|Distributed Systems Overview]], level senior). Dua penggunaan istilah yang sama ini sering membingungkan justru karena namanya identik tapi maknanya berbeda konteks — consistency ACID soal "tidak melanggar aturan skema/bisnis", consistency distributed systems soal "seberapa cepat dan dalam urutan apa perubahan terlihat di seluruh node."

## How It Works

| Jaminan | Menjawab kegagalan | Mekanisme umum |
|---|---|---|
| Atomicity | Kegagalan di tengah proses (crash, error) | Write-ahead log — perubahan dicatat dulu sebelum diterapkan permanen, memungkinkan rollback. |
| Consistency | Transaction yang menghasilkan data melanggar aturan | Constraint (`CHECK`, `FOREIGN KEY`, `UNIQUE`) diperiksa sebelum `COMMIT` diizinkan berhasil. |
| Isolation | Transaction lain "mengintip" perubahan yang belum final | Locking dan/atau [[MVCC]] (Multi-Version Concurrency Control, level intermediate). |
| Durability | Kehilangan data setelah `COMMIT` karena crash | Data yang di-`COMMIT` ditulis ke disk (bukan hanya memori) sebelum `COMMIT` dianggap berhasil. |

Poin penting: keempat jaminan ini **bekerja bersama** tapi menjawab pertanyaan yang berbeda. Sebuah sistem bisa saja atomik (tidak ada state setengah jadi) tapi tidak durable (data hilang kalau server mati sebelum ditulis ke disk) — dua properti yang independen secara konsep, meski database relasional matang seperti MariaDB/PostgreSQL menjamin keduanya sekaligus untuk transaction yang sudah `COMMIT`.

## In Go

```go
package main

import (
	"context"
	"database/sql"
	"errors"
	"fmt"
)

// BuatPermohonan menunjukkan bagaimana Consistency (constraint database)
// dan Atomicity (transaction) bekerja bersama: kalau constraint dilanggar,
// database menolak COMMIT, dan seluruh transaction otomatis batal.
func BuatPermohonan(ctx context.Context, db *sql.DB, instansiID int, judul string) error {
	tx, err := db.BeginTx(ctx, nil)
	if err != nil {
		return fmt.Errorf("mulai transaction buat permohonan: %w", err)
	}
	defer tx.Rollback()

	_, err = tx.ExecContext(ctx, `
		INSERT INTO permohonan (instansi_id, judul, status) VALUES (?, ?, 'diajukan')
	`, instansiID, judul)
	if err != nil {
		// Kalau instansi_id melanggar FOREIGN KEY (instansi tidak ada),
		// database menolak INSERT ini — Consistency ditegakkan di sini,
		// dan Atomicity memastikan tidak ada perubahan lain yang "bocor".
		var mysqlErr interface{ Number() uint16 }
		if errors.As(err, &mysqlErr) {
			return fmt.Errorf("permohonan melanggar constraint database: %w", err)
		}
		return fmt.Errorf("insert permohonan: %w", err)
	}

	if err := tx.Commit(); err != nil {
		return fmt.Errorf("commit buat permohonan: %w", err)
	}
	return nil
}
```

## In His Stack

MariaDB dengan storage engine InnoDB (default untuk tabel baru di hampir semua instalasi modern) mendukung ACID penuh — tapi ini **bukan** properti universal MySQL/MariaDB: storage engine lama seperti MyISAM (masih ditemukan di sistem legacy yang belum dimigrasi) **tidak** mendukung transaction sama sekali, artinya tidak ada Atomicity maupun Isolation dalam pengertian ACID. Kalau kamu mewarisi tabel di sistem lama yang ternyata masih memakai MyISAM, ini bukan detail sepele — kode yang mengasumsikan `ROLLBACK` akan membatalkan perubahan pada tabel itu akan diam-diam salah, karena MyISAM tidak punya mekanisme untuk itu.

> [!question] Perlu diverifikasi
> Klaim: InnoDB adalah storage engine default untuk tabel baru di instalasi MariaDB modern.
> Kenapa ragu: default storage engine bisa dikonfigurasi berbeda di level server (`my.cnf`), dan sistem legacy yang pernah dikonfigurasi manual bertahun-tahun lalu mungkin tidak mengikuti default versi terbaru.
> Cara verifikasi: jalankan `SHOW TABLE STATUS` atau `SELECT ENGINE FROM information_schema.TABLES WHERE table_name = '...'` pada tabel yang relevan di instance yang sebenarnya dipakai.

## Trade-offs and When Not To Use It

ACID penuh bukan "selalu benar secara default" — ia adalah pilihan desain dengan biaya nyata: menegakkan Isolation dan Consistency ketat membatasi seberapa banyak transaction bisa berjalan konkuren tanpa saling memblokir, dan Durability (menulis ke disk sebelum `COMMIT` dianggap selesai) menambah latensi dibanding menulis ke memori saja. Sistem yang butuh throughput sangat tinggi dengan toleransi kehilangan data kecil pada skenario kegagalan langka (misalnya cache terdistribusi, metrik agregat yang tidak kritis) kadang secara sadar memilih sistem penyimpanan yang melonggarkan sebagian jaminan ACID demi kecepatan — trade-off yang valid **selama** dipilih secara sadar, bukan karena tidak tahu apa yang dikorbankan.

## Common Mistakes

> [!warning] Jebakan
> Mengasumsikan "database" secara otomatis berarti ACID penuh — banyak sistem penyimpanan modern (termasuk beberapa key-value dan document store) secara sengaja melonggarkan sebagian jaminan ACID demi performa dan skalabilitas, dan ini didokumentasikan secara eksplisit di masing-masing produk.

> [!warning] Jebakan
> Mencampuradukkan "Consistency" di ACID dengan "consistency" di teori distributed systems — keduanya istilah yang sama tapi menjawab pertanyaan yang sepenuhnya berbeda konteks.

> [!warning] Jebakan
> Mengasumsikan seluruh tabel di sebuah database relasional otomatis mendapat jaminan ACID penuh, tanpa memeriksa storage engine yang sebenarnya dipakai — beberapa storage engine lama (seperti MyISAM di MySQL/MariaDB) tidak mendukung transaction sama sekali.

## Exercises

1. Jelaskan dengan kata-katamu sendiri apa yang dijamin masing-masing dari empat huruf ACID, tanpa melihat catatan.
2. Kenapa "Consistency" di ACID berbeda maknanya dari "consistency" di teori distributed systems, meski istilahnya sama persis?
3. Berikan satu contoh konkret pelanggaran Consistency yang seharusnya dicegah constraint database (bisa dari domain kerja pemerintahan).
4. Desain terbuka: tim mempertimbangkan memindahkan sebagian data statistik (yang dibaca sangat sering, ditulis sangat sering, tapi sedikit "basi" bisa diterima) dari MariaDB ke sistem penyimpanan yang secara eksplisit hanya menjanjikan *eventual consistency* (bukan ACID penuh). Diskusikan pertimbangan apa yang harus dievaluasi sebelum keputusan ini diambil, dan jenis data seperti apa yang **tidak boleh** ikut dipindahkan dengan alasan yang sama.

> [!success]- Kunci jawaban
> **3.** Constraint `CHECK (saldo >= 0)` yang dibahas di [[Data Types and Constraints]] adalah contoh langsung — kalau sebuah transaction mencoba `COMMIT` perubahan yang membuat `saldo` negatif, Consistency ACID mengharuskan database menolak `COMMIT` itu sepenuhnya (dan Atomicity memastikan seluruh transaction dibatalkan, bukan hanya bagian yang melanggar).
> **4.** Pertimbangan utama: data statistik yang "sedikit basi masih bisa diterima" (misalnya jumlah kunjungan hari ini, yang toleran terhadap keterlambatan beberapa detik) adalah kandidat yang wajar untuk sistem eventual consistency, karena dampak bisnis dari sesekali membaca nilai yang sedikit tertinggal sangat kecil. Data yang **tidak boleh** ikut dipindahkan dengan alasan yang sama adalah apa pun yang butuh Atomicity dan Isolation ketat karena konsekuensi kesalahannya besar dan tidak bisa "diperbaiki nanti" — saldo keuangan, status persetujuan dokumen resmi, atau apa pun yang jadi dasar keputusan hukum/administratif. Aturan praktisnya: pindahkan data yang errornya murah untuk ditoleransi atau diperbaiki belakangan, pertahankan ACID penuh untuk data yang errornya mahal atau tidak bisa diperbaiki setelah fakta.

## Self-Check

- Sebutkan empat jaminan ACID dan kegagalan spesifik yang masing-masing cegah.
- Kenapa "Consistency" ACID berbeda dari "consistency" di distributed systems?
- Storage engine apa di MySQL/MariaDB yang tidak mendukung transaction sama sekali?
- Kenapa ACID penuh bukan pilihan yang selalu tepat untuk semua jenis data?

## Connected Notes

- [[Database Transactions]] — ACID adalah kumpulan jaminan yang berlaku pada unit transaction yang dibahas di note itu; prasyarat langsung untuk note ini.
- [[Basic Isolation Levels]] — pembahasan lebih dalam tentang huruf "I" (Isolation) di ACID, termasuk trade-off antar level.
- [[Data Types and Constraints]] — constraint adalah mekanisme konkret yang menegakkan huruf "C" (Consistency) di ACID.
- [[Beyond Relational - Document, Key-Value, Wide-Column, Graph, and Time-Series Stores]] — pembahasan sistem penyimpanan yang secara sengaja melonggarkan sebagian jaminan ACID, disinggung di "The Problem".
- [[../60 Distributed Systems/_Overview|Distributed Systems Overview]] — tempat istilah "consistency" muncul kembali dengan makna yang berbeda dari ACID, penting dipahami agar tidak tertukar.

## Further Reading

- Jim Gray, *"The Transaction Concept: Virtues and Limitations"* (1981) — paper awal yang merumuskan konsep transaction yang mendasari ACID.

## Catatan Saya

*Tulis di sini sistem penyimpanan di kerjaanmu yang kamu belum yakin benar-benar menjamin ACID penuh atau tidak — dan bagaimana memverifikasinya.*
