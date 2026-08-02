---
title: Case - A Government Partner's API That Only Accepts SOAP
type: case
level: intermediate
domain: architecture
status: unread
difficulty: 3
est_minutes: 14
source: synthesized
concepts: ["[[../30 APIs and Web/Designing an API for a Partner You Do Not Control|Designing an API for a Partner You Do Not Control]]", "[[../90 Architecture and Design/Hexagonal and Clean Architecture in Go|Hexagonal and Clean Architecture in Go]]", "[[../30 APIs and Web/Contract Negotiation and Versioning|Contract Negotiation and Versioning]]"]
prerequisites: []
next: []
tags: [backend, architecture, apis, gov-context, integration]
created: 2026-08-02
---

## The Situation

Sebuah sistem modern berbasis Go dan REST perlu berintegrasi dengan sistem milik kementerian lain yang dibangun bertahun-tahun lalu dan hanya menyediakan antarmuka SOAP/XML — protokol yang sudah dianggap kuno di kebanyakan konteks pengembangan modern, tapi tidak bisa diubah karena sistem itu melayani puluhan instansi lain dan biaya migrasinya (baik teknis maupun birokrasi) dianggap tidak sepadan oleh pemiliknya.

## What Was Actually Asked For

Permintaan dari tim internal: "kita perlu terintegrasi dengan sistem itu." Godaan yang muncul dari sebagian developer: menulis seluruh kode aplikasi memakai struktur data dan pola yang meniru SOAP/XML, supaya "konsisten" dengan API yang dipanggil. Kebutuhan sebenarnya justru sebaliknya: **kode aplikasi internal harus tetap bersih dan idiomatic** (REST/JSON di dalam), dan kompleksitas SOAP/XML partner harus **diisolasi** di satu titik integrasi, tidak menyebar ke seluruh basis kode.

## Options

### Option 1 — Panggil SOAP langsung dari setiap tempat yang membutuhkannya

Setiap handler atau service yang butuh data dari partner ini langsung membangun request XML dan mem-parsing response SOAP di tempat. Tercepat diimplementasikan untuk kebutuhan pertama, tapi kompleksitas SOAP (namespace XML, envelope, fault handling yang berbeda dari error HTTP biasa) menyebar ke banyak bagian kode, membuat basis kode makin sulit dipahami dan diuji seiring bertambahnya titik integrasi dengan partner ini.

### Option 2 — Adapter/anti-corruption layer terpisah

Bangun satu lapisan (package) khusus yang menjadi satu-satunya titik yang tahu tentang SOAP/XML — menerjemahkan panggilan dari kode internal (yang memakai struct Go biasa dan interface bersih) menjadi request SOAP, dan menerjemahkan response SOAP kembali jadi struct Go yang sama sekali tidak "berbau" XML. Mengikuti prinsip [[../90 Architecture and Design/Hexagonal and Clean Architecture in Go|Hexagonal and Clean Architecture in Go]] — logika bisnis internal tidak pernah tahu atau peduli bahwa di baliknya ada SOAP, hanya berinteraksi lewat interface Go biasa.

### Option 3 — Wrapper service terpisah (microservice kecil khusus SOAP)

Alih-alih package internal, bangun service kecil terpisah yang **hanya** tugasnya menerjemahkan antara REST/JSON (dipanggil aplikasi internal) dan SOAP/XML (dipanggil ke partner) — isolasi lebih ekstrem lagi, sampai ke level proses terpisah, bukan hanya package. Berguna kalau integrasi SOAP ini dipakai **banyak** aplikasi berbeda dari 13 aplikasi yang ada (satu wrapper melayani semuanya), tapi menambah infrastruktur dan titik kegagalan baru untuk kebutuhan yang mungkin masih sederhana.

## The Decision

