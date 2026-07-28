---
title: Locking and Row Locks
type: concept
level: intermediate
domain: databases
status: unread
difficulty: 3
est_minutes: 16
prerequisites: ["[[MVCC]]"]
next: ["[[Deadlocks]]"]
tags: [backend, databases]
created: 2026-07-29
---

## TL;DR

[[MVCC]] menyelesaikan konflik baca-tulis tanpa locking, tapi tidak menyelesaikan konflik **tulis-tulis** — dua transaction yang sama-sama ingin mengubah baris yang sama tetap butuh mekanisme yang memutuskan siapa menunggu siapa. Row-level locking mengunci **baris spesifik** yang sedang diubah, bukan seluruh tabel, memungkinkan transaction lain tetap bebas mengubah baris **lain** secara bersamaan. `SELECT ... FOR UPDATE` secara eksplisit mengunci baris yang dibaca untuk mencegah transaction lain mengubahnya sampai transaction pertama selesai — alat yang tepat untuk pola "baca, putuskan, lalu tulis berdasarkan apa yang dibaca" yang rentan terhadap race condition kalau hanya mengandalkan `UPDATE` biasa.

## The Problem

Dua worker job memproses antrean permohonan secara konkuren untuk mempercepat throughput. Masing-masing worker menjalankan `SELECT * FROM permohonan WHERE status = 'menunggu' LIMIT 1`, mendapati baris yang sama (karena keduanya membaca hampir bersamaan sebelum salah satu sempat mengubah status), lalu masing-masing memproses permohonan yang **sama** dan mengubah statusnya jadi "diproses" — job yang sama dikerjakan dua kali, menghasilkan duplikasi kerja atau, lebih buruk, efek samping ganda (misalnya email notifikasi terkirim dua kali, atau saldo dipotong dua kali kalau job ini melibatkan pembayaran).

Akar masalahnya: `SELECT` biasa tidak memberi tahu database "saya berniat mengubah baris ini setelah membacanya" — kedua worker sama-sama berhasil membaca baris yang sama tanpa saling menghalangi, karena `SELECT` biasa (di bawah MVCC) tidak pernah memblokir `SELECT` lain. Yang dibutuhkan adalah cara memberi tahu database secara eksplisit: "kunci baris ini begitu saya membacanya, supaya worker lain yang mencoba membaca baris yang sama untuk tujuan yang sama harus menunggu sampai saya selesai."

## Intuition

Bayangkan row-level locking seperti **mengambil nomor antrean fisik dari sebuah dokumen**, bukan sekadar melihatnya dari kejauhan. Kalau dua orang sama-sama melihat map di dinding (`SELECT` biasa), keduanya bisa melihat lokasi yang sama tanpa saling menghalangi — tapi kalau seseorang **mengambil** dokumen fisik itu dari rak (`SELECT ... FOR UPDATE`), orang lain yang ingin mengambil dokumen fisik yang sama harus menunggu sampai dokumen itu dikembalikan. `FOR UPDATE` mengubah tindakan "melihat" menjadi tindakan "mengklaim", dan klaim itu ditahan sampai transaction selesai (`COMMIT`/`ROLLBACK`).

Analogi ini bocor pada satu hal: mengambil dokumen fisik dari rak terlihat jelas bagi siapa pun yang datang setelahnya (rak itu kosong). Row lock di database **tidak terlihat** lewat `SELECT` biasa — worker lain yang menjalankan `SELECT` biasa (tanpa `FOR UPDATE`) tetap bisa "melihat" baris yang sedang dikunci (berkat MVCC, mereka melihat versi snapshot mereka sendiri), dan hanya `SELECT ... FOR UPDATE` (atau operasi tulis) terhadap baris yang sama yang benar-benar terhalang menunggu. Row lock adalah kontrak antara operasi-operasi yang secara eksplisit ikut dalam "permainan" locking yang sama, bukan penghalang universal terhadap semua akses.

## How It Works

```mermaid
sequenceDiagram
    participant W1 as Worker 1
    participant DB as Database
    participant W2 as Worker 2

    W1->>DB: BEGIN; SELECT * FROM permohonan WHERE id=7 FOR UPDATE
    DB-->>W1: Baris dikunci untuk W1
    W2->>DB: BEGIN; SELECT * FROM permohonan WHERE id=7 FOR UPDATE
    Note over DB: W2 DITAHAN (menunggu),\nbaris id=7 sudah dikunci W1
    W1->>DB: UPDATE status='diproses' WHERE id=7; COMMIT
    Note over DB: Lock W1 dilepas
    DB-->>W2: Baris dikunci untuk W2 (giliran W2)
    W2->>DB: SELECT status -> 'diproses' (BUKAN 'menunggu' lagi)
    Note over W2: W2 melihat status sudah berubah,\nmelewati baris ini, tidak diproses dobel
```

