---
title: Keeping Search in Sync with the Source of Truth
type: concept
level: intermediate
domain: databases
status: unread
difficulty: 3
est_minutes: 16
prerequisites: ["[[Relevance Scoring]]"]
next: []
tags: [backend, databases, integration]
created: 2026-07-29
---

## TL;DR

Elasticsearch (atau sistem pencarian terpisah manapun) hampir tidak pernah menjadi sumber kebenaran (source of truth) data — database relasional tetap memegang peran itu, dan indeks pencarian adalah **salinan** yang dioptimalkan khusus untuk kebutuhan pencarian. Begitu ada dua salinan data yang harus tetap sinkron, muncul kelas masalah baru yang tidak ada saat data hanya hidup di satu tempat: **search index yang diam-diam melenceng dari database sumbernya** — dokumen yang sudah dihapus/diubah di database tapi masih muncul (atau muncul dengan data lama) di hasil pencarian, karena mekanisme sinkronisasinya gagal, tertunda, atau tidak pernah dibangun dengan benar sejak awal.

## The Problem

Sebuah petugas menghapus permohonan yang salah input, memverifikasi penghapusan itu berhasil di database (data benar-benar hilang saat di-query langsung), tapi permohonan itu **masih muncul** di hasil pencarian dashboard selama beberapa hari. Investigasi menemukan bahwa proses sinkronisasi ke Elasticsearch (job terjadwal yang berjalan setiap malam) gagal diam-diam beberapa malam berturut-turut karena perubahan skema kecil di database yang tidak diantisipasi kode sinkronisasi — tidak ada yang memantau kesehatan proses ini secara eksplisit, sehingga kegagalan itu berlangsung tanpa terdeteksi sampai pengguna melaporkannya sebagai bug "data hantu" yang membingungkan.

Masalah kedua yang lebih halus dan lebih sulit didiagnosis: sebuah dokumen yang statusnya baru saja diubah dari "menunggu" ke "disetujui" di database, masih muncul dengan status "menunggu" di hasil pencarian selama beberapa menit — bukan karena proses sinkronisasi gagal total, tapi karena ia berjalan dengan **jeda** (mirip replication lag yang dibahas di [[Read Replicas and Replication Lag]], tapi kali ini antara database dan search index, bukan antara primary dan replica database yang sama). Untuk kebanyakan kebutuhan pencarian, jeda beberapa menit ini bisa diterima — tapi kalau tidak dikomunikasikan atau dipahami sebagai trade-off yang disengaja, ia terasa seperti bug yang "kadang muncul, kadang tidak" tergantung seberapa cepat pengguna memeriksa ulang setelah mengubah data.

## Intuition

Bayangkan search index seperti **katalog perpustakaan yang harus diperbarui manual setiap kali ada buku baru masuk atau buku lama dikeluarkan dari rak**. Kalau petugas yang memperbarui katalog itu absen atau lalai satu hari, katalog itu **diam-diam** tidak lagi mencerminkan isi rak yang sebenarnya — buku yang sudah dikeluarkan masih tercantum di katalog, buku baru belum tercantum sama sekali. Masalahnya bukan katalog itu "rusak" secara mekanis — ia berfungsi normal, hanya berisi informasi yang sudah usang, dan tidak ada yang tahu sampai seseorang mencari buku yang menurut katalog "ada" tapi ternyata sudah tidak ada di rak.

Analogi ini bocor pada satu hal: petugas perpustakaan yang lalai biasanya terlihat jelas tidak bekerja (absen, meja kosong). Proses sinkronisasi search index yang gagal **tidak selalu terlihat jelas** — job yang gagal bisa saja tetap "berjalan" tanpa error yang eksplisit terlihat (misalnya berhasil terhubung ke Elasticsearch tapi gagal memproses sebagian dokumen karena format data yang berubah), membuat kegagalan sinkronisasi menjadi kelas bug yang secara khusus butuh observability eksplisit (lihat [[../70 Infrastructure and Delivery/The Three Pillars of Observability|The Three Pillars of Observability]]) untuk terdeteksi sebelum pengguna yang menemukannya lebih dulu.

## How It Works

