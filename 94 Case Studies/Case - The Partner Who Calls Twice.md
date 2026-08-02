---
title: Case - The Partner Who Calls Twice
type: case
level: intermediate
domain: architecture
status: unread
difficulty: 2
est_minutes: 13
source: synthesized
concepts: ["[[../30 APIs and Web/Idempotency|Idempotency]]", "[[../60 Distributed Systems/Idempotency Keys|Idempotency Keys]]", "[[../30 APIs and Web/Timeouts in HTTP Servers|Timeouts in HTTP Servers]]"]
prerequisites: []
next: []
tags: [backend, architecture, apis, gov-context, integration]
created: 2026-08-02
---

## The Situation

Sebuah endpoint yang menerima pengajuan permohonan dari sistem partner (instansi lain) kadang menerima **dua permintaan identik** untuk satu pengajuan yang sama, beberapa detik terpisah. Investigasi menunjukkan penyebabnya: sistem partner mengonfigurasi timeout klien mereka di 5 detik, sementara endpoint ini kadang butuh 6-8 detik untuk memproses (verifikasi dokumen yang cukup berat). Partner, mengikuti praktik retry yang wajar di sisi mereka, mengirim ulang permintaan yang sama begitu timeout mereka habis — tidak tahu bahwa permintaan pertama sebenarnya sedang (atau sudah) diproses dengan sukses di sisi endpoint ini.

## What Was Actually Asked For

Tim internal awalnya meminta "minta partner menaikkan timeout mereka" — permintaan yang secara teknis masuk akal tapi realistisnya sulit dipenuhi cepat (perubahan konfigurasi di sisi partner butuh proses persetujuan mereka sendiri, dan tidak ada jaminan mereka akan menyetujuinya). Kebutuhan sebenarnya bukan "partner harus menunggu lebih lama" — kebutuhan sebenarnya adalah **sistem ini harus aman menerima permintaan duplikat**, terlepas dari apa pun konfigurasi timeout di sisi partner, karena duplikasi semacam ini adalah kenyataan yang akan terus terjadi di integrasi lintas organisasi mana pun.

## Options

### Option 1 — Minta partner menaikkan timeout dan menambah retry logic yang lebih sabar

Solusi yang menyerahkan masalah ke pihak lain. Kalaupun disetujui, tidak menghilangkan kemungkinan duplikasi sepenuhnya (jaringan tetap bisa memutus koneksi meski partner menunggu lebih lama), hanya mengurangi frekuensinya. Butuh koordinasi dan persetujuan pihak partner yang di luar kendali langsung tim ini.

### Option 2 — Mempercepat pemrosesan di bawah 5 detik

Optimasi verifikasi dokumen supaya selalu selesai di bawah threshold timeout partner. Solusi tekstual yang terdengar benar, tapi tidak menghilangkan akar masalah — timeout partner bisa berubah kapan saja tanpa pemberitahuan, dan sistem lain (bukan partner ini) mungkin punya timeout yang lebih pendek lagi. Mengejar target performa berdasarkan konfigurasi satu partner tertentu adalah solusi yang rapuh terhadap perubahan yang di luar kendali.

### Option 3 — Idempotency key di endpoint

Endpoint mewajibkan header `Idempotency-Key` (partner diminta menyertakan ID unik per pengajuan, bukan per percobaan request) — lihat [[../60 Distributed Systems/Idempotency Keys|Idempotency Keys]]. Permintaan kedua dengan key yang sama tidak diproses ulang, cukup mengembalikan hasil dari permintaan pertama. Solusi ini aman terlepas dari timeout di sisi mana pun, dan tidak butuh koordinasi mengubah perilaku partner selain menambahkan satu header di permintaan mereka (perubahan kecil dan masuk akal diajukan sebagai bagian dari spesifikasi API).

## The Decision

Option 3 dipilih sebagai solusi struktural, dikombinasikan dengan optimasi ringan dari Option 2 (mempercepat pemrosesan sebagai manfaat tambahan, bukan solusi utama). Constraint penentunya: idempotency key menyelesaikan masalah secara mendasar dan berlaku untuk **semua** partner masa depan, bukan hanya partner yang timeout-nya kebetulan pendek sekarang — investasi satu kali yang menghilangkan seluruh kelas masalah, dibanding solusi 1 dan 2 yang hanya mengurangi gejala untuk satu partner spesifik.

## What Breaks at 10x and 100x

Pada volume 10x, tanpa idempotency key, biaya operasional menangani duplikasi manual (tim yang harus mendeteksi dan membersihkan data duplikat) tumbuh proporsional dengan jumlah request — sesuatu yang tidak terjadi lagi begitu idempotency key diterapkan, karena penanganan duplikasi jadi otomatis di level sistem. Pada 100x dengan banyak partner berbeda (bukan cuma satu), solusi 1 dan 2 sama sekali tidak skalabel — tidak mungkin mengoordinasikan timeout setiap partner satu per satu; idempotency key tetap bekerja sama baiknya tidak peduli berapa banyak partner yang terlibat.

## What You Would Do Differently

Menetapkan idempotency key sebagai syarat wajib sejak spesifikasi API pertama kali dirancang, bukan ditambahkan reaktif setelah masalah duplikasi ditemukan — partner yang sudah terintegrasi tanpa header ini butuh koordinasi ulang untuk menambahkannya, proses yang lebih mudah dilakukan di awal sebelum banyak pihak bergantung pada kontrak lama.

## Concepts Exercised

- [[../30 APIs and Web/Idempotency|Idempotency]] — konsep dasar yang jadi fondasi solusi struktural kasus ini.
- [[../60 Distributed Systems/Idempotency Keys|Idempotency Keys]] — mekanisme konkret yang diterapkan sebagai solusi.
- [[../30 APIs and Web/Timeouts in HTTP Servers|Timeouts in HTTP Servers]] — akar masalah teknis (selisih timeout) yang memicu duplikasi.

## Your Turn

Endpoint yang sama, setelah idempotency key diterapkan, menerima permintaan kedua dengan `Idempotency-Key` yang sama tapi payload yang **berbeda** (nomor pengajuan sama, tapi nominal berbeda). Apa yang seharusnya dilakukan sistem?

> [!success]- Kunci jawaban
> Sistem harus **menolak** permintaan kedua dengan error eksplisit (bukan memproses ulang, dan bukan diam-diam mengembalikan hasil permintaan pertama) — payload berbeda dengan idempotency key sama adalah indikasi kesalahan di sisi partner (memakai ulang key untuk pengajuan yang sebenarnya berbeda), bukan retry yang sah dari permintaan yang sama. Sistem yang benar menyimpan hash payload asli bersama idempotency key, membandingkannya di setiap permintaan berikutnya, dan mengembalikan error jelas ("idempotency key sudah dipakai untuk payload berbeda") supaya partner segera menyadari kesalahannya, bukan menyembunyikan inkonsistensi ini secara diam-diam.

## Catatan Saya

*Kalau kamu punya kejadian nyata dari kerjaan yang mirip tema ini, tulis sebagai case study baru dengan `source: personal`.*