Diagram ini menunjukkan bahwa `FOR UPDATE` memaksa `Worker 2` **menunggu** sampai `Worker 1` selesai (bukan langsung membaca versi lama seperti `SELECT` biasa di bawah MVCC) — begitu giliran `Worker 2` tiba, ia melihat status yang **sudah diperbarui** `Worker 1`, sehingga tahu untuk tidak memproses baris yang sama lagi.

**Varian penting locking eksplisit:**

- **`FOR UPDATE`** — exclusive lock, mencegah transaction lain membaca (lewat `FOR UPDATE`/`FOR SHARE`) atau menulis baris yang sama sampai lock dilepas.
- **`FOR SHARE`** (`LOCK IN SHARE MODE` di MySQL) — shared lock, mengizinkan transaction lain juga mengambil shared lock (baca bersama) tapi mencegah siapa pun menulis baris itu sampai semua shared lock dilepas.
- **`SKIP LOCKED`** — alih-alih menunggu baris yang sudah terkunci, transaction langsung **melewati** baris itu dan mengambil baris berikutnya yang belum terkunci — pola inilah yang membuat "antrean job di database" (worker mengambil job berikutnya yang tersedia, bukan menunggu antrean) bisa diimplementasikan tanpa duplikasi kerja dan tanpa saling menunggu antar worker.
- **`NOWAIT`** — alih-alih menunggu, transaction langsung gagal dengan error kalau baris yang diminta sudah terkunci transaction lain, memberi aplikasi kesempatan menangani situasi itu secara eksplisit (misalnya, retry dengan strategi berbeda) daripada diam menunggu.

## Under The Hood

InnoDB (MySQL/MariaDB) mengimplementasikan row lock lewat mekanisme yang terikat pada **index** — locking sebenarnya terjadi pada entri index yang dilalui untuk menemukan baris, bukan langsung pada baris data itu sendiri. Konsekuensi praktis yang mengejutkan banyak orang: `UPDATE tabel SET x = 1 WHERE kolom_tanpa_index = 'y'` di InnoDB bisa mengunci **jauh lebih banyak baris** dari yang diharapkan (bahkan mendekati keseluruhan tabel dalam kasus tertentu), karena tanpa index yang tepat pada `kolom_tanpa_index`, InnoDB harus memindai dan mengunci setiap baris yang diperiksa selama pencarian, bukan hanya baris yang akhirnya cocok — locking granularity di InnoDB pada praktiknya sangat bergantung pada keberadaan index yang tepat, bukan murni "per baris" seperti namanya menyiratkan.

InnoDB juga mengimplementasikan **next-key locking** — kombinasi row lock pada baris yang ditemukan **dan** gap lock pada celah sebelum baris itu, mekanisme yang (seperti disinggung di [[Isolation Levels and Their Anomalies]]) menjadi alasan `REPEATABLE READ` InnoDB secara praktis mencegah sebagian besar phantom read, berbeda dari standar SQL murni. PostgreSQL tidak memakai gap lock — ia mengandalkan mekanisme SSI (disinggung di note yang sama) untuk mendeteksi konflik yang setara pada level `SERIALIZABLE`.

## In Go

```go
package worker

import (
	"context"
	"database/sql"
	"errors"
	"fmt"
)

var ErrTidakAdaJobTersedia = errors.New("tidak ada job tersedia untuk diproses")

// AmbilJobBerikutnya memakai FOR UPDATE SKIP LOCKED — pola "queue di dalam
// database" yang aman untuk banyak worker konkuren. Worker yang barisnya
// sedang dikunci worker lain TIDAK menunggu; ia langsung melompat ke baris
// berikutnya yang belum dikunci siapa pun, menghindari duplikasi kerja
// tanpa satu worker pun perlu menunggu worker lain selesai.
func AmbilJobBerikutnya(ctx context.Context, db *sql.DB) (int64, error) {
	tx, err := db.BeginTx(ctx, nil)
	if err != nil {
		return 0, fmt.Errorf("mulai transaction ambil job: %w", err)
	}
	defer tx.Rollback()

	var id int64
	err = tx.QueryRowContext(ctx, `
		SELECT id FROM permohonan
		WHERE status = 'menunggu'
		ORDER BY tanggal_dibuat ASC
		LIMIT 1
		FOR UPDATE SKIP LOCKED
	`).Scan(&id)
	if errors.Is(err, sql.ErrNoRows) {
		return 0, ErrTidakAdaJobTersedia
	}
	if err != nil {
		return 0, fmt.Errorf("ambil job berikutnya: %w", err)
	}

	if _, err := tx.ExecContext(ctx, `UPDATE permohonan SET status = 'diproses' WHERE id = ?`, id); err != nil {
		return 0, fmt.Errorf("tandai job sedang diproses: %w", err)
	}

	if err := tx.Commit(); err != nil {
		return 0, fmt.Errorf("commit ambil job: %w", err)
	}
	return id, nil
}
```

