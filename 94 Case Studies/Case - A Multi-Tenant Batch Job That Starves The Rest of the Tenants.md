---
title: Case - A Multi-Tenant Batch Job That Starves The Rest of the Tenants
type: case
level: intermediate
domain: architecture
status: unread
difficulty: 3
est_minutes: 14
source: synthesized
concepts: ["[[../60 Distributed Systems/Multi-Tenancy|Multi-Tenancy]]", "[[../30 APIs and Web/Rate Limiting Algorithms|Rate Limiting Algorithms]]", "[[../30 APIs and Web/Bulkheads|Bulkheads]]"]
prerequisites: []
next: []
tags: [backend, architecture, distributed, gov-context]
created: 2026-08-02
---

## The Situation

Sistem multi-tenant yang melayani 13 instansi dari satu infrastruktur bersama menjalankan job batch malam hari untuk masing-masing instansi (menghasilkan laporan, menyinkronkan data). Satu instansi — yang menangani volume kasus jauh lebih besar dari 12 instansi lain gabungan — punya job batch yang butuh waktu jauh lebih lama untuk selesai. Suatu malam, job instansi besar ini mengalami masalah (data yang diproses lebih kompleks dari biasanya) dan berjalan sangat lama, menghabiskan hampir seluruh kapasitas resource bersama (CPU, koneksi database) — job instansi-instansi kecil lain yang seharusnya selesai dalam hitungan menit malah tertunda berjam-jam, dan sebagian gagal karena timeout menunggu resource yang tidak pernah tersedia.

## What Was Actually Asked For

Permintaan awal dari instansi kecil yang terdampak: "job kami harus tetap selesai tepat waktu." Kebutuhan sebenarnya adalah **isolasi resource** — satu tenant (instansi) yang bermasalah tidak seharusnya bisa menghabiskan kapasitas bersama sampai mengganggu tenant lain, terlepas seberapa besar atau kompleks beban kerja tenant itu.

## Options

### Option 1 — Tambah kapasitas infrastruktur secara keseluruhan

Naikkan kapasitas CPU dan koneksi database yang tersedia, memberi lebih banyak "ruang" untuk semua job berjalan bersamaan tanpa saling mengganggu. Solusi termudah, tapi tidak menyelesaikan akar masalah — kalau job instansi besar tumbuh lebih besar lagi (atau mengalami masalah serupa lagi), ia tetap bisa menghabiskan kapasitas tambahan itu, hanya menunda kapan masalah yang sama terjadi lagi, bukan mencegahnya.

### Option 2 — Bulkhead: alokasi resource terpisah per tenant

Terapkan pola [[../30 APIs and Web/Bulkheads|Bulkheads]] — setiap tenant mendapat kuota resource yang terisolasi (connection pool terpisah, batas CPU/memori terpisah kalau infrastruktur mendukung), memastikan job satu tenant yang bermasalah hanya menghabiskan **kuotanya sendiri**, tidak bisa menyentuh kuota tenant lain. Menyelesaikan akar masalah isolasi, tapi butuh kapasitas total yang dialokasikan lebih hati-hati (kuota tetap per tenant bisa berarti sebagian kapasitas menganggur saat tenant kecil tidak memakainya penuh).

### Option 3 — Antrean job dengan prioritas dan batas waktu (timeout) per job

Jalankan seluruh job batch lewat sistem antrean terpusat yang membatasi jumlah job yang boleh berjalan bersamaan, dengan batas waktu maksimum per job (job yang melebihi batas waktu dihentikan paksa, bukan dibiarkan terus berjalan tanpa batas) — mencegah satu job "liar" menghabiskan resource selamanya, meski tidak sepenuhnya mengisolasi resource per tenant seperti Option 2.

## The Decision

