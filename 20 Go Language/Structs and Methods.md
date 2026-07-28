---
title: Structs and Methods
type: concept
level: junior
domain: go
status: unread
difficulty: 1
est_minutes: 12
prerequisites: ["[[The Go Type System]]"]
next: ["[[Pointer vs Value Receivers]]", "[[Interfaces and Implicit Satisfaction]]"]
tags: [backend, go]
created: 2026-07-26
---

## TL;DR

`struct` di Go adalah cara utama mengelompokkan data terkait jadi satu tipe — mirip class tanpa inheritance. `Method` adalah function yang "menempel" pada sebuah tipe lewat *receiver*, memberinya perilaku. Go sengaja tidak punya kelas dan inheritance seperti PHP — komposisi (menyematkan satu struct di dalam struct lain, lihat [[Embedding]]) dan interface adalah cara Go mencapai hal yang biasanya dipecahkan lewat inheritance di bahasa lain, dengan trade-off yang jauh lebih eksplisit dan lebih sedikit "magic" tersembunyi.

## The Problem

Bayangkan seorang engineer yang terbiasa dengan class PHP mencoba memodelkan `Dokumen` di Go dengan mengharapkan sesuatu seperti class dengan constructor, private property, dan method yang otomatis "milik" object begitu instance dibuat. Kebingungan pertama muncul: Go tidak punya `class`, tidak punya `constructor` bawaan, dan tidak punya keyword `private`/`public` — visibilitas ditentukan oleh huruf besar/kecil di awal nama (`Dokumen` exported, `dokumen` tidak), bukan keyword eksplisit.

Kebingungan kedua, yang lebih dalam: karena Go tidak punya inheritance, engineer itu mencoba memaksakan hierarki class ala PHP (`class DokumenPDF extends Dokumen`) lewat embedding struct, dan terkejut menemukan bahwa embedding di Go bukan inheritance sungguhan — method yang "diwariskan" lewat embedding tidak bisa di-override dengan polymorphism dinamis seperti OOP klasik. Tanpa memahami bahwa Go memang sengaja tidak menyediakan inheritance (menggantinya dengan komposisi dan interface), desain yang dihasilkan akan terus terasa "dipaksakan" dan tidak idiomatic.

## Intuition

Bayangkan `struct` seperti **formulir dengan kolom-kolom yang sudah ditentukan** — `Dokumen` punya kolom `ID`, `Status`, `TanggalDibuat`. `Method` adalah **prosedur standar yang bisa dilakukan pada formulir jenis itu** — misalnya "cara memvalidasi formulir Dokumen" adalah method yang menempel khusus pada tipe `Dokumen`, tidak bisa dipanggil sembarangan pada formulir jenis lain.

Analogi ini bocor pada soal pewarisan: formulir di dunia nyata tidak punya konsep "formulir turunan yang otomatis mewarisi semua prosedur formulir induknya sambil bisa mengganti sebagian prosedur itu" (itulah inheritance dengan polymorphism). Go dengan sengaja tidak menyediakan hal ini — kalau kamu butuh "formulir yang mirip tapi sedikit berbeda perilakunya", Go mengarahkanmu untuk secara eksplisit menyusunnya lewat komposisi (menyematkan satu struct di struct lain) atau mendefinisikan interface bersama, bukan lewat mekanisme pewarisan otomatis.

## How It Works

```go
type Dokumen struct {
    ID            string
    Status        string
    TanggalDibuat time.Time
}

// Method dengan pointer receiver — bisa mengubah field Dokumen asli.
func (d *Dokumen) Verifikasi() {
    d.Status = "diverifikasi"
}

// Method dengan value receiver — hanya membaca, tidak mengubah apa pun.
func (d Dokumen) SudahDiverifikasi() bool {
    return d.Status == "diverifikasi"
}
```

`(d *Dokumen)` dan `(d Dokumen)` di depan nama function itu disebut **receiver** — inilah yang membedakan method biasa dari function biasa: method "menempel" pada satu tipe tertentu dan dipanggil lewat instance tipe itu (`doc.Verifikasi()`), bukan dipanggil sebagai function lepas (`Verifikasi(doc)`). Pilihan antara pointer receiver dan value receiver punya konsekuensi nyata pada apakah perubahan di dalam method terlihat oleh pemanggil — dibahas penuh di [[Pointer vs Value Receivers]].

Tidak ada `constructor` bawaan di Go — pola yang idiomatic adalah function biasa bernama `NewX` yang mengembalikan instance yang sudah diinisialisasi dengan benar:

```go
func NewDokumen(id string) *Dokumen {
    return &Dokumen{
        ID:            id,
        Status:        "draft",
        TanggalDibuat: time.Now(),
    }
}
```

## In Go

Contoh lengkap yang menunjukkan struct, method, dan pola `NewX` bekerja bersama:

```go
package main

import (
    "fmt"
    "time"
)

type Dokumen struct {
    ID            string
    Status        string
    TanggalDibuat time.Time
}

func NewDokumen(id string) *Dokumen {
    return &Dokumen{ID: id, Status: "draft", TanggalDibuat: time.Now()}
}

func (d *Dokumen) Verifikasi() error {
    if d.Status == "diverifikasi" {
        return fmt.Errorf("dokumen %s sudah diverifikasi sebelumnya", d.ID)
    }
    d.Status = "diverifikasi"
    return nil
}

func main() {
    doc := NewDokumen("A-001")
    if err := doc.Verifikasi(); err != nil {
        fmt.Println("gagal:", err)
        return
    }
    fmt.Printf("dokumen %s statusnya sekarang: %s\n", doc.ID, doc.Status)
}
```

