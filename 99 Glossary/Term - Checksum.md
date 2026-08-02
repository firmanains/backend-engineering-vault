---
title: Term - Checksum
type: term
level: junior
domain: security
status: unread
difficulty: 1
est_minutes: 3
prerequisites: []
next: []
tags: [backend, security]
created: 2026-08-02
---

**Checksum** adalah nilai kecil (biasanya hasil fungsi hash) yang dihitung dari sebuah data untuk memverifikasi integritasnya — kalau data berubah sedikit pun (rusak selama transfer, korup di penyimpanan), checksum yang dihitung ulang dari data itu akan berbeda dari checksum asli, memberi sinyal jelas bahwa data tidak lagi utuh seperti aslinya.

Ini kenapa istilah ini penting dipahami: checksum adalah lapisan verifikasi murah yang sering luput ditambahkan pada transfer file besar atau integrasi lintas sistem — tanpa itu, kerusakan data selama transfer (jaringan yang tidak sempurna) bisa tidak terdeteksi sampai data itu benar-benar dipakai dan menghasilkan hasil yang salah.

## Muncul Di

- [[../94 Case Studies/Case - Sending PDFs Through a JSON API|Case - Sending PDFs Through a JSON API]] — checksum sebagai jaminan integritas minimal untuk transfer file lintas sistem.
- [[../94 Case Studies/Case - Daily CSV Over SFTP in 2026|Case - Daily CSV Over SFTP in 2026]] — checksum dan penanda selesai sebagai mekanisme verifikasi transfer file.

## Catatan Saya

*Kosong — diisi pembaca.*
