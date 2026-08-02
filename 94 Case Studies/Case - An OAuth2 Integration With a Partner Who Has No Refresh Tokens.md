---
title: Case - An OAuth2 Integration With a Partner Who Has No Refresh Tokens
type: case
level: intermediate
domain: architecture
status: unread
difficulty: 3
est_minutes: 14
source: synthesized
concepts: ["[[../80 Security/OAuth2 Overview|OAuth2 Overview]]", "[[../80 Security/JWT - Structure, Signature, and When It Is The Wrong Tool|JWT - Structure, Signature, and When It Is The Wrong Tool]]", "[[../30 APIs and Web/Designing an API for a Partner You Do Not Control|Designing an API for a Partner You Do Not Control]]"]
prerequisites: []
next: []
tags: [backend, architecture, security, gov-context, integration]
created: 2026-08-02
---

## The Situation

Sebuah integrasi dengan partner instansi memakai OAuth2 untuk otorisasi — tapi implementasi OAuth2 milik partner ini, karena keterbatasan sistem lama mereka, **tidak menyediakan refresh token** sama sekali. Access token yang mereka terbitkan berlaku 1 jam, dan satu-satunya cara mendapat token baru setelah kedaluwarsa adalah mengulang seluruh alur otorisasi dari awal (yang, dalam implementasi mereka, butuh interaksi manual — bukan panggilan API otomatis sederhana).

## What Was Actually Asked For

Permintaan tim internal: "integrasi ini harus bisa berjalan otomatis tanpa campur tangan manual setiap jam." Yang sebenarnya dibutuhkan bukan mengubah perilaku partner (di luar kendali) — tapi merancang cara sistem sendiri **beroperasi wajar** dalam batasan token yang hanya berlaku 1 jam tanpa mekanisme refresh otomatis standar.

## Options

### Option 1 — Minta manusia melakukan reotorisasi manual setiap jam

Terima batasan apa adanya, jadwalkan seseorang di tim untuk melakukan ulang proses otorisasi setiap jam. Jelas tidak realistis untuk operasi 24/7 — solusi ini disebutkan hanya sebagai baseline yang menunjukkan kenapa solusi otomatis benar-benar dibutuhkan.

### Option 2 — Otomasi alur otorisasi lewat scripting/headless browser

Kalau alur otorisasi partner sebenarnya berbasis web (form login), otomasi proses ini lewat script yang mensimulasikan interaksi manual (mengisi form, klik tombol) menggunakan headless browser. Secara teknis bisa berjalan otomatis tanpa manusia, tapi rapuh (bergantung struktur halaman web partner yang bisa berubah kapan saja tanpa pemberitahuan) dan berpotensi melanggar ketentuan penggunaan resmi partner yang mengasumsikan alur ini dilakukan manusia.

### Option 3 — Negosiasi ulang dengan partner untuk skema token yang lebih sesuai

Ajukan permintaan resmi ke partner untuk skema alternatif yang lebih ramah otomasi — client credentials grant (skema OAuth2 untuk komunikasi machine-to-machine yang tidak butuh refresh token manual sama sekali karena tidak melibatkan otorisasi pengguna individual), atau minimal token dengan masa berlaku lebih panjang untuk kebutuhan integrasi sistem-ke-sistem. Solusi paling matang, tapi butuh waktu dan bergantung kesediaan partner mengubah implementasi mereka.

## The Decision

Option 3 diajukan resmi ke partner (dengan justifikasi teknis yang jelas: kebutuhan integrasi sistem-ke-sistem, bukan otorisasi pengguna individual, sehingga client credentials grant secara konseptual lebih tepat dari awal), sambil Option 2 (otomasi terbatas dan hati-hati) diterapkan sebagai solusi sementara selama menunggu respons partner — dengan monitoring ketat dan siap beralih ke fallback manual kalau otomasi ini gagal karena perubahan di sisi partner. Constraint penentunya: kebutuhan operasional 24/7 tidak bisa menunggu proses negosiasi partner yang mungkin memakan waktu berbulan-bulan, tapi solusi sementara (Option 2) diterapkan dengan kesadaran penuh akan risikonya, bukan dianggap solusi permanen.

## What Breaks at 10x and 100x

Pada volume 10x (integrasi ini makin kritis, dipakai lebih banyak alur bisnis), risiko Option 2 (otomasi yang rapuh terhadap perubahan halaman partner) jadi semakin tidak bisa diterima — tekanan untuk menyelesaikan Option 3 (negosiasi resmi) makin besar, dan mungkin perlu eskalasi ke level manajemen antar instansi untuk mempercepat proses. Pada 100x (skenario di mana integrasi serupa dengan keterbatasan yang sama terjadi dengan banyak partner lain, bukan cuma satu), pola ini cukup umum untuk dijadikan standar penanganan — dokumentasi internal tentang "cara menangani partner tanpa refresh token" sebagai pengetahuan institusional, bukan solusi ad-hoc yang ditemukan ulang setiap kali.

## What You Would Do Differently

Menanyakan detail skema otorisasi partner (termasuk keberadaan refresh token) **sebelum** komitmen desain sistem final, bukan menemukan keterbatasan ini setelah integrasi mulai dibangun — pertanyaan yang seharusnya jadi bagian standar due diligence teknis di awal setiap integrasi baru dengan partner eksternal.

## Concepts Exercised

- [[../80 Security/OAuth2 Overview|OAuth2 Overview]] — konsep dasar OAuth2, termasuk perbedaan grant type yang relevan untuk kasus ini (client credentials vs authorization code).
- [[../80 Security/JWT - Structure, Signature, and When It Is The Wrong Tool|JWT - Structure, Signature, and When It Is The Wrong Tool]] — relevan memahami masa berlaku token dan implikasi desainnya.
- [[../30 APIs and Web/Designing an API for a Partner You Do Not Control|Designing an API for a Partner You Do Not Control]] — kerangka umum yang berlaku langsung: batasan partner yang tidak bisa diubah harus diadaptasi, bukan dilawan.

## Your Turn

Partner akhirnya merespons dan menyetujui permintaan client credentials grant (Option 3), tapi butuh waktu tiga bulan untuk implementasi di sisi mereka. Selama tiga bulan itu, tim harus terus memakai solusi otomasi sementara (Option 2). Apa yang seharusnya dipantau ketat selama periode transisi ini?

> [!success]- Kunci jawaban
> Tiga hal yang perlu dipantau ketat: (1) tingkat keberhasilan otomasi setiap jam — kegagalan yang mendadak meningkat adalah sinyal dini bahwa halaman/alur otorisasi partner mungkin berubah, butuh perbaikan segera sebelum integrasi benar-benar terputus; (2) waktu eksekusi otomasi — perlambatan bertahap bisa menandakan partner menambah langkah verifikasi baru (captcha, misalnya) yang akan membuat otomasi ini gagal total; (3) komunikasi proaktif dengan partner kalau ada perubahan besar di sisi mereka yang direncanakan selama periode tiga bulan ini — memastikan tim tidak terkejut oleh perubahan yang sebenarnya sudah diketahui partner tapi belum dikomunikasikan. Selain itu, siapkan fallback manual (Option 1, meski tidak ideal) sebagai jaring pengaman darurat kalau otomasi benar-benar berhenti berfungsi sebelum solusi permanen (client credentials) benar-benar siap dipakai.

## Catatan Saya

*Kalau kamu punya kejadian nyata dari kerjaan yang mirip tema ini, tulis sebagai case study baru dengan `source: personal`.*
