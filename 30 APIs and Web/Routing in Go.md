---
title: Routing in Go
type: concept
level: junior
domain: apis
status: unread
difficulty: 2
est_minutes: 13
prerequisites: ["[[net-http Handlers and Middleware]]"]
next: ["[[Context Propagation in HTTP Servers]]", "[[Timeouts in HTTP Servers]]"]
tags: [backend, go, apis]
created: 2026-07-26
---

## TL;DR

Routing mencocokkan sebuah request masuk (method + path) ke handler yang tepat. `http.ServeMux` bawaan Go (sejak versi yang mendukung pattern method dan wildcard path, misalnya `"GET /dokumen/{id}"`) sudah menangani mayoritas kebutuhan routing tanpa dependency tambahan sama sekali. Router pihak ketiga (seperti `chi`) menambah kenyamanan untuk kasus yang lebih kompleks — pengelompokan rute dengan middleware bersama, pattern matching yang lebih ekspresif — tapi keputusan memakainya sebaiknya berdasarkan **kompleksitas struktur routing yang sungguh dibutuhkan**, bukan sekadar kebiasaan lama sebelum stdlib mendukung fitur ini secara native.

## The Problem

Bayangkan sebuah tim di versi Go yang lebih lama (sebelum `http.ServeMux` mendukung pattern method dan wildcard secara native) menulis routing manual — memecah string path dengan `strings.Split`, memeriksa method lewat `switch r.Method`, dan mencocokkan segmen path satu per satu secara manual. Ini bekerja untuk beberapa rute pertama, tapi seiring API bertambah puluhan endpoint dengan variasi path yang kompleks, logic manual ini menjadi rapuh — mudah salah menangani kasus tepi (trailing slash, segmen kosong), dan sulit dipelihara.

Di sisi lain, bayangkan tim lain yang sudah bermigrasi ke versi Go modern (dengan `ServeMux` yang sudah mendukung method dan wildcard native) tapi tetap mempertahankan dependency router pihak ketiga yang berat, murni karena "sudah terbiasa begitu" — menambah dependency yang sebenarnya tidak lagi dibutuhkan untuk kebutuhan routing mereka yang sesungguhnya sederhana.

## Intuition

Bayangkan routing seperti **sistem sortir pos** yang mencocokkan kombinasi alamat dan jenis pengiriman (method + path) sebuah amplop ke jalur pengiriman yang benar. Kantor pos kecil bisa cukup dengan aturan sortir manual sederhana ("cek huruf pertama nama jalan"); hub pengiriman besar dengan struktur jauh lebih kompleks butuh sistem conveyor otomatis (router library penuh).

Analogi "kantor pos kecil vs hub besar" ini bisa menyiratkan bahwa **volume** trafik yang menentukan pilihan — padahal faktor sesungguhnya biasanya adalah **kompleksitas struktural** aturan routing yang dibutuhkan (pengelompokan bertingkat, middleware berbeda per grup rute, constraint pattern yang rumit), bukan seberapa banyak request yang diterima. Sebuah API dengan trafik rendah tapi struktur routing yang sangat kompleks bisa lebih diuntungkan router library dibanding API bertrafik tinggi dengan struktur rute yang sederhana dan datar.

## How It Works

```go
mux := http.NewServeMux()
mux.HandleFunc("GET /dokumen/{id}", getDokumenHandler)
mux.HandleFunc("PATCH /dokumen/{id}", updateDokumenHandler)
mux.HandleFunc("GET /pemohon/{pemohonID}/dokumen", listDokumenByPemohonHandler)
```

`r.PathValue("id")` di dalam handler mengambil nilai wildcard yang cocok dari path. Saat beberapa pattern terdaftar bisa sama-sama cocok dengan satu request, `ServeMux` punya aturan presedensi (pattern yang lebih spesifik umumnya menang) — detail persisnya sebaiknya diperiksa langsung di dokumentasi resmi versi Go yang dipakai, bukan ditebak, karena aturan ini bisa halus dan berpengaruh nyata saat pattern-pattern yang terdaftar tumpang tindih.

## In Go

Mengelompokkan rute terkait tanpa dependency tambahan, cukup dengan function biasa:

```go
func registerDokumenRoutes(mux *http.ServeMux, svc *DokumenService) {
    mux.HandleFunc("GET /dokumen/{id}", svc.handleGet)
    mux.HandleFunc("PATCH /dokumen/{id}", svc.handleUpdate)
    mux.HandleFunc("DELETE /dokumen/{id}", svc.handleDelete)
}

func registerPemohonRoutes(mux *http.ServeMux, svc *PemohonService) {
    mux.HandleFunc("GET /pemohon/{id}", svc.handleGet)
    mux.HandleFunc("GET /pemohon/{id}/dokumen", svc.handleListDokumen)
}

func main() {
    mux := http.NewServeMux()
    registerDokumenRoutes(mux, dokumenService)
    registerPemohonRoutes(mux, pemohonService)

    handler := chain(mux, loggingMiddleware, recoverMiddleware) // lihat net-http Handlers and Middleware
    http.ListenAndServe(":8080", handler)
}
```

Pola `registerXxxRoutes` ini memberi pengelompokan yang cukup rapi tanpa perlu fitur "grouping" bawaan dari router pihak ketiga — untuk struktur rute yang tidak terlalu kompleks (tidak butuh middleware yang berbeda per grup, misalnya), ini sudah cukup.

