---
title: The Go Type System
type: concept
level: junior
domain: go
status: unread
difficulty: 2
est_minutes: 15
prerequisites: []
next: ["[[Structs and Methods]]", "[[Interfaces and Implicit Satisfaction]]"]
tags: [backend, go]
created: 2026-07-26
---

## TL;DR

Go adalah bahasa **statically typed** — setiap tipe diketahui saat kompilasi, dan tidak ada konversi implisit antar tipe seperti di PHP, bahkan antara `int` dan `int64` sekalipun. Tipe bernama (`struct`, tipe yang didefinisikan lewat `type X Y`) dicocokkan secara **nominal** (harus persis sama jenisnya, atau dikonversi eksplisit), sementara `interface` dipenuhi secara **struktural** — tipe apa pun yang punya method yang sesuai otomatis memenuhi interface itu, tanpa deklarasi `implements` seperti di bahasa lain. Setiap tipe juga selalu punya **zero value** yang jelas (bukan `null`/`undefined`) — `0` untuk angka, `""` untuk string, `nil` untuk pointer/slice/map/channel/interface. Ini semua bukan detail sepele: ia mengubah cara berpikir soal bug, dari "ditemukan saat runtime" menjadi "ditolak saat kompilasi".

## The Problem

Bayangkan seorang engineer yang terbiasa dengan PHP menulis kode Go pertamanya, dan menulis sesuatu seperti menjumlahkan dua nilai bertipe berbeda dengan asumsi Go akan otomatis mengonversinya seperti PHP melakukan type juggling pada `"5" + 5`. Compiler Go menolak ini mentah-mentah — bahkan menjumlahkan `int` dengan `int64` pun ditolak tanpa konversi eksplisit, meski keduanya "hanya angka". Awalnya ini terasa seperti bahasa yang terlalu cerewet.

Masalah yang lebih halus muncul di tempat lain: engineer yang sama menulis function yang menerima `struct` sebagai parameter, mengubah salah satu field-nya di dalam function, lalu bingung kenapa perubahan itu tidak terlihat di pemanggil — persis seperti bug yang seharusnya tidak mungkin terjadi kalau berpikir dengan model PHP, di mana object selalu diakses lewat semacam handle/referensi. Di Go, `struct` di-passing **by value** secara default — function menerima **salinan**, bukan struct aslinya, kecuali secara eksplisit memakai pointer. Tanpa memahami perbedaan mendasar ini, bug seperti ini akan terus muncul dalam bentuk berbeda-beda.

## Intuition

Bayangkan sistem tipe Go seperti **petugas bea cukai yang sangat ketat** — tidak ada barang (value) yang boleh melewati batas tanpa dokumen yang jelas (konversi tipe eksplisit), berapa pun miripnya barang itu terlihat dari luar. Ini kontras dengan PHP yang lebih seperti perbatasan yang sangat longgar — hampir semua hal diterima dan "diterjemahkan" otomatis di tempat.

Analogi ini bocor di satu hal penting: `interface` di Go tidak mengikuti "aturan bea cukai" yang sama dengan tipe bernama lainnya. Untuk `interface`, Go memakai aturan yang jauh lebih longgar — **struktural**, bukan nominal: siapa pun yang punya "surat-surat" (method) yang sesuai boleh lewat, tanpa perlu mendaftar terlebih dulu (tidak ada kata kunci `implements`). Jadi sebenarnya ada dua "kebijakan perbatasan" berbeda dalam satu sistem tipe — ketat untuk tipe konkret, longgar untuk interface — dan detail ini dibahas penuh di [[Interfaces and Implicit Satisfaction]].

## How It Works

Beberapa properti inti sistem tipe Go yang membentuk semua yang lain:

- **Static typing** — tipe setiap variable ditentukan saat kompilasi, tidak berubah selama program berjalan.
- **Tidak ada konversi implisit** — mengubah `int` jadi `float64`, atau `MyInt` (tipe custom berbasis `int`) jadi `int` biasa, harus dilakukan eksplisit lewat konversi (`float64(x)`, `int(myInt)`).
- **Nominal typing untuk tipe bernama** — dua struct dengan field yang identik tapi nama tipe berbeda tetap dianggap tipe yang berbeda.
- **Structural typing untuk interface** — sebuah tipe memenuhi interface hanya dengan memiliki method yang sesuai, tanpa deklarasi eksplisit.
- **Zero value untuk setiap tipe** — variable yang dideklarasikan tanpa nilai awal otomatis mendapat zero value tipenya (`0`, `""`, `false`, `nil`), bukan keadaan "tidak terdefinisi".

