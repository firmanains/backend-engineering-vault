---
title: My Zettels Overview
type: concept
level: junior
domain: foundations
status: mastered
difficulty: 1
est_minutes: 5
prerequisites: ["[[How To Read This Vault]]"]
next: []
tags: [backend, meta]
created: 2026-07-26
---

## Folder Ini Berbeda Dari Semua Folder Lain

Setiap folder lain di vault ini ditulis lebih dulu, sebelum kamu membacanya. Folder ini kebalikannya: **kosong sampai kamu sendiri yang mengisinya**. Tidak ada AI, template, atau proses otomatis yang pernah menulis di sini — kalau suatu hari folder ini berisi zettel yang tidak kamu ingat menulisnya, itu tanda ada yang salah dengan proses penulisan vault ini.

Ini adalah benih dari **Zettelkasten riset** yang akan kamu butuhkan untuk kerja pascasarjana di distributed systems. Bedanya dengan note lain di vault: note di `10` sampai `90` menjelaskan apa yang sudah diketahui dunia. Zettel di sini adalah klaim yang **kamu sendiri** bisa bela.

## Kapan Sesuatu Layak Dipromosikan Ke Sini

Setiap note di vault punya bagian `## Catatan Saya` di baris paling bawah — tempat kosong untuk pertanyaan, contoh nyata dari kerjaanmu, atau bagian yang tidak nyambung. Sebagian besar isi bagian itu akan tetap jadi catatan kecil selamanya, dan itu wajar.

Tapi sesekali, satu baris di `## Catatan Saya` akan berkembang jadi lebih dari sekadar pertanyaan — ia jadi **klaim yang bisa kamu pertahankan**. Bedanya:

- *Pertanyaan* — "Kenapa isolation level default MariaDB beda dari PostgreSQL?" → tetap di `## Catatan Saya`, cari jawabannya, mungkin tambahkan sebagai catatan singkat di note yang sama.
- *Klaim yang bisa dibela* — "Di sistem legal-services kami, `REPEATABLE READ` MariaDB sebenarnya menyembunyikan bug konkurensi yang baru kelihatan setelah load naik 5x, karena gap lock membuat kami mengira race condition sudah aman padahal hanya kebetulan tidak pernah teruji di volume rendah." → ini sudah jadi ide orisinal dengan observasi spesifik, bukan sekadar pertanyaan. **Ini layak jadi zettel.**

Cek ini setiap bulan sebagai bagian dari ritual mingguan (lihat [[How To Read This Vault]]).

## Aturan Menulis Zettel

- Bebas menautkan ke mana saja di vault — zettel di sini justru diharapkan menautkan lintas domain, karena ide orisinal biasanya lahir dari menghubungkan dua hal yang di vault utama sengaja dipisah demi kejelasan.
- Tidak perlu mengikuti template concept note. Tulis seatomik mungkin — satu zettel, satu klaim.
- Frontmatter tetap dianjurkan (minimal `title`, `type: term` atau biarkan kosong sesuai kebutuhanmu, `tags`, `created`) supaya tetap searchable, tapi tidak wajib mengikuti skema penuh di `Vault Conventions.md`.

## Connected Notes

- [[How To Read This Vault]] — aturan `## Catatan Saya` dan ritme review bulanan yang menjadi sumber promosi ke folder ini.
- [[Read Me First]] — ambisi gelar master di distributed systems yang menjadi alasan folder ini ada sejak awal.
- [[Vault Manifest]] — daftar seluruh note lain di vault; folder ini sengaja tidak masuk manifest karena isinya tidak direncanakan, hanya ditunggu.

## Catatan Saya

*(Folder ini, dan bagian ini secara khusus, tidak pernah diisi oleh proses penulisan vault. Kosong secara permanen sampai kamu mengisinya sendiri.)*
