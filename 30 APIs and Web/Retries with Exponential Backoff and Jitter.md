---
title: Retries with Exponential Backoff and Jitter
type: concept
level: intermediate
domain: apis
status: unread
difficulty: 3
est_minutes: 14
prerequisites: ["[[Timeout Budgets]]", "[[Idempotent Consumers]]"]
next: ["[[Circuit Breakers]]"]
tags: [backend, apis, go, resilience]
created: 2026-08-02
---

## TL;DR

Retry naif — mencoba ulang segera setelah gagal, tanpa jeda — sering memperburuk masalah yang sedang ia coba selesaikan, terutama ketika kegagalan disebabkan layanan tujuan sedang kewalahan: seribu client yang sama-sama retry serentak justru menambah beban tepat pada layanan yang sedang berjuang pulih. **Exponential backoff** memperbaiki ini dengan menaikkan jeda antar percobaan secara eksponensial (1 detik, 2 detik, 4 detik, 8 detik, dan seterusnya), memberi layanan tujuan waktu untuk pulih. **Jitter** — variasi acak kecil yang ditambahkan ke setiap jeda — menyelesaikan masalah kedua yang lebih halus: tanpa jitter, banyak client yang gagal pada saat bersamaan akan tetap retry pada saat yang **hampir sama** persis, sekadar bergeser waktunya secara sinkron, menciptakan gelombang beban berulang alih-alih menyebar merata.

## The Problem

Layanan verifikasi dokumen eksternal yang dipakai sistem legal-services mengalami downtime singkat selama tiga puluh detik akibat deployment mereka sendiri. Seratus consumer Go yang sedang memproses dokumen pada saat itu semuanya menerima error koneksi pada detik yang hampir sama, dan semuanya ditulis dengan logika retry sederhana: `time.Sleep(1 * time.Second)` lalu coba lagi. Begitu layanan verifikasi kembali online, keseratus consumer itu mengirim request secara bersamaan lagi (karena mereka semua menunggu durasi tetap yang sama), membebani layanan yang baru saja pulih dengan lonjakan traffic mendadak — persis yang disebut **retry storm**, kondisi di mana usaha untuk pulih dari kegagalan justru memicu kegagalan baru yang lebih besar.

Masalah kedua: retry dengan jeda tetap 1 detik tidak memberi ruang bagi layanan yang butuh waktu lebih lama untuk pulih — kalau downtime berlangsung tiga puluh detik, consumer yang retry setiap 1 detik akan mencoba tiga puluh kali dalam periode itu, semuanya sia-sia sampai tepat layanan itu benar-benar kembali, membebani jaringan dan layanan tujuan dengan percobaan yang secara statistik hampir pasti gagal di detik-detik awal downtime.

## Intuition

Cara paling mudah memahaminya lewat dua orang yang mencoba menelepon nomor yang sedang sibuk. Retry naif seperti menelepon ulang setiap detik tanpa henti — kalau nomor itu sibuk karena memang sedang menerima banyak panggilan, menelepon lebih sering hanya menambah antrean panggilan yang harus ditangani, memperlambat semua orang termasuk diri sendiri. Exponential backoff seperti menunggu lebih lama setiap kali gagal — coba lagi setelah satu menit, lalu dua menit, lalu empat menit — memberi waktu bagi pihak yang dihubungi untuk menyelesaikan panggilan-panggilan lain lebih dulu. Jitter seperti tidak semua orang menunggu **persis** durasi yang sama — kalau seratus orang semuanya menunggu tepat dua menit lalu menelepon lagi bersamaan, mereka tetap menciptakan lonjakan panggilan serentak; menambah variasi kecil (menunggu antara 100 dan 140 detik, bukan tepat 120 detik) menyebar panggilan-panggilan itu, mengurangi lonjakan.

Analogi ini tidak benar-benar bocor di sini — perilaku retry storm di sistem terdistribusi memang persis fenomena "semua orang menelepon ulang bersamaan" yang terjadi nyata di sistem telepon lama, salah satu alasan pola ini sudah dipelajari lama sebelum era microservices.

## How It Works

```mermaid
flowchart LR
    A["Percobaan 1: gagal"] -->|"tunggu ~1s + jitter"| B["Percobaan 2: gagal"]
    B -->|"tunggu ~2s + jitter"| C["Percobaan 3: gagal"]
    C -->|"tunggu ~4s + jitter"| D["Percobaan 4: berhasil"]
```

