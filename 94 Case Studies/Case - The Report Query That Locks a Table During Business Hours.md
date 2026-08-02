---
title: Case - The Report Query That Locks a Table During Business Hours
type: case
level: intermediate
domain: architecture
status: unread
difficulty: 3
est_minutes: 14
source: synthesized
concepts: ["[[../40 Databases/Read Replicas and Replication Lag|Read Replicas and Replication Lag]]", "[[../40 Databases/OLTP vs OLAP vs HTAP|OLTP vs OLAP vs HTAP]]", "[[../40 Databases/Basic Isolation Levels|Basic Isolation Levels]]"]
prerequisites: []
next: []
tags: [backend, architecture, databases, gov-context, performance]
created: 2026-08-02
---

## The Situation

Seorang staf administrasi di sebuah instansi biasa menjalankan laporan bulanan langsung dari aplikasi produksi di jam kerja — query yang menggabungkan data dari beberapa tabel besar untuk menghitung ringkasan seluruh kasus bulan berjalan. Suatu siang, laporan ini dijalankan bersamaan dengan lonjakan traffic petugas yang mengajukan dan memverifikasi kasus baru — dan tiba-tiba seluruh sistem terasa lambat, beberapa petugas melaporkan permintaan mereka timeout. Investigasi menunjukkan query laporan itu menahan lock pada tabel `kasus` cukup lama, memblokir tulisan baru dari petugas yang sedang aktif bekerja.

## What Was Actually Asked For

Permintaan awal: "laporan ini penting, jangan sampai tidak bisa dijalankan." Tapi yang sebenarnya dibutuhkan staf administrasi bukan "menjalankan query tepat sekarang di jam kerja" — mereka butuh **angka laporan yang akurat, tersedia dalam waktu wajar**, tidak harus real-time detik ke detik. Kebutuhan operasional petugas lapangan (bisa mengajukan dan memproses kasus tanpa gangguan) jauh lebih sensitif terhadap waktu dibanding kebutuhan laporan bulanan staf administrasi.

## Options

### Option 1 — Jadwalkan ulang laporan di luar jam kerja

Solusi paling murah: pindahkan eksekusi laporan ke malam hari lewat cron job, saat traffic petugas minimal. Tidak butuh perubahan arsitektur apa pun. Tapi tidak menyelesaikan masalah kalau suatu saat ada kebutuhan laporan ad-hoc mendesak di jam kerja (permintaan atasan yang butuh angka "sekarang juga"), dan tidak mengatasi akar masalah query berat yang menahan lock lama.

### Option 2 — Jalankan laporan di read replica

Arahkan query laporan ke [[../40 Databases/Read Replicas and Replication Lag|Read Replicas and Replication Lag]], bukan database utama — lock yang ditahan query laporan di replica tidak memengaruhi transaksi tulis di database utama sama sekali. Butuh infrastruktur replica yang mungkin belum ada, dan menerima replication lag (data laporan mungkin tertinggal beberapa detik dari kondisi live) — trade-off yang hampir selalu bisa diterima untuk laporan bulanan.

### Option 3 — Pisahkan ke database analitik terpisah (ClickHouse atau sejenis)

Untuk kebutuhan laporan yang makin sering dan makin kompleks, replikasi data (lewat CDC) ke database analitik yang memang dirancang untuk query agregat berat, lihat [[../40 Databases/OLTP vs OLAP vs HTAP|OLTP vs OLAP vs HTAP]]. Solusi paling matang jangka panjang, tapi investasi infrastruktur dan proses sinkronisasi yang jauh lebih besar dari kebutuhan sekarang.

## The Decision

Option 2 dipilih sebagai solusi langsung — read replica yang cukup murah diaktifkan (banyak platform database sudah menyediakan fitur ini) dan langsung menyelesaikan masalah lock tanpa investasi besar. Option 1 diterapkan sebagai lapisan tambahan (laporan rutin tetap dijadwalkan malam hari sebagai kebiasaan baik), tapi Option 2 memberi fleksibilitas menjalankan laporan ad-hoc kapan saja tanpa risiko mengganggu operasi. Option 3 dicatat sebagai langkah berikutnya kalau kebutuhan laporan terus bertambah kompleks, bukan langkah yang perlu diambil sekarang.

## What Breaks at 10x and 100x

Pada volume 10x (lebih banyak instansi, lebih banyak laporan berbeda dijalankan bersamaan), satu read replica bisa mulai kewalahan menampung beban query analitik yang makin beragam dan makin sering — titik ini yang membenarkan Option 3 (database analitik terpisah) sebagai investasi yang sudah sepadan, bukan berlebihan lagi. Pada 100x, query laporan yang makin kompleks (join lintas banyak tabel, agregasi periode panjang) pada replica relasional biasa mulai terasa lambat secara inheren — bukan lagi soal lock, tapi soal karakteristik penyimpanan row-oriented yang tidak optimal untuk beban analitik seberat itu.

## What You Would Do Differently

Menetapkan aturan eksplisit sejak awal — "query yang menyentuh lebih dari N baris atau berjalan lebih dari M detik wajib memakai read replica, tidak boleh langsung ke database utama" — sebagai kebijakan tim, bukan menunggu insiden nyata untuk menyadarinya.

## Concepts Exercised

- [[../40 Databases/Read Replicas and Replication Lag|Read Replicas and Replication Lag]] — solusi inti yang diterapkan untuk memisahkan beban baca berat dari database utama.
- [[../40 Databases/OLTP vs OLAP vs HTAP|OLTP vs OLAP vs HTAP]] — kerangka memahami kenapa beban laporan analitik dan beban transaksional harus dipisah pada skala besar.
- [[../40 Databases/Basic Isolation Levels|Basic Isolation Levels]] — mekanisme locking yang menjelaskan kenapa query laporan bisa memblokir tulisan di database utama.

## Your Turn

Staf administrasi mengeluh bahwa angka di laporan yang dijalankan dari read replica kadang "sedikit berbeda" dari yang mereka lihat di aplikasi utama untuk kasus yang baru saja diubah statusnya. Apa penjelasan paling mungkin, dan apakah ini masalah yang perlu diperbaiki?

> [!success]- Kunci jawaban
> Penjelasan paling mungkin adalah replication lag — perubahan terbaru di database utama butuh waktu (biasanya sangat singkat, tapi tidak nol) untuk sampai ke replica, dan laporan yang dijalankan tepat setelah perubahan terjadi bisa melihat data yang sedikit tertinggal. Ini biasanya **bukan** masalah yang perlu "diperbaiki" untuk laporan bulanan — selisih beberapa detik tidak relevan untuk data yang diagregasi selama sebulan penuh. Yang perlu dilakukan adalah mengomunikasikan eksplisit ke staf administrasi bahwa laporan ini bersifat eventual consistent dengan jeda singkat (lihat [[../60 Distributed Systems/Defensible Eventual Consistency|Defensible Eventual Consistency]]), bukan mencoba memaksakan laporan selalu 100% real-time yang akan mengembalikan masalah lock yang justru ingin dihindari.

## Catatan Saya

*Kalau kamu punya kejadian nyata dari kerjaan yang mirip tema ini, tulis sebagai case study baru dengan `source: personal`.*
