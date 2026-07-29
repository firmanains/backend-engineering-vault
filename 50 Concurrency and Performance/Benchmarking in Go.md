---
title: Benchmarking in Go
type: concept
level: intermediate
domain: concurrency
status: unread
difficulty: 2
est_minutes: 15
prerequisites: ["[[pprof Profiling]]"]
next: ["[[Reducing Allocations]]"]
tags: [backend, concurrency, go, performance]
created: 2026-07-29
---

## TL;DR

[[pprof Profiling]] menjawab "di mana waktu/memori dihabiskan" untuk aplikasi yang sudah berjalan. Benchmark menjawab pertanyaan yang berbeda tapi saling melengkapi: "seberapa cepat/efisien **fungsi spesifik ini**, diukur dengan metodologi yang konsisten dan bisa dibandingkan dari waktu ke waktu?" Go menyediakan dukungan benchmark bawaan lewat `go test -bench`, mengukur bukan hanya kecepatan eksekusi tapi juga (lewat flag `-benchmem`) jumlah alokasi memori per operasi — dua angka yang sama pentingnya untuk memahami dampak sebuah perubahan kode terhadap performa nyata, bukan sekadar perasaan "kelihatannya lebih cepat".

## The Problem

Dua developer berdebat soal implementasi mana yang lebih cepat untuk menggabungkan string dalam sebuah loop — satu memakai operator `+=` berulang, satu lagi memakai `strings.Builder`. Tanpa pengukuran konkret, argumen ini hanya berdasarkan intuisi ("kayaknya Builder lebih cepat karena..."), yang meski sering benar arahnya, tidak memberi angka pasti seberapa besar perbedaannya, atau apakah perbedaan itu signifikan untuk ukuran data yang sebenarnya relevan dengan kasus penggunaan mereka. Keputusan performa yang dibuat tanpa data konkret berisiko salah — kadang perbedaan yang "terasa jelas" secara teori ternyata tidak signifikan dalam praktik untuk ukuran data kecil, atau sebaliknya, sesuatu yang dianggap remeh ternyata berdampak besar pada skala data yang sebenarnya dipakai production.

Masalah kedua: sebuah tim melakukan "optimasi" pada fungsi kritis tanpa mengukur baseline sebelum perubahan — setelah perubahan, mereka **yakin** kodenya lebih cepat berdasarkan perasaan, tapi tanpa angka before-after yang konkret, klaim ini tidak bisa diverifikasi, dan yang lebih berbahaya, tidak ada cara mendeteksi kalau ternyata perubahan itu (mungkin karena efek samping yang tidak terduga pada escape analysis atau alokasi) justru membuatnya **lebih lambat**.

## Intuition

Bayangkan benchmark seperti **uji coba mobil di dyno (dynamometer)** — alat yang mengukur performa mesin dalam kondisi terkontrol dan konsisten, memberi angka pasti (tenaga kuda, torsi) yang bisa dibandingkan secara objektif antar mesin berbeda atau antar versi mesin yang sama setelah modifikasi. Tanpa dyno, dua montir bisa berdebat "mesin ini kayaknya lebih bertenaga" berdasarkan perasaan mengendarai, yang subjektif dan sulit diperbandingkan secara adil. Benchmark Go memberi "dyno" untuk kode — mengeksekusi fungsi yang sama berulang kali dalam kondisi terkontrol, memberi angka rata-rata yang bisa dipercaya dan dibandingkan.

Analogi ini bocor pada satu hal: dyno mobil mengukur dalam kondisi yang **selalu** terkontrol (suhu, beban) untuk hasil yang bisa dibandingkan lintas waktu. Benchmark kode bisa terpengaruh oleh kondisi mesin yang menjalankannya (proses lain yang sedang berjalan, suhu CPU, bahkan versi Go yang dipakai) — inilah kenapa membandingkan angka benchmark **absolut** dari mesin berbeda kurang bermakna, sementara membandingkan **before-after** pada mesin yang sama, dengan kondisi yang relatif konsisten, jauh lebih dapat diandalkan.

## How It Works

```go
package stringutil

import "strings"

func GabungDenganPlus(bagian []string) string {
	var hasil string
	for _, b := range bagian {
		hasil += b
	}
	return hasil
}

func GabungDenganBuilder(bagian []string) string {
	var sb strings.Builder
	for _, b := range bagian {
		sb.WriteString(b)
	}
	return sb.String()
}
```

```go
package stringutil

import "testing"

var dataUji = []string{"a", "b", "c", "d", "e", "f", "g", "h", "i", "j"}

// Nama fungsi HARUS diawali "Benchmark", menerima *testing.B —
// b.N adalah jumlah iterasi yang DITENTUKAN OTOMATIS oleh testing
// framework, disesuaikan sampai hasil pengukuran stabil secara statistik.
func BenchmarkGabungDenganPlus(b *testing.B) {
	for i := 0; i < b.N; i++ {
		GabungDenganPlus(dataUji)
	}
}

func BenchmarkGabungDenganBuilder(b *testing.B) {
	for i := 0; i < b.N; i++ {
		GabungDenganBuilder(dataUji)
	}
}
```

