---
title: Change Data Capture
type: concept
level: senior
domain: distributed
status: unread
difficulty: 3
est_minutes: 16
prerequisites: ["[[Event Schema Evolution]]", "[[../40 Databases/Read Replicas and Replication Lag|Read Replicas and Replication Lag]]"]
next: ["[[Defensible Eventual Consistency]]"]
tags: [backend, distributed, databases]
created: 2026-08-02
---

## TL;DR

Change Data Capture (CDC) menangkap setiap perubahan (insert, update, delete) yang terjadi di database dan mengubahnya jadi aliran event, **tanpa** aplikasi yang menulis ke database itu perlu tahu atau melakukan apa pun secara eksplisit. Bedanya dengan menambahkan kode "kirim event setiap kali ada perubahan" secara manual di aplikasi: CDC membaca perubahan langsung dari **transaction log** database (mekanisme internal yang sudah ada untuk keperluan replikasi dan recovery, lihat [[../40 Databases/Read Replicas and Replication Lag|Read Replicas and Replication Lag]]) — sumber yang **dijamin** mencatat setiap perubahan yang benar-benar commit, tidak bergantung pada aplikasi mengingat untuk memanggil kode pengiriman event di setiap tempat yang mengubah data.

## The Problem

Sebuah tim ingin menyinkronkan data dari database MariaDB ke Elasticsearch untuk kebutuhan pencarian (lihat [[../40 Databases/Keeping Search in Sync with the Source of Truth|Keeping Search in Sync with the Source of Truth]]). Pendekatan awal: menambahkan kode di setiap endpoint yang mengubah data untuk juga mengirim event ke Elasticsearch — "setelah `UPDATE` berhasil, kirim event sinkronisasi". Pendekatan ini bekerja untuk endpoint yang sudah diketahui dan diubah kodenya, tapi beberapa bulan kemudian ditemukan celah: ada proses batch internal yang mengubah data langsung lewat query SQL manual (untuk perbaikan data massal), dan proses itu **lupa** menyertakan kode pengiriman event — data di database berubah, tapi Elasticsearch tidak pernah tahu, dan index pencarian diam-diam mulai menyimpang dari sumber kebenaran tanpa ada yang menyadarinya sampai pengguna melaporkan hasil pencarian yang aneh.

Masalah ini adalah kelemahan struktural dari pendekatan "aplikasi mengirim event manual setiap kali mengubah data" — pendekatan ini bergantung pada **setiap** titik yang mengubah data (termasuk yang belum ada, seperti proses batch baru, migrasi manual, atau perbaikan darurat lewat query langsung) mengingat untuk menyertakan kode pengiriman event. Satu titik yang terlewat berarti drift yang diam-diam terjadi tanpa terdeteksi, persis seperti masalah [[../70 Infrastructure and Delivery/State Files and Drift|State Files and Drift]] tapi untuk data aplikasi, bukan infrastruktur.

## Intuition

Cara paling mudah memahaminya: mengirim event manual di setiap tempat yang mengubah data seperti **meminta setiap karyawan melapor manual ke resepsionis** setiap kali mereka masuk atau keluar gedung — bekerja selama semua karyawan ingat melakukannya, tapi begitu ada satu karyawan (atau tamu, atau kontraktor) yang lupa atau tidak tahu aturan ini, catatan resepsionis jadi tidak akurat tanpa ada yang sadar. CDC seperti **memasang sensor otomatis di pintu masuk gedung** — sensor ini mencatat setiap orang yang lewat, tidak peduli siapa mereka atau apakah mereka "ingat" untuk melapor, karena pencatatan terjadi di titik fisik yang **pasti** dilewati semua orang, bukan bergantung pada disiplin masing-masing individu mengingat aturan.

Analogi ini nyaris sepenuhnya menangkap esensi CDC. Yang tidak sepenuhnya tertangkap: sensor pintu gedung mencatat kejadian secara real-time seketika. Transaction log database yang dibaca CDC punya sedikit jeda alami (mirip replication lag di [[../40 Databases/Read Replicas and Replication Lag|Read Replicas and Replication Lag]]) — CDC bukan mekanisme instan sempurna, hanya jauh lebih andal dibanding mengandalkan aplikasi mengirim event manual di setiap titik.

## How It Works

```mermaid
flowchart LR
    App["Aplikasi\n(INSERT/UPDATE/DELETE biasa,\nTIDAK tahu ada CDC)"] --> DB[("Database")]
    DB -->|"transaction log\n(binlog di MariaDB/MySQL,\nWAL di PostgreSQL)"] --> CDC["CDC Connector\n(misalnya Debezium)"]
    CDC -->|"event perubahan"| Stream["Message Broker\n(Kafka)"]
    Stream --> Consumer1["Elasticsearch\n(sinkronisasi index)"]
    Stream --> Consumer2["Cache\n(invalidasi)"]
```
Aplikasi menulis ke database dengan cara yang **persis sama seperti biasa** — tidak ada kode tambahan, tidak ada kesadaran bahwa CDC sedang berjalan. CDC connector membaca transaction log database (mekanisme internal yang sudah dipakai database itu sendiri untuk replikasi), mengubah setiap perubahan jadi event terstruktur, dan mempublikasikannya ke message broker — dari sana, **siapa pun** yang butuh tahu perubahan data (Elasticsearch, cache, sistem analitik) bisa berlangganan tanpa aplikasi asli perlu tahu mereka ada, dan tanpa risiko ada titik perubahan data yang "terlewat" seperti di "The Problem", karena CDC membaca dari sumber yang mencatat **semua** perubahan yang benar-benar commit, apa pun jalur atau proses yang menyebabkannya.

