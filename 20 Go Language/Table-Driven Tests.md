---
title: Table-Driven Tests
type: concept
level: junior
domain: go
status: unread
difficulty: 2
est_minutes: 13
prerequisites: ["[[The Go Toolchain]]", "[[Errors as Values]]"]
next: ["[[Stdlib Testing vs Testify]]", "[[Mocking Through Interfaces]]"]
tags: [backend, go]
created: 2026-07-26
---

## TL;DR

Table-driven test adalah pola testing paling idiomatic di Go: definisikan sebuah **tabel** (slice of struct) berisi kasus-kasus test — input, hasil yang diharapkan, nama kasus — lalu loop di atasnya sekali, memanggil `t.Run(tc.name, ...)` untuk setiap baris. Pola ini menghindari duplikasi logic assertion di banyak function `TestXxx` terpisah, memberi output kegagalan yang jelas per kasus (lewat subtest bernama), dan membuat menambah kasus baru semudah menambah satu baris di tabel.

## The Problem

Bayangkan sebuah function `ValidasiNIK` yang harus menangani banyak variasi input — NIK valid, NIK terlalu pendek, NIK mengandung huruf, NIK kosong, dan seterusnya. Cara naif menulis test-nya adalah satu function `TestXxx` terpisah untuk setiap variasi, masing-masing mengulang logic assertion yang nyaris identik. Menambah satu variasi baru berarti menyalin-tempel satu function test lagi, dan kalau logic assertion-nya perlu diubah (misalnya format pesan error diperbarui), perubahan itu harus disebarkan manual ke semua function test yang sudah ada — rawan ada satu yang terlewat dan menjadi tidak sinkron.

Table-driven test mengubah ini jadi satu function test dengan satu logic assertion, dijalankan berulang untuk setiap baris data di tabel — menambah kasus baru semudah menambah satu baris, dan perubahan logic assertion cukup diubah di satu tempat.

## Intuition

Bayangkan table-driven test seperti **daftar periksa kontrol kualitas di pabrik** — ada satu prosedur pemeriksaan standar (logic assertion), diterapkan mekanis ke setiap baris di lembar data (tabel kasus test) yang mencatat varian produk apa saja yang perlu diperiksa. Menambah varian produk baru cukup menambah satu baris di lembar itu, tanpa perlu menulis prosedur pemeriksaan baru dari nol.

Analogi ini bocor tepat di satu gotcha teknis: baris-baris di lembar periksa sungguhan independen dan urutan pemeriksaannya tidak penting. Di Go, kalau subtest dijalankan **paralel** (`t.Parallel()`) di dalam loop, ada jebakan klasik terkait bagaimana variable loop ditangkap oleh closure setiap iterasi — sebelum Go 1.22, semua goroutine subtest paralel bisa diam-diam berakhir memakai data dari iterasi **terakhir** kalau variable loop tidak ditangkap ulang secara eksplisit di setiap iterasi.

## How It Works

```go
func TestValidasiNIK(t *testing.T) {
    cases := []struct {
        nama    string
        input   string
        wantErr bool
    }{
        {nama: "NIK valid", input: "3175012345678901", wantErr: false},
        {nama: "NIK terlalu pendek", input: "12345", wantErr: true},
        {nama: "NIK kosong", input: "", wantErr: true},
        {nama: "NIK mengandung huruf", input: "31750123ABC78901", wantErr: true},
    }

    for _, tc := range cases {
        tc := tc // lihat catatan versi Go di bawah soal baris ini
        t.Run(tc.nama, func(t *testing.T) {
            err := ValidasiNIK(tc.input)
            if (err != nil) != tc.wantErr {
                t.Errorf("ValidasiNIK(%q) error = %v, wantErr %v", tc.input, err, tc.wantErr)
            }
        })
    }
}
```

`t.Run(tc.nama, ...)` membuat subtest bernama — kalau kasus "NIK terlalu pendek" gagal, output test akan menyebutnya persis, misalnya `--- FAIL: TestValidasiNIK/NIK_terlalu_pendek`, bukan sekadar "test gagal" tanpa konteks kasus mana yang bermasalah.

> [!question] Perlu diverifikasi
> Klaim: baris `tc := tc` di dalam loop dibutuhkan untuk menghindari bug capture variable loop **hanya** kalau subtest dijalankan paralel (`t.Parallel()`) pada Go versi sebelum 1.22; Go 1.22 mengubah semantik variable loop menjadi per-iterasi secara default.
> Kenapa ragu: perilaku ini bergantung versi Go yang dipakai proyek — proyek yang di-pin ke versi lebih lama dari 1.22 tetap butuh baris `tc := tc` ini secara eksplisit untuk subtest paralel.
> Cara verifikasi: periksa `go.mod` untuk versi Go yang dipakai proyek, dan baca release notes Go 1.22 soal perubahan semantik variable loop `for`.

## In Go

Versi dengan subtest paralel, menunjukkan pentingnya capture variable loop untuk Go versi lama:

```go
func TestValidasiNIKParalel(t *testing.T) {
    cases := []struct {
        nama    string
        input   string
        wantErr bool
    }{
        {nama: "valid", input: "3175012345678901", wantErr: false},
        {nama: "kosong", input: "", wantErr: true},
    }

    for _, tc := range cases {
        tc := tc // WAJIB di Go < 1.22 sebelum t.Parallel() di bawah;
                 // tanpa ini, semua subtest paralel bisa memakai
                 // data dari iterasi TERAKHIR saja.
        t.Run(tc.nama, func(t *testing.T) {
            t.Parallel()
            err := ValidasiNIK(tc.input)
            if (err != nil) != tc.wantErr {
                t.Errorf("ValidasiNIK(%q) error = %v, wantErr %v", tc.input, err, tc.wantErr)
            }
        })
    }
}
```