```go
type Celsius float64
type Fahrenheit float64

func contoh() {
    var c Celsius = 100
    var f Fahrenheit = 212

    // c + f TIDAK BISA dikompilasi meski keduanya berbasis float64 —
    // ini SENGAJA, supaya kamu tidak pernah tidak sadar mencampur
    // satuan yang berbeda secara implisit.
    // total := c + f // compile error: mismatched types

    total := float64(c) + float64(f) // harus eksplisit
    _ = total
}
```

## Under The Hood

Zero value bukan sekadar kenyamanan sintaks — ia adalah keputusan desain yang menghilangkan satu kelas bug penuh (null reference/undefined behavior) dengan cara memastikan **setiap** value, bahkan yang belum diinisialisasi secara eksplisit, tetap punya bentuk yang valid dan bisa dipakai. Trade-off-nya: zero value bisa **ambigu** secara makna. `var jumlahRetry int` bernilai `0` secara default — tapi apakah itu berarti "belum ada retry" atau "memang sengaja diset nol"? Go tidak membedakan keduanya secara otomatis; kalau perbedaan itu penting, harus dimodelkan eksplisit (misalnya lewat pointer `*int`, atau pola dua nilai `(T, bool)` seperti pada map lookup).

## In Go

```go
package main

import "fmt"

type Dokumen struct {
    ID     string
    Status string
}

// Menerima SALINAN Dokumen — mengubah field di sini TIDAK memengaruhi
// struct asli di pemanggil.
func ubahStatusSalah(d Dokumen) {
    d.Status = "diverifikasi" // hanya mengubah salinan lokal
}

// Menerima POINTER ke Dokumen — mengubah field di sini MEMENGARUHI
// struct asli di pemanggil, karena keduanya menunjuk memori yang sama.
func ubahStatusBenar(d *Dokumen) {
    d.Status = "diverifikasi"
}

func main() {
    doc := Dokumen{ID: "A-001", Status: "draft"}

    ubahStatusSalah(doc)
    fmt.Println(doc.Status) // tetap "draft" — mengejutkan bagi yang terbiasa PHP

    ubahStatusBenar(&doc)
    fmt.Println(doc.Status) // sekarang "diverifikasi"
}
```

Perbedaan `ubahStatusSalah` dan `ubahStatusBenar` bukan soal gaya penulisan — ini adalah konsekuensi langsung dari value semantics Go: `struct` disalin penuh saat di-passing kecuali kamu secara eksplisit memakai pointer. Ini kebalikan dari model PHP, di mana object (bukan array/primitif) selalu diakses lewat semacam handle, sehingga perubahan di dalam function selalu "terlihat" oleh pemanggil.

## In His Stack

**PHP (Yii1/Yii2)** memakai typing yang jauh lebih longgar: variable bisa berubah tipe kapan saja, perbandingan `==` melakukan type juggling yang terkenal jadi sumber bug (`"0" == false` bernilai `true` dalam banyak versi PHP), dan object model PHP selalu bersifat seperti reference — mengubah object di dalam function selalu terlihat oleh pemanggil. Kedua kebiasaan ini adalah sumber bug paling umum saat engineer PHP pindah ke Go: mengharapkan konversi otomatis yang tidak pernah terjadi, dan mengharapkan struct berperilaku seperti PHP object padahal defaultnya adalah value semantics.

## Trade-offs and When Not To Use It

Static typing yang ketat menambah verbosity — setiap konversi harus eksplisit, setiap tipe harus dipikirkan di muka. Ini bukan trade-off yang bisa "dimatikan" karena ini adalah bahasa itu sendiri, bukan fitur opsional. Yang perlu dipahami adalah kapan memakai value semantics (struct kecil, immutable, aman dibagi lewat salinan) vs pointer semantics (struct besar yang mahal disalin, atau memang butuh perubahan terlihat oleh pemanggil) — keputusan ini dibahas lebih dalam di [[Pointer vs Value Receivers]].

## Common Mistakes

