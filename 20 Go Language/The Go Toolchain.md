---
title: The Go Toolchain
type: concept
level: junior
domain: go
status: unread
difficulty: 1
est_minutes: 12
prerequisites: ["[[Packages and Modules]]"]
next: ["[[Table-Driven Tests]]", "[[../70 Infrastructure and Delivery/CI-CD Pipelines|CI-CD Pipelines]]"]
tags: [backend, go]
created: 2026-07-26
---

## TL;DR

Perintah `go` adalah satu toolchain terpadu: `go build` (kompilasi jadi binary), `go run` (kompilasi lalu langsung jalankan, binary-nya dibuang), `go install` (kompilasi lalu simpan binary-nya untuk dipakai ulang), `go test` (jalankan test), `go vet` (analisis statis menangkap kesalahan umum), `gofmt`/`goimports` (format kode secara kanonik, tanpa opsi konfigurasi sama sekali), dan `go mod tidy` (menyelaraskan `go.mod`/`go.sum` dengan import yang sungguh-sungguh dipakai). Ini kontras jauh dengan ekosistem PHP yang menyatukan Composer, PHPUnit, PHP-CS-Fixer, dan PHPStan/Psalm sebagai tool-tool terpisah yang harus dipilih dan dikonfigurasi sendiri-sendiri agar bekerja selaras.

## The Problem

Bayangkan seorang engineer baru bergabung ke tim yang menulis Go, dan setiap pull request-nya memicu diskusi panjang soal gaya format kode — tab vs spasi, di mana kurung kurawal diletakkan, urutan import — persis diskusi yang biasa terjadi di codebase PHP tanpa `php-cs-fixer` yang dikonfigurasi ketat. Diskusi ini sepenuhnya sia-sia di Go: `gofmt` tidak punya opsi konfigurasi sama sekali — ada **satu** format kanonik, dan setiap file Go seharusnya selalu melewatinya sebelum di-commit. Kalau tim ini tidak menjadikan `gofmt` bagian dari alur kerja wajib (pre-commit hook atau CI), mereka menciptakan ulang masalah yang sebenarnya sudah diselesaikan bahasa itu sendiri.

Masalah kedua: engineer yang sama menulis kode dengan `fmt.Printf("%d", namaDokumen)` — mencampur verb format `%d` (untuk angka) dengan variable string — dan bug ini lolos tanpa terdeteksi sampai muncul di output production yang aneh. `go vet`, yang seharusnya dijalankan otomatis di setiap build atau CI, akan menangkap kesalahan tipe format string seperti ini **sebelum kode pernah dijalankan** — tapi hanya kalau memang dijalankan sebagai bagian dari alur kerja, bukan sekadar tersedia sebagai perintah yang tidak pernah dipanggil.

## Intuition

Bayangkan toolchain Go seperti **satu perkakas multifungsi yang seluruh bagiannya sudah dikalibrasi produsen yang sama** untuk bekerja konsisten satu sama lain — beda dengan merakit sendiri obeng dari satu merek, gunting dari merek lain, dan pisau dari merek ketiga (ekosistem PHP: Composer, PHPUnit, PHP-CS-Fixer, PHPStan masing-masing proyek terpisah yang perlu dipilih dan diselaraskan sendiri).

Analogi "perkakas terkalibrasi" ini bocor pada soal fleksibilitas: perkakas multifungsi biasanya tetap punya beberapa pengaturan (sudut, kekencangan). `gofmt` sengaja **tidak** punya pengaturan sama sekali — bukan karena keterbatasan teknis, tapi keputusan filosofis untuk menghapus total ruang perdebatan gaya kode. Ini terasa membatasi bagi yang punya preferensi format kuat, tapi manfaatnya nyata: kode Go dari tim mana pun, di proyek mana pun, terlihat familiar karena semuanya melewati formatter yang identik.

## How It Works

