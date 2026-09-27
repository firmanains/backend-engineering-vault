---
title: Profiling a Real Application
type: concept
level: intermediate
domain: concurrency
status: unread
difficulty: 3
est_minutes: 16
prerequisites: ["[[Capacity Planning]]", "[[pprof Profiling]]", "[[Benchmarking in Go]]", "[[Load Testing]]"]
next: ["[[Cache-Aside, Write-Through, and Write-Behind]]"]
tags: [backend, concurrency, performance, go]
created: 2026-07-29
---

## TL;DR

Setiap note sebelumnya di domain ini membahas satu alat atau konsep secara terisolasi — pprof, benchmark, load test, escape analysis, masing-masing menjawab pertanyaan spesifik. Note penutup ini menunjukkan bagaimana semuanya dipakai **bersama**, dalam urutan yang masuk akal, untuk satu latihan optimasi performa yang utuh dari awal sampai akhir: mulai dari gejala yang dilaporkan, sampai perbaikan yang diverifikasi dengan data, bukan sekadar tebakan yang "terasa" sudah membaik.

## The Problem

Sebuah endpoint API yang menampilkan daftar permohonan dengan filter kompleks mulai menerima keluhan "terasa lambat" dari pengguna, tapi tim tidak punya metodologi jelas untuk mendiagnosis dan memperbaikinya secara sistematis — mereka mencoba beberapa perubahan berdasarkan tebakan (menambah index, mengubah beberapa query), masing-masing memakan waktu tim tapi tidak ada yang benar-benar diverifikasi dampaknya secara terukur. Tanpa metodologi yang konsisten, tim tidak tahu apakah perbaikan yang sudah dilakukan benar-benar berdampak, atau hanya kebetulan bersamaan dengan penurunan traffic sesaat yang membuat endpoint itu **terlihat** lebih cepat tanpa hubungan sebab-akibat yang jelas.

## Intuition

Bayangkan latihan profiling menyeluruh seperti **diagnosis medis yang sistematis**, dibanding menebak penyakit dari gejala permukaan lalu mencoba obat secara acak. Dokter yang baik mengikuti urutan: dengarkan gejala (laporan pengguna), lakukan pemeriksaan yang tepat untuk mempersempit kemungkinan (profiling untuk menemukan lokasi masalah), konfirmasi diagnosis dengan tes spesifik (benchmark untuk memverifikasi hipotesis), berikan pengobatan yang ditargetkan (perbaikan kode), lalu **verifikasi** hasilnya dengan pemeriksaan ulang (benchmark/load test setelah perbaikan) — bukan berhenti setelah memberi obat dan berasumsi pasien pasti sembuh.

## How It Works

```mermaid
flowchart TD
    A["1. Gejala dilaporkan\n(endpoint lambat, p99 tinggi)"] --> B["2. Ukur baseline SEBELUM\nperubahan apa pun\n(latency, alokasi, CPU)"]
    B --> C["3. Profiling: pprof CPU + heap\nuntuk menemukan LOKASI masalah"]
    C --> D["4. Bentuk HIPOTESIS spesifik\n(bukan tebakan umum)"]
    D --> E["5. Verifikasi hipotesis dengan\nbenchmark terisolasi"]
    E --> F["6. Terapkan perbaikan\nyang DITARGETKAN"]
    F --> G["7. Ukur ULANG, bandingkan\ndengan baseline langkah 2"]
    G --> H{"Perbaikan signifikan\nDAN terverifikasi?"}
    H -->|"Ya"| I["Load test untuk konfirmasi\ndi skala production"]
    H -->|"Tidak"| C
```

Diagram ini menunjukkan siklus lengkap yang mengulang kembali ke profiling kalau perbaikan pertama tidak memberi hasil yang diharapkan — optimasi performa jarang selesai dalam satu iterasi, dan siklus yang disiplin (bukan mencoba banyak perubahan sekaligus tanpa mengukur masing-masing secara terpisah) memastikan setiap perbaikan yang diterapkan benar-benar bisa dipertanggungjawabkan dampaknya.