```mermaid
flowchart LR
    DB[("Database\n(source of truth)")] -->|"Strategi 1: Dual write"| ES1[("Elasticsearch")]
    DB -->|"Strategi 2: Batch sync\n(cron job berkala)"| ES2[("Elasticsearch")]
    DB -->|"Strategi 3: CDC\n(binlog/WAL streaming)"| ES3[("Elasticsearch")]
```

Diagram ini menunjukkan tiga strategi umum menjaga sinkronisasi, masing-masing dengan trade-off berbeda:

- **Dual write** — aplikasi menulis ke database **dan** ke Elasticsearch dalam operasi yang sama (biasanya berurutan, bukan atomik). Paling sederhana diimplementasikan, tapi paling rapuh: kalau tulisan ke database berhasil tapi tulisan ke Elasticsearch gagal (jaringan terputus, Elasticsearch sedang down), kedua sistem melenceng seketika, dan tidak ada mekanisme bawaan untuk mendeteksi atau memperbaiki drift ini secara otomatis.
- **Batch sync** — job terjadwal (cron) yang secara berkala membaca perubahan dari database dan mendorongnya ke Elasticsearch. Lebih tahan terhadap kegagalan sesaat (percobaan berikutnya bisa memperbaiki apa yang terlewat), tapi memperkenalkan jeda yang jelas antara perubahan data dan perubahan hasil pencarian, sebesar interval job itu berjalan.
- **Change Data Capture (CDC)** — membaca perubahan langsung dari **binlog/WAL** database (dibahas lebih dalam di [[../60 Distributed Systems/Change Data Capture|Change Data Capture]], level senior) dan mengalirkannya ke Elasticsearch secara near-real-time. Paling andal dan minim jeda, tapi butuh infrastruktur tambahan (Debezium atau setara) dan kompleksitas operasional yang lebih tinggi dibanding dua pendekatan lainnya.

## Under The Hood

**Dual write** punya masalah struktural yang tidak bisa ditambal sepenuhnya tanpa mekanisme tambahan: tidak ada transaction yang mencakup **dua sistem berbeda** (database relasional dan Elasticsearch) sekaligus — kalaupun aplikasi menunggu konfirmasi dari keduanya sebelum menganggap operasi "sukses", tetap ada jendela waktu di antara kedua tulisan itu di mana kegagalan pada satu sisi meninggalkan sistem dalam keadaan tidak konsisten. Pola **transactional outbox** (dibahas lebih dalam di [[../30 APIs and Web/The Transactional Outbox Pattern|The Transactional Outbox Pattern]]) adalah salah satu cara memperbaiki ini — menulis "niat untuk sinkronisasi" sebagai bagian dari transaction database yang sama, lalu proses terpisah yang membaca niat itu dan benar-benar mendorongnya ke Elasticsearch, dengan jaminan **at-least-once** (bisa dicoba ulang kalau gagal) alih-alih dual write yang bisa gagal diam-diam tanpa jejak.

**Full reindex** — proses membangun ulang seluruh index dari nol berdasarkan seluruh data di database — adalah jaring pengaman yang harus selalu tersedia terlepas dari strategi sinkronisasi harian yang dipakai, karena drift yang terakumulasi dari waktu ke waktu (bug kecil di CDC, kegagalan batch sync yang tidak terdeteksi lama) pada akhirnya butuh cara untuk "menyamakan ulang" kedua sistem dari awal. Full reindex pada dataset besar bisa memakan waktu signifikan dan membebani database sumber (harus membaca seluruh data) — perlu direncanakan sebagai operasi terjadwal dengan sengaja, bukan dijalankan reaktif secara panik saat drift sudah parah.

## In Go