- **`go build`** — mengompilasi package jadi binary yang disimpan di direktori saat ini (atau path yang ditentukan), **tidak** menjalankannya.
- **`go run`** — mengompilasi ke lokasi sementara, langsung menjalankannya, lalu membuang binary-nya. Cocok untuk iterasi cepat saat development, **tidak** cocok untuk artifact yang akan di-deploy.
- **`go install`** — mengompilasi dan menyimpan binary-nya ke `$GOBIN` (atau `$GOPATH/bin`) untuk dipakai ulang sebagai command-line tool.
- **`go test`** — menjalankan semua function `TestXxx` di package yang dituju (lihat [[Table-Driven Tests]]).
- **`go vet`** — analisis statis yang menangkap kesalahan umum yang **diketahui polanya** (format string yang salah tipe, struct tag yang typo, dll) — bukan linter serba bisa yang menangkap semua kelas bug; untuk cakupan analisis lebih luas, tim biasanya menambahkan `golangci-lint` (kumpulan banyak linter) sebagai lapisan tambahan, bukan pengganti.
- **`gofmt`/`goimports`** — memformat kode ke bentuk kanonik tunggal, tanpa opsi konfigurasi; `goimports` juga merapikan urutan dan pengelompokan import.
- **`go mod tidy`** — menyelaraskan `go.mod`/`go.sum` (lihat [[Packages and Modules]]) dengan import yang sungguh dipakai kode, menambah yang kurang dan membuang yang tidak terpakai.

## In Go

Contoh bug yang ditangkap `go vet` sebelum kode dijalankan:

```go
func cetakJumlah(nama string, jumlah int) {
    fmt.Printf("dokumen %s: %d\n", jumlah, nama) // urutan argumen tertukar!
}
```

Menjalankan `go vet ./...` pada kode ini akan mencetak peringatan eksplisit seperti `Printf format %d has arg jumlah of wrong type int` atau serupa — mendeteksi bahwa verb `%d` dipasangkan dengan argumen yang salah tipe, murni lewat analisis statis, tanpa perlu menjalankan program sama sekali.

Alur kerja CI yang umum untuk service Go, menggabungkan semua tool di atas secara berurutan:

```sh
#!/bin/sh
set -e

gofmt -l . | grep -q . && { echo "kode belum diformat, jalankan gofmt -w ."; exit 1; }
go vet ./...
go build ./...
go test ./...
```

Skrip ini menolak build kalau ada file yang belum diformat (`gofmt -l .` mencetak nama file yang belum sesuai format kanonik), lalu menjalankan `vet`, `build`, dan `test` berurutan — pola dasar yang biasanya diformalkan lebih jauh di [[../70 Infrastructure and Delivery/CI-CD Pipelines|CI-CD Pipelines]].

## In His Stack

**PHP (Yii1/Yii2)** bergantung pada beberapa tool terpisah yang harus dipilih dan dikonfigurasi masing-masing tim: Composer untuk dependency, PHPUnit untuk testing, PHP-CS-Fixer atau PHP_CodeSniffer untuk format/gaya kode, PHPStan atau Psalm untuk analisis statis — masing-masing punya file konfigurasi sendiri, dan tim yang berbeda bisa mengonfigurasinya secara berbeda pula, membuat "standar" itu sendiri jadi keputusan tim, bukan bawaan bahasa. Go menyatukan sebagian besar kebutuhan ini dalam satu perintah `go` yang identik di proyek mana pun — mengurangi variasi konfigurasi antar tim secara signifikan, meski analisis statis mendalam (setara PHPStan level tinggi) tetap butuh tool tambahan seperti `golangci-lint`.

## Trade-offs and When Not To Use It

Ketiadaan opsi konfigurasi di `gofmt` kadang terasa membatasi bagi individu dengan preferensi gaya kuat, tapi dalam konteks tim (apalagi tim besar dengan 10+ developer lintas banyak aplikasi), keseragaman mutlak ini jauh lebih berharga daripada fleksibilitas personal — ia menghapus total satu kategori diskusi code review yang tidak produktif. `go vet` sengaja terbatas cakupannya (hanya pola yang benar-benar diketahui dan hampir pasti bug) — jangan mengandalkannya sebagai satu-satunya lapisan analisis statis; untuk cakupan lebih luas (unused variable yang lebih agresif, kompleksitas cyclomatic, dll), tambahkan `golangci-lint` sebagai standar tambahan di CI.

## Common Mistakes

> [!warning] Jebakan
> Tidak menjadikan `gofmt`/`goimports` bagian wajib dari alur kerja (pre-commit hook atau CI gate), sehingga diskusi gaya format kode yang seharusnya sudah dihapus oleh bahasa ini justru terulang lagi di code review.