Diagram ini menunjukkan jeda yang berlipat ganda di setiap percobaan (exponential backoff), dengan tambahan variasi acak kecil di setiap jeda (jitter) yang tidak terlihat di diagram tapi selalu ada di implementasi nyata. Rumus dasar backoff eksponensial: `jeda = basis × faktor_pengali^percobaan`, biasanya dengan **batas atas** (cap) supaya jeda tidak tumbuh tak terbatas untuk kegagalan yang berlangsung sangat lama.

Ada beberapa varian jitter yang umum dipakai, masing-masing dengan trade-off:

- **Full jitter** — jeda dipilih acak antara 0 dan nilai backoff eksponensial penuh. Paling efektif menyebar beban, tapi variasinya besar sehingga sebagian percobaan bisa retry lebih cepat dari yang diharapkan.
- **Equal jitter** — separuh dari nilai backoff tetap, separuh lagi acak. Kompromi antara prediktabilitas dan penyebaran beban.
- **Decorrelated jitter** — jeda berikutnya dihitung berdasarkan jeda percobaan sebelumnya dikali faktor acak, cenderung menghasilkan penyebaran yang lebih baik untuk kasus retry berulang dalam jumlah banyak.

## In Go

```go
package main

import (
	"context"
	"errors"
	"fmt"
	"math/rand/v2"
	"time"
)

const (
	percobaanMaksimal = 5
	jedaBasis          = 1 * time.Second
	jedaMaksimal        = 30 * time.Second
)

func panggilDenganRetry(ctx context.Context, operasi func(context.Context) error) error {
	var errTerakhir error

	for percobaan := 0; percobaan < percobaanMaksimal; percobaan++ {
		if percobaan > 0 {
			jeda := hitungBackoffDenganJitter(percobaan)
			select {
			case <-time.After(jeda):
			case <-ctx.Done():
				return fmt.Errorf("dibatalkan saat menunggu retry: %w", ctx.Err())
			}
		}

		err := operasi(ctx)
		if err == nil {
			return nil
		}

		if !bolehDiretry(err) {
			// Kegagalan permanen (misalnya validasi input salah)
			// tidak akan pernah berhasil walau diulang — hentikan
			// retry segera, jangan buang waktu menunggu.
			return fmt.Errorf("kegagalan permanen, tidak diulang: %w", err)
		}
		errTerakhir = err
	}

	return fmt.Errorf("gagal setelah %d percobaan: %w", percobaanMaksimal, errTerakhir)
}

func hitungBackoffDenganJitter(percobaan int) time.Duration {
	// Exponential backoff: 1s, 2s, 4s, 8s, ... dibatasi jedaMaksimal.
	backoff := jedaBasis * time.Duration(1<<uint(percobaan))
	if backoff > jedaMaksimal {
		backoff = jedaMaksimal
	}

	// Full jitter: pilih acak antara 0 dan backoff penuh.
	return time.Duration(rand.Int64N(int64(backoff)))
}

func bolehDiretry(err error) bool {
	// Contoh sederhana: kegagalan jaringan layak diretry,
	// error validasi tidak. Implementasi sungguhan biasanya
	// memeriksa tipe error atau status code secara eksplisit.
	var errJaringan *NetworkError
	return errors.As(err, &errJaringan)
}
```

Perhatikan `bolehDiretry` — retry tanpa membedakan jenis kegagalan adalah kesalahan tersendiri: mengulang error validasi input (yang pasti akan gagal lagi persis sama) hanya membuang waktu dan resource tanpa peluang berhasil sama sekali.

## In His Stack

Retry dengan backoff dan jitter relevan langsung untuk setiap panggilan ke layanan eksternal yang tidak sepenuhnya bisa diandalkan — layanan OCR, gateway pembayaran, API instansi lain — kasus yang sudah dibahas dari sudut pandang desain umum di [[Handling an Unreliable Counterparty]]. Penting diingat: retry hanya aman diterapkan pada operasi yang **idempotent** (dibahas di [[Idempotent Consumers]]) — mengulang panggilan yang tidak idempotent, seperti "buat transaksi baru", tanpa idempotency key yang tepat bisa menghasilkan efek ganda yang sama berbahayanya dengan duplikasi pesan yang dibahas di note-note messaging sebelumnya.

## Trade-offs and When Not To Use It

Retry dengan backoff menambah latency untuk kasus kegagalan — trade-off yang harus disadari sejak awal, terutama kalau ada [[Timeout Budgets]] yang ketat di lapisan luar: retry lima kali dengan backoff eksponensial bisa dengan mudah menghabiskan seluruh budget waktu yang tersedia untuk satu request, membuat retry sendiri jadi penyebab timeout, bukan solusinya. Untuk operasi yang tidak idempotent dan tidak punya mekanisme idempotency key, retry otomatis sebaiknya dihindari sepenuhnya sampai mekanisme itu ada — lebih baik gagal cepat dan jelas daripada retry yang berisiko menghasilkan efek samping ganda.