## Under The Hood

CDC bekerja dengan membaca **transaction log** — di MariaDB/MySQL disebut binary log (binlog), di PostgreSQL disebut write-ahead log (WAL). Log ini sudah ada dan berjalan untuk keperluan lain (replikasi ke replica, recovery setelah crash) — CDC connector pada dasarnya "menumpang" mekanisme yang sudah ada ini, membaca log yang sama yang dipakai replica database untuk tahu perubahan apa yang terjadi, tanpa menambah beban query langsung ke database utama (berbeda dari pendekatan polling yang berulang kali query tabel untuk mendeteksi perubahan, yang jauh lebih membebani database).

Poin penting yang membedakan CDC dari sekadar "trigger database": trigger dijalankan **dalam** transaksi yang sama dengan perubahan asli, menambah latency dan risiko ke operasi tulis utama (trigger yang gagal bisa membatalkan transaksi asli). CDC membaca log **setelah** transaksi commit, sepenuhnya asinkron dan tidak menambah beban atau risiko apa pun ke jalur tulis utama aplikasi — perubahan sudah aman tersimpan di database sebelum CDC bahkan mulai memprosesnya.

## In Go

```go
package cdc

import "context"

// ChangeEvent merepresentasikan SATU perubahan yang ditangkap dari
// transaction log — TIDAK PERNAH ditulis manual oleh aplikasi,
// dihasilkan otomatis dari mekanisme CDC.
type ChangeEvent struct {
	Table     string
	Operation string // "insert", "update", "delete"
	Before    map[string]any
	After     map[string]any
}

// Consumer TIDAK PERNAH tahu proses apa yang menyebabkan perubahan
// ini di database — bisa dari endpoint API, bisa dari batch job,
// bisa dari query manual darurat. CDC membuat SEMUANYA tertangkap
// sama konsistennya.
type ChangeConsumer interface {
	Handle(ctx context.Context, event ChangeEvent) error
}

// SearchIndexSync menunjukkan konsumen konkret yang menjaga
// Elasticsearch tetap sinkron TANPA aplikasi asli perlu tahu index
// pencarian ini ada.
type SearchIndexSync struct{}

func (s *SearchIndexSync) Handle(ctx context.Context, event ChangeEvent) error {
	switch event.Operation {
	case "insert", "update":
		return indexDocument(ctx, event.Table, event.After)
	case "delete":
		return removeFromIndex(ctx, event.Table, event.Before)
	}
	return nil
}

func indexDocument(ctx context.Context, table string, data map[string]any) error { return nil }
func removeFromIndex(ctx context.Context, table string, data map[string]any) error { return nil }
```

## In His Stack

Untuk 13 aplikasi yang memakai MariaDB dan mungkin butuh menyinkronkan data ke Elasticsearch untuk pencarian, atau ke sistem analitik terpisah, [[../92 Tools/Debezium|Debezium]] adalah tool CDC paling umum dipakai di ekosistem open-source — mengintegrasikannya berarti mengaktifkan binlog di MariaDB (kalau belum aktif) dan menjalankan connector yang membaca binlog itu, mempublikasikan perubahan ke Kafka. Ini juga menjadi mekanisme inti untuk [[Zero-Downtime Database Migration Using CDC]] — memindahkan data ke sistem baru sambil sistem lama tetap melayani traffic, dibahas lebih dalam di klaster migrasi.

## Trade-offs and When Not To Use It

CDC menambah komponen infrastruktur baru yang harus dioperasikan (connector, message broker) dan butuh akses ke transaction log database (yang di beberapa lingkungan managed database, aksesnya dibatasi atau butuh konfigurasi khusus). Untuk kebutuhan sinkronisasi sederhana dengan sedikit titik perubahan data yang sudah diketahui semua (bukan proses batch atau query manual yang bisa muncul kapan saja), mengirim event manual di kode aplikasi mungkin cukup dan lebih sederhana untuk diimplementasikan pertama kali. CDC bernilai jelas begitu jumlah titik yang bisa mengubah data bertambah (termasuk proses yang di luar kendali langsung tim, atau yang mungkin ditambahkan orang lain di masa depan tanpa koordinasi ketat) — situasi di mana keandalan "menangkap semua perubahan tanpa kecuali" menjadi jauh lebih penting daripada kesederhanaan implementasi awal.

## Common Mistakes

> [!warning] Jebakan
> Mengandalkan aplikasi mengirim event manual di setiap titik yang mengubah data, tanpa mempertimbangkan proses batch, migrasi manual, atau perbaikan darurat yang mungkin mengubah data lewat jalur lain — persis masalah di "The Problem", drift yang diam-diam terjadi tanpa terdeteksi.

