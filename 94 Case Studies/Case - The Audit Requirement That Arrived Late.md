---
title: Case - The Audit Requirement That Arrived Late
type: case
level: intermediate
domain: architecture
status: unread
difficulty: 3
est_minutes: 14
source: synthesized
concepts: ["[[../80 Security/Audit Logging|Audit Logging]]", "[[../80 Security/Compliance Trails for Government Systems|Compliance Trails for Government Systems]]", "[[../60 Distributed Systems/Event Sourcing|Event Sourcing]]"]
prerequisites: []
next: []
tags: [backend, architecture, security, gov-context]
created: 2026-08-02
---

## The Situation

Sebuah sistem sudah berjalan tiga tahun tanpa audit logging formal — hanya log aplikasi biasa dengan retensi 30 hari. Unit pengawasan internal instansi tiba-tiba meminta bukti riwayat lengkap siapa mengubah data kasus tertentu selama dua tahun terakhir, sebagai bagian investigasi dugaan penyalahgunaan wewenang. Tim teknis menyadari sistem tidak pernah dirancang untuk menjawab pertanyaan ini — sebagian tabel memakai hard delete (riwayat perubahan tertimpa `UPDATE` biasa), dan log aplikasi yang ada sudah lama terhapus kebijakan retensi.

## What Was Actually Asked For

Permintaan unit pengawasan: "berikan riwayat lengkap perubahan data kasus ini." Permintaan ini secara implisit mengasumsikan riwayat itu **ada** dan bisa diambil — realitasnya, sebagian riwayat itu sudah hilang permanen, dan kebutuhan sebenarnya sekarang bergeser jadi dua hal: (1) berikan sebanyak mungkin bukti yang **masih ada** dengan jujur tentang keterbatasannya, dan (2) pastikan pertanyaan serupa di masa depan bisa dijawab lengkap.

## Options

### Option 1 — Coba rekonstruksi riwayat dari sumber lain yang tersisa

Periksa apakah ada sumber lain yang mungkin masih menyimpan sebagian riwayat — backup database lama, log aplikasi yang mungkin belum sepenuhnya terhapus di beberapa server, atau catatan manual dari petugas yang terlibat. Solusi jangka pendek yang mencoba memaksimalkan bukti yang tersedia sekarang, tapi hasilnya hampir pasti tidak lengkap, dan proses ini sendiri butuh waktu tanpa jaminan hasil yang memuaskan permintaan investigasi.

### Option 2 — Akui keterbatasan secara jujur, mulai audit logging sekarang

Sampaikan secara eksplisit dan tertulis ke unit pengawasan bahwa riwayat sebelum tanggal tertentu tidak bisa direkonstruksi lengkap karena keterbatasan desain sistem lama, sambil segera menerapkan audit logging (lihat [[../80 Security/Audit Logging|Audit Logging]]) untuk semua perubahan ke depan. Solusi paling jujur dan realistis, tapi tidak menjawab kebutuhan investigasi yang sedang berjalan sekarang untuk data historis yang sudah terlanjur hilang.

### Option 3 — Kombinasi: maksimalkan bukti yang ada, plus audit logging permanen, plus tinjau ulang desain skema untuk mencegah hilangnya riwayat lagi

Jalankan Option 1 dan 2 bersamaan, ditambah tinjauan desain skema database untuk tabel-tabel kritis — mengubah dari hard delete jadi soft delete atau pola yang menyimpan riwayat versi (mendekati filosofi [[../60 Distributed Systems/Event Sourcing|Event Sourcing]]), memastikan kejadian serupa tidak terulang untuk modul-modul lain yang mungkin juga akan diminta audit serupa di masa depan.

## The Decision