```go
package outbox

import (
	"context"
	"database/sql"
	"errors"
	"fmt"
	"log/slog"
)

// UbahStatusDenganOutbox mencatat entri outbox dalam transaction yang sama
// dengan perubahan data utama. Kalau UPDATE berhasil di-commit, niat untuk
// sinkronisasi ke Elasticsearch juga pasti tercatat, tidak pernah hanya
// salah satunya (berbeda dari dual write).
func UbahStatusDenganOutbox(ctx context.Context, db *sql.DB, permohonanID int64, statusBaru string) error {
	tx, err := db.BeginTx(ctx, nil)
	if err != nil {
		return fmt.Errorf("mulai transaction: %w", err)
	}
	defer tx.Rollback()

	if _, err := tx.ExecContext(ctx, `UPDATE permohonan SET status = ? WHERE id = ?`, statusBaru, permohonanID); err != nil {
		return fmt.Errorf("update status permohonan: %w", err)
	}

	// Entri outbox sengaja hanya berisi ID entitas, bukan isi dokumen.
	// Worker akan membaca keadaan terbaru dari database saat memprosesnya.
	if _, err := tx.ExecContext(ctx, `
		INSERT INTO outbox (entitas, entitas_id, diproses)
		VALUES ('permohonan', ?, false)
	`, permohonanID); err != nil {
		return fmt.Errorf("catat entri outbox: %w", err)
	}

	if err := tx.Commit(); err != nil {
		return fmt.Errorf("commit perubahan status: %w", err)
	}
	return nil
}

// Dokumen adalah bentuk permohonan yang dikirim ke index pencarian.
type Dokumen struct {
	ID     int64
	Nomor  string
	Status string
}

// Indexer adalah abstraksi kecil atas klien Elasticsearch.
type Indexer interface {
	Simpan(ctx context.Context, d Dokumen) error
	Hapus(ctx context.Context, id int64) error
}

// ProsesOutbox membaca entri yang belum diproses, lalu untuk setiap entri
// membaca keadaan TERBARU entitas dari database dan menyalinnya ke index.
// Karena yang dikirim selalu keadaan terbaru, urutan pemrosesan dan
// pengulangan tidak berbahaya: entri lama yang dicoba ulang belakangan
// tidak bisa menimpa index dengan data usang. Entitas yang sudah dihapus
// dari database ikut dihapus dari index.
func ProsesOutbox(ctx context.Context, db *sql.DB, idx Indexer, logger *slog.Logger) error {
	rows, err := db.QueryContext(ctx, `
		SELECT id, entitas_id FROM outbox
		WHERE diproses = false AND entitas = 'permohonan'
		ORDER BY id ASC LIMIT 100
	`)
	if err != nil {
		return fmt.Errorf("ambil entri outbox: %w", err)
	}
	type entri struct{ id, entitasID int64 }
	var antrean []entri
	for rows.Next() {
		var e entri
		if err := rows.Scan(&e.id, &e.entitasID); err != nil {
			rows.Close()
			return fmt.Errorf("scan entri outbox: %w", err)
		}
		antrean = append(antrean, e)
	}
	if err := rows.Err(); err != nil {
		rows.Close()
		return fmt.Errorf("iterasi entri outbox: %w", err)
	}
	if err := rows.Close(); err != nil {
		return fmt.Errorf("tutup hasil outbox: %w", err)
	}

	for _, e := range antrean {
		if err := sinkronkanSatu(ctx, db, idx, e.entitasID); err != nil {
			// Entri tetap belum diproses dan akan dicoba lagi di putaran
			// berikutnya. Karena worker selalu membaca keadaan terbaru,
			// melanjutkan ke entri lain tidak merusak urutan.
			logger.Warn("gagal sinkronkan permohonan ke index",
				"outbox_id", e.id, "permohonan_id", e.entitasID, "error", err)
			continue
		}
		if _, err := db.ExecContext(ctx, `UPDATE outbox SET diproses = true WHERE id = ?`, e.id); err != nil {
			return fmt.Errorf("tandai outbox %d selesai: %w", e.id, err)
		}
	}
	return nil
}

func sinkronkanSatu(ctx context.Context, db *sql.DB, idx Indexer, id int64) error {
	var d Dokumen
	err := db.QueryRowContext(ctx,
		`SELECT id, nomor_permohonan, status FROM permohonan WHERE id = ?`, id,
	).Scan(&d.ID, &d.Nomor, &d.Status)
	if errors.Is(err, sql.ErrNoRows) {
		// Baris sudah tidak ada di sumber kebenaran: hapus juga dari index.
		if err := idx.Hapus(ctx, id); err != nil {
			return fmt.Errorf("hapus dokumen %d dari index: %w", id, err)
		}
		return nil
	}
	if err != nil {
		return fmt.Errorf("baca permohonan %d: %w", id, err)
	}
	if err := idx.Simpan(ctx, d); err != nil {
		return fmt.Errorf("simpan dokumen %d ke index: %w", id, err)
	}
	return nil
}
```

