---
title: Reading By Domain Depth
type: concept
level: junior
domain: foundations
status: mastered
difficulty: 1
est_minutes: 10
prerequisites: ["[[How To Read This Vault]]"]
next: []
tags: [backend, meta]
created: 2026-07-29
---

## Kenapa File Ini Ada

[[How To Read This Vault]] menjelaskan tiga cara membaca vault ini — linear (ikuti `01 Maps/Level N - ...md`), graph-driven, dan problem-driven. Ketiganya masih berlaku. Tapi cara vault ini **ditulis** berubah setelah Level 1 (seluruh 89 note junior) selesai: alih-alih menulis lebar dulu (semua domain di level intermediate, baru semua domain di level senior), penulisan sekarang berjalan **dalam per domain** — satu folder domain ditulis sampai kedalaman penuhnya (junior yang sudah ada → seluruh intermediate → seluruh senior) sebelum domain berikutnya disentuh. Alasannya tercatat di `CLAUDE.md` §14: pembaca secara eksplisit lebih suka menuntaskan satu topik sampai benar-benar dikuasai, daripada menyapu lebar di satu level lalu mengulang proses yang sama di level berikutnya.

File ini menjelaskan cara membaca yang cocok dengan pola penulisan itu — bukan pengganti [[How To Read This Vault]], tapi pelengkap yang menjawab pertanyaan konkret: "kalau vault ditulis per domain, gimana cara bacanya?"

## Cara Baca Lama vs Cara Baca Ini

**Cara lama (`01 Maps/Level N - ...md`)**: baca seluruh note junior lintas semua domain, baru seluruh note intermediate lintas semua domain, baru seluruh note senior. Cocok kalau seluruh vault (atau seluruh satu level) sudah selesai ditulis, karena setiap lompatan antar domain dijamin punya tujuan yang sudah ada isinya.

**Cara ini (per domain, habis-habisan)**: pilih **satu** folder domain (misalnya `40 Databases/`), baca `_Overview.md`-nya, lalu ikuti reading order di sana dari atas ke bawah — junior dulu (kemungkinan besar sudah kamu baca), lalu terus ke intermediate, lalu senior kalau domain itu punya tingkat senior sendiri — sebelum pindah ke domain lain sama sekali. Cocok justru **karena** vault ditulis dengan urutan yang sama: begitu kamu selesai membaca satu domain penuh, penulisannya juga baru saja selesai penuh untuk domain itu — tidak ada bagian yang "hilang" di tengah jalan.

## Domain Mana Duluan, dan Alasannya

Pertanyaan yang sering muncul di titik ini: "oke, satu domain habis-habisan sebelum pindah — tapi domain mana yang pertama, dan kenapa bukan domain lain?" Ini rekomendasiku sebagai mentor, bukan aturan kaku vault — kalau kamu punya alasan kuat menukar urutan (misalnya ada kebutuhan mendesak di kerjaan yang memaksa loncat ke `80 Security` lebih awal), lakukan saja. Tapi kalau kamu tidak punya alasan spesifik, ikuti urutan ini.

Per Agustus 2026, seluruh domain `10` sampai `90` — plus `92 Tools`, `94 Case Studies`, `95 Projects`, dan `99 Glossary` — sudah lengkap ditulis sampai kedalaman maksimalnya. Jadi pertanyaan ini sekarang murni soal urutan baca yang paling masuk akal, bukan lagi soal "domain mana yang sudah ada isinya".

**Urutan yang aku rekomendasikan, dan kenapa setiap perpindahan masuk akal:**

