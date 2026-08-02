---
title: Case - The Timezone Bug in a Nationwide Report
type: case
level: intermediate
domain: architecture
status: unread
difficulty: 2
est_minutes: 13
source: synthesized
concepts: ["[[../20 Go Language/Struct Tags and JSON Marshalling|Struct Tags and JSON Marshalling]]", "[[../40 Databases/Data Types and Constraints|Data Types and Constraints]]", "[[../30 APIs and Web/API Versioning|API Versioning]]"]
prerequisites: []
next: []
tags: [backend, architecture, gov-context]
created: 2026-08-02
---

## The Situation

Laporan nasional bulanan yang menghitung jumlah kasus per hari menunjukkan angka yang aneh — beberapa hari terakhir setiap bulan selalu menunjukkan jumlah kasus jauh lebih sedikit dari hari-hari sebelumnya, pola yang tidak masuk akal secara operasional. Investigasi menemukan akar masalahnya: sistem menyimpan timestamp dalam UTC (praktik yang benar), tapi laporan menghitung "kasus per hari" dengan mengelompokkan berdasarkan tanggal UTC — sementara Indonesia berada di zona waktu UTC+7 (WIB) sampai UTC+9 (WIT), membuat kasus yang dibuat petugas jam 11 malam waktu lokal (yang sudah jadi tanggal berikutnya di UTC+7, tapi baru jam 4 sore UTC) terhitung di tanggal yang salah menurut sudut pandang lokal.

## What Was Actually Asked For

Permintaan awal: "angka laporan salah, tolong perbaiki." Yang sebenarnya dibutuhkan: laporan yang mengelompokkan kasus berdasarkan **tanggal menurut zona waktu lokal Indonesia yang relevan**, bukan tanggal UTC — kebutuhan yang terdengar sederhana tapi menyentuh keputusan desain yang lebih dalam, terutama karena Indonesia sendiri punya tiga zona waktu berbeda (WIB, WITA, WIT) tergantung wilayah instansi.

## Options

### Option 1 — Konversi ke satu zona waktu tetap (WIB) untuk semua laporan

Kelompokkan semua kasus berdasarkan tanggal WIB, terlepas dari wilayah asal kasus itu. Solusi termudah diimplementasikan, tapi secara halus tidak akurat untuk kasus dari wilayah WITA/WIT — kasus yang dibuat jam 11 malam WIT (yang sudah tanggal berikutnya menurut waktu lokal petugas itu) mungkin masih terhitung tanggal sebelumnya kalau dipaksa dikonversi ke WIB.

### Option 2 — Konversi berdasarkan zona waktu wilayah asal masing-masing kasus

Setiap kasus dikelompokkan berdasarkan tanggal menurut zona waktu **wilayah asal instansi yang mengajukannya** (WIB untuk Jawa, WITA untuk sebagian besar Kalimantan/Sulawesi, WIT untuk Papua, dst.) — akurat secara lokal untuk setiap wilayah, tapi menambah kompleksitas: laporan "nasional" sekarang menggabungkan hari yang secara teknis dikelompokkan dengan definisi berbeda per wilayah, sesuatu yang perlu dijelaskan jelas ke pembaca laporan supaya tidak disalahpahami sebagai angka yang tidak konsisten.

### Option 3 — Simpan timestamp UTC, tapi tambahkan kolom eksplisit "tanggal lokal" saat data dibuat

Alih-alih menghitung ulang zona waktu setiap kali laporan dibuat, simpan tanggal lokal (dihitung sekali, saat data pertama kali dibuat, berdasarkan zona waktu wilayah petugas yang mengajukan) sebagai kolom terpisah eksplisit — lihat pertimbangan tipe data yang tepat di [[../40 Databases/Data Types and Constraints|Data Types and Constraints]]. Laporan tinggal mengelompokkan berdasarkan kolom ini langsung, tanpa perlu logika konversi zona waktu berulang setiap kali laporan dijalankan, dan nilainya konsisten meski aturan konversi zona waktu (kalau pernah berubah) tidak memengaruhi data historis yang sudah tersimpan.

## The Decision

