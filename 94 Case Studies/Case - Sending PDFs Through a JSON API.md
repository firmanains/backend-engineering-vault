---
title: Case - Sending PDFs Through a JSON API
type: case
level: intermediate
domain: architecture
status: unread
difficulty: 2
est_minutes: 14
source: synthesized
concepts: ["[[../30 APIs and Web/Content Types and multipart-form-data|Content Types and multipart-form-data]]", "[[../30 APIs and Web/Pre-signed URLs|Pre-signed URLs]]", "[[../30 APIs and Web/Request Size Limits Along The Path|Request Size Limits Along The Path]]"]
prerequisites: []
next: []
tags: [backend, architecture, apis, gov-context, integration]
created: 2026-08-02
---

## The Situation

Sebuah instansi pemerintah daerah perlu mengirim hasil pindaian dokumen persetujuan (file PDF, ukuran bervariasi dari 200KB sampai 15MB tergantung jumlah halaman dan resolusi pindaian) ke sistem pusat milik kementerian setiap kali sebuah kasus disetujui. Sistem pusat menyediakan satu API — dan API itu **hanya** menerima `application/json`, hasil desain lama yang tidak bisa diubah karena dipakai puluhan instansi lain dan proses perubahan kontrak butuh persetujuan berlapis yang memakan waktu berbulan-bulan. Tenggat integrasi ini satu bulan, terikat program nasional yang sudah diumumkan ke publik.

## What Was Actually Asked For

Permintaan literalnya: "kirim file PDF ke endpoint ini." Kebutuhan sebenarnya, setelah digali lebih dalam bersama tim sistem pusat: mereka butuh **bukti bahwa dokumen itu benar-benar dikirim instansi yang berwenang, utuh tanpa kerusakan, dan bisa diaudit siapa mengirim apa kapan** — bukan sekadar "file sampai". Format JSON adalah batasan kontrak yang tidak bisa dihindari, tapi cara data biner masuk ke dalam JSON itu sepenuhnya di tangan tim integrasi.

## Options

### Option 1 — Base64 encode langsung di body JSON

Encode seluruh file PDF jadi base64, sisipkan sebagai satu field string dalam payload JSON. Paling sederhana diimplementasikan, tidak butuh perubahan di sisi manapun selain encoding/decoding. Tapi base64 menambah ukuran sekitar 33% (lihat [[../30 APIs and Web/Content Types and multipart-form-data|Content Types and multipart-form-data]]), dan untuk file 15MB, payload JSON membengkak jadi sekitar 20MB — berisiko melampaui batas ukuran request di berbagai titik jalur (load balancer, reverse proxy, API gateway) yang mungkin belum dikonfigurasi menerima payload sebesar itu.

### Option 2 — Pre-signed URL: unggah dulu, kirim referensinya

Instansi mengunggah PDF ke object storage terlebih dulu lewat pre-signed URL (lihat [[../30 APIs and Web/Pre-signed URLs|Pre-signed URLs]]), lalu payload JSON ke sistem pusat hanya berisi metadata (nomor kasus, checksum, dan URL/ID referensi file itu). Sistem pusat mengambil file dari storage kapan pun ia siap memprosesnya. Paling ringan di jalur jaringan API, dan memisahkan masalah "transfer file besar" dari "transfer metadata terstruktur" — tapi butuh infrastruktur object storage yang bisa diakses kedua pihak, dan proses persetujuan tambahan (siapa boleh mengunggah ke storage siapa) yang mungkin juga butuh waktu birokrasi.

### Option 3 — Base64 dengan chunking dan checksum eksplisit

Kalau opsi 2 tidak realistis dalam tenggat satu bulan (butuh persetujuan infrastruktur tambahan lintas instansi), file besar dipecah jadi beberapa chunk, masing-masing dikirim sebagai request JSON terpisah dengan nomor urut dan checksum per-chunk, disusun ulang di sisi penerima. Lebih rumit diimplementasikan dibanding opsi 1, tapi menghindari masalah ukuran payload tunggal yang terlalu besar, sekaligus memberi titik pemulihan (retry per-chunk, bukan mengirim ulang seluruh file) kalau koneksi terputus di tengah jalan.

