---
title: Project 3 - A Small Distributed System
type: project
level: senior
domain: architecture
status: unread
difficulty: 5
est_minutes: 0
concepts: ["[[../60 Distributed Systems/Sagas - Orchestration vs Choreography|Sagas - Orchestration vs Choreography]]", "[[../30 APIs and Web/The Transactional Outbox Pattern|The Transactional Outbox Pattern]]", "[[../60 Distributed Systems/Idempotency Keys|Idempotency Keys]]", "[[../60 Distributed Systems/Chaos Engineering|Chaos Engineering]]", "[[../60 Distributed Systems/Writing Architecture Decision Records|Writing Architecture Decision Records]]"]
prerequisites: ["[[Project 2 - Worker Pool, Cache, and Streaming Service]]"]
next: []
tags: [backend, architecture, go, distributed]
created: 2026-08-02
---

## Learning Objectives

Membangun sistem yang benar-benar terdiri dari **beberapa service independen** yang harus saling koordinasi lewat jaringan yang tidak sempurna — proyek ini adalah tempat seluruh teori distributed systems yang dipelajari (saga, outbox, idempotency, kegagalan parsial) berhenti jadi abstraksi dan mulai jadi kenyataan yang harus benar-benar ditangani kodenya. Setelah menyelesaikan proyek ini, kamu harus bisa: merancang saga lintas service dengan compensating action yang benar, mengimplementasikan outbox pattern untuk menghindari dual write, menjamin idempotency di titik-titik kritis, sengaja menguji ketahanan sistem lewat skenario chaos, dan mendokumentasikan keputusan arsitektural lewat ADR yang bisa dipertahankan.

## The Brief

Bangun sistem pengajuan permohonan **lintas tiga service independen**, masing-masing dengan database sendiri: **Service Pengajuan** (menerima permohonan baru, mengunci kuota harian), **Service Verifikasi** (memvalidasi dokumen yang dilampirkan), dan **Service Pencatatan Resmi** (mencatat hasil akhir setelah kedua langkah sebelumnya berhasil). Proses pengajuan penuh adalah saga tiga langkah — kalau salah satu langkah gagal setelah langkah sebelumnya berhasil, sistem harus membatalkan langkah-langkah sebelumnya lewat compensating action, bukan meninggalkan sistem dalam keadaan tidak konsisten.

## Requirements

**Arsitektur tiga service**
- Ketiga service berjalan sebagai proses/container terpisah, masing-masing dengan database sendiri (boleh instance PostgreSQL/MariaDB yang sama, tapi schema/database logic terpisah — bukan berbagi tabel).
- Komunikasi antar service lewat HTTP atau messaging (pilih dan jelaskan alasannya) — tidak boleh ada satu service yang langsung query database service lain.

**Saga dan compensating action**
- Implementasikan orchestrator (di service terpisah atau di salah satu dari tiga service) yang menjalankan ketiga langkah berurutan, dengan compensating action untuk setiap langkah (lihat [[../60 Distributed Systems/Sagas - Orchestration vs Choreography|Sagas - Orchestration vs Choreography]]).
- Simulasikan kegagalan di langkah kedua atau ketiga (lewat flag/endpoint uji yang sengaja membuatnya gagal), dan verifikasi compensating action benar-benar mengembalikan sistem ke keadaan konsisten.

**Outbox pattern**
- Setiap service yang perlu memberi tahu service lain tentang perubahan data menulis ke tabel outbox dalam transaksi yang sama dengan perubahan data itu sendiri (lihat [[../30 APIs and Web/The Transactional Outbox Pattern|The Transactional Outbox Pattern]]) — tidak ada dual write langsung ke dua tempat berbeda tanpa lewat outbox.
- Proses relay terpisah membaca outbox dan mengirim ke service tujuan, dengan retry kalau gagal.

**Idempotency**
- Setiap endpoint yang menerima permintaan dari orchestrator atau service lain menerima `Idempotency-Key` (lihat [[../60 Distributed Systems/Idempotency Keys|Idempotency Keys]]) — permintaan yang sama dikirim ulang (simulasikan retry karena timeout) tidak menghasilkan efek ganda.

**Chaos scenario**
- Rancang dan jalankan minimal tiga eksperimen chaos (lihat [[../60 Distributed Systems/Chaos Engineering|Chaos Engineering]]): (1) matikan Service Verifikasi di tengah proses saga berjalan, (2) simulasikan latency tinggi di panggilan ke Service Pencatatan Resmi, (3) jalankan dua permintaan pengajuan bersamaan yang mendekati batas kuota harian, verifikasi tidak ada race condition yang membuat kuota terlampaui.