Option 3 dipilih sebagai respons lengkap. Constraint penentunya: investigasi yang sedang berjalan butuh jawaban **sekarang** (tidak bisa menunggu perbaikan desain selesai), tapi menyelesaikan hanya untuk investigasi ini tanpa memperbaiki akar masalah berarti pertanyaan serupa di masa depan (untuk modul lain, atau periode waktu lain) akan menghadapi masalah yang persis sama. Tim mengomunikasikan eksplisit ke unit pengawasan tentang apa yang bisa dan tidak bisa dipenuhi dari data historis, sambil menunjukkan langkah konkret yang sudah diambil untuk mencegah masalah ini berulang — respons yang jujur dan proaktif, bukan defensif.

## What Breaks at 10x and 100x

Pada skala lebih besar (lebih banyak modul dan tabel yang perlu ditinjau ulang skemanya), migrasi dari hard delete ke soft delete/event-sourced untuk **seluruh** sistem sekaligus tidak realistis dilakukan cepat — perlu diprioritaskan berdasarkan sensitivitas data (modul yang menangani data paling sensitif atau paling berpotensi jadi objek investigasi lebih dulu). Pada skala organisasi (13 aplikasi, bukan hanya satu), kebutuhan audit trail yang konsisten lintas semua sistem membenarkan investasi kebijakan dan tooling audit logging terpusat, bukan setiap aplikasi menyelesaikan masalah ini sendiri-sendiri secara terpisah.

## What You Would Do Differently

Menjadikan audit logging sebagai bagian standar desain sejak sistem pertama kali dibangun untuk data sensitif seperti kasus hukum — bukan menunggu permintaan investigasi eksternal untuk menyadari kebutuhannya, karena di titik itu kerusakan (riwayat yang sudah hilang) sudah terjadi dan tidak bisa diperbaiki lagi.

## Concepts Exercised

- [[../80 Security/Audit Logging|Audit Logging]] — mekanisme yang seharusnya sudah ada sejak awal, dan diterapkan sekarang untuk ke depan.
- [[../80 Security/Compliance Trails for Government Systems|Compliance Trails for Government Systems]] — kerangka kebutuhan compliance yang secara langsung relevan dengan skenario investigasi unit pengawasan ini.
- [[../60 Distributed Systems/Event Sourcing|Event Sourcing]] — pola desain yang dipertimbangkan sebagai perbaikan struktural jangka panjang untuk mencegah kehilangan riwayat berulang.

## Your Turn

Setelah audit logging diterapkan untuk semua perubahan ke depan, seorang developer mengusulkan agar audit log baru ini juga mencatat **nilai lama dan baru secara lengkap**, termasuk untuk kolom yang berisi data sangat sensitif (nomor identitas kependudukan). Apakah ini keputusan yang tepat tanpa pertimbangan tambahan?

> [!success]- Kunci jawaban
> Tidak sepenuhnya tepat tanpa pertimbangan tambahan — mencatat nilai sensitif mentah di audit log berarti audit log itu sendiri sekarang menjadi target yang sama sensitifnya dengan data aslinya, dan butuh perlindungan yang setara (enkripsi, kontrol akses ketat, lihat [[../80 Security/Encryption at Rest vs In Transit|Encryption at Rest vs In Transit]]) — perlindungan yang mungkin belum diterapkan pada infrastruktur logging yang ada. Pendekatan yang lebih aman: audit log mencatat **bahwa** kolom sensitif tertentu berubah dan **siapa** yang mengubahnya, tanpa selalu menyertakan nilai mentahnya, kecuali untuk kasus investigasi spesifik yang memang membutuhkan detail penuh dan sudah melalui proses otorisasi tambahan untuk mengaksesnya. Trade-off ini perlu didiskusikan eksplisit dengan tim keamanan dan compliance sebelum diimplementasikan, bukan diputuskan sepihak oleh satu developer berdasarkan asumsi "lebih lengkap selalu lebih baik".

## Catatan Saya

*Kalau kamu punya kejadian nyata dari kerjaan yang mirip tema ini, tulis sebagai case study baru dengan `source: personal`.*