## Common Mistakes

> [!warning] Jebakan
> Menerapkan retry dengan jeda tetap (bukan eksponensial) untuk kegagalan yang disebabkan layanan tujuan kewalahan — jeda tetap tidak memberi cukup waktu pemulihan dan berkontribusi pada retry storm yang memperpanjang downtime, bukan mempercepat pemulihan.

> [!warning] Jebakan
> Menerapkan exponential backoff tanpa jitter, sehingga banyak client yang gagal bersamaan tetap retry pada waktu yang hampir sama persis di setiap putaran — backoff eksponensial saja tidak cukup untuk mencegah retry storm; jitter adalah bagian yang sama pentingnya, bukan tambahan opsional.

> [!warning] Jebakan
> Meretry operasi yang tidak idempotent tanpa mekanisme deteksi duplikasi di sisi penerima — kegagalan jaringan yang terjadi **setelah** operasi sebenarnya berhasil di sisi server tapi response-nya hilang di jalan akan membuat retry mengulang operasi yang sebenarnya sudah berhasil.

## Exercises

1. Jelaskan kenapa retry dengan jeda tetap bisa memperburuk retry storm dibanding tidak retry sama sekali.
2. Bandingkan full jitter dan equal jitter dari sisi trade-off prediktabilitas jeda versus efektivitas menyebar beban.
3. Sebuah operasi mengembalikan error validasi input (data yang dikirim memang salah format). Jelaskan kenapa menerapkan retry pada kasus ini adalah kesalahan, dan bagaimana kode retry sebaiknya membedakan kasus ini dari kegagalan jaringan sementara.
4. **(Open-ended)** Timmu punya timeout budget total 5 detik untuk sebuah panggilan ke layanan eksternal (dari [[Timeout Budgets]]), dan ingin menerapkan retry dengan exponential backoff di dalam budget itu. Rancang parameter retry (jumlah percobaan maksimal, jeda basis, jeda maksimal) yang masuk akal supaya retry tidak menghabiskan seluruh budget tanpa sisa waktu untuk percobaan terakhir benar-benar dieksekusi.

> [!success]- Kunci jawaban
> Untuk soal 4: dengan budget 5 detik, retry harus dirancang supaya total waktu tunggu antar percobaan jauh lebih kecil dari 5 detik, menyisakan waktu untuk percobaan itu sendiri dieksekusi (bukan hanya menunggu). Misalnya: jeda basis 200ms, faktor pengali 2, jeda maksimal 1 detik, dengan maksimal tiga percobaan — total waktu tunggu kasar sekitar 200ms + 400ms + 800ms (dengan jitter, bervariasi di sekitar itu) sekitar 1.4 detik, menyisakan lebih dari 3.5 detik untuk ketiga percobaan itu sendiri benar-benar dieksekusi. Parameter ini harus dihitung mundur dari budget yang tersedia, bukan dipilih dari angka default yang umum dipakai tanpa mempertimbangkan konteks — timeout budget yang sempit butuh parameter retry yang lebih agresif (jeda lebih pendek, percobaan lebih sedikit) dibanding budget yang longgar.

## Self-Check

- Kenapa retry naif tanpa backoff bisa memperburuk kegagalan yang disebabkan layanan tujuan kewalahan?
- Apa masalah yang diselesaikan jitter, yang tidak diselesaikan exponential backoff saja?
- Kenapa retry hanya aman diterapkan pada operasi yang idempotent?

## Connected Notes

- [[Timeout Budgets]] — retry harus dirancang di dalam batas timeout budget yang tersedia, bukan menghabiskannya tanpa sisa waktu eksekusi.
- [[Idempotent Consumers]] — prasyarat keamanan retry: operasi yang tidak idempotent berisiko menghasilkan efek ganda kalau diretry.
- [[Circuit Breakers]] — kelanjutan langsung: mekanisme yang menghentikan retry sama sekali ketika kegagalan sudah terbukti persisten, alih-alih terus mencoba.
- [[Handling an Unreliable Counterparty]] — retry dengan backoff adalah salah satu pertahanan konkret terhadap partner eksternal yang tidak sepenuhnya bisa diandalkan.

## Further Reading

- Artikel AWS Architecture Blog, "Exponential Backoff And Jitter" — sumber yang memperkenalkan istilah full/equal/decorrelated jitter secara luas di industri.

## Catatan Saya

*Tulis pertanyaanmu sendiri, contoh kode dari pekerjaanmu, atau bagian yang belum jelas di sini.*
