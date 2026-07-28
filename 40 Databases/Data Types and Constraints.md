---
title: Data Types and Constraints
type: concept
level: junior
domain: databases
status: unread
difficulty: 2
est_minutes: 15
prerequisites: ["[[Deliberate Denormalisation]]"]
next: ["[[Index Basics]]"]
tags: [backend, databases]
created: 2026-07-28
---

## TL;DR

Tipe data dan constraint (`NOT NULL`, `UNIQUE`, `CHECK`, `FOREIGN KEY`) adalah cara database **menegakkan invariant** — aturan yang harus selalu benar — di titik data ditulis, bukan berharap setiap baris kode aplikasi mengingat untuk memvalidasinya. Validasi di lapisan aplikasi bisa dilewati (bug, endpoint baru yang lupa memanggil validator yang sama, akses langsung ke database untuk perbaikan data darurat); constraint di database tidak bisa dilewati siapa pun yang menulis lewat jalur SQL apa pun. Memilih tipe data yang tepat juga bukan detail kosmetik — tipe yang salah (misalnya `VARCHAR` untuk uang, `INT` untuk sesuatu yang bisa melebihi jangkauannya) melahirkan kelas bug yang baru terlihat setelah data bertumbuh.

## The Problem

Kolom `saldo` di tabel `rekening` awalnya didefinisikan sebagai `FLOAT`. Validasi "saldo tidak boleh negatif" hanya diterapkan di lapisan aplikasi Go, lewat satu fungsi yang dipanggil sebelum setiap transaksi penarikan. Suatu hari, tim menambahkan endpoint baru untuk "penyesuaian saldo manual oleh admin" (fitur darurat untuk memperbaiki kesalahan pencatatan) — endpoint ini ditulis terpisah, terburu-buru, dan lupa memanggil validator yang sama. Hasilnya: sebuah saldo bisa menjadi negatif tanpa terdeteksi sama sekali, karena tidak ada apa pun di database yang menolaknya.

Masalah kedua yang independen: `FLOAT` adalah tipe angka desimal biner yang **tidak presisi** untuk nilai uang — operasi aritmatika berulang pada `FLOAT` bisa menghasilkan galat pembulatan kecil yang terakumulasi (misalnya `0.1 + 0.2` tidak persis `0.3` dalam representasi biner). Untuk saldo rekening yang diaudit dan harus presisi ke rupiah, ini bukan detail teoretis — ini bug nyata yang bisa membuat total saldo sistem tidak pernah persis cocok dengan jumlah seharusnya. Perbaikan yang benar butuh dua hal sekaligus: tipe data yang tepat (`DECIMAL`, bukan `FLOAT`) dan `CHECK` constraint (`saldo >= 0`) yang ditegakkan database sendiri, terlepas dari jalur mana pun yang menulis ke tabel itu.

## Intuition

Bayangkan constraint database seperti **gerbang otomatis di pabrik**, dibandingkan validasi aplikasi yang seperti **instruksi SOP di papan pengumuman**. SOP hanya efektif kalau setiap pekerja membacanya dan mematuhinya secara sukarela setiap saat — pekerja baru yang belum sempat dilatih, atau pekerja lama yang terburu-buru, bisa melewatkannya tanpa ada yang menyadari sampai terlambat. Gerbang otomatis secara fisik menolak apa pun yang tidak memenuhi syarat, tidak peduli siapa yang mencoba lewat atau seberapa terburu-buru mereka.

Analogi ini bocor pada satu hal: gerbang pabrik hanya perlu dipasang sekali dan bekerja secara mekanis selamanya, sementara constraint database **tetap bisa keliru dirancang** — `CHECK` yang terlalu longgar, atau `UNIQUE` yang lupa menyertakan kolom yang seharusnya jadi bagian keunikan (misalnya unique per instansi, bukan unique global). Constraint memindahkan penegakan aturan dari "harapan setiap pengembang ingat" ke "definisi eksplisit yang diperiksa mesin" — tapi definisi itu sendiri tetap harus dirancang benar oleh manusia.