## In His Stack

**Yii2** dengan `UrlManager` dan array `rules` di konfigurasinya adalah pendekatan yang jauh lebih deklaratif — rute didefinisikan sebagai data konfigurasi yang dipetakan ke controller/action, bukan dipanggil sebagai baris kode seperti `mux.HandleFunc`. Pergeseran ke Go berarti routing jadi lebih "kode-sentris" — didaftarkan lewat pemanggilan function langsung, bukan array konfigurasi — yang untuk sebagian orang terasa kurang deklaratif, tapi lebih mudah ditelusuri langsung (tidak ada lapisan konfigurasi tambahan yang perlu "diterjemahkan" runtime).

## Trade-offs and When Not To Use It

`http.ServeMux` bawaan (versi modern) menutupi kebutuhan mayoritas API tanpa dependency tambahan. Router pihak ketiga menambah kenyamanan nyata untuk struktur routing yang benar-benar kompleks — banyak grup rute bersarang, masing-masing butuh kombinasi middleware yang berbeda, atau constraint pattern yang lebih kaya dari yang didukung stdlib. Menambahnya "hanya karena kebiasaan" tanpa kebutuhan struktural yang nyata adalah dependency yang tidak perlu.

## Common Mistakes

> [!warning] Jebakan
> Menulis parsing routing manual (memecah string path, switch method) di versi Go yang stdlib-nya belum mendukung pattern matching yang dibutuhkan, alih-alih memakai router library yang tersedia untuk versi itu — logic manual seperti ini rapuh dan sulit dipelihara seiring rute bertambah.

> [!warning] Jebakan
> Mempertahankan dependency router pihak ketiga yang berat murni karena kebiasaan, meski sudah bermigrasi ke versi Go yang stdlib mux-nya sudah cukup untuk kebutuhan routing yang sesungguhnya sederhana.

> [!warning] Jebakan
> Tidak memahami aturan presedensi saat beberapa pattern rute yang terdaftar bisa sama-sama cocok dengan satu request, menebak-nebak pattern mana yang akan dipilih alih-alih memeriksa dokumentasi resmi — bisa menyebabkan kebingungan soal handler mana yang sebenarnya dipanggil.

## Exercises

1. Apa yang ditambahkan `http.ServeMux` versi modern dibanding versi lama yang hanya mencocokkan path tanpa method?
2. Kapan router pihak ketiga benar-benar memberi manfaat dibanding stdlib mux?
3. Kenapa keputusan memakai router library sebaiknya berdasarkan kompleksitas struktural, bukan volume trafik semata?
4. Desain terbuka: sebuah API sedang berkembang dari 10 endpoint jadi lebih dari 100 endpoint, terorganisir dalam beberapa grup resource (dokumen, pemohon, permohonan, laporan) yang masing-masing butuh kombinasi middleware berbeda (misalnya laporan butuh middleware rate limiting tambahan yang tidak dibutuhkan endpoint lain). Rancang keputusan apakah tetap memakai stdlib `ServeMux` atau bermigrasi ke router library, dengan alasan konkret.

> [!success]- Kunci jawaban
> Dengan lebih dari 100 endpoint dan kebutuhan middleware yang berbeda **per grup resource** (bukan seragam untuk semua endpoint), ini adalah titik di mana router library dengan dukungan grouping middleware bawaan mulai memberi manfaat nyata — stdlib `ServeMux` bisa menangani ini juga (dengan pola `registerXxxRoutes` plus membungkus grup tertentu dengan middleware tambahan secara manual), tapi router library biasanya menyediakan API yang lebih ringkas untuk pola "grup rute + middleware khusus grup itu" ini. Keputusan konkretnya bergantung pada apakah kompleksitas ini akan terus bertambah (migrasi ke router library lebih terbayar untuk investasi jangka panjang) atau ini kompleksitas puncak yang tidak akan tumbuh lebih jauh (stdlib plus sedikit boilerplate tambahan tetap memadai) — bukan keputusan yang harus diambil hanya berdasarkan jumlah endpoint semata.

## Self-Check

- Apa yang ditambahkan pattern method dan wildcard path di `ServeMux` modern?
- Kapan router pihak ketiga benar-benar memberi manfaat?
- Kenapa kompleksitas struktural lebih relevan daripada volume trafik untuk keputusan ini?
- Apa risiko tidak memahami aturan presedensi pattern routing?

## Connected Notes

- [[net-http Handlers and Middleware]] — prasyarat: handler dan middleware yang didaftarkan lewat router di note ini.
- [[Context Propagation in HTTP Servers]] — nilai yang di-set middleware sering diakses lewat context di handler yang dipilih router.
- [[Resource Modelling]] — struktur URL yang dibahas di domain resource modelling langsung menentukan bagaimana rute disusun di sini.
- [[Timeouts in HTTP Servers]] — konfigurasi timeout server yang bekerja berdampingan dengan router dan handler.

## Further Reading

- Dokumentasi resmi package `net/http`, function `ServeMux.Handle` (pkg.go.dev/net/http) — detail aturan presedensi pattern yang akurat sesuai versi Go yang dipakai.

## Catatan Saya

*Tulis di sini apakah service Go-mu memakai stdlib ServeMux atau router pihak ketiga, dan alasan pemilihannya.*
