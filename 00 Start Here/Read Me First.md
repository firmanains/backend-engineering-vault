---
title: Read Me First
type: concept
level: junior
domain: foundations
status: mastered
difficulty: 1
est_minutes: 8
prerequisites: []
next: ["[[How To Read This Vault]]"]
tags: [backend, meta]
created: 2026-07-26
---

## Apa Ini

Ini adalah **Backend Engineering Vault** — sebuah jalur belajar yang tersimpan sebagai kumpulan catatan Obsidian yang saling terhubung, dirancang untuk membawa seorang backend engineer dari level junior menuju level senior, dengan **Go** sebagai bahasa utama.

Vault ini bukan dokumentasi referensi yang dibaca sekali lalu ditutup. Ia dirancang untuk dibaca ulang, ditandai, dan ditulisi selama bertahun-tahun — sampai bagian `## Catatan Saya` di setiap note penuh dengan pertanyaan dan contoh nyata dari pekerjaanmu sendiri.

## Untuk Siapa Vault Ini Dibuat

Vault ini dibuat untuk satu pembaca spesifik: seorang backend engineer dengan pengalaman produksi sekitar satu tahun, yang juga menjadi koordinator teknis untuk lebih dari 10 developer di lebih dari 13 aplikasi legal-services milik pemerintah. Stack harian: **PHP (Yii1/Yii2) + MariaDB**, dikelilingi ekosistem **Go, Kafka, Elasticsearch, Kubernetes, Jenkins, Next.js**.

Karena itu, isi vault ini tidak menjelaskan dasar-dasar pemrograman, HTTP, SQL, atau Git — pembaca sudah menguasainya. Yang ditulis di sini dimulai dari level seorang working engineer, dan secara sengaja memberi bobot lebih besar pada **integrasi antar sistem**, karena itulah pekerjaan sehari-hari yang sebenarnya: berbicara dengan sistem milik pihak lain yang tidak bisa kamu ubah.

## Tiga Tujuan

1. **Menguasai backend engineering secara konseptual sampai level senior** — bukan hanya tahu istilahnya, tapi paham kenapa setiap konsep itu lahir, dan kapan konsep itu adalah pilihan yang salah.
2. **Mastery sungguhan dalam Go** — bukan sekadar bisa menulis handler HTTP, tapi paham scheduler, memory model, dan idiom yang membuat kode Go idiomatic, bukan Java atau PHP yang ditulis ulang dengan sintaks Go.
3. **Membangun judgement** — kemampuan untuk mengambil dan mempertahankan keputusan arsitektural di bawah tekanan, dengan alasan yang bisa dijelaskan ke orang lain.

Di baliknya ada satu ambisi jangka panjang: **gelar master di bidang distributed systems**. Folder `60 Distributed Systems/` diperlakukan sebagai jalur serius, bukan bab pengantar yang ditulis untuk sekadar melengkapi kurikulum.

## Bahasa

Penjelasan ditulis dalam **Bahasa Indonesia**. Istilah teknis tetap dalam **Bahasa Inggris** dan tidak pernah diterjemahkan — kamu akan bertemu istilah-istilah itu dalam bahasa Inggris di dokumentasi resmi, pesan error, wawancara kerja, dan paper akademis, jadi menerjemahkannya justru membuatnya lebih sulit dikenali, bukan lebih mudah. Lihat `Vault Conventions.md` untuk daftar lengkap istilah yang tidak diterjemahkan dan aturan tata bahasanya.

## Mulai Dari Mana

Jangan mulai dari sini dengan membaca semua isi vault sekaligus. Urutannya:

1. Selesaikan file ini.
2. Baca [[How To Read This Vault]] — cara membaca, ritme belajar, dan siklus status note.
3. Baca `Vault Conventions.md` untuk memahami frontmatter, tag, dan penamaan file.
4. Buka `01 Maps/Level 1 - Junior Path.md` dan mulai dari note pertama di sana.

## Peta Folder, Singkat

- `00 Start Here/` — kamu sedang di sini.
- `01 Maps/` — peta baca lintas domain, per level.
- `02 Templates/` — template mentah, hanya untuk referensi struktur.
- `03 My Zettels/` — **milikmu sepenuhnya**. Vault ini tidak pernah menulis apa pun di sini.
- `10` sampai `90` — folder domain, dari yang paling dekat dengan mesin (`10 Foundations`) sampai yang paling dekat dengan organisasi (`90 Architecture and Design`).
- `92 Tools/` — produk konkret (PostgreSQL, Kafka, Kubernetes, dst). Ini pengetahuan yang cepat basi, dipisahkan sengaja dari konsep yang bertahan lama.
- `94 Case Studies/` — keputusan nyata di bawah constraint nyata. Ini tempat konsep berubah menjadi judgement.
- `95 Projects/` — satu proyek capstone per level, semuanya dalam Go.
- `99 Glossary/` — satu istilah per note, untuk lookup cepat dan backlink.

## Connected Notes

- [[How To Read This Vault]] — langkah konkret setelah file ini: cara membaca, ritme, dan siklus status.
- [[Backend Engineering Map]] — peta lengkap seluruh vault dalam satu halaman.
- [[Level 1 - Junior Path]] — urutan baca konkret untuk mulai belajar hari ini.

## Catatan Saya

*Tulis di sini kenapa kamu memulai proyek ini, dan apa yang ingin kamu bisa jawab dengan percaya diri enam bulan dari sekarang.*