## How It Works

`NOT NULL` — kolom wajib diisi, database menolak `INSERT`/`UPDATE` yang mencoba menyisakan `NULL`.

`UNIQUE` — tidak boleh ada dua baris dengan nilai sama di kolom (atau kombinasi kolom) itu. Bisa lebih dari satu kolom sekaligus: `UNIQUE (instansi_id, tanggal)` berarti keunikan berlaku per kombinasi, bukan per kolom individual.

`CHECK` — syarat kondisi arbitrer yang harus selalu benar untuk setiap baris.

```sql
CREATE TABLE rekening (
    id INT PRIMARY KEY AUTO_INCREMENT,
    pemilik_id INT NOT NULL,
    saldo DECIMAL(15, 2) NOT NULL DEFAULT 0,
    CHECK (saldo >= 0),
    FOREIGN KEY (pemilik_id) REFERENCES pengguna(id)
);
```

`FOREIGN KEY` — menegakkan bahwa nilai di kolom ini **harus** cocok dengan nilai yang ada di tabel lain, mencegah "baris yatim" yang menunjuk ke entitas yang tidak pernah ada.

Pilihan tipe data yang paling sering keliru:

| Kebutuhan | Tipe yang salah | Tipe yang benar | Alasan |
|---|---|---|---|
| Nilai uang | `FLOAT`/`DOUBLE` | `DECIMAL(p, s)` | `FLOAT` adalah representasi biner aproksimatif, rentan galat pembulatan yang terakumulasi pada operasi berulang. |
| Timestamp lintas zona waktu | `DATETIME` tanpa info zona waktu | `TIMESTAMP` (dengan zona waktu tersimpan/dikonversi eksplisit) | `DATETIME` polos ambigu soal zona waktu mana yang dimaksud, sumber [[../94 Case Studies/Case - The Timezone Bug in a Nationwide Report|bug batas tanggal lintas zona waktu]]. |
| ID yang bertumbuh sangat besar | `INT` (jangkauan terbatas) | `BIGINT` | `INT` standar punya batas atas sekitar 2.1 miliar — tabel log atau transaksi bervolume tinggi bisa mendekatinya lebih cepat dari perkiraan. |
| Status/kategori tetap | `VARCHAR` bebas | `ENUM` atau tabel referensi + `FOREIGN KEY` | String bebas membuka celah salah ketik (`'aktif'` vs `'Aktif'` vs `'active'`) yang tidak akan pernah tertangkap constraint apa pun. |

## In Go

```go
package main

import (
	"context"
	"database/sql"
	"errors"
	"fmt"

	"github.com/go-sql-driver/mysql"
)

// TarikSaldo mencoba mengandalkan CHECK constraint di database sebagai
// pertahanan terakhir — bukan satu-satunya validasi, tapi jaring pengaman
// yang tidak bisa dilewati jalur penulisan mana pun.
func TarikSaldo(ctx context.Context, db *sql.DB, rekeningID int, jumlah string) error {
	_, err := db.ExecContext(ctx, `
		UPDATE rekening SET saldo = saldo - ? WHERE id = ?
	`, jumlah, rekeningID)
	if err != nil {
		var mysqlErr *mysql.MySQLError
		if errors.As(err, &mysqlErr) && mysqlErr.Number == 3819 { // CHECK constraint violation
			return fmt.Errorf("penarikan ditolak, saldo tidak boleh negatif: %w", err)
		}
		return fmt.Errorf("tarik saldo rekening %d: %w", rekeningID, err)
	}
	return nil
}
```

Perhatikan kode ini tidak mengganti validasi di lapisan aplikasi (mengecek saldo cukup sebelum mencoba menarik tetap penting untuk memberi pesan error yang baik ke pengguna) — `CHECK` constraint di sini adalah **lapisan kedua**, jaring pengaman kalau validasi aplikasi entah bagaimana terlewati atau punya bug.