Option 3 dipilih — menyimpan tanggal lokal sebagai fakta eksplisit saat data dibuat, bukan dihitung ulang setiap kali dibutuhkan, adalah solusi yang paling tahan terhadap perubahan aturan di masa depan dan paling cepat untuk laporan yang sering dijalankan. Constraint penentunya: laporan nasional ini dijalankan rutin dan sering dirujuk untuk keputusan kebijakan — akurasi dan konsistensi jangka panjang lebih penting dibanding kemudahan implementasi awal yang ditawarkan Option 1.

## What Breaks at 10x and 100x

Pada volume 10x (lebih banyak jenis laporan yang butuh pengelompokan tanggal serupa), pola menyimpan "tanggal lokal" sebagai kolom eksplisit perlu diterapkan konsisten di semua tabel yang relevan, bukan hanya satu tabel yang kebetulan jadi sumber laporan pertama yang bermasalah — kebijakan desain ini perlu didokumentasikan sebagai standar tim, bukan solusi ad-hoc satu tabel. Pada 100x dengan ekspansi ke sistem yang melayani wilayah lintas negara (skenario hipotetis tapi relevan untuk sistem yang mungkin diperluas), asumsi "tiga zona waktu Indonesia" perlu digeneralisasi jadi penyimpanan zona waktu eksplisit per record, bukan hardcoded ke wilayah domestik saja.

## What You Would Do Differently

Menetapkan konvensi eksplisit sejak awal proyek tentang bagaimana tanggal "hari ini" didefinisikan untuk kebutuhan pelaporan — pertanyaan yang terlihat sepele tapi punya jawaban yang tidak sesederhana kelihatannya untuk negara dengan banyak zona waktu, dan seharusnya didiskusikan sebelum laporan pertama dibangun, bukan ditemukan lewat bug setelah production.

## Concepts Exercised

- [[../20 Go Language/Struct Tags and JSON Marshalling|Struct Tags and JSON Marshalling]] — relevan untuk memastikan representasi waktu (termasuk info zona waktu) ditangani benar saat data dipertukarkan lintas sistem.
- [[../40 Databases/Data Types and Constraints|Data Types and Constraints]] — pertimbangan tipe data yang tepat untuk menyimpan tanggal lokal secara eksplisit dan konsisten.
- [[../30 APIs and Web/API Versioning|API Versioning]] — relevan kalau perbaikan ini butuh perubahan kontrak API yang sudah dipakai sistem lain, butuh proses migrasi kontrak yang terkendali.

## Your Turn

Setelah kolom "tanggal lokal" diterapkan untuk kasus baru, tim mempertimbangkan apakah perlu menghitung ulang kolom ini untuk seluruh data historis yang sudah ada (jutaan baris lama yang belum punya kolom ini). Apa pertimbangan yang menentukan keputusan ini?

> [!success]- Kunci jawaban
> Pertimbangan utamanya adalah seberapa jauh ke belakang laporan historis benar-benar dibutuhkan akurat — kalau laporan bulanan hanya pernah dirujuk untuk periode beberapa bulan terakhir, backfill penuh ke seluruh riwayat data mungkin tidak sepadan biayanya (mengikuti prinsip backfill aman skala besar, lihat [[../60 Distributed Systems/Backfilling Large Datasets Safely|Backfilling Large Datasets Safely]]). Kalau ada kebutuhan nyata melihat tren jangka panjang (laporan tahunan yang membandingkan beberapa tahun), backfill penuh jadi perlu, dijalankan per-batch dengan hati-hati mengikuti strategi yang aman untuk tabel besar. Keputusan ini sebaiknya dikonfirmasi eksplisit dengan pihak yang memakai laporan itu — "apakah data historis sebelum perbaikan ini akan dirujuk lagi, dan seberapa penting akurasinya" — bukan diasumsikan sepihak oleh tim teknis tanpa mengecek kebutuhan nyata pengguna laporan.

## Catatan Saya

*Kalau kamu punya kejadian nyata dari kerjaan yang mirip tema ini, tulis sebagai case study baru dengan `source: personal`.*