> [!warning] Jebakan
> Mencampur `go build`, `go run`, dan `go install` tanpa memahami perbedaan output masing-masing — misalnya mengira `go run` menghasilkan binary yang bisa disimpan dan di-deploy, padahal binary-nya sementara dan dibuang setelah eksekusi selesai.

> [!warning] Jebakan
> Menganggap `go vet` sebagai linter serba bisa yang menangkap semua kelas bug. Ia hanya memeriksa pola yang sudah diketahui dan hampir pasti kesalahan (format string, struct tag) — untuk cakupan analisis statis yang lebih luas, dibutuhkan tool tambahan seperti `golangci-lint`.

## Exercises

1. Apa perbedaan `go build`, `go run`, dan `go install`, dan kapan masing-masing tepat dipakai?
2. Kenapa `gofmt` sengaja tidak menyediakan opsi konfigurasi, berbeda dari PHP-CS-Fixer yang sangat bisa dikustomisasi?
3. Apa yang bisa (dan tidak bisa) ditangkap oleh `go vet`?
4. Desain terbuka: sebuah tim baru mulai menulis service Go untuk salah satu dari 13 aplikasi legal-services, dan ingin memastikan kualitas kode terjaga otomatis tanpa bergantung pada disiplin manual setiap developer. Rancang alur CI minimal yang memakai toolchain Go bawaan (plus linter tambahan kalau relevan) untuk menjamin kode yang masuk selalu terformat, lolos analisis statis dasar, dan lolos test — sebelum bisa di-merge.

> [!success]- Kunci jawaban
> Alur CI minimal: (1) jalankan `gofmt -l .` dan gagalkan build kalau ada file yang terdaftar (belum terformat); (2) jalankan `go vet ./...` untuk menangkap kesalahan pola umum; (3) tambahkan `golangci-lint run` sebagai lapisan analisis statis yang lebih luas (unused code, kompleksitas berlebihan, dan aturan tambahan yang disepakati tim lewat file konfigurasi `.golangci.yml`); (4) jalankan `go build ./...` untuk memastikan seluruh codebase benar-benar bisa dikompilasi; (5) jalankan `go test ./... -race` (menyertakan race detector, lihat [[../50 Concurrency and Performance/Race Conditions and the Race Detector|Race Conditions and the Race Detector]]) sebagai gate terakhir sebelum merge diizinkan. Jadikan seluruh urutan ini gate wajib di pipeline CI (lihat [[../70 Infrastructure and Delivery/CI-CD Pipelines|CI-CD Pipelines]]), bukan langkah opsional yang bergantung pada inisiatif masing-masing developer.

## Self-Check

- Apa perbedaan `go build`, `go run`, dan `go install`?
- Kenapa `gofmt` tidak punya opsi konfigurasi sama sekali?
- Apa yang bisa ditangkap `go vet`, dan apa yang tidak?
- Apa fungsi `go mod tidy`?

## Connected Notes

- [[Packages and Modules]] — prasyarat: `go.mod`/`go.sum` yang dikelola `go mod tidy` dijelaskan penuh di note itu.
- [[Table-Driven Tests]] — kelanjutan langsung penggunaan `go test`, salah satu bagian toolchain yang dibahas di note ini.
- [[../70 Infrastructure and Delivery/CI-CD Pipelines|CI-CD Pipelines]] — alur CI yang menyatukan seluruh perintah toolchain di note ini jadi satu pipeline otomatis.
- [[../50 Concurrency and Performance/Race Conditions and the Race Detector|Race Conditions and the Race Detector]] — `-race` sebagai flag tambahan pada `go test` dan `go build` yang penting untuk kode konkuren.
- [[../50 Concurrency and Performance/Benchmarking in Go|Benchmarking in Go]] — `go test -bench` sebagai bagian lain dari toolchain testing yang sama.

## Further Reading

- Dokumentasi resmi *"Command go"* (pkg.go.dev/cmd/go) — referensi lengkap seluruh sub-perintah toolchain, sumber kebenaran paling akurat untuk versi Go yang sedang dipakai.

## Catatan Saya

*Tulis di sini bug yang pernah ditangkap `go vet` (atau seharusnya ditangkap, kalau kamu belum menjalankannya secara rutin) di kodemu sendiri.*
