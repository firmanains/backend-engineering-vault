---
title: Case - Daily CSV Over SFTP in 2026
type: case
level: intermediate
domain: architecture
status: unread
difficulty: 2
est_minutes: 13
source: synthesized
concepts: ["[[../30 APIs and Web/File-Based Integration|File-Based Integration]]", "[[../30 APIs and Web/Idempotency|Idempotency]]", "[[../30 APIs and Web/Integration Testing Across an Organisational Boundary|Integration Testing Across an Organisational Boundary]]"]
prerequisites: []
next: []
tags: [backend, architecture, apis, gov-context, integration]
created: 2026-08-02
---

## The Situation

Sebuah instansi mitra bersikeras integrasi data kasus dilakukan lewat file CSV harian yang diunggah ke server SFTP mereka — bukan API modern, bukan webhook, bukan pesan real-time. Tim teknis awalnya menganggap ini sebagai kemunduran ("kenapa tidak API saja seperti biasa?"), tapi setelah didalami, alasan partner cukup konkret: tim mereka kecil, tidak punya kapasitas mengoperasikan atau memantau integrasi API yang lebih kompleks, dan proses CSV/SFTP sudah jadi standar operasional mereka selama bertahun-tahun untuk semua integrasi dengan pihak luar.

## What Was Actually Asked For

Permintaan partner: "kirim file CSV setiap hari lewat SFTP." Yang sebenarnya mereka butuhkan (dan tidak secara eksplisit mereka minta, tapi jadi kebutuhan implisit siapa pun yang menerima data lewat file): file yang **selalu ada tepat waktu**, **isinya bisa dipercaya lengkap dan benar**, dan **jelas siapa yang bertanggung jawab kalau prosesnya gagal** — kebutuhan yang sama persis dengan integrasi API modern, hanya medianya berbeda.

## Options

### Option 1 — Kirim file apa adanya, tanpa jaminan tambahan

Job harian yang men-generate CSV dan mengunggahnya ke SFTP, tanpa mekanisme verifikasi lebih lanjut. Termudah diimplementasikan, tapi rapuh — kalau job gagal di tengah jalan, file yang terunggah bisa tidak lengkap tanpa ada yang tahu sampai partner komplain data yang mereka proses ternyata salah atau kurang.

### Option 2 — File dengan checksum dan penanda "selesai" terpisah

Setelah file CSV selesai ditulis penuh, hitung checksum-nya dan unggah **dua** file: CSV itu sendiri, dan file penanda kecil (misalnya `.done` atau berisi checksum) yang hanya diunggah **setelah** CSV selesai sepenuhnya. Partner tahu untuk hanya memproses file CSV yang sudah punya penanda selesai yang sesuai — mencegah mereka memproses file yang masih setengah jalan diunggah (kondisi yang bisa terjadi kalau mereka mulai membaca file tepat saat proses unggah masih berlangsung).

### Option 3 — File dengan nomor urut dan idempotency di sisi penerima

Selain checksum dan penanda selesai (Option 2), sertakan nomor urut atau tanggal eksplisit di nama file (`kasus_2026-08-02.csv`), dan pastikan proses di sisi partner (kalau bisa dikoordinasikan) idempoten terhadap pemrosesan ulang file yang sama — kalau file yang sama tidak sengaja diproses dua kali (misalnya job diunggah ulang manual), hasilnya tidak menghasilkan data ganda.

## The Decision

Kombinasi Option 2 dan 3 dipilih. Constraint penentunya: meski medianya "kuno" (file dan SFTP dibanding API modern), prinsip keandalan yang sama tetap berlaku — integritas data (checksum), kejelasan kapan file benar-benar siap diproses (penanda selesai), dan keamanan terhadap pemrosesan ganda (idempotency) adalah kebutuhan universal untuk integrasi lintas organisasi, tidak peduli protokolnya API real-time atau file harian. Menolak solusi ini karena "medianya ketinggalan zaman" akan mengabaikan constraint nyata partner (kapasitas tim mereka) demi preferensi teknis yang tidak menyelesaikan masalah nyata mereka.

## What Breaks at 10x and 100x

Pada volume 10x (file yang jauh lebih besar, atau lebih banyak file per hari untuk kategori data berbeda), transfer SFTP bisa mulai memakan waktu signifikan dan job yang menunggu transfer selesai penuh sebelum menandai "selesai" perlu dipastikan tidak timeout prematur. Pada 100x (integrasi serupa dengan puluhan partner berbeda, semuanya lewat file), mengelola job terjadwal dan verifikasi untuk setiap partner secara manual jadi tidak terkelola — titik ini membenarkan investasi platform integrasi file terpusat yang menstandarkan proses verifikasi, retry, dan monitoring lintas semua partner file-based sekaligus, alih-alih setiap integrasi punya skrip sendiri-sendiri.

## What You Would Do Differently

Mendokumentasikan format file, penanda selesai, dan kebijakan retensi secara eksplisit dan tertulis sejak kesepakatan awal dengan partner — bukan asumsi informal yang mudah disalahpahami kedua pihak, terutama untuk integrasi yang akan berjalan bertahun-tahun dengan kemungkinan pergantian personel di kedua sisi.

## Concepts Exercised

- [[../30 APIs and Web/File-Based Integration|File-Based Integration]] — konsep dasar yang membingkai seluruh pendekatan kasus ini.
- [[../30 APIs and Web/Idempotency|Idempotency]] — prinsip yang diterapkan mencegah pemrosesan ganda file yang sama.
- [[../30 APIs and Web/Integration Testing Across an Organisational Boundary|Integration Testing Across an Organisational Boundary]] — kebutuhan pengujian lintas dua tim berbeda yang relevan untuk memverifikasi kesepakatan format file ini benar-benar dipatuhi.

## Your Turn

Setelah beberapa bulan berjalan lancar, partner melaporkan bahwa file CSV kemarin "kosong" (hanya header, tanpa data), padahal seharusnya ada ratusan baris kasus. Bagaimana desain di atas (checksum, penanda selesai) membantu mendiagnosis apa yang sebenarnya terjadi?

> [!success]- Kunci jawaban
> Periksa dulu apakah penanda "selesai" untuk file itu benar-benar sesuai dengan checksum file CSV yang diterima partner — kalau cocok, berarti file memang benar-benar kosong saat proses generate selesai (masalah di sisi query/data sumber, bukan di proses transfer), dan investigasi harus fokus ke kenapa query yang menghasilkan CSV itu tidak menemukan data pada hari itu (mungkin bug di filter tanggal, atau memang kebetulan tidak ada kasus baru). Kalau checksum tidak cocok, berarti file rusak atau tidak lengkap selama proses transfer meski penanda selesai sudah ada — menunjukkan bug di urutan penulisan file dan penanda (penanda seharusnya hanya ditulis **setelah** checksum dihitung dari file final, bukan sebelum atau bersamaan). Kedua skenario ini butuh perbaikan yang sama sekali berbeda, dan desain checksum-plus-penanda yang sudah ada memberi cara cepat membedakan keduanya tanpa harus menebak-nebak.

## Catatan Saya

*Kalau kamu punya kejadian nyata dari kerjaan yang mirip tema ini, tulis sebagai case study baru dengan `source: personal`.*
