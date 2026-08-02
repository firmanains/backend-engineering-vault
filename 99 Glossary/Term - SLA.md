---
title: Term - SLA
type: term
level: intermediate
domain: architecture
status: unread
difficulty: 1
est_minutes: 3
prerequisites: []
next: []
tags: [backend, architecture]
created: 2026-08-02
---

**SLA (Service Level Agreement)** adalah kontrak formal — sering punya konsekuensi kontraktual atau finansial — antara penyedia layanan dan penggunanya, menyatakan tingkat layanan yang dijanjikan (misalnya "99.9% uptime bulanan") beserta konsekuensi kalau janji itu tidak terpenuhi. SLA berbeda dari [[../70 Infrastructure and Delivery/SLIs and SLOs|SLI dan SLO]]: SLO adalah target internal yang dipakai tim untuk mengambil keputusan teknis (lihat [[../60 Distributed Systems/Error Budgets|Error Budgets]]), sementara SLA adalah janji **eksternal** ke pelanggan atau partner, biasanya diatur sedikit lebih longgar dari SLO internal untuk memberi ruang aman sebelum benar-benar melanggar kontrak.

Ini kenapa istilah ini penting dipahami: mencampur SLA dengan SLO/SLI adalah kesalahan umum — tim yang menetapkan SLO sama persis dengan SLA berisiko sudah "melanggar" secara kontraktual tepat saat mereka baru menyadari ada masalah secara internal, tanpa ruang bereaksi lebih dulu.

## Muncul Di

- [[../70 Infrastructure and Delivery/SLIs and SLOs|SLIs and SLOs]] — perbedaan SLA, SLI, dan SLO dibahas dalam konteks disiplin reliability.
- [[../60 Distributed Systems/Error Budgets|Error Budgets]] — SLO internal (yang biasanya lebih ketat dari SLA) jadi dasar error budget.

## Catatan Saya

*Kosong — diisi pembaca.*
