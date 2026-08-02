---
title: Case - The Counter That Undercounts
type: case
level: intermediate
domain: architecture
status: unread
difficulty: 3
est_minutes: 14
source: synthesized
concepts: ["[[../40 Databases/Locking and Row Locks|Locking and Row Locks]]", "[[../92 Tools/PostgreSQL - Locking and SELECT FOR UPDATE|PostgreSQL - Locking and SELECT FOR UPDATE]]", "[[../50 Concurrency and Performance/Race Conditions and the Race Detector|Race Conditions and the Race Detector]]"]
prerequisites: []
next: []
tags: [backend, architecture, databases, concurrency, gov-context]
created: 2026-08-02
---

## The Situation

Sebuah dashboard menampilkan jumlah kasus yang diproses per hari, dihitung lewat counter yang di-increment setiap kali sebuah kasus selesai diproses. Kode yang menangani ini terlihat sederhana: baca nilai counter saat ini, tambah satu, simpan kembali. Tim mulai menjalankan dua worker paralel untuk mempercepat pemrosesan kasus (mengejar lonjakan traffic musim pengajuan). Beberapa minggu kemudian, staf administrasi menyadari angka di dashboard **lebih kecil** dari jumlah kasus yang benar-benar selesai diproses hari itu — counter kehilangan hitungan tanpa penjelasan yang jelas.

## What Was Actually Asked For

Permintaan awal: "counter-nya salah, tolong perbaiki." Setelah investigasi, ternyata bukan bug logika sederhana — akar masalahnya adalah **race condition** klasik: dua worker membaca nilai counter yang sama (misalnya 100) hampir bersamaan, masing-masing menambah satu di memori mereka sendiri (jadi 101), dan menyimpan kembali — hasil akhirnya tetap 101, bukan 102 seperti seharusnya, karena tulisan kedua **menimpa** tulisan pertama tanpa tahu tulisan itu terjadi.

## Options

### Option 1 — Kunci di level aplikasi (mutex Go)

Menambahkan `sync.Mutex` di kode aplikasi untuk memastikan hanya satu goroutine yang bisa membaca-menambah-menyimpan counter di satu waktu. Solusi sederhana kalau seluruh proses increment terjadi dalam satu instance aplikasi yang sama — tapi tidak berfungsi kalau ada lebih dari satu instance aplikasi (mutex Go hanya berlaku dalam satu proses, tidak lintas proses/mesin), yang justru skenario nyata di sini (dua worker paralel bisa berarti dua proses/Pod berbeda).

### Option 2 — Row locking di database (`SELECT FOR UPDATE`)

Menggunakan lock eksplisit database (lihat [[../92 Tools/PostgreSQL - Locking and SELECT FOR UPDATE|PostgreSQL - Locking and SELECT FOR UPDATE]]) — worker mengunci baris counter sebelum membaca, menambah, dan menyimpan, memastikan tidak ada worker lain yang bisa membaca nilai yang sama sampai lock dilepas. Bekerja lintas proses/instance karena lock-nya di level database, bukan aplikasi. Butuh transaksi eksplisit dan sedikit lebih lambat dibanding operasi atomik murni, tapi benar untuk kasus lintas instance.

### Option 3 — Operasi atomik `UPDATE counter SET value = value + 1`

Alih-alih baca-tambah-simpan sebagai tiga langkah terpisah di aplikasi, jadikan satu statement SQL atomik yang menyuruh database sendiri yang melakukan penambahan — `UPDATE counters SET value = value + 1 WHERE id = ?`. Database menjamin operasi ini atomik secara internal, tanpa butuh lock eksplisit dari aplikasi. Paling sederhana dan paling cepat di antara ketiga opsi, karena tidak butuh baca-lalu-tulis sebagai dua langkah terpisah sama sekali.

## The Decision

Option 3 dipilih — untuk kasus counter sederhana seperti ini, database sudah menyediakan cara menjamin atomicity tanpa perlu logika locking eksplisit tambahan di aplikasi. Constraint penentunya: kesederhanaan dan performa — statement `UPDATE ... SET value = value + 1` mengeliminasi seluruh kelas race condition ini tanpa transaksi tambahan atau kompleksitas lock manual, jauh lebih murah dibanding Option 2 untuk kasus yang memang sesederhana increment counter.

## What Breaks at 10x and 100x

Pada volume 10x (banyak worker menambah counter yang sama secara sangat sering), statement `UPDATE` atomik tetap benar, tapi bisa menjadi titik kontensi (banyak transaksi menunggu giliran mengunci baris yang sama) — solusi di titik ini biasanya beralih ke pendekatan sharded counter (counter dipecah jadi beberapa baris, dijumlahkan saat dibaca) untuk mengurangi kontensi pada satu baris tunggal. Pada 100x lintas banyak instance database (skenario multi-region), atomic update di satu database tidak lagi cukup — butuh strategi agregasi terdistribusi yang lebih kompleks, kembali ke pertimbangan trade-off consistency-availability yang dibahas di [[../60 Distributed Systems/CAP Theorem and PACELC|CAP Theorem and PACELC]].

## What You Would Do Differently

Menjalankan `go test -race` (lihat [[../50 Concurrency and Performance/Race Conditions and the Race Detector|Race Conditions and the Race Detector]]) secara rutin di CI sejak awal — race detector kemungkinan besar akan menangkap pola read-modify-write yang tidak aman ini jauh sebelum sampai ke production, kalau ada test yang mensimulasikan akses konkuren ke fungsi ini.

## Concepts Exercised

- [[../40 Databases/Locking and Row Locks|Locking and Row Locks]] — mekanisme locking yang jadi alternatif solusi (Option 2) untuk kasus ini.
- [[../92 Tools/PostgreSQL - Locking and SELECT FOR UPDATE|PostgreSQL - Locking and SELECT FOR UPDATE]] — implementasi konkret row locking yang dipertimbangkan.
- [[../50 Concurrency and Performance/Race Conditions and the Race Detector|Race Conditions and the Race Detector]] — alat yang seharusnya menangkap masalah ini sebelum production.

## Your Turn

Tim mengganti solusi jadi Option 3 (`UPDATE ... SET value = value + 1`), tapi seminggu kemudian menemukan bahwa mereka juga butuh membaca nilai counter **sebelum** increment untuk keperluan logging ("counter berubah dari X ke Y"). Bagaimana mereka mendapatkan nilai lama dengan tetap aman dari race condition?

> [!success]- Kunci jawaban
> Memakai `RETURNING` (di PostgreSQL) atau pola serupa yang tersedia di database yang dipakai — `UPDATE counters SET value = value + 1 WHERE id = ? RETURNING value` mengembalikan nilai **setelah** update dalam satu operasi atomik yang sama, tanpa perlu query baca terpisah yang kembali membuka celah race condition. Untuk mendapatkan nilai sebelum dan sesudah sekaligus, sebagian database mendukung mengembalikan kedua nilai dalam satu statement, atau alternatifnya menghitung nilai lama dari nilai baru dikurangi satu (`value_lama = value_baru - 1`) tanpa perlu query tambahan sama sekali — tetap dalam satu operasi atomik, tidak membuka kembali kerentanan read-modify-write yang baru saja diperbaiki.

## Catatan Saya

*Kalau kamu punya kejadian nyata dari kerjaan yang mirip tema ini, tulis sebagai case study baru dengan `source: personal`.*