Option 2 dipilih untuk kebutuhan saat ini — adapter internal sebagai package terpisah cukup untuk mengisolasi kompleksitas tanpa menambah infrastruktur baru yang belum tentu dibutuhkan. Constraint penentunya: integrasi ini saat ini hanya dipakai satu aplikasi, membuat Option 3 (wrapper service terpisah) sebagai overhead yang belum sepadan — tapi desain adapter (Option 2) sengaja dibuat dengan interface yang jelas, membuat migrasi ke Option 3 relatif mudah kalau di masa depan aplikasi lain juga butuh integrasi yang sama.

## What Breaks at 10x and 100x

Pada volume 10x (beberapa aplikasi lain dari 13 aplikasi juga butuh integrasi dengan partner SOAP yang sama), duplikasi adapter di setiap aplikasi jadi tidak efisien — titik ini membenarkan migrasi ke Option 3 (wrapper service terpusat) yang melayani semua aplikasi sekaligus, menghindari setiap tim membangun dan memelihara adapter SOAP-nya sendiri secara terpisah. Pada 100x volume panggilan (bukan jumlah aplikasi, tapi frekuensi panggilan), karakteristik performa SOAP/XML (parsing yang lebih berat dibanding JSON) mulai jadi bottleneck nyata — pertimbangan caching hasil panggilan yang jarang berubah, atau negosiasi ulang dengan partner untuk endpoint alternatif yang lebih ringan, jadi relevan.

## What You Would Do Differently

Menulis test yang mencakup adapter SOAP secara menyeluruh (termasuk skenario fault/error SOAP yang formatnya berbeda dari error HTTP biasa) sejak awal — kompleksitas yang diisolasi di satu tempat tetap butuh diuji dengan teliti, karena satu-satunya titik ini menjadi kritis bagi seluruh integrasi yang bergantung padanya.

## Concepts Exercised

- [[../30 APIs and Web/Designing an API for a Partner You Do Not Control|Designing an API for a Partner You Do Not Control]] — prinsip dasar yang berlaku langsung: kontrak partner tidak bisa diubah, jadi desain internal harus beradaptasi dengan cara yang tidak mengorbankan kualitas kode sendiri.
- [[../90 Architecture and Design/Hexagonal and Clean Architecture in Go|Hexagonal and Clean Architecture in Go]] — pola arsitektur yang mendasari keputusan mengisolasi kompleksitas SOAP di satu adapter.
- [[../30 APIs and Web/Contract Negotiation and Versioning|Contract Negotiation and Versioning]] — relevan untuk memahami kenapa kontrak partner (SOAP) di sini benar-benar tidak bisa dinegosiasikan, berbeda dari kasus integrasi lain yang mungkin masih punya ruang negosiasi.

## Your Turn

Partner mengumumkan bahwa API SOAP mereka akan "di-deprecate" dalam dua tahun, digantikan API REST baru — tapi detail teknis API baru itu belum tersedia sekarang. Bagaimana desain adapter (Option 2) yang sudah ada membantu (atau tidak membantu) migrasi ini nanti?

> [!success]- Kunci jawaban
> Desain adapter yang mengisolasi kompleksitas SOAP di satu titik (Option 2) membantu signifikan — karena kode internal aplikasi hanya berinteraksi dengan interface Go bersih (bukan langsung dengan detail SOAP), migrasi nanti ke API REST baru partner **hanya perlu mengubah implementasi di dalam adapter itu saja** (mengganti logika yang membangun request SOAP jadi logika yang membangun request REST), tanpa menyentuh kode aplikasi internal yang memanggil adapter itu sama sekali — interface-nya tetap sama, hanya implementasi di baliknya yang berubah. Ini adalah manfaat konkret dari isolasi yang sudah diterapkan sejak awal: keputusan Option 2 yang awalnya dipilih untuk kebutuhan kebersihan kode sekarang juga memberi manfaat tambahan berupa migrasi yang jauh lebih murah di masa depan, meski manfaat itu tidak sepenuhnya diantisipasi saat keputusan awal dibuat.

## Catatan Saya

*Kalau kamu punya kejadian nyata dari kerjaan yang mirip tema ini, tulis sebagai case study baru dengan `source: personal`.*