**Langkah 2 (baseline) adalah langkah yang paling sering dilewati** — tanpa angka "sebelum" yang jelas dan terukur dalam kondisi yang konsisten, klaim "sudah lebih cepat" setelah perubahan tidak bisa diverifikasi secara objektif, hanya berdasarkan perasaan atau anekdot.

## Under The Hood

**Kenapa urutan profiling sebelum benchmark penting**: profiling (langkah 3) pada aplikasi yang benar-benar berjalan (atau load test yang mensimulasikan kondisi nyata) menunjukkan **di mana** masalah sesungguhnya berada di antara ribuan baris kode — tanpa ini, benchmark yang ditulis berdasarkan tebakan bisa saja mengukur fungsi yang sebenarnya tidak relevan dengan masalah nyata, membuang waktu untuk mengoptimasi sesuatu yang tidak pernah menjadi bottleneck sesungguhnya. Benchmark (langkah 5) baru bernilai penuh **setelah** profiling mempersempit area yang perlu diperiksa mendalam.

**Menghubungkan berbagai jenis profil untuk gambaran lengkap**: CPU profile menunjukkan di mana waktu dihabiskan; heap profile menunjukkan di mana memori dialokasikan (yang mungkin **menyebabkan** waktu CPU tinggi secara tidak langsung, lewat tekanan GC yang meningkat, lihat [[Garbage Collection in Go]]); goroutine profile menunjukkan kalau ada goroutine yang macet atau bocor yang berkontribusi pada masalah. Masalah performa nyata seringkali adalah kombinasi dari beberapa faktor ini yang saling terkait, bukan satu penyebab tunggal yang berdiri sendiri.

## In Go

```go
package main

import (
	"encoding/json"
	"fmt"
	"log"
	"os"
	"runtime/pprof"
	"time"
)

type Permohonan struct {
	ID     int    `json:"id"`
	Nomor  string `json:"nomor"`
	Status string `json:"status"`
}

// LatihanOptimasiLengkap menunjukkan struktur latihan profiling: ukur
// baseline, jalankan beban yang sama dengan CPU profiling aktif, simpan
// profilnya untuk dibandingkan dengan profil setelah perbaikan.
func LatihanOptimasiLengkap(namaProfil string) error {
	data := buatDataUji(5000)

	// Langkah 2: baseline sebelum perubahan apa pun.
	mulai := time.Now()
	if err := jalankanEndpointSimulasi(data); err != nil {
		return err
	}
	fmt.Printf("baseline satu eksekusi: %v\n", time.Since(mulai))

	// Langkah 3: CPU profiling untuk menemukan lokasi masalah.
	f, err := os.Create(namaProfil)
	if err != nil {
		return fmt.Errorf("buat file profil: %w", err)
	}
	defer f.Close()

	if err := pprof.StartCPUProfile(f); err != nil {
		return fmt.Errorf("mulai profiling: %w", err)
	}
	defer pprof.StopCPUProfile()

	for i := 0; i < 200; i++ {
		if err := jalankanEndpointSimulasi(data); err != nil {
			return err
		}
	}
	return nil
}

// jalankanEndpointSimulasi harus melakukan kerja CPU sungguhan. Simulasi
// dengan time.Sleep tidak akan muncul di CPU profile sama sekali, karena
// goroutine yang tidur tidak memakai CPU.
func jalankanEndpointSimulasi(data []Permohonan) error {
	if _, err := json.Marshal(data); err != nil {
		return fmt.Errorf("marshal response: %w", err)
	}
	return nil
}

func buatDataUji(n int) []Permohonan {
	hasil := make([]Permohonan, n)
	for i := range hasil {
		hasil[i] = Permohonan{ID: i, Nomor: fmt.Sprintf("P-%06d", i), Status: "menunggu"}
	}
	return hasil
}

func main() {
	if err := LatihanOptimasiLengkap("cpu_sebelum.prof"); err != nil {
		log.Fatal(err)
	}
	// Analisis: go tool pprof cpu_sebelum.prof. Setelah hipotesis
	// dibentuk (langkah 4), diverifikasi dengan benchmark (langkah 5), dan
	// perbaikan diterapkan (langkah 6), jalankan ulang dengan
	// "cpu_sesudah.prof" lalu bandingkan: go tool pprof -diff_base
	// cpu_sebelum.prof cpu_sesudah.prof.
}
```