## In His Stack

Yii2 punya lapisan validasi sendiri lewat `rules()` di `ActiveRecord` — ini **tidak** sama dengan constraint database, meski sering disalahpahami sebagai pengganti. Validasi Yii2 hanya berjalan kalau kode benar-benar memanggil `$model->validate()` atau `save()` lewat jalur `ActiveRecord` itu — migrasi data manual, query raw lewat `createCommand()`, atau skrip perbaikan darurat yang langsung menyentuh database sama sekali tidak melewati `rules()` Yii2. Constraint yang benar-benar ditegakkan di skema MariaDB adalah satu-satunya jaminan yang berlaku di **semua** jalur itu sekaligus — pertimbangan ini penting khususnya di sistem dengan banyak titik akses data (skrip migrasi, tools admin internal, integrasi partner) di luar alur `ActiveRecord` normal.

> [!question] Perlu diverifikasi
> Klaim: dukungan `CHECK` constraint yang benar-benar ditegakkan (bukan diterima secara sintaks tapi diabaikan) di MariaDB/MySQL bergantung versi.
> Kenapa ragu: riwayat MySQL sempat menerima sintaks `CHECK` tanpa benar-benar menegakkannya di versi-versi lama, sebelum penegakan sungguhan ditambahkan — versi persis kapan ini berubah mudah salah diingat.
> Cara verifikasi: dokumentasi resmi MySQL/MariaDB bagian "CHECK Constraints", periksa versi yang sedang dipakai produksi.

## Trade-offs and When Not To Use It

Constraint menambah friksi nyata ke operasi tulis: setiap `INSERT`/`UPDATE` sekarang harus lolos pemeriksaan, dan constraint yang terlalu ketat bisa menghambat kasus penggunaan yang sah tapi tidak terduga saat skema dirancang (misalnya `CHECK` yang terlalu spesifik untuk aturan bisnis yang ternyata sering berubah). Untuk data yang benar-benar sementara atau eksperimental (tabel staging untuk impor data mentah sebelum divalidasi dan dipindah ke tabel utama), constraint ketat justru menghalangi tujuan tabel itu — data mentah yang belum tentu bersih memang perlu bisa masuk dulu sebelum disaring.

## Common Mistakes

> [!warning] Jebakan
> Mengandalkan validasi di lapisan aplikasi (seperti `rules()` Yii2) sebagai satu-satunya pertahanan, tanpa constraint yang setara di database — jalur penulisan yang melewati lapisan aplikasi (skrip migrasi, akses langsung) akan lolos tanpa validasi apa pun.

> [!warning] Jebakan
> Memakai `FLOAT`/`DOUBLE` untuk nilai uang — galat pembulatan biner yang terakumulasi bisa membuat total tidak pernah persis cocok, terutama setelah banyak operasi aritmatika berulang.

> [!warning] Jebakan
> Membuat `UNIQUE` constraint pada kolom tunggal padahal keunikan yang sebenarnya diinginkan adalah per kombinasi kolom (misalnya unique per instansi, bukan unique global) — constraint yang salah cakupan bisa terlalu longgar (tidak mencegah duplikat yang seharusnya dicegah) atau terlalu ketat (menolak kombinasi valid yang seharusnya diperbolehkan).

## Exercises

1. Kenapa validasi "saldo tidak boleh negatif" yang hanya ada di kode aplikasi tidak cukup untuk sistem dengan banyak jalur penulisan data?
2. Jelaskan kenapa `FLOAT` bermasalah untuk menyimpan nilai uang, dan tipe apa yang seharusnya dipakai.
3. Rancang constraint (`NOT NULL`, `UNIQUE`, `CHECK`, `FOREIGN KEY`) yang tepat untuk tabel `pendaftaran_pelatihan (pegawai_id, pelatihan_id, tanggal_daftar)` — pastikan satu pegawai tidak bisa mendaftar dua kali ke pelatihan yang sama.
4. Desain terbuka: sebuah tabel `permohonan` punya kolom `nomor_registrasi` yang harus unik **per tahun**, bukan unik selamanya (nomor registrasi boleh mulai dari 1 lagi di tahun berikutnya, format seperti `2026-00042`). Rancang constraint yang tepat untuk kasus ini, dan diskusikan apakah nomor registrasi ini sebaiknya di-generate aplikasi atau database, dengan mempertimbangkan race condition antara pengecekan keunikan dan penulisan.