## The Decision

Opsi 1 (base64 langsung) dipilih untuk fase pertama, dengan syarat eksplisit: batas ukuran file dinaikkan di **semua** titik jalur jaringan (dikoordinasikan dan diverifikasi satu per satu, bukan diasumsikan), dan file di atas ambang tertentu (misalnya 8MB) dikompres dulu sebelum dikirim kalau memungkinkan. Constraint penentunya adalah tenggat satu bulan — opsi 2 secara teknis lebih baik tapi butuh koordinasi infrastruktur lintas instansi yang realistisnya tidak selesai dalam waktu itu, dan opsi 3 menambah kompleksitas implementasi yang juga berisiko terhadap tenggat. Base64 dipilih sebagai solusi "cukup baik sekarang", dengan checksum (hash SHA-256 dari file asli) disertakan di payload sebagai jaminan integritas minimal — kompromi yang defensible, bukan solusi ideal.

## What Breaks at 10x and 100x

Pada volume 10x (dari puluhan kasus per hari jadi ratusan): payload base64 besar mulai membebani bandwidth dan waktu proses kedua sisi secara nyata, dan biaya penyimpanan sementara di sisi sistem pusat (yang harus menyimpan payload JSON mentah sebelum diproses) mulai terasa. Pada 100x (ribuan kasus per hari, skenario realistis kalau program ini diperluas nasional): base64 langsung di JSON menjadi jelas tidak sepadan — titik ini adalah alasan kuat bermigrasi ke opsi 2 (pre-signed URL), dan proses birokrasi yang tadinya "terlalu lambat untuk tenggat satu bulan" sekarang punya waktu dan justifikasi jelas untuk diselesaikan dengan benar.

## What You Would Do Differently

Dengan waktu lebih banyak sejak awal, mengajukan opsi pre-signed URL ke tim sistem pusat lebih dulu (bahkan sebelum tenggat mendesak) mungkin bisa mempercepat persetujuan infrastrukturnya — birokrasi lintas instansi sering lebih lambat justru karena diajukan mendadak di bawah tekanan, bukan direncanakan lebih awal dengan justifikasi teknis yang matang.

## Concepts Exercised

- [[../30 APIs and Web/Content Types and multipart-form-data|Content Types and multipart-form-data]] — biaya base64 encoding yang jadi pertimbangan inti keputusan ini.
- [[../30 APIs and Web/Pre-signed URLs|Pre-signed URLs]] — solusi jangka panjang yang lebih matang untuk transfer file besar.
- [[../30 APIs and Web/Request Size Limits Along The Path|Request Size Limits Along The Path]] — kebutuhan memverifikasi batas ukuran di setiap titik jalur, bukan diasumsikan.

## Your Turn

Instansi lain mengalami situasi serupa, tapi filenya adalah video rekaman sidang (rata-rata 200MB), bukan PDF. Apakah base64 langsung di JSON masih opsi yang defensible untuk kasus ini?

> [!success]- Kunci jawaban
> Tidak — base64 langsung sudah tidak defensible untuk ukuran sebesar ini bahkan sebagai solusi sementara. Penambahan 33% dari 200MB menjadi hampir 270MB jauh melampaui batas wajar payload JSON di hampir semua infrastruktur, dan risiko kegagalan di tengah transfer (koneksi terputus, timeout) jauh lebih tinggi untuk file sebesar ini. Pre-signed URL (opsi 2) menjadi pilihan yang hampir wajib di sini, bukan sekadar "lebih baik" — kalau infrastrukturnya belum siap, solusi sementara yang lebih realistis adalah menolak permintaan sampai infrastruktur itu ada, bukan memaksakan base64 untuk ukuran video sebesar ini.

## Catatan Saya

*Kalau kamu punya kejadian nyata dari kerjaan yang mirip tema ini, tulis sebagai case study baru dengan `source: personal`.*