Dua keputusan desain di kode ini sering terlewat. Pertama, entri outbox hanya membawa ID, dan worker membaca ulang keadaan terbaru. Kalau entri membawa payload (misalnya status saat itu), entri lama yang gagal lalu dicoba ulang setelah entri yang lebih baru berhasil akan menimpa index dengan status usang. Kedua, penghapusan ditangani dengan cara yang sama: `sql.ErrNoRows` berarti baris sudah dihapus, jadi dokumen di index ikut dihapus. Kalau lebih dari satu worker berjalan bersamaan, ambil entri dengan `FOR UPDATE SKIP LOCKED` di dalam transaction (lihat [[Locking and Row Locks]]).

## In His Stack

Untuk sistem legal-services yang menggabungkan MariaDB dan Elasticsearch, "data hantu" di hasil pencarian (dokumen yang seharusnya sudah tidak ada tapi masih muncul) adalah kelas bug yang sangat mengganggu kepercayaan pengguna terhadap sistem, khususnya untuk data yang sensitif secara hukum — petugas yang mempercayai hasil pencarian sebagai representasi akurat dari database bisa membuat keputusan berdasarkan informasi usang. Menjadikan kesehatan proses sinkronisasi (lag antara database dan index, tingkat kegagalan job sinkronisasi) sebagai metrik yang eksplisit dipantau — bukan diasumsikan "pasti jalan karena sudah di-deploy" — adalah kebiasaan operasional yang sepadan dengan risiko yang ditanggung.

## Trade-offs and When Not To Use It

Membangun mekanisme sinkronisasi yang benar-benar andal (outbox pattern, CDC) menambah kompleksitas infrastruktur yang signifikan — untuk sistem pencarian yang kebutuhan kesegarannya longgar (misalnya pencarian arsip dokumen lama yang jarang berubah), batch sync sederhana yang berjalan tiap beberapa jam sudah lebih dari cukup, dan membangun CDC penuh untuk kasus ini adalah over-engineering. Sebaliknya, untuk sistem yang butuh hasil pencarian mencerminkan perubahan hampir seketika (misalnya pencarian status permohonan yang aktif diproses), batch sync sekali sehari jelas tidak memadai, dan investasi ke CDC atau setidaknya transactional outbox jadi sepadan. Keputusan strategi sinkronisasi harus mengikuti kebutuhan kesegaran data yang **sesungguhnya** dibutuhkan pengguna, bukan default ke solusi paling canggih atau paling sederhana tanpa mempertimbangkan kebutuhan nyata.

## Common Mistakes

> [!warning] Jebakan
> Memakai dual write murni (tulis ke database, lalu tulis ke Elasticsearch) tanpa mekanisme retry atau pendeteksian kegagalan — begitu salah satu tulisan gagal, kedua sistem melenceng tanpa jejak yang jelas, dan drift ini bisa terakumulasi diam-diam untuk waktu lama.

> [!warning] Jebakan
> Tidak memantau kesehatan proses sinkronisasi secara eksplisit (lag, tingkat kegagalan) — kegagalan sinkronisasi sering tidak menghasilkan error yang mencolok, dan baru terdeteksi setelah pengguna melaporkan data yang tidak sesuai.

> [!warning] Jebakan
> Tidak punya mekanisme full reindex yang siap dipakai sebagai jaring pengaman — begitu drift yang terakumulasi terlalu parah untuk diperbaiki inkremental, tim tidak punya cara cepat menyamakan ulang kedua sistem dari awal.

## Exercises

1. Jelaskan kenapa dual write murni (tanpa outbox pattern) rentan terhadap drift yang tidak terdeteksi antara database dan search index.
2. Apa perbedaan mendasar batch sync dan CDC dalam hal kesegaran data dan kompleksitas infrastruktur?
3. Kenapa full reindex tetap dibutuhkan sebagai jaring pengaman, terlepas dari strategi sinkronisasi harian yang dipakai?
4. Desain terbuka: sistem pencarianmu memakai batch sync setiap 30 menit, dan kamu baru menyadari proses ini sudah gagal diam-diam selama tiga hari terakhir tanpa terdeteksi (tidak ada alert yang terpasang). Rancang dua perbaikan: (a) bagaimana mendeteksi kegagalan semacam ini lebih cepat di masa depan, dan (b) bagaimana memperbaiki drift yang sudah terlanjur terjadi selama tiga hari itu, tanpa harus melakukan full reindex penuh yang memakan waktu lama kalau memang tidak perlu.