```bash
# -bench=. menjalankan semua benchmark; -benchmem menambah kolom
# alokasi memori (jumlah alokasi dan total byte per operasi).
go test -bench=. -benchmem ./...

# Output tipikal:
# BenchmarkGabungDenganPlus-8       500000    2400 ns/op    480 B/op    9 allocs/op
# BenchmarkGabungDenganBuilder-8   2000000     600 ns/op     64 B/op    1 allocs/op
```

Output ini memberi angka **konkret**: `GabungDenganBuilder` sekitar 4x lebih cepat (600ns vs 2400ns) dan mengalokasikan jauh lebih sedikit (1 alokasi vs 9 alokasi) — bukan lagi perdebatan intuisi, tapi data yang bisa langsung dipakai membuat keputusan.

## Under The Hood

**`b.N`** ditentukan otomatis oleh testing framework — ia menjalankan benchmark dengan `N` yang meningkat bertahap (dimulai kecil, terus digandakan) sampai total waktu eksekusi cukup lama untuk memberi pengukuran yang stabil secara statistik (mengurangi noise dari variasi kecil antar run) — developer tidak perlu (dan tidak boleh) menentukan `b.N` secara manual, karena angka ini disesuaikan otomatis berdasarkan kecepatan kode yang diukur.

**`b.ResetTimer()`** penting dipanggil kalau ada setup yang mahal sebelum loop benchmark yang sebenarnya (misalnya membuat data uji besar) — tanpa ini, waktu setup ikut terhitung sebagai bagian dari pengukuran, mendistorsi hasil terutama kalau `b.N` kecil (setup mendominasi) dibanding besar (setup relatif kecil dibanding total waktu).

**Benchmark yang melibatkan operasi paralel** (`b.RunParallel`) mengukur throughput saat dijalankan dari banyak goroutine sekaligus — relevan untuk kode yang di production memang dipanggil konkuren (handler HTTP, misalnya), memberi gambaran performa yang lebih representatif dibanding benchmark sekuensial murni untuk kode yang karakteristik pemakaian sebenarnya selalu konkuren.

## In Go

```go
package handler_test

import (
	"testing"
)

func BenchmarkParseRequestBesar(b *testing.B) {
	dataBesar := buatDataUjiBesar() // setup MAHAL, dijalankan SEKALI

	b.ResetTimer() // reset timer SETELAH setup, SEBELUM loop pengukuran

	for i := 0; i < b.N; i++ {
		ParseRequest(dataBesar)
	}
}

// BenchmarkKonkurenParalel mengukur throughput saat dipanggil dari
// BANYAK goroutine sekaligus — representatif untuk kode yang memang
// dipanggil konkuren di production (seperti handler HTTP).
func BenchmarkKonkurenParalel(b *testing.B) {
	b.RunParallel(func(pb *testing.PB) {
		for pb.Next() {
			ParseRequest([]byte(`{"contoh":"data"}`))
		}
	})
}

func buatDataUjiBesar() []byte { return make([]byte, 1024*1024) }
func ParseRequest(data []byte) {}
```

## In His Stack

Untuk fungsi kritis yang dipanggil sangat sering (parsing payload webhook dari partner eksternal, serialisasi/deserialisasi JSON untuk API bervolume tinggi), menulis benchmark sebagai bagian dari test suite (bukan hanya unit test kebenaran fungsional) memberi jaring pengaman terhadap regresi performa — perubahan kode di masa depan yang tidak sengaja memperlambat fungsi kritis ini bisa terdeteksi lewat perbandingan angka benchmark sebelum dan sesudah perubahan, sebelum sampai ke code review, apalagi production.

## Trade-offs and When Not To Use It

Menulis benchmark untuk setiap fungsi kecil adalah usaha yang tidak selalu sepadan — benchmark paling bernilai untuk fungsi yang **benar-benar** kritis terhadap performa (dipanggil sangat sering, atau bagian dari hot path yang sudah teridentifikasi lewat profiling), bukan untuk kode yang jarang dipanggil atau yang perbedaan performanya tidak akan pernah terasa dalam praktik. Hasil benchmark juga bisa menyesatkan kalau tidak representatif terhadap kondisi nyata — benchmark dengan data uji yang jauh lebih kecil (atau lebih besar) dari data production sesungguhnya bisa memberi kesimpulan yang tidak berlaku untuk kondisi nyata, sehingga data uji benchmark idealnya mendekati karakteristik data production yang sebenarnya.

## Common Mistakes

> [!warning] Jebakan
> Membuat keputusan optimasi performa berdasarkan intuisi tanpa benchmark konkret — perbedaan yang "terasa jelas" secara teori kadang tidak signifikan dalam praktik, atau sebaliknya.