1. **`10 Foundations`** — mulai di sini karena inilah yang paling dekat dengan mesin (lihat prinsip penomoran di `CLAUDE.md` §5: angka kecil dekat dengan mesin, angka besar dekat dengan organisasi). Kamu sudah paham HTTP dan SQL, jadi bacaan ini akan terasa cepat — tapi TCP handshake, TLS, dan cara OS menangani koneksi adalah kosakata yang dipakai diam-diam di hampir semua domain lain (terutama `70` dan `60`), jadi lebih baik dipastikan solid sekarang daripada ditambal belakangan.
2. **`20 Go Language`** — bahasa dulu, sebelum runtime-nya. Kamu akan menulis (atau membaca) kode Go di *setiap* domain berikutnya lewat bagian "In Go" tiap note — belajar interface, generics, dan error handling sambil juga mencoba mencerna goroutine scheduler di domain lain hanya memecah perhatian tanpa perlu.
3. **`50 Concurrency and Performance`** — lanjutan langsung dari `20`, dan ini domain yang **secara harfiah** menjawab goal keduamu di `CLAUDE.md` §2: "mastery in Go". Goroutine, channel, GMP scheduler, pprof — bacaan ini punya momentum penuh kalau langsung disambung dari bahasa dasarnya, bukan diselingi domain lain dulu.
4. **`40 Databases`** — sengaja ditaruh setelah Go, bukan sebelumnya, supaya kode Go di bagian "In Go" tiap note database (connection pooling, query dengan `sqlx`) langsung terasa seperti kode yang bisa kamu tempel ke kerjaan nyata, bukan pseudo-code. Ini juga domain dengan jarak paling dekat ke pekerjaan harianmu (MariaDB) — momentum belajar biasanya lebih kuat kalau langsung bisa dicocokkan ke sistem yang kamu pegang setiap hari.
5. **`30 APIs and Web`** — domain terbesar, dan langsung menjawab instruksi paling eksplisit di `CLAUDE.md` §2: integrasi sistem-ke-sistem dengan partner eksternal dibobot berat karena itulah pekerjaan hariannya. Ditaruh di sini (bukan lebih awal) karena separuh isinya — resilience pattern, messaging semantics — baru benar-benar berbunyi setelah kamu paham concurrency (`50`) dan trade-off database (`40`) yang mendasarinya.
6. **`90 Architecture and Design`** — begitu kamu sudah membangun dan mengintegrasikan cukup banyak (`20`–`30`), pertanyaan "bagaimana menyusun ini semua dengan baik" jadi jauh lebih konkret dibanding dibaca lebih awal sebagai teori abstrak. Tingkat seniornya (API Governance, RFC process, mentoring) juga langsung relevan untuk peran koordinator teknismu — tidak perlu menunggu apa pun lagi untuk mulai dipraktikkan.
7. **`80 Security`** — dibaca setelah `30` dan `90` karena banyak isinya (OAuth2, webhook security, API governance untuk standar keamanan lintas tim) baru punya konteks penuh setelah kedua domain itu. Sudut compliance-nya (STRIDE, audit trail, compliance trail pemerintah) juga eksplisit relevan untuk konteks kerjamu di layanan hukum pemerintah.
8. **`70 Infrastructure and Delivery`** — klaster observability-nya (metrics, tracing, dashboard) jauh lebih mudah dicerna setelah kamu sudah paham `pprof` dan performance dari `50` — konsepnya bertumpuk, bukan berdiri sendiri. Ini juga domain yang paling cocok dengan ekosistem harianmu (Kubernetes, Jenkins).
9. **`60 Distributed Systems`** — ditaruh terakhir dengan sengaja, bukan karena kurang penting (ini justru domain yang paling serius untuk ambisi S2 distributed systems-mu, lihat `CLAUDE.md` §2), tapi karena hampir setiap note di domain ini secara eksplisit dibangun di atas fondasi dari `50`, `30`, `70`, `80`, dan `40` — membacanya di akhir berarti nyaris tidak ada forward-reference yang belum kamu kenal, jadi setiap konsep berat (Raft, saga, CAP theorem) langsung punya tempat berpijak yang sudah familiar.

**Di mana `92 Tools`, `94 Case Studies`, `95 Projects`, dan `99 Glossary` masuk?** Ketiga folder pertama bukan "domain kesepuluh" yang dibaca terpisah — selipkan sepanjang jalan, persis seperti dijelaskan di [[How To Read This Vault]]: begitu selesai satu concept note yang punya implementasi nyata (misalnya [[../60 Distributed Systems/Change Data Capture|Change Data Capture]]), langsung lompat baca tool note-nya ([[../92 Tools/Debezium|Debezium]]), lalu case study yang memakainya kalau ada. Glossary otomatis kamu temui lewat link kemunculan pertama sebuah istilah — tidak perlu dijadwalkan sendiri.