## In His Stack

Metodologi ini paling relevan justru saat menghadapi tekanan waktu — "endpoint ini lambat, perbaiki sekarang" adalah instruksi yang umum diterima, dan disiplin mengikuti urutan profiling-hipotesis-verifikasi (bukan langsung mencoba perbaikan pertama yang terpikirkan) justru **menghemat** waktu dalam skala lebih besar, karena mencegah tim menghabiskan berhari-hari mencoba perbaikan yang ternyata tidak menyentuh akar masalah sesungguhnya. Untuk koordinator teknis yang mengarahkan 10+ developer, metodologi yang konsisten ini juga memudahkan komunikasi lintas tim — "kami menemukan lewat profiling bahwa 60% waktu dihabiskan di parsing JSON" adalah klaim yang jauh lebih meyakinkan dan actionable dibanding "sepertinya endpoint ini lambat karena banyak hal".

## Trade-offs and When Not To Use It

Mengikuti metodologi lengkap ini untuk setiap masalah performa kecil adalah usaha berlebihan — untuk perbaikan yang jelas dan berisiko rendah (misalnya menambah index yang jelas-jelas hilang dari `EXPLAIN`, lihat [[../40 Databases/Reading EXPLAIN|Reading EXPLAIN]]), langsung menerapkan perbaikan dan mengamati dampaknya lebih praktis dibanding menjalankan seluruh siklus profiling formal. Metodologi penuh ini paling bernilai untuk masalah performa yang **tidak jelas** penyebabnya, atau untuk optimasi pada kode yang benar-benar kritis (hot path bervolume sangat tinggi) di mana kesalahan diagnosis bisa memakan waktu tim yang signifikan kalau diselesaikan lewat trial-and-error tanpa data.

## Common Mistakes

> [!warning] Jebakan
> Melompat langsung ke "perbaikan" tanpa mengukur baseline atau melakukan profiling terlebih dahulu — klaim "sudah lebih cepat" tidak bisa diverifikasi secara objektif tanpa data before-after yang konsisten.

> [!warning] Jebakan
> Menerapkan banyak perubahan sekaligus tanpa mengukur dampak masing-masing secara terpisah — kalau performa membaik, tidak jelas perubahan mana yang sebenarnya berkontribusi, dan kalau ada perubahan yang justru memperburuk sesuatu, sulit diidentifikasi di antara banyak perubahan yang digabung sekaligus.

> [!warning] Jebakan
> Berhenti setelah satu iterasi perbaikan tanpa mengukur ulang untuk verifikasi — asumsi bahwa perbaikan yang terlihat masuk akal secara teori pasti berhasil dalam praktik, tanpa konfirmasi data yang sebenarnya.

## Exercises

1. Jelaskan kenapa mengukur baseline sebelum melakukan perubahan apa pun adalah langkah yang tidak boleh dilewati dalam optimasi performa.
2. Kenapa profiling sebaiknya dilakukan sebelum menulis benchmark yang ditargetkan pada fungsi tertentu?
3. Kenapa menerapkan banyak perubahan sekaligus tanpa mengukur masing-masing secara terpisah bermasalah?
4. Desain terbuka: kamu menerima laporan "endpoint pencarian dokumen lambat" tanpa detail lebih lanjut. Jalankan seluruh metodologi di note ini secara naratif — jelaskan langkah demi langkah apa yang akan kamu lakukan dari menerima laporan ini sampai bisa mengonfirmasi perbaikan yang diterapkan benar-benar berdampak, sebutkan alat spesifik (pprof, benchmark, load test) yang dipakai di setiap langkah.

