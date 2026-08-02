---
title: Project 2 - Worker Pool, Cache, and Streaming Service
type: project
level: intermediate
domain: architecture
status: unread
difficulty: 4
est_minutes: 0
concepts: ["[[../50 Concurrency and Performance/Worker Pools|Worker Pools]]", "[[../50 Concurrency and Performance/Cache-Aside, Write-Through, and Write-Behind|Cache-Aside, Write-Through, and Write-Behind]]", "[[../30 APIs and Web/Queue vs Log Semantics|Queue vs Log Semantics]]", "[[../70 Infrastructure and Delivery/The Three Pillars of Observability|The Three Pillars of Observability]]", "[[../30 APIs and Web/Circuit Breakers|Circuit Breakers]]"]
prerequisites: ["[[Project 1 - Authenticated REST API]]"]
next: ["[[Project 3 - A Small Distributed System]]"]
tags: [backend, architecture, go, concurrency, apis, infrastructure]
created: 2026-08-02
---

## Learning Objectives

Membangun service yang harus benar-benar berhadapan dengan konkurensi, beban tinggi, dan kegagalan parsial — bukan lagi sekadar CRUD sederhana seperti Project 1. Setelah menyelesaikan proyek ini, kamu harus bisa: merancang worker pool yang aman dan efisien, menerapkan strategi caching yang tepat, memproduksi dan mengonsumsi pesan Kafka dengan benar (termasuk menangani redelivery), menginstrumentasi observability yang benar-benar berguna saat insiden, dan menerapkan pola resiliensi yang membuat service ini tidak memperparah kegagalan yang sudah terjadi.

## The Brief

Bangun service yang memproses dokumen yang diunggah (melanjutkan dari Project 1, atau berdiri sendiri dengan endpoint upload sederhana) — setiap dokumen yang masuk perlu diproses (simulasikan proses berat: OCR sederhana, ekstraksi metadata, atau validasi format yang memakan waktu beberapa detik), hasilnya di-cache untuk pembacaan ulang cepat, dan setiap tahap pemrosesan mengirim event ke Kafka yang dikonsumsi service notifikasi terpisah (bisa berupa consumer sederhana dalam proyek yang sama).

## Requirements

**Worker pool**
- Endpoint upload menerima dokumen dan **tidak** memprosesnya secara sinkron di request-response yang sama — dokumen masuk antrean internal, diproses oleh sekumpulan worker goroutine dengan jumlah tetap (lihat [[../50 Concurrency and Performance/Worker Pools|Worker Pools]]).
- Worker pool harus bisa dihentikan dengan graceful shutdown — job yang sedang diproses saat sinyal shutdown diterima harus sempat selesai (atau dibatalkan dengan bersih), tidak dipotong paksa di tengah jalan.
- Terapkan batas antrean (backpressure) — kalau antrean penuh, permintaan baru ditolak dengan jelas (bukan menumpuk tanpa batas sampai memori habis).

**Caching**
- Hasil pemrosesan dokumen di-cache di Redis (lihat [[../50 Concurrency and Performance/Cache-Aside, Write-Through, and Write-Behind|Cache-Aside, Write-Through, and Write-Behind]]) — pilih strategi (cache-aside, write-through) dan jelaskan alasannya.
- TTL yang masuk akal, dan invalidasi eksplisit saat dokumen diproses ulang.

**Kafka producer dan consumer**
- Setiap kali status pemrosesan dokumen berubah (diterima, sedang diproses, selesai, gagal), publikasikan event ke topic Kafka.
- Bangun consumer terpisah (bisa dalam proses yang sama sebagai goroutine, atau binary terpisah) yang mengonsumsi event ini dan melakukan sesuatu dengannya (simulasikan: kirim notifikasi, log ke tempat terpisah) — consumer ini **harus idempoten** terhadap redelivery (lihat [[../30 APIs and Web/Idempotent Consumers|Idempotent Consumers]]).

**Observability**
- Structured logging dengan level yang tepat di seluruh alur.
- Metrik dasar RED (request rate, error rate, duration) untuk endpoint upload, dan metrik USE untuk worker pool (utilization, saturation dari panjang antrean).
- Endpoint `/metrics` yang bisa di-scrape Prometheus.