**Kapan mengerjakan `95 Projects`?** Ketiganya adalah checkpoint, bukan bacaan — kerjakan, jangan cuma dibaca.
- **Project 1** (REST API terautentikasi) bisa langsung dikerjakan begitu kamu selesai level junior di `10`, `20`, sebagian `30`/`40`/`80`/`90` — semua prasyaratnya junior, jadi kamu tidak perlu menunggu sampai langkah 6 di atas selesai.
- **Project 2** (worker pool, cache, Kafka) paling pas setelah langkah 8 — butuh `50` (worker pool), `30` (messaging), dan klaster observability `70` sekaligus.
- **Project 3** (sistem terdistribusi kecil) adalah penutup sesungguhnya — kerjakan setelah langkah 9, karena saga, outbox, dan chaos engineering-nya memang didesain sebagai ujian akhir seluruh urutan ini.

## Langkah Konkret

1. Buka `_Overview.md` domain yang sedang kamu baca (misalnya `40 Databases/_Overview.md`). Bagian "Reading Order" di situ sudah mengelompokkan note per level (Junior / Intermediate / Senior) secara berurutan — ikuti persis urutan itu, dari nomor 1 sampai selesai, tanpa loncat.
2. Kalau domain itu **tidak** punya bagian "Senior" di `_Overview.md`-nya (contoh: `40 Databases` sengaja hanya junior+intermediate), itu bukan berarti topiknya berhenti di situ — cek bagian "Bagaimana Domain Ini Terhubung Dengan Domain Lain" di `_Overview.md` yang sama, biasanya sudah menunjuk domain lain (paling sering `60 Distributed Systems`) tempat kelanjutan level seniornya berada.
3. Setelah satu domain tuntas dibaca dari junior sampai kedalaman maksimalnya, pindah ke domain berikutnya sesuai urutan di atas.
4. Sepanjang jalan, selipkan tool note dan case study yang relevan begitu concept note yang mengimplementasikannya selesai dibaca (lihat bagian sebelumnya) — jangan menunda seluruh `92 Tools`/`94 Case Studies` sampai akhir.

## Kalau Kamu Ingin Melihat Bentuk Lengkap Sebuah Topik Sekaligus

Karena domain folder sudah dirancang mencampur junior/intermediate/senior dalam satu tempat (lihat CLAUDE.md §5), tidak perlu berpindah folder sama sekali untuk melihat perjalanan lengkap satu topik dari dasar sampai dalam — cukup gulir `_Overview.md` domain itu dari atas ke bawah. Bandingkan ini dengan cara baca lama yang mengharuskan membuka tiga file peta berbeda (`Level 1`, `Level 2`, `Level 3`) dan mencari-cari bagian mana di masing-masing yang membahas domain yang sama.

## Connected Notes

- [[How To Read This Vault]] — tiga cara baca dasar (linear, graph-driven, problem-driven) yang tetap berlaku; file ini hanya menambahkan cara keempat yang cocok dengan pola penulisan per domain.
- [[Backlog]] — riwayat urutan domain saat vault ini ditulis, kalau kamu penasaran bagaimana rekomendasi di atas dibandingkan dengan urutan penulisannya (keduanya hampir sama persis, dan bukan kebetulan).
- [[Progress Tracker]] — tempat menandai status baca (`unread`/`learning`/`understood`/`mastered`) tidak berubah, terlepas dari cara baca mana yang dipakai.
- [[Vault Manifest]] — daftar lengkap note per domain, termasuk yang belum ditulis; dipakai saat mengecek apakah sebuah domain benar-benar sudah tuntas atau masih ada sisa.

## Catatan Saya

*Tulis di sini kalau kamu menemukan urutan baca yang menurutmu lebih masuk akal dari yang disarankan file ini, untuk domain tertentu.*
