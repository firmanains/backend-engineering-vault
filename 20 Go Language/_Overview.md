---
title: Go Language Overview
type: concept
level: junior
domain: go
status: mastered
difficulty: 1
est_minutes: 8
prerequisites: ["[[../10 Foundations/_Overview|Foundations Overview]]"]
next: ["[[The Go Type System]]"]
tags: [backend, go]
created: 2026-07-26
---

## Apa Ini

Domain ini adalah bahasa Go itu sendiri: sistem tipenya, struct dan method, interface dan bagaimana ia dipenuhi secara implisit, internal slice dan map, filosofi error handling, packages dan modules, testing, sampai fitur lanjutan seperti generics dan reflection. Ini murni tentang **bagaimana menulis Go yang idiomatic** — bukan concurrency (lihat `50 Concurrency and Performance`) dan bukan bagaimana Go dipakai membangun HTTP server (lihat `30 APIs and Web`).

Pemisahan ini disengaja: mekanik bahasa (bab ini) berbeda dari cara memakai bahasa itu untuk concurrency (`50`) dan untuk web (`30`). Kalau ketiganya digabung, folder ini akan membengkak jadi campur aduk topik yang seharusnya masing-masing berdiri sendiri.

## Kenapa Ini Penting

"Bisa menulis Go" dan "menguasai Go" adalah dua hal berbeda. Engineer yang datang dari PHP sering menulis Go dengan pola pikir PHP — memakai `interface{}` di mana generics lebih tepat, menyalahgunakan pointer receiver, mengabaikan bahwa slice yang di-pass ke function bisa mengubah data pemanggilnya. Mastery sungguhan dalam Go — salah satu dari tiga tujuan utama vault ini — dimulai persis di domain ini.

## Reading Order

### Junior — Go core dan testing

1. [[The Go Type System]] — dasar untuk semua yang berikutnya.
2. [[Structs and Methods]] — unit dasar pemodelan data di Go.
3. [[Pointer vs Value Receivers]] — keputusan yang salah di sini menyebabkan bug aliasing yang sulit dilacak.
4. [[Interfaces and Implicit Satisfaction]] — kenapa Go tidak butuh kata kunci `implements`, dan apa konsekuensinya untuk desain API.
5. [[Slice Internals]] — length, capacity, dan aliasing bug klasik saat slice dioper sebagai argumen.
6. [[Map Internals]] — bagaimana map bekerja di bawah, dan kenapa iterasi map tidak pernah berurutan.
7. [[Defer, Panic, and Recover]] — mekanisme cleanup dan error handling terakhir sebelum goroutine mati.
8. [[Errors as Values]] — filosofi error handling Go yang paling sering disalahpahami oleh pendatang dari bahasa lain.
9. [[Error Wrapping]] — `fmt.Errorf("...: %w", err)` dan kenapa itu penting untuk debugging di production.
10. [[Packages and Modules]] — unit organisasi kode dan dependency management.
11. [[The Go Toolchain]] — `go build`, `go test`, `go vet`, dan alat lain yang dipakai setiap hari.
12. [[Struct Tags and JSON Marshalling]] — bagaimana data Go berubah jadi payload API dan sebaliknya.
13. [[Table-Driven Tests]] — pola testing paling idiomatic di Go.
14. [[Stdlib Testing vs Testify]] — kapan assertion library membantu, kapan ia menyembunyikan informasi.
15. [[Mocking Through Interfaces]] — cara Go melakukan dependency injection untuk testing tanpa framework mocking berat.
16. [[Testing HTTP Handlers with httptest]] — menguji handler tanpa server sungguhan berjalan.
17. [[Meaningful Test Coverage]] — kenapa angka coverage 100% bisa berbahaya, bukan aman.

### Intermediate — Advanced Go

18. [[Generics]] — abstraksi tipe tanpa `interface{}` dan type assertion di mana-mana.
19. [[Reflection and Its Costs]] — kekuatan yang datang dengan biaya performa dan keamanan tipe.
20. [[Embedding]] — komposisi ala Go, dan bedanya dari inheritance.
21. [[Functional Options]] — pola konfigurasi API yang bisa berkembang tanpa breaking change.
22. [[Designing Stable Library APIs]] — menulis package yang aman dipakai orang lain bertahun-tahun.
23. [[Sentinel Errors vs Error Types]] — dua cara membedakan jenis error, dan kapan masing-masing tepat.

> [!info] Ke mana perginya "Go concurrency" dan "Go runtime"?
> Kurikulum sumber menyebut goroutine, channel, `sync`, GMP scheduler, GC, dan `pprof` sebagai bagian dari "Go". Di vault ini, semua itu ditempatkan di [[../50 Concurrency and Performance/_Overview|Concurrency and Performance Overview]] mengikuti deskripsi folder itu sendiri. Lihat `Curriculum Changelog.md` untuk alasan lengkap keputusan ini.

## Bagaimana Domain Ini Terhubung Dengan Domain Lain

- [[../10 Foundations/_Overview|Foundations Overview]] — model memori (stack/heap) dan I/O di domain itu menjelaskan kenapa Go mendesain slice, map, dan GC seperti sekarang.
- [[../50 Concurrency and Performance/_Overview|Concurrency and Performance Overview]] — melanjutkan bahasa ini ke goroutine, channel, dan runtime.
- [[../30 APIs and Web/_Overview|APIs and Web Overview]] — struct tag dan JSON marshalling di domain ini langsung dipakai untuk desain payload API.
- [[../92 Tools/_Overview|Tools Overview]] — tidak ada tool note untuk Go sendiri (Go adalah bahasa, bukan produk perishable), tapi hampir semua contoh kode di tool note memakai idiom dari domain ini.

## Progress

```dataview
TABLE status, level, difficulty
FROM #go
WHERE type = "concept"
SORT level ASC, difficulty ASC
```

## Catatan Saya

*Tulis di sini kebiasaan menulis kode dari PHP yang ternyata jadi anti-pattern di Go.*