Kombinasi Option 2 dan 3 dipilih. Constraint penentunya: isolasi resource murni (Option 2) penting untuk mencegah tenant besar mengganggu tenant kecil dalam kondisi normal, tapi tidak cukup sendirian untuk menangani kasus job yang benar-benar "liar" (berjalan jauh lebih lama dari yang diharapkan bahkan dalam kuotanya sendiri) — batas waktu eksplisit (Option 3) memberi jaring pengaman tambahan yang mencegah satu job buruk terus berjalan tanpa henti, apa pun penyebabnya. Option 1 (tambah kapasitas) tetap dilakukan secara wajar sebagai bagian dari perencanaan kapasitas normal, tapi tidak dianggap solusi utama untuk masalah isolasi ini.

## What Breaks at 10x and 100x

Pada volume 10x (lebih banyak instansi bergabung ke sistem multi-tenant ini), kuota tetap per tenant (Option 2 versi sederhana) mulai tidak efisien — instansi baru yang kecil mendapat kuota yang sama besarnya dengan instansi besar yang sudah ada, atau sebaliknya kuota yang terlalu kecil untuk kebutuhan nyata mereka. Di titik ini, kuota dinamis berdasarkan ukuran/kebutuhan historis tenant (bukan angka tetap seragam) jadi pendekatan yang lebih matang. Pada 100x, isolasi di level job scheduling saja mungkin tidak cukup — pertimbangan model isolasi yang lebih ketat (lihat [[../60 Distributed Systems/Multi-Tenancy|Multi-Tenancy]], model database/schema terpisah per tenant) jadi relevan kalau kebutuhan isolasi meluas melampaui sekadar job batch, ke seluruh operasi sistem.

## What You Would Do Differently

Menetapkan kuota resource per tenant sejak sistem multi-tenant pertama kali dirancang, bukan menambahkannya reaktif setelah satu tenant besar terbukti bisa mengganggu yang lain — asumsi "semua tenant akan berperilaku wajar" adalah asumsi yang rapuh begitu jumlah dan keragaman ukuran tenant bertambah.

## Concepts Exercised

- [[../60 Distributed Systems/Multi-Tenancy|Multi-Tenancy]] — konsep dasar isolasi tenant yang jadi akar masalah dan solusi kasus ini.
- [[../30 APIs and Web/Rate Limiting Algorithms|Rate Limiting Algorithms]] — prinsip pembatasan yang relevan diterapkan untuk membatasi laju/jumlah job per tenant.
- [[../30 APIs and Web/Bulkheads|Bulkheads]] — pola isolasi resource yang jadi inti solusi kasus ini.

## Your Turn

Setelah bulkhead diterapkan (kuota resource terpisah per tenant), instansi besar mengeluh job mereka sekarang **lebih lambat** dari sebelumnya, karena dibatasi kuota yang lebih kecil dari kapasitas yang dulu bisa mereka pakai bebas. Bagaimana ini seharusnya ditangani?

> [!success]- Kunci jawaban
> Ini adalah trade-off yang memang harus dikomunikasikan eksplisit, bukan dianggap sebagai bug — isolasi yang melindungi tenant lain dari gangguan berarti tenant besar tidak lagi bisa "meminjam" kapasitas bersama sebebas dulu, bahkan untuk kebutuhan yang sah. Solusinya bukan menghapus bulkhead (yang akan mengembalikan risiko yang sudah diperbaiki), melainkan mengalokasikan kuota yang **proporsional** dengan kebutuhan nyata masing-masing tenant — instansi besar yang memang konsisten butuh resource lebih banyak mendapat kuota lebih besar dari instansi kecil, bukan kuota seragam untuk semua. Ini butuh data historis penggunaan resource per tenant untuk menentukan alokasi yang adil, dan idealnya dikomunikasikan sebagai kebijakan yang transparan ke semua instansi — bukan kuota yang terasa sewenang-wenang tanpa penjelasan jelas.

## Catatan Saya

*Kalau kamu punya kejadian nyata dari kerjaan yang mirip tema ini, tulis sebagai case study baru dengan `source: personal`.*