> [!warning] Jebakan
> Memakai trigger database sebagai pengganti CDC tanpa mempertimbangkan dampaknya ke transaksi utama — trigger yang gagal atau lambat menambah risiko langsung ke operasi tulis aplikasi, berbeda dari CDC yang sepenuhnya asinkron dan tidak menyentuh jalur tulis utama.

> [!warning] Jebakan
> Tidak mempertimbangkan perubahan skema tabel sumber yang bisa mematahkan consumer CDC — perubahan struktur tabel (menambah/menghapus kolom) mengubah bentuk event yang dihasilkan CDC, dan consumer yang tidak siap menangani ini bisa gagal memproses event baru, mirip masalah yang dibahas di [[Event Schema Evolution]].

## Exercises

1. Jelaskan perbedaan mendasar CDC dengan mengirim event manual di kode aplikasi setiap kali data berubah.
2. Kenapa CDC membaca transaction log, bukan melakukan polling berulang ke tabel database?
3. Kenapa trigger database bukan pengganti yang setara untuk CDC, meski keduanya bisa "menangkap" perubahan data?
4. Desain terbuka: 13 aplikasimu punya beberapa proses batch internal yang mengubah data langsung lewat query SQL manual untuk perbaikan data massal, dan tim baru menyadari index pencarian Elasticsearch sering menyimpang dari database karena proses batch ini tidak mengirim event sinkronisasi. Rancang solusi memakai CDC untuk masalah ini, dan jelaskan kenapa solusi ini lebih tahan terhadap proses baru yang mungkin ditambahkan di masa depan dibanding memperbaiki setiap proses batch satu per satu.

> [!success]- Kunci jawaban
> **1.** Mengirim event manual butuh setiap titik kode yang mengubah data secara eksplisit menyertakan kode pengiriman event — rentan terlewat untuk titik yang belum diketahui atau ditambahkan belakangan. CDC menangkap perubahan langsung dari transaction log database, sumber yang mencatat **semua** perubahan yang benar-benar commit, apa pun proses yang menyebabkannya, tanpa aplikasi perlu tahu atau melakukan apa pun secara eksplisit.
> **4.** Pasang CDC connector (Debezium) yang membaca binlog MariaDB dan mempublikasikan setiap perubahan ke Kafka, lalu consumer yang menyinkronkan ke Elasticsearch berlangganan aliran event itu — solusi ini menangkap perubahan dari **manapun** asalnya (endpoint API, proses batch lama, proses batch baru yang belum ditulis sekarang, bahkan perbaikan darurat lewat query manual) karena CDC bekerja di level transaction log database, bukan di level kode aplikasi spesifik. Ini lebih tahan terhadap proses baru di masa depan karena solusinya tidak butuh setiap proses baru "diingatkan" untuk menyertakan kode sinkronisasi — cukup mengubah data di database seperti biasa, dan CDC otomatis menangkapnya, menghilangkan seluruh kelas bug "lupa mengirim event" yang jadi akar masalah drift index pencarian yang dialami sekarang.

## Self-Check

- Apa perbedaan mendasar CDC dengan mengirim event manual di kode aplikasi?
- Kenapa CDC membaca transaction log, bukan polling tabel?
- Kenapa trigger database bukan pengganti yang setara untuk CDC?
- Kapan CDC adalah investasi yang sepadan dibanding mengirim event manual?

## Connected Notes

- [[Event Schema Evolution]] — perubahan skema tabel sumber memengaruhi bentuk event yang dihasilkan CDC, membutuhkan disiplin evolusi skema yang sama.
- [[../40 Databases/Read Replicas and Replication Lag|Read Replicas and Replication Lag]] — CDC membaca sumber data yang sama (transaction log) yang dipakai mekanisme replikasi database, dibahas dasarnya di note itu.
- [[../40 Databases/Keeping Search in Sync with the Source of Truth|Keeping Search in Sync with the Source of Truth]] — CDC adalah salah satu solusi paling andal untuk masalah sinkronisasi index pencarian yang dibahas di note itu.
- [[Zero-Downtime Database Migration Using CDC]] — kelanjutan langsung: penerapan CDC untuk kebutuhan migrasi database skala besar tanpa downtime, dibahas di klaster migrasi.
- [[../92 Tools/Debezium|Debezium]] — tool konkret paling umum dipakai mengimplementasikan CDC di ekosistem open-source.

## Further Reading

- Dokumentasi resmi Debezium bagian "How Debezium Works" — penjelasan mendalam mekanisme membaca transaction log untuk berbagai database.
- Martin Kleppmann, "Turning the Database Inside Out" — presentasi berpengaruh yang menghubungkan CDC dengan gagasan event-driven architecture secara lebih luas.

## Catatan Saya

*Tulis di sini apakah ada data di salah satu dari 13 aplikasimu yang disinkronkan ke sistem lain lewat kode manual, dan apakah pernah ditemukan drift karena ada titik perubahan yang terlewat.*