## In His Stack

`SELECT ... FOR UPDATE SKIP LOCKED` adalah pola yang sangat relevan sebagai alternatif ringan dari message queue sungguhan (Kafka, RabbitMQ) untuk kasus antrean sederhana yang volumenya tidak terlalu ekstrem — banyak sistem PHP/Yii2 lama memakai polling sederhana tanpa locking eksplisit sama sekali (`SELECT ... WHERE status = 'menunggu' LIMIT 1` tanpa `FOR UPDATE`), yang persis rentan terhadap race condition seperti di "The Problem" begitu lebih dari satu worker/cron job berjalan bersamaan — kondisi yang mudah terjadi tanpa disadari begitu sebuah cron job di-scale jadi berjalan di lebih dari satu server, atau lebih dari satu instance Kubernetes.

## Trade-offs and When Not To Use It

Row lock yang ditahan terlalu lama (transaction yang lambat menyelesaikan pekerjaannya setelah mengambil lock) memblokir transaction lain yang butuh baris yang sama, dan dalam kasus terburuk berkontribusi pada deadlock (dibahas di [[Deadlocks]]). Untuk beban kerja dengan konkurensi sangat tinggi terhadap baris yang sama (misalnya counter yang di-increment ribuan kali per detik), locking eksplisit per baris bisa menjadi bottleneck — pola atomic increment (`UPDATE counter SET nilai = nilai + 1`, yang di banyak database dieksekusi atomik tanpa perlu `SELECT ... FOR UPDATE` terpisah) atau pendekatan lain seperti sharding counter (memecah satu counter jadi beberapa baris yang dijumlahkan saat dibaca) lebih sesuai dibanding menahan row lock secara eksplisit untuk operasi sesederhana itu. Row lock adalah alat yang tepat untuk pola "baca, putuskan berdasarkan kondisi kompleks, lalu tulis" — bukan untuk operasi tulis sederhana yang bisa dilakukan atomik dalam satu statement.

## Common Mistakes

> [!warning] Jebakan
> Menjalankan `UPDATE`/`SELECT ... FOR UPDATE` dengan kondisi `WHERE` pada kolom tanpa index — di InnoDB, ini bisa mengunci jauh lebih banyak baris dari yang diperlukan karena locking terjadi mengikuti jalur pencarian index (atau full scan kalau tidak ada index), bukan hanya baris yang akhirnya cocok.

> [!warning] Jebakan
> Memakai `SELECT` biasa (tanpa `FOR UPDATE`) untuk pola "baca kondisi, lalu tulis berdasarkan kondisi itu", berharap MVCC akan mencegah race condition — MVCC mencegah konflik baca-tulis, bukan konflik dua transaction yang sama-sama membaca kondisi yang sama lalu sama-sama menulis berdasarkan kondisi itu (persis kasus dua worker di "The Problem").

> [!warning] Jebakan
> Menahan row lock terlalu lama dengan melakukan operasi mahal (memanggil API eksternal, menunggu I/O lambat) di dalam transaction yang sudah mengambil lock — memperbesar jendela waktu transaction lain harus menunggu, meningkatkan risiko timeout dan deadlock.

## Exercises

1. Jelaskan kenapa `SELECT` biasa tidak cukup untuk mencegah race condition pada pola "baca kondisi, lalu tulis berdasarkan kondisi itu", meski database memakai MVCC.
2. Apa fungsi `SKIP LOCKED`, dan kenapa itu berbeda dari perilaku default `FOR UPDATE` yang menunggu?
3. Kenapa locking pada kolom tanpa index di InnoDB bisa mengunci jauh lebih banyak baris dari yang diharapkan?
4. Desain terbuka: sistemmu punya cron job yang berjalan di tiga server berbeda secara independen (tanpa koordinasi eksplisit antar server), masing-masing menjalankan query yang sama untuk mengambil dan memproses maksimal 10 permohonan berstatus "menunggu" setiap menit. Rancang query yang aman dari duplikasi kerja antar ketiga server ini, dan jelaskan kenapa pendekatanmu tidak menyebabkan ketiga server saling menunggu tanpa perlu.