**Resiliensi**
- Kalau proyek ini memanggil dependency eksternal apa pun (bisa disimulasikan sebagai service palsu yang kadang sengaja gagal), terapkan circuit breaker (lihat [[../30 APIs and Web/Circuit Breakers|Circuit Breakers]]) dan retry dengan exponential backoff.

## Staged Milestones

1. **Worker pool dasar**: endpoint upload menerima file, memasukkannya ke antrean internal, worker goroutine memprosesnya (simulasi pemrosesan dengan `time.Sleep` yang bervariasi), hasil disimpan ke database.
2. **Caching**: hasil pemrosesan di-cache, endpoint baca mengecek cache dulu sebelum ke database, invalidasi saat data berubah.
3. **Kafka**: producer mempublikasikan event perubahan status, consumer terpisah mengonsumsinya dengan penanganan idempoten.
4. **Observability**: logging terstruktur, metrik RED/USE, endpoint `/metrics` berfungsi dan bisa di-scrape.
5. **Resiliensi dan Chaos ringan**: circuit breaker dan retry pada panggilan eksternal (atau simulasi), uji dengan sengaja mematikan dependency (Redis, Kafka) sesaat dan verifikasi service tidak crash total.

## Definition of Done

- Upload dokumen langsung mendapat respons cepat (tidak menunggu pemrosesan selesai), status bisa dicek lewat endpoint terpisah.
- Worker pool menangani beban konkuren (uji dengan mengunggah puluhan dokumen sekaligus) tanpa race condition (`go test -race` lulus).
- Cache benar-benar mengurangi beban database yang terlihat (verifikasi lewat log/metrik jumlah query database turun signifikan untuk pembacaan berulang).
- Consumer Kafka terbukti aman diproses ulang (uji manual: proses ulang event yang sama, pastikan tidak ada efek samping ganda).
- Dashboard (Grafana sederhana, atau minimal query PromQL yang didokumentasikan) menunjukkan metrik RED/USE yang masuk akal.
- Service tetap merespons (meski terdegradasi) saat salah satu dependency (Redis atau Kafka) dimatikan sesaat dalam pengujian manual.

## Self-Review Checklist

- [ ] Apakah worker pool benar-benar berhenti menerima job baru dan menunggu job berjalan selesai saat shutdown, bukan langsung mati paksa?
- [ ] Apakah ada race condition yang terlewat — sudah dijalankan `go test -race` di seluruh test suite?
- [ ] Apakah consumer Kafka benar-benar diverifikasi idempoten, bukan diasumsikan aman tanpa pengujian nyata?
- [ ] Apakah metrik yang diinstrumentasi benar-benar menjawab pertanyaan operasional nyata (bisa dipakai mendiagnosis masalah), bukan sekadar angka yang ada tapi tidak berguna?
- [ ] Apakah circuit breaker benar-benar diuji dengan mematikan dependency sungguhan, bukan hanya dibaca kodenya dan diasumsikan berfungsi?
- [ ] Apakah ada goroutine yang berpotensi bocor (dimulai tapi tidak pernah dijamin selesai atau dibatalkan)?

## Concepts Exercised

- [[../50 Concurrency and Performance/Worker Pools|Worker Pools]] — pola konkurensi inti yang membentuk arsitektur pemrosesan dokumen.
- [[../50 Concurrency and Performance/Cache-Aside, Write-Through, and Write-Behind|Cache-Aside, Write-Through, and Write-Behind]] — strategi caching yang diterapkan dan dijustifikasi.
- [[../30 APIs and Web/Queue vs Log Semantics|Queue vs Log Semantics]] — pemahaman Kafka sebagai log, relevan untuk desain consumer yang benar.
- [[../70 Infrastructure and Delivery/The Three Pillars of Observability|The Three Pillars of Observability]] — instrumentasi log, metrik yang diterapkan penuh.
- [[../30 APIs and Web/Circuit Breakers|Circuit Breakers]] — pola resiliensi yang diuji dengan skenario kegagalan nyata.

## Catatan Saya

*Tulis di sini strategi caching dan ukuran worker pool yang kamu pilih, dan bagaimana kamu memverifikasi keduanya benar-benar bekerja seperti diharapkan di bawah beban.*