> [!warning] Jebakan
> Mengharapkan konversi implisit antara tipe numerik yang "terlihat sama" (`int` dan `int64`, atau dua tipe custom yang sama-sama berbasis `float64`). Go menolak semuanya tanpa konversi eksplisit — ini desain sengaja, bukan kekurangan.

> [!warning] Jebakan
> Menyalahartikan zero value sebagai "tidak ada nilai". `0`, `""`, dan `false` adalah nilai yang sah dan tidak bisa dibedakan dari "belum diisi" kecuali kamu memodelkannya eksplisit (pointer, atau pola `(value, ok)`).

> [!warning] Jebakan
> Mem-passing `struct` besar sebagai value ke function yang sering dipanggil, tanpa sadar setiap pemanggilan menyalin seluruh isi struct itu — baik dari sisi performa (lihat [[../50 Concurrency and Performance/Reducing Allocations|Reducing Allocations]]) maupun dari sisi mengharapkan perubahan yang tidak akan pernah terlihat pemanggil.

## Exercises

1. Kenapa `int` dan `int64` di Go tidak bisa dijumlahkan langsung tanpa konversi eksplisit, padahal keduanya "angka"?
2. Jelaskan bedanya bagaimana `struct` dan `interface` masing-masing dicocokkan tipenya di Go.
3. Tulis dua versi function yang menerima `Dokumen` — satu lewat value, satu lewat pointer — dan jelaskan kapan masing-masing tepat dipakai.
4. Desain terbuka: sebuah tim yang baru pindah dari PHP ke Go terus-menerus menulis bug di mana perubahan pada struct di dalam function tidak "terlihat" oleh kode pemanggil. Rancang panduan singkat (aturan praktis, bukan sekadar "gunakan pointer selalu") untuk membantu tim ini memutuskan kapan memakai value dan kapan memakai pointer receiver/parameter, termasuk kapan value semantics justru diinginkan.

> [!success]- Kunci jawaban
> Aturan praktis yang bisa dipakai: pakai **pointer** ketika function memang dimaksudkan untuk mengubah state struct yang dilihat pemanggil (semantik "mutasi"), atau ketika struct-nya besar dan menyalinnya berulang kali mahal secara performa. Pakai **value** ketika struct kecil (beberapa field sederhana), dan terutama ketika kamu justru **ingin** isolasi — memberi pemanggil jaminan bahwa function ini tidak akan mengubah data aslinya, membuat kode lebih mudah dinalar (tidak ada efek samping tersembunyi). Untuk tim yang baru pindah dari PHP, tekankan bahwa "PHP object selalu seperti reference" tidak berlaku di Go untuk struct — kebiasaan lama ini harus sengaja dipatahkan, bukan diasumsikan berlaku sama.

## Self-Check

- Kenapa Go menolak konversi implisit antara `int` dan `int64`?
- Apa perbedaan cara Go mencocokkan tipe untuk `struct` dibanding untuk `interface`?
- Apa itu zero value, dan kenapa ia bisa ambigu maknanya?
- Kenapa mengubah field struct di dalam function yang menerima struct sebagai value tidak terlihat oleh pemanggil?

## Connected Notes

- [[Structs and Methods]] — kelanjutan langsung: bagaimana struct (tipe bernama paling umum di Go) mendapat perilaku lewat method.
- [[Pointer vs Value Receivers]] — pembahasan penuh kapan memilih pointer vs value, yang disinggung sekilas di note ini.
- [[Interfaces and Implicit Satisfaction]] — sisi "longgar" dari sistem tipe Go yang dikontraskan dengan nominal typing di note ini.
- [[Slice Internals]] — salah satu tipe yang perilakunya sering disalahpahami justru karena tidak sepenuhnya value maupun reference semantics secara naif.
- [[../90 Architecture and Design/Manual Dependency Injection in Go|Manual Dependency Injection in Go]] — desain berbasis interface yang memanfaatkan langsung structural typing yang dijelaskan di note ini.

## Further Reading

- Dokumentasi resmi *The Go Programming Language Specification* (go.dev/ref/spec), bagian "Types" — sumber kebenaran paling akurat untuk aturan konversi tipe.

## Catatan Saya

*Tulis di sini kebiasaan dari PHP yang paling sering membuatmu salah menulis kode Go di awal, dan bagaimana akhirnya terbiasa.*
