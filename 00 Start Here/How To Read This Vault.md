---
title: How To Read This Vault
type: concept
level: junior
domain: foundations
status: mastered
difficulty: 1
est_minutes: 12
prerequisites: ["[[Read Me First]]"]
next: ["[[Vault Conventions]]"]
tags: [backend, meta]
created: 2026-07-26
---

## Jam Pertama

Buka empat file ini, dalam urutan ini, dan tidak lebih:

1. `00 Start Here/Read Me First.md` — sudah kamu baca.
2. `01 Maps/Level 1 - Junior Path.md` — daftar note pertama yang akan kamu baca, sudah diurutkan.
3. `_Overview.md` dari domain pertama di jalur itu (biasanya `10 Foundations/_Overview.md`).
4. Note pertama di folder itu.

Jangan buka folder lain di jam pertama. Rasa penasaran untuk menjelajah seluruh vault itu wajar, tapi tunda — vault ini akan tetap di sini bulan depan, dan membaca satu note sampai tuntas lebih berharga daripada membuka sepuluh tab.

## Tiga Cara Membaca, dan Kapan Memakainya

**Linear** — mengikuti reading order di `01 Maps/Level N - ...md` dari atas ke bawah. Ini adalah default, dan inilah yang membangun fondasi. Pakai cara ini untuk 90% waktu belajarmu, terutama di level junior.

**Graph-driven** — mengikuti sebuah `[[wikilink]]` begitu ia membuat penasaran, lalu kembali ke note asal. Ini cara yang baik untuk **konsolidasi** — setelah kamu paham satu topik, menjelajahi koneksinya memperkuat mental model. Ini cara yang buruk untuk first pass, karena kamu akan berhenti di tengah jalan dengan sepuluh note setengah terbaca dan tidak satu pun benar-benar dipahami.

**Problem-driven** — ketika sesuatu rusak di kantor, cari di vault, baca hanya bagian yang menjawab masalah itu, lalu tandai note itu `learning`. Ini cara vault ini tetap hidup setelah first pass selesai — pengetahuan yang dipakai untuk memecahkan masalah nyata menempel jauh lebih kuat daripada yang dibaca tanpa tekanan.

## Bagaimana Concept, Tool, dan Case Study Saling Melengkapi

Tiga jenis note ini membentuk satu siklus, dan melewatkan salah satunya menghasilkan pemahaman yang timpang:

- **Concept note** menjawab *kenapa* — masalah apa yang membuat ide ini perlu ada, dan bagaimana mekanismenya bekerja secara umum, lepas dari produk mana pun.
- **Tool note** menjawab *bagaimana produk tertentu mengimplementasikan ide itu* — konfigurasi, operasional, debugging.
- **Case study** melatih *memilih* — menerapkan beberapa concept sekaligus di bawah constraint yang tidak kamu pilih sendiri: deadline, partner yang tidak fleksibel, tim yang tidak punya skill untuk mengoperasikan solusi yang elegan.

Seseorang yang membaca tool note tanpa concept note-nya akan bisa mengonfigurasi Kafka, tapi tidak bisa menjelaskan kapan queue justru jawaban yang salah. Selalu baca concept dulu, baru tool, baru case study yang memakainya.

## Siklus Status

Setiap note punya field `status` di frontmatter, dengan bar yang jelas untuk tiap tahap:

- `unread` — belum dibuka sama sekali.
- `learning` — sudah dibaca sekali, masih samar. Kamu bisa mengenali istilahnya tapi belum bisa menjelaskan ulang tanpa membuka note.
- `understood` — bisa menjelaskan ulang idenya dengan lisan, tanpa melihat note, **termasuk minimal satu trade-off**. Ini bar minimum untuk bilang "aku paham ini."
- `mastered` — sudah menyelesaikan exercise open-ended di note itu, dan bisa mengajarkannya ke junior engineer. Ini bar untuk topik yang benar-benar jadi milikmu.

Jangan menaikkan status hanya karena sudah membaca sampai akhir. Naikkan status karena kamu sudah mengujinya — dengan menjelaskan ke orang lain, atau ke dirimu sendiri dengan lantang.

## Ritme Review

Recall mengalahkan re-reading. Karena itu setiap note diakhiri dengan bagian **Self-Check** — bukan untuk dibaca, tapi untuk dijawab dari ingatan. Jadwal review yang disarankan untuk note baru:

- Hari 1 — baca penuh, kerjakan exercises.
- Hari 3 — jawab Self-Check tanpa membuka note. Kalau macet, baru buka.
- Hari 7 — ulangi.
- Hari 21 — ulangi sekali lagi. Kalau di titik ini kamu masih lancar menjawab semua Self-Check dan bisa menyebutkan trade-off-nya, note itu layak naik ke `understood` atau `mastered`.

## Pace

Maksimal **dua sampai tiga note baru per hari**. Ini batas, bukan target yang harus selalu dikejar. Satu note yang dibaca dalam-dalam dengan exercise-nya benar-benar dikerjakan mengalahkan lima note yang hanya dilewati matanya. Vault ini akan tetap ada bulan depan — tidak ada alasan untuk terburu-buru sampai pemahamannya dangkal.

## Ritual Mingguan

Sekali seminggu:

1. Buka `Progress Tracker.md`. Naikkan status dengan jujur — jangan naikkan `understood` menjadi `mastered` hanya karena minggu itu terasa produktif.
2. Pilih note untuk minggu berikutnya dari `01 Maps/Level N - ...md` yang sedang kamu jalani.
3. Buka graph view, filter ke level yang sedang kamu baca, dan cari cluster yang terisolasi — node yang hampir tidak terhubung ke apa pun. Itu biasanya tanda ada pemahaman yang bolong, bukan sekadar note yang belum ditulis.
4. Sekali sebulan, tinjau ulang bagian `## Catatan Saya` di note-note yang sudah kamu tulisi. Kalau ada satu baris yang berkembang menjadi klaim yang bisa kamu bela sendiri — bukan sekadar pertanyaan — pindahkan (promote) ke `03 My Zettels/` sebagai zettel baru.

## Aturan `## Catatan Saya`

Setiap note diakhiri dengan bagian `## Catatan Saya` yang sengaja dikosongkan. Note itu **belum benar-benar jadi milikmu** sampai sesuatu tertulis di sana — sebuah pertanyaan yang belum terjawab, tautan ke kode nyata di kantor yang memakai (atau melanggar) konsep itu, atau catatan tentang bagian penjelasan yang tidak nyambung untukmu. Inilah yang membedakan vault ini dari sekadar buku teks yang diunduh: buku teks tidak pernah berubah karena dibaca; vault ini seharusnya berubah.

## Connected Notes

- [[Read Me First]] — konteks awal tentang tujuan vault ini secara keseluruhan.
- [[Vault Conventions]] — detail frontmatter, tag, dan penamaan yang membuat semua query Dataview di vault ini bekerja.
- [[Progress Tracker]] — tempat ritual mingguan di atas benar-benar dijalankan.
- [[Backend Engineering Map]] — peta lengkap untuk mode graph-driven ketika kamu sudah siap menjelajah.

## Catatan Saya

*Tulis di sini kalau ritme dua-tiga note per hari ternyata tidak cocok denganmu, dan apa yang kamu ganti.*