**Dokumen desain dengan ADR**
- Tulis dokumen desain (README terpisah, `DESIGN.md`) yang menjelaskan arsitektur sistem secara keseluruhan.
- Sertakan minimal tiga ADR (lihat [[../60 Distributed Systems/Writing Architecture Decision Records|Writing Architecture Decision Records]]) untuk keputusan signifikan: (1) orchestration vs choreography untuk saga, (2) strategi memastikan idempotency, (3) keputusan lain yang kamu anggap signifikan (pembagian tanggung jawab antar service, strategi komunikasi, dsb.).

## Staged Milestones

1. **Tiga service berdiri sendiri**: masing-masing service berjalan independen dengan database sendiri, endpoint dasar berfungsi (belum ada koordinasi saga).
2. **Saga happy path**: orchestrator menjalankan ketiga langkah berurutan untuk kasus sukses penuh, data konsisten di ketiga service setelah selesai.
3. **Compensating action**: simulasikan kegagalan di berbagai titik, verifikasi compensating action benar-benar membatalkan langkah sebelumnya dengan benar.
4. **Outbox dan idempotency**: ganti komunikasi langsung dengan pola outbox, tambahkan idempotency key di semua endpoint yang menerima permintaan dari orchestrator.
5. **Chaos dan dokumentasi**: jalankan ketiga eksperimen chaos, catat hasilnya, tulis dokumen desain lengkap dengan ADR.

## Definition of Done

- Saga berjalan sukses penuh untuk kasus normal, dan compensating action terbukti bekerja benar untuk minimal dua skenario kegagalan berbeda (diverifikasi lewat pengujian manual, bukan cuma dibaca kodenya).
- Tidak ada dual write langsung — semua komunikasi antar service lewat outbox atau panggilan yang idempoten.
- Ketiga eksperimen chaos dijalankan, hasilnya (termasuk kalau ada yang mengungkap bug) didokumentasikan di `DESIGN.md`.
- Race condition pada kuota harian (chaos scenario ketiga) terbukti tidak terjadi lewat pengujian konkuren nyata, bukan asumsi.
- `DESIGN.md` berisi minimal tiga ADR yang lengkap (context, decision, consequences) dan bisa dipertahankan kalau ditanya "kenapa keputusan ini, bukan alternatif lain".

## Self-Review Checklist

- [ ] Apakah compensating action benar-benar diuji dengan skenario kegagalan nyata (bukan cuma ditulis kodenya tanpa pernah benar-benar dipicu)?
- [ ] Apakah ada titik dual write yang terlewat (menulis ke dua tempat tanpa lewat outbox atau transaksi yang menjamin atomicity)?
- [ ] Apakah idempotency key benar-benar mencegah efek ganda saat diuji dengan mengirim permintaan yang sama dua kali secara sengaja?
- [ ] Apakah eksperimen chaos dijalankan dengan hipotesis eksplisit sebelum dijalankan (bukan sekadar "matikan dan lihat apa yang terjadi" tanpa ekspektasi jelas)?
- [ ] Apakah ADR yang ditulis benar-benar menjelaskan trade-off (apa yang didapat dan dikorbankan), bukan hanya menyatakan keputusan tanpa justifikasi?
- [ ] Kalau salah satu dari tiga service ini harus diganti total oleh orang lain enam bulan dari sekarang, apakah dokumen desain yang ditulis cukup untuk mereka memahami kontrak dan asumsi yang harus dijaga?

## Concepts Exercised

- [[../60 Distributed Systems/Sagas - Orchestration vs Choreography|Sagas - Orchestration vs Choreography]] — pola inti yang membentuk seluruh alur pengajuan lintas tiga service.
- [[../30 APIs and Web/The Transactional Outbox Pattern|The Transactional Outbox Pattern]] — mekanisme yang mencegah dual write di komunikasi antar service.
- [[../60 Distributed Systems/Idempotency Keys|Idempotency Keys]] — jaminan keamanan terhadap retry yang diterapkan di semua endpoint antar-service.
- [[../60 Distributed Systems/Chaos Engineering|Chaos Engineering]] — metodologi pengujian ketahanan yang diterapkan lewat tiga eksperimen wajib.
- [[../60 Distributed Systems/Writing Architecture Decision Records|Writing Architecture Decision Records]] — praktik dokumentasi keputusan yang dituntut sebagai bagian definisi selesai.

## Catatan Saya

*Tulis di sini hasil dari ketiga eksperimen chaos-mu — apa yang sesuai hipotesis, dan apa yang mengejutkan (kalau ada).*