> [!success]- Kunci jawaban
> **1.** MVCC memastikan `SELECT` tidak pernah diblokir oleh transaction lain yang sedang menulis (dan sebaliknya) — tapi itu berarti dua transaction yang **sama-sama hanya membaca** kondisi yang sama (`status = 'menunggu'`) bisa sama-sama berhasil membacanya sebagai `true` secara independen, tanpa saling tahu, sebelum salah satu dari mereka sempat menulis perubahan yang seharusnya membuat kondisi itu jadi `false` bagi yang lain. MVCC tidak dirancang mencegah ini karena ini bukan konflik baca-tulis dalam pengertian MVCC — ini konflik keputusan yang diambil dari pembacaan yang sama-sama valid pada titik waktunya masing-masing.
> **4.** Query yang aman: `SELECT id FROM permohonan WHERE status = 'menunggu' ORDER BY tanggal_dibuat ASC LIMIT 10 FOR UPDATE SKIP LOCKED`, diikuti `UPDATE ... SET status = 'diproses'` dalam transaction yang sama, lalu `COMMIT`. Ketiga server bisa menjalankan query yang identik ini secara bersamaan tanpa duplikasi: `SKIP LOCKED` memastikan begitu server pertama mengunci 10 baris tertentu, server kedua dan ketiga yang menjalankan query hampir bersamaan akan **melompati** 10 baris yang sudah terkunci itu dan langsung mengambil 10 baris **berikutnya** yang masih tersedia — bukan menunggu (yang akan sia-sia karena begitu server pertama commit, baris itu sudah berubah status jadi "diproses" dan tidak lagi cocok kondisi `WHERE`), dan bukan pula mengambil baris yang sama (yang akan menyebabkan duplikasi kerja). Ini yang membuat `SKIP LOCKED` secara khusus lebih tepat dibanding `FOR UPDATE` polos untuk pola multi-worker/multi-server semacam ini.

## Self-Check

- Kenapa MVCC saja tidak cukup mencegah race condition pada pola "baca kondisi, lalu tulis"?
- Apa perbedaan `FOR UPDATE`, `FOR SHARE`, dan `SKIP LOCKED`?
- Kenapa locking di InnoDB sangat bergantung pada keberadaan index yang tepat?
- Kapan row lock eksplisit adalah alat yang salah dipakai, dan pendekatan apa yang lebih tepat untuk kasus itu?

## Connected Notes

- [[MVCC]] — row lock menyelesaikan konflik tulis-tulis yang tidak diselesaikan MVCC sendirian, dua mekanisme yang saling melengkapi.
- [[Isolation Levels and Their Anomalies]] — write skew adalah anomali yang bisa dicegah lewat locking eksplisit sebagai alternatif menaikkan isolation level ke `SERIALIZABLE`.
- [[Deadlocks]] — risiko langsung dari row lock yang ditahan lebih dari satu transaction secara bersilangan, dibahas di note berikutnya.
- [[../92 Tools/PostgreSQL - Locking and SELECT FOR UPDATE|PostgreSQL - Locking and SELECT FOR UPDATE]] — detail operasional lengkap `FOR UPDATE`, `SKIP LOCKED`, `NOWAIT`, dan advisory lock di PostgreSQL secara spesifik.
- [[../30 APIs and Web/Idempotency|Idempotency]] — pola queue-in-database dengan `SKIP LOCKED` di note ini adalah salah satu cara konkret mencegah job diproses dua kali, prinsip yang sama dengan idempotency di level API.

## Further Reading

- Dokumentasi resmi PostgreSQL, bagian "Explicit Locking" dan "SELECT ... FOR UPDATE".
- Dokumentasi resmi MySQL/InnoDB, bagian "Locking Reads" dan "Locks Set by Different SQL Statements".

## Catatan Saya

*Tulis di sini apakah ada cron job atau worker di kerjaanmu yang berjalan lebih dari satu instance tanpa locking eksplisit — dan apakah pernah terjadi duplikasi kerja yang bisa dijelaskan oleh race condition di note ini.*
