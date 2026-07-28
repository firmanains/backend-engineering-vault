---
title: Progress Tracker
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

Halaman ini tidak dibaca seperti note lain — ia dibuka sekali seminggu sebagai bagian dari ritual mingguan di [[How To Read This Vault]]. Naikkan status dengan jujur; lihat definisi tiap tahap di note itu sebelum mencentang apa pun.

## Ritual Mingguan (checklist)

- [ ] Buka tabel di bawah, naikkan `status` note yang sudah benar-benar dikuasai minggu ini.
- [ ] Pilih 10–15 note untuk minggu depan dari `01 Maps/Level N - ...md` yang sedang dijalani.
- [ ] Buka graph view Obsidian, filter tag level yang sedang dijalani, cari cluster terisolasi.
- [ ] Sekali sebulan: tinjau `## Catatan Saya` di note yang sudah ditulisi, promosikan yang layak ke `03 My Zettels/`.

## Ringkasan per Level

```dataview
TABLE
  length(filter(rows.status, (s) => s = "mastered")) AS Mastered,
  length(filter(rows.status, (s) => s = "understood")) AS Understood,
  length(filter(rows.status, (s) => s = "learning")) AS Learning,
  length(filter(rows.status, (s) => s = "unread")) AS Unread,
  length(rows) AS Total
FROM #backend
WHERE type = "concept"
GROUP BY level
```

## Note Yang Sedang Dipelajari (`learning`)

```dataview
TABLE level, domain, difficulty
FROM #backend
WHERE status = "learning"
SORT level ASC, difficulty ASC
```

## Junior — Belum Disentuh

```dataview
TABLE domain, difficulty, est_minutes
FROM #backend
WHERE level = "junior" AND status = "unread"
SORT domain ASC, difficulty ASC
```

## Semua Note Level Junior, Diurutkan Sulit

```dataview
TABLE domain, status, difficulty
FROM #backend
WHERE level = "junior"
SORT difficulty ASC
```

## Tool Notes — Status per Depth Tier

```dataview
TABLE depth, volatility, status
FROM #backend
WHERE type = "tool"
SORT depth ASC
```

## Case Studies — Belum Dibaca

```dataview
TABLE concepts, status
FROM #backend
WHERE type = "case" AND status = "unread"
```

## Connected Notes

- [[How To Read This Vault]] — definisi setiap nilai `status` dan ritme review yang dijalankan lewat halaman ini.
- [[Vault Conventions]] — field frontmatter yang menjadi sumber semua kolom di atas.
- [[Backlog]] — note yang sudah ditautkan tapi belum ditulis; bukan bagian dari tracker ini karena belum punya frontmatter.

## Catatan Saya

*Tulis di sini pola belajar yang kamu sadari dari minggu ke minggu — misalnya domain mana yang selalu tertunda.*