## In His Stack

**Yii1/Yii2** dibangun sepenuhnya di atas class PHP dengan inheritance (model `ActiveRecord` mewarisi banyak perilaku dari base class-nya), constructor eksplisit, dan visibility keyword (`private`, `protected`, `public`). Go menolak seluruh model ini secara sengaja: tidak ada `extends`, tidak ada constructor bawaan, dan visibilitas hanya ditentukan huruf besar/kecil nama. Ini bukan Go yang "kurang lengkap" — ini keputusan desain untuk menghindari hierarki class dalam yang sulit dinalar, dengan harga berupa sedikit lebih banyak boilerplate eksplisit (`NewX` function) untuk hal yang di PHP datang gratis lewat constructor.

## Trade-offs and When Not To Use It

Struct dan method tanpa inheritance memaksa desain yang lebih eksplisit — setiap "berbagi perilaku" antar tipe harus disengaja lewat komposisi atau interface, bukan otomatis lewat hierarki. Ini butuh sedikit lebih banyak kode di awal, tapi menghindari masalah klasik OOP dengan inheritance dalam (fragile base class problem — perubahan di base class yang tidak terduga merusak semua turunannya). Untuk pemodelan data yang benar-benar sederhana (DTO, value object), struct polos tanpa method sama sekali juga valid — tidak semua struct butuh method.

## Common Mistakes

> [!warning] Jebakan
> Mencoba memaksakan pola inheritance ala OOP klasik lewat embedding struct, mengharapkan polymorphism dinamis (method di struct "induk" otomatis memanggil versi yang di-override di struct "anak"). Embedding di Go tidak bekerja seperti itu — method yang dipanggil selalu method milik tipe yang secara eksplisit dipanggil, bukan yang di-resolve secara dinamis.

> [!warning] Jebakan
> Lupa bahwa Go tidak punya constructor bawaan yang otomatis dipanggil, sehingga struct yang dibuat lewat `Dokumen{}` polos punya semua field di zero value-nya — kalau ada invariant yang harus dijaga sejak awal, gunakan pola `NewX` yang eksplisit, jangan andalkan struct literal langsung di seluruh codebase.

> [!warning] Jebakan
> Mencampur pointer receiver dan value receiver untuk method-method berbeda pada tipe yang sama tanpa alasan jelas. Konvensi umum: kalau satu method butuh pointer receiver (karena perlu mengubah state), sebaiknya semua method pada tipe itu konsisten memakai pointer receiver — detail lengkap ada di [[Pointer vs Value Receivers]].

## Exercises

1. Apa perbedaan antara `struct` di Go dan `class` di PHP dari segi pewarisan?
2. Kenapa Go tidak menyediakan constructor bawaan, dan pola apa yang menggantikannya?
3. Tulis satu struct dengan dua method — satu yang mengubah state (pointer receiver), satu yang hanya membaca (value receiver).
4. Desain terbuka: sebuah tim ingin memodelkan beberapa jenis dokumen (`DokumenPDF`, `DokumenWord`, `DokumenGambar`) yang punya sebagian perilaku sama (validasi ukuran file, cek ekstensi) dan sebagian berbeda (cara mengekstrak teks untuk pencarian). Di PHP, ini biasanya dipecahkan lewat abstract class dan inheritance. Rancang bagaimana ini dimodelkan secara idiomatic di Go tanpa inheritance.

> [!success]- Kunci jawaban
> Pisahkan perilaku bersama dari perilaku yang berbeda: buat satu struct `Dokumen` yang menampung field dan method bersama (validasi ukuran file, cek ekstensi) sebagai komposisi yang disematkan (embedding) di setiap tipe dokumen spesifik, ATAU lebih baik lagi, definisikan sebuah `interface` (misalnya `EkstraktorTeks` dengan method `EkstrakTeks() (string, error)`) yang diimplementasikan berbeda oleh `DokumenPDF`, `DokumenWord`, `DokumenGambar` masing-masing secara independen. Kode yang memproses dokumen untuk pencarian cukup bergantung pada interface itu, bukan pada tipe konkret mana pun — inilah cara Go mencapai polymorphism tanpa inheritance, dibahas lebih dalam di [[Interfaces and Implicit Satisfaction]].

## Self-Check

- Apa yang dimaksud "receiver" pada sebuah method di Go?
- Kenapa Go tidak punya constructor bawaan, dan apa pola yang menggantikannya?
- Apa perbedaan mendasar antara embedding di Go dan inheritance di PHP?
- Bagaimana Go menentukan apakah sebuah field atau method itu exported atau tidak?

## Connected Notes

- [[The Go Type System]] — prasyarat: struct adalah tipe bernama paling umum, dicocokkan secara nominal seperti dijelaskan di note itu.
- [[Pointer vs Value Receivers]] — pembahasan penuh konsekuensi memilih pointer vs value receiver untuk method.
- [[Interfaces and Implicit Satisfaction]] — cara Go mencapai polymorphism tanpa inheritance, memakai struct dan method yang dijelaskan di note ini.
- [[Embedding]] — mekanisme komposisi Go yang sering disalahartikan sebagai inheritance.
- [[../90 Architecture and Design/Handler-Service-Repository Layering|Handler-Service-Repository Layering]] — struct dan method di note ini adalah blok bangunan dasar untuk layering ini.

## Further Reading

- Dokumentasi resmi *A Tour of Go* (go.dev/tour), bagian "Methods" untuk latihan interaktif dasar.

## Catatan Saya

*Tulis di sini kalau kamu pernah mencoba memaksakan pola inheritance PHP ke Go, dan bagaimana akhirnya menemukan cara yang lebih idiomatic.*