> [!warning] Jebakan
> Lupa memanggil `b.ResetTimer()` setelah setup yang mahal di dalam fungsi benchmark — waktu setup ikut terhitung sebagai bagian pengukuran, mendistorsi hasil terutama untuk `b.N` yang kecil.

> [!warning] Jebakan
> Menulis benchmark dengan data uji yang tidak representatif terhadap kondisi production nyata (terlalu kecil, terlalu seragam) — kesimpulan performa yang didapat mungkin tidak berlaku untuk beban kerja sesungguhnya.

## Exercises

1. Jelaskan kenapa `b.N` ditentukan otomatis oleh testing framework, bukan ditetapkan manual oleh developer.
2. Kenapa `b.ResetTimer()` penting dipanggil setelah setup yang mahal di dalam fungsi benchmark?
3. Kenapa membandingkan angka benchmark absolut dari mesin yang berbeda kurang bermakna dibanding membandingkan before-after pada mesin yang sama?
4. Desain terbuka: kamu ingin memverifikasi klaim bahwa mengganti `interface{}` dengan generics (lihat [[../20 Go Language/Generics|Generics]]) pada sebuah fungsi utilitas yang dipanggil sangat sering benar-benar meningkatkan performa, bukan sekadar lebih type-safe. Rancang benchmark yang membandingkan kedua versi ini secara adil, dan sebutkan metrik apa saja (selain waktu eksekusi) yang perlu dibandingkan untuk mendapat gambaran lengkap.

> [!success]- Kunci jawaban
> **1.** Kecepatan eksekusi kode bisa sangat bervariasi (dari nanodetik untuk operasi sederhana sampai detik untuk operasi kompleks) — angka `N` yang tetap (misalnya selalu 1000 iterasi) akan memberi pengukuran yang terlalu singkat (noise besar) untuk kode yang sangat cepat, atau terlalu lama (buang waktu) untuk kode yang lambat. Testing framework menyesuaikan `b.N` secara otomatis, meningkatkannya bertahap sampai total waktu eksekusi cukup untuk memberi pengukuran yang stabil secara statistik, terlepas dari seberapa cepat atau lambat kode yang diukur.
> **4.** Tulis dua fungsi benchmark terpisah, satu memanggil versi `interface{}`, satu memanggil versi generic, keduanya dengan input data yang **identik** (representatif terhadap ukuran/jenis data production nyata). Jalankan `go test -bench=. -benchmem` untuk kedua fungsi pada mesin yang sama dalam kondisi yang relatif tenang (tidak ada proses berat lain berjalan bersamaan). Bandingkan tiga metrik dari output: **ns/op** (waktu eksekusi rata-rata per operasi), **B/op** (byte yang dialokasikan per operasi — penting karena versi generic seringkali mengurangi alokasi yang sebelumnya dibutuhkan untuk boxing/unboxing `interface{}`), dan **allocs/op** (jumlah alokasi terpisah per operasi, relevan untuk tekanan GC meski total byte-nya sama). Ketiga metrik ini bersama memberi gambaran lengkap — kadang satu versi lebih cepat dalam ns/op tapi tidak banyak berbeda dalam alokasi, atau sebaliknya, dan keputusan optimasi yang baik mempertimbangkan ketiganya, bukan hanya kecepatan mentah.

## Self-Check

- Kenapa `b.N` ditentukan otomatis, bukan manual?
- Apa fungsi `b.ResetTimer()` dan kapan ia dibutuhkan?
- Kenapa data uji benchmark harus representatif terhadap kondisi production?
- Tiga metrik apa yang ditampilkan `-benchmem`, dan kenapa ketiganya penting?

## Connected Notes

- [[pprof Profiling]] — benchmark sering dikombinasikan dengan pprof (`go test -bench -cpuprofile=cpu.prof`) untuk profiling yang lebih terarah pada fungsi spesifik.
- [[Escape Analysis]] — hasil `-benchmem` adalah cara konkret memverifikasi dampak keputusan escape analysis terhadap jumlah alokasi.
- [[Reducing Allocations]] — benchmark adalah alat verifikasi utama untuk mengonfirmasi teknik pengurangan alokasi benar-benar berdampak, dibahas di note berikutnya.
- [[Load Testing]] — benchmark mengukur performa fungsi individual; load testing mengukur performa sistem secara keseluruhan di bawah beban nyata, dibahas di note lain domain ini.
- [[../20 Go Language/Generics|Generics]] — klaim performa generics vs interface{} yang disinggung di exercise note ini perlu diverifikasi lewat benchmark, bukan diasumsikan.

## Further Reading

- Dokumentasi resmi Go, package `testing`, bagian benchmark.
- Dave Cheney, "How to write benchmarks in Go" (artikel blog yang banyak dirujuk komunitas Go).

## Catatan Saya

*Tulis di sini apakah ada fungsi kritis di kerjaanmu yang belum pernah di-benchmark — dan klaim performa apa (kalau ada) yang selama ini hanya berdasarkan asumsi.*
