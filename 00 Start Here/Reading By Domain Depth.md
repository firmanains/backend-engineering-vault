---
title: Reading By Domain Depth
type: concept
level: junior
domain: foundations
status: mastered
difficulty: 1
est_minutes: 6
prerequisites: ["[[How To Read This Vault]]"]
next: []
tags: [backend, meta]
created: 2026-07-29
---

## Kenapa File Ini Ada

[[How To Read This Vault]] menjelaskan tiga cara membaca vault ini — linear (ikuti `01 Maps/Level N - ...md`), graph-driven, dan problem-driven. Ketiganya masih berlaku. Tapi cara vault ini **ditulis** berubah setelah Level 1 (seluruh 89 note junior) selesai: alih-alih menulis lebar dulu (semua domain di level intermediate, baru semua domain di level senior), penulisan sekarang berjalan **dalam per domain** — satu folder domain ditulis sampai kedalaman penuhnya (junior yang sudah ada → seluruh intermediate → seluruh senior) sebelum domain berikutnya disentuh. Alasannya tercatat di `CLAUDE.md` §14: pembaca secara eksplisit lebih suka menuntaskan satu topik sampai benar-benar dikuasai, daripada menyapu lebar di satu level lalu mengulang proses yang sama di level berikutnya.

File ini menjelaskan cara membaca yang cocok dengan pola penulisan itu — bukan pengganti [[How To Read This Vault]], tapi pelengkap yang menjawab pertanyaan konkret: "kalau vault ditulis per domain, gimana cara bacanya?"

## Cara Baca Lama vs Cara Baca Ini

**Cara lama (`01 Maps/Level N - ...md`)**: baca seluruh note junior lintas semua domain, baru seluruh note intermediate lintas semua domain, baru seluruh note senior. Cocok kalau seluruh vault (atau seluruh satu level) sudah selesai ditulis, karena setiap lompatan antar domain dijamin punya tujuan yang sudah ada isinya.

**Cara ini (per domain, habis-habisan)**: pilih **satu** folder domain (misalnya `40 Databases/`), baca `_Overview.md`-nya, lalu ikuti reading order di sana dari atas ke bawah — junior dulu (kemungkinan besar sudah kamu baca), lalu terus ke intermediate, lalu senior kalau domain itu punya tingkat senior sendiri — sebelum pindah ke domain lain sama sekali. Cocok justru **karena** vault ditulis dengan urutan yang sama: begitu kamu selesai membaca satu domain penuh, penulisannya juga baru saja selesai penuh untuk domain itu — tidak ada bagian yang "hilang" di tengah jalan.

## Langkah Konkret

1. Buka `00 Start Here/Backlog.md`, bagian "Titik Lanjutan (Resume Point)" — di situ tercatat domain mana yang sudah selesai penuh, dan domain mana yang sedang dikerjakan sekarang. Baca domain yang **sudah selesai penuh** dulu; domain yang masih setengah jalan berisiko punya link ke note yang belum ada.
2. Buka `_Overview.md` domain itu (misalnya `40 Databases/_Overview.md`). Bagian "Reading Order" di situ sudah mengelompokkan note per level (Junior / Intermediate / Senior) secara berurutan — ikuti persis urutan itu, dari nomor 1 sampai selesai, tanpa loncat.
3. Kalau domain itu **tidak** punya bagian "Senior" di `_Overview.md`-nya (contoh: `40 Databases` sengaja hanya junior+intermediate), itu bukan berarti topiknya berhenti di situ — cek bagian "Bagaimana Domain Ini Terhubung Dengan Domain Lain" di `_Overview.md` yang sama, biasanya sudah menunjuk domain lain (paling sering `60 Distributed Systems`) tempat kelanjutan level seniornya berada. Baca domain itu belakangan, khusus untuk note yang memang disebut sebagai kelanjutan topik ini — bukan seluruh domain itu.
4. Setelah satu domain tuntas dibaca dari junior sampai kedalaman maksimalnya, **baru** pindah ke domain berikutnya. Domain mana yang berikutnya mengikuti urutan yang sama seperti urutan penulisan — lihat aturan urutan domain di `CLAUDE.md` §14 (atau tanyakan langsung kalau kamu memakai asisten yang sama untuk melanjutkan menulis; ia akan tahu domain apa yang sedang/akan dikerjakan berikutnya).

## Kalau Kamu Membaca Lebih Cepat Dari Penulisannya

Wajar terjadi: kamu sudah menuntaskan satu domain, tapi domain berikutnya di urutan belum selesai ditulis. Dua pilihan yang sama-sama valid:

- **Tunggu** — minta domain berikutnya diselesaikan dulu sebelum melanjutkan baca, supaya pengalamannya tetap "satu topik penuh, tanpa jeda".
- **Beralih ke graph-driven/problem-driven sementara** — sambil menunggu, pakai dua cara baca lain dari [[How To Read This Vault]]: ikuti link yang membuatmu penasaran dari note yang sudah ada, atau cari note spesifik kalau ada masalah nyata di kerjaan yang butuh dijawab sekarang. Note yang dibaca dengan cara ini tetap ditandai statusnya seperti biasa di [[Progress Tracker]].

Yang sebaiknya dihindari: memaksa lompat ke domain yang belum ditulis sama sekali hanya karena urutannya "berikutnya" — kamu hanya akan menemukan folder kosong atau `_Overview.md` yang seluruh isinya masih daftar rencana, bukan note sungguhan.

## Kalau Kamu Ingin Melihat Bentuk Lengkap Sebuah Topik Sekaligus

Karena domain folder sudah dirancang mencampur junior/intermediate/senior dalam satu tempat (lihat CLAUDE.md §5), tidak perlu berpindah folder sama sekali untuk melihat perjalanan lengkap satu topik dari dasar sampai dalam — cukup gulir `_Overview.md` domain itu dari atas ke bawah. Bandingkan ini dengan cara baca lama yang mengharuskan membuka tiga file peta berbeda (`Level 1`, `Level 2`, `Level 3`) dan mencari-cari bagian mana di masing-masing yang membahas domain yang sama.

## Connected Notes

- [[How To Read This Vault]] — tiga cara baca dasar (linear, graph-driven, problem-driven) yang tetap berlaku; file ini hanya menambahkan cara keempat yang cocok dengan pola penulisan per domain.
- [[Backlog]] — sumber kebenaran untuk domain mana yang sudah selesai penuh dan mana yang sedang dikerjakan, dicek di langkah 1 di atas.
- [[Progress Tracker]] — tempat menandai status baca (`unread`/`learning`/`understood`/`mastered`) tidak berubah, terlepas dari cara baca mana yang dipakai.
- [[Vault Manifest]] — daftar lengkap note per domain, termasuk yang belum ditulis; dipakai saat mengecek apakah sebuah domain benar-benar sudah tuntas atau masih ada sisa.

## Catatan Saya

*Tulis di sini kalau kamu menemukan urutan baca yang menurutmu lebih masuk akal dari yang disarankan file ini, untuk domain tertentu.*