> [!success]- Kunci jawaban
> **1.** Dual write menulis ke dua sistem berbeda secara berurutan tanpa transaksi yang mencakup keduanya sekaligus — kalau tulisan pertama (database) berhasil tapi tulisan kedua (Elasticsearch) gagal karena alasan apa pun (jaringan, Elasticsearch down, timeout), tidak ada mekanisme otomatis yang mencatat bahwa sinkronisasi ini gagal dan perlu dicoba ulang. Aplikasi mungkin bahkan menganggap seluruh operasi "sukses" kalau error dari sisi Elasticsearch tidak ditangani dengan benar (misalnya di-log tapi tidak menggagalkan response ke pengguna) — drift terjadi tanpa jejak yang mudah ditemukan sampai seseorang secara spesifik membandingkan kedua sistem.
> **4.** (a) Tambahkan metrik eksplisit untuk proses batch sync: jumlah dokumen berhasil disinkronkan per run, waktu proses terakhir kali berhasil (`last_successful_sync`), dan alert otomatis kalau `last_successful_sync` lebih tua dari beberapa kali interval normal (misalnya lebih dari 2 jam untuk job yang seharusnya jalan tiap 30 menit) — mengubah kegagalan diam-diam menjadi sinyal yang eksplisit terlihat operator, bukan bergantung pada laporan pengguna. (b) Alih-alih full reindex penuh, jalankan sinkronisasi **bertarget** untuk baris yang berubah selama tiga hari itu (`WHERE updated_at >= NOW() - INTERVAL 3 DAY`), dengan asumsi kolom `updated_at` bisa diandalkan (diperbarui oleh setiap jalur tulis, termasuk script manual). Ada satu lubang besar di pendekatan ini: **baris yang dihapus tidak punya `updated_at`**, karena barisnya sudah tidak ada. Penghapusan selama tiga hari itu harus ditangani terpisah. Pilihannya: pakai soft delete (`dihapus_pada`) sehingga penghapusan ikut terlihat oleh query `updated_at`; atau bandingkan daftar ID. Ambil semua ID dokumen di index untuk rentang yang terdampak, cocokkan dengan ID yang masih ada di database, lalu hapus dari index ID yang tidak ditemukan. Tanpa langkah ini, "data hantu" seperti di The Problem tetap bertahan setelah perbaikan.

## Self-Check

- Kenapa dual write murni rentan terhadap drift yang tidak terdeteksi?
- Apa perbedaan trade-off batch sync dan CDC?
- Kenapa full reindex tetap perlu ada sebagai jaring pengaman?
- Metrik apa yang perlu dipantau untuk mendeteksi kegagalan sinkronisasi lebih cepat?

## Connected Notes

- [[Relevance Scoring]] — hasil pencarian yang relevan tidak berguna kalau datanya sendiri sudah usang atau salah karena drift sinkronisasi, penutup alami dari rangkaian topik search di domain ini.
- [[Read Replicas and Replication Lag]] — lag antara database dan search index adalah masalah yang secara konseptual identik dengan replication lag, hanya antara dua sistem yang sepenuhnya berbeda alih-alih dua instance database yang sama.
- [[../60 Distributed Systems/Change Data Capture|Change Data Capture]] — strategi sinkronisasi paling andal yang dibahas mendalam di level senior, disinggung sebagai salah satu dari tiga strategi di note ini.
- [[../30 APIs and Web/The Transactional Outbox Pattern|The Transactional Outbox Pattern]] — pola yang dipakai untuk sinkronisasi andal di note ini, dibahas lengkap sebagai pola integrasi di domain APIs.
- [[../70 Infrastructure and Delivery/The Three Pillars of Observability|The Three Pillars of Observability]] — pemantauan kesehatan proses sinkronisasi (lag, tingkat kegagalan) adalah aplikasi langsung dari prinsip observability di domain itu.

## Further Reading

- Dokumentasi resmi Debezium (implementasi CDC open-source populer) sebagai referensi konkret arsitektur CDC untuk sinkronisasi database-ke-search-index.

## Catatan Saya

*Tulis di sini strategi sinkronisasi yang dipakai fitur pencarian di kerjaanmu saat ini (dual write, batch sync, atau CDC) — dan apakah pernah ada insiden "data hantu" seperti di note ini.*