> [!success]- Kunci jawaban
> **1.** Tanpa baseline yang terukur dalam kondisi yang jelas dan konsisten (traffic yang sama, waktu yang sama, kondisi sistem yang sebanding), tidak ada dasar objektif untuk membandingkan "sebelum" dan "sesudah" — perbaikan yang terlihat berhasil bisa jadi kebetulan bersamaan dengan faktor lain yang berubah (traffic turun, komponen lain yang lebih cepat sesaat), bukan benar-benar disebabkan perubahan yang diterapkan. Baseline memberi titik pembanding yang solid untuk mengklaim perbaikan secara meyakinkan.
> **4.** (1) Ukur baseline latency endpoint pencarian dokumen dari metrik yang sudah ada (dashboard p50/p95/p99, lihat [[Latency Percentiles (p50, p95, p99)]]) atau, kalau belum ada, jalankan load test singkat untuk mendapat angka awal yang konsisten. (2) Tentukan dulu apakah waktunya habis untuk **menghitung** atau **menunggu**. Ambil CPU profile selama traffic normal atau load test (`go tool pprof http://host/debug/pprof/profile?seconds=30`). Kalau CPU aplikasi rendah dan profilnya tidak menunjukkan hotspot, sementara latency tinggi, waktunya habis untuk menunggu sesuatu di luar proses. (3) Cari yang ditunggu: span database di distributed tracing, metrik durasi query, atau slow query log MariaDB. Misalnya ternyata query pencarian memakai `LIKE '%kata%'` yang memindai seluruh tabel (lihat [[../40 Databases/Inverted Indexes and How Search Engines Work|Inverted Indexes and How Search Engines Work]]). Kalau sebaliknya CPU profile didominasi fungsi aplikasi (misalnya serialisasi hasil), lanjutkan dengan `top10` dan `list` di pprof. (4) Bentuk hipotesis spesifik: "full table scan akibat `LIKE` tanpa index yang bisa dipakai adalah penyebab utama". (5) Verifikasi dengan `EXPLAIN` pada query itu, dan pastikan full table scan benar-benar terjadi. (6) Terapkan perbaikan yang ditargetkan (full-text index atau search engine terpisah). (7) Ukur ulang latency endpoint yang sama dalam kondisi yang sebanding dengan baseline. (8) Kalau perbaikannya signifikan, jalankan load test untuk mengonfirmasi hasilnya bertahan di bawah beban production yang representatif.

## Self-Check

- Kenapa mengukur baseline adalah langkah pertama yang wajib sebelum optimasi apa pun?
- Kenapa profiling sebaiknya mendahului penulisan benchmark yang ditargetkan?
- Kenapa menerapkan banyak perubahan sekaligus menyulitkan verifikasi dampak masing-masing?
- Apa langkah terakhir yang mengonfirmasi perbaikan benar-benar siap untuk kondisi production?

## Connected Notes

- [[pprof Profiling]] — alat utama langkah 3 dan 7 dalam metodologi yang dijelaskan di note ini.
- [[Benchmarking in Go]] — alat verifikasi hipotesis pada langkah 5, mengukur fungsi terisolasi yang sudah dipersempit lewat profiling.
- [[Load Testing]] — konfirmasi akhir bahwa perbaikan bertahan pada skala production, langkah penutup metodologi ini.
- [[Capacity Planning]] — data dari latihan profiling menyeluruh sering menjadi input untuk perencanaan kapasitas yang lebih akurat.
- [[Latency Percentiles (p50, p95, p99)]] — metrik yang tepat untuk mengukur baseline dan hasil akhir, bukan sekadar rata-rata yang bisa menyesatkan.

## Further Reading

- Brendan Gregg, *Systems Performance: Enterprise and the Cloud* (edisi ke-2, 2020), dan artikelnya tentang **USE Method** (Utilization, Saturation, Errors) di brendangregg.com — metodologi sistematis untuk menemukan bottleneck resource, relevan lintas bahasa pemrograman.

## Catatan Saya

*Tulis di sini satu masalah performa nyata yang pernah kamu hadapi di kerjaan — jalankan mundur, apakah metodologi di note ini akan mengubah cara kamu mendiagnosisnya waktu itu.*