## In His Stack

**PHPUnit** punya `@dataProvider`, konsep yang serupa — satu method test dijalankan berulang dengan data dari method provider terpisah. Bedanya, `@dataProvider` bekerja lewat annotation/atribut yang "magic" dihubungkan reflection PHPUnit, sementara table-driven test Go hanyalah struct dan slice biasa, tanpa framework atau reflection tersembunyi — lebih transparan dan mudah ditelusuri langsung dari kode.

## Trade-offs and When Not To Use It

Table-driven test paling pas untuk function dengan input-output yang jelas (pure function, atau function dengan sedikit dependency eksternal). Untuk test yang butuh setup state kompleks berbeda-beda per kasus, tabelnya bisa ditambah field function setup (`setup func(t *testing.T)`) sebagai bagian dari struct kasus — tetap table-driven, hanya lebih kaya. Kalau kasus-kasus test benar-benar tidak berbagi logic assertion sama sekali (menguji hal yang sepenuhnya berbeda), memaksakan semuanya ke satu tabel hanya menambah kompleksitas tanpa manfaat — pisahkan jadi test function terpisah kalau memang tidak ada logic yang dibagi.

## Common Mistakes

> [!warning] Jebakan
> Tidak memakai `t.Run` dan hanya melakukan assertion langsung di dalam loop. Kegagalan pada kasus mana pun akan tercampur jadi satu laporan tanpa nama kasus yang jelas, menyulitkan diagnosis kasus mana yang sebenarnya gagal.

> [!warning] Jebakan
> Lupa menangkap ulang variable loop (`tc := tc`) sebelum memanggil `t.Parallel()` di dalam subtest, pada Go versi sebelum 1.22 — semua subtest paralel bisa diam-diam memakai data dari iterasi terakhir tabel, bukan data masing-masing.

> [!warning] Jebakan
> Menjejalkan terlalu banyak concern berbeda ke satu tabel test yang sama (menguji beberapa function sekaligus, atau menguji hal-hal yang tidak berbagi logic assertion). Table-driven bermanfaat karena satu logic assertion dipakai berulang — kalau logic-nya beda-beda tiap baris, manfaat itu hilang.

## Exercises

1. Apa manfaat utama menulis test dalam bentuk tabel dibanding banyak function `TestXxx` terpisah?
2. Kenapa `t.Run(tc.nama, ...)` memberi manfaat dibanding memanggil assertion langsung di dalam loop tanpa subtest?
3. Kenapa `tc := tc` di dalam loop penting saat subtest dijalankan paralel di Go versi sebelum 1.22?
4. Desain terbuka: sebuah function `HitungBiayaLayanan` menghitung biaya berdasarkan kombinasi jenis layanan, jumlah dokumen, dan status prioritas — kombinasi variabelnya cukup banyak (puluhan kombinasi valid). Rancang struktur table-driven test untuk function ini yang tetap mudah dibaca meski jumlah kasusnya banyak, dan jelaskan bagaimana menjaga tabel ini tetap terawat seiring aturan bisnis bertambah kompleks di masa depan.

> [!success]- Kunci jawaban
> Definisikan struct kasus dengan field yang jelas mewakili setiap dimensi variasi (`jenisLayanan`, `jumlahDokumen`, `prioritas`, `wantBiaya`, `wantErr`), dan kelompokkan baris-baris tabel dengan komentar yang menjelaskan kelompok skenario apa yang sedang diuji (misalnya "--- kombinasi normal ---", "--- kombinasi edge case: jumlah dokumen nol ---") supaya tabel tetap mudah dinavigasi meski panjang. Untuk menjaga keterawatan seiring aturan bisnis bertambah, sertakan nama kasus (`tc.nama`) yang deskriptif dan spesifik (bukan sekadar "kasus 1", "kasus 2") sehingga kegagalan test langsung menunjukkan skenario bisnis mana yang rusak, dan pertimbangkan memecah tabel jadi beberapa function test terpisah kalau ia mulai menguji lebih dari satu aspek/tanggung jawab function yang sama.

## Self-Check

- Apa yang membedakan table-driven test dari menulis banyak function `TestXxx` terpisah?
- Apa manfaat `t.Run` dibanding assertion langsung di dalam loop?
- Kenapa capture variable loop (`tc := tc`) penting untuk subtest paralel di Go versi lama?
- Kapan table-driven test kurang cocok dipakai?

## Connected Notes

- [[The Go Toolchain]] — prasyarat: `go test` yang menjalankan pola table-driven test ini.
- [[Errors as Values]] — table-driven test sering dipakai untuk memverifikasi perilaku error sebuah function secara sistematis lintas banyak kasus.
- [[Stdlib Testing vs Testify]] — pola table-driven bisa dipakai baik dengan `testing` polos maupun dengan assertion library tambahan.
- [[Mocking Through Interfaces]] — table-driven test sering dikombinasikan dengan mock untuk menguji berbagai skenario dependency eksternal.
- [[../50 Concurrency and Performance/Benchmarking in Go|Benchmarking in Go]] — pola tabel yang sama juga bisa dipakai untuk benchmark dengan berbagai ukuran input.

## Further Reading

- Wiki resmi Go, halaman *"TableDrivenTests"* (github.com/golang/go/wiki/TableDrivenTests) — contoh dan konvensi resmi dari tim Go.

## Catatan Saya

*Tulis di sini function di kerjaanmu yang paling cocok ditulis ulang sebagai table-driven test.*
