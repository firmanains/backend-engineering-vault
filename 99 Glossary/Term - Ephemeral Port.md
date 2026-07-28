---
title: Term - Ephemeral Port
type: term
level: junior
domain: foundations
status: unread
difficulty: 1
est_minutes: 3
prerequisites: []
next: []
tags: [backend, foundations]
created: 2026-07-26
---

**Ephemeral port** adalah nomor port sementara yang dipilih otomatis oleh sistem operasi di sisi **client** setiap kali sebuah koneksi keluar dibuka — berbeda dari port tetap yang dipakai server untuk `listen` (misalnya port 443 untuk HTTPS). Rentangnya biasanya beberapa ribu nomor tinggi (dikonfigurasi lewat parameter kernel, berbeda-beda antar OS), dan begitu koneksi ditutup, port itu dikembalikan ke pool untuk dipakai koneksi lain — tapi tidak seketika, karena bisa tertahan di state [[../10 Foundations/TCP Handshake and Connection Lifecycle|TIME_WAIT]] terlebih dulu.

Ini kenapa istilah ini penting dipahami: sebuah service yang membuka banyak koneksi keluar berumur pendek tanpa memakai ulang koneksi (lihat [[../10 Foundations/TCP Handshake and Connection Lifecycle|TCP Handshake and Connection Lifecycle]]) bisa kehabisan ephemeral port yang tersedia jauh sebelum kehabisan memori atau CPU — gejalanya adalah error seperti "cannot assign requested address" yang sering membingungkan karena tidak terlihat seperti masalah resource biasa.

## Muncul Di

- [[../10 Foundations/TCP Handshake and Connection Lifecycle|TCP Handshake and Connection Lifecycle]] — penjelasan penuh kenapa `TIME_WAIT` menahan ephemeral port dan bagaimana connection pooling mencegah kehabisannya.
- [[../10 Foundations/How An OS Handles Network Connections|How An OS Handles Network Connections]] — ephemeral port adalah bagian dari pasangan alamat yang dipakai kernel untuk mencocokkan koneksi masuk/keluar.

## Catatan Saya

*Kosong — diisi pembaca.*