> [!success]- Kunci jawaban
> **3.** `UNIQUE (pegawai_id, pelatihan_id)` — bukan `UNIQUE (pegawai_id)` (yang salah, akan mencegah satu pegawai ikut pelatihan apa pun lebih dari sekali) atau `UNIQUE (pelatihan_id)` (yang juga salah, akan mencegah lebih dari satu pegawai ikut pelatihan yang sama). Kombinasi kedua kolom itulah yang harus unik, ditambah `NOT NULL` pada keduanya dan `FOREIGN KEY` masing-masing ke tabel `pegawai` dan `pelatihan`.
> **4.** Kalau format `nomor_registrasi` sudah menyertakan tahun sebagai bagian stringnya (`'2026-00042'`), constraint `UNIQUE (nomor_registrasi)` sederhana sudah cukup — keunikan per tahun otomatis terjaga karena tahunnya jadi bagian nilai itu sendiri. Yang lebih sulit adalah **generate** nomor urutnya secara aman dari race condition: pola naif "`SELECT MAX(nomor_urut) + 1`, baru `INSERT`" punya race condition yang sama persis dengan yang dibahas di [[Upserts]] — dua permohonan yang datang bersamaan bisa sama-sama membaca `MAX()` yang sama sebelum salah satunya sempat menulis. Solusi yang aman: gunakan mekanisme auto-increment terpisah per tahun (misalnya tabel counter kecil `registrasi_counter (tahun, urutan_terakhir)` yang di-upsert atomik `urutan_terakhir = urutan_terakhir + 1` dan hasilnya dibaca dalam transaksi yang sama), bukan menghitung `MAX()` dari tabel `permohonan` itu sendiri setiap kali nomor baru dibutuhkan.

## Self-Check

- Kenapa constraint database lebih andal dibanding validasi yang hanya ada di lapisan aplikasi?
- Kenapa `FLOAT` tidak cocok untuk menyimpan nilai uang?
- Apa beda `UNIQUE` pada satu kolom dan `UNIQUE` pada kombinasi beberapa kolom?
- Sebutkan satu skenario di mana constraint yang terlalu ketat justru menghalangi kebutuhan yang sah.

## Connected Notes

- [[Deliberate Denormalisation]] — constraint (terutama `CHECK` dan trigger terkait) sering jadi bagian dari mekanisme menjaga data terdenormalisasi tetap konsisten.
- [[Relational Modelling]] — `FOREIGN KEY` adalah representasi teknis langsung dari kardinalitas relasi yang dibahas di note itu.
- [[Upserts]] — race condition pada penomoran/penghitungan yang dibahas di jawaban exercise butuh pola atomik yang sama dengan upsert.
- [[Index Basics]] — banyak constraint (`UNIQUE`, `PRIMARY KEY`) secara implisit membuat index; keduanya saling terkait erat di lapisan implementasi.
- [[../80 Security/_Overview|Security Overview]] — constraint adalah salah satu lapisan pertahanan defense-in-depth, prinsip yang juga mendasari banyak topik keamanan di domain itu.

## Further Reading

- Dokumentasi resmi MySQL/MariaDB, bagian "CHECK Constraints" dan "Data Types" — referensi lengkap tipe data dan batasannya.
- Dokumentasi resmi PostgreSQL, bagian "Constraints" di manual DDL.

## Catatan Saya

*Tulis di sini kolom di kerjaanmu yang tipenya menurutmu salah pilih (misalnya FLOAT untuk uang), dan seberapa besar risikonya kalau belum diperbaiki.*
