---
title: Case - The Cache That Returned Stale Data After a Write
type: case
level: intermediate
domain: architecture
status: unread
difficulty: 3
est_minutes: 14
source: synthesized
concepts: ["[[../50 Concurrency and Performance/Cache-Aside, Write-Through, and Write-Behind|Cache-Aside, Write-Through, and Write-Behind]]", "[[../50 Concurrency and Performance/Cache Invalidation Strategies|Cache Invalidation Strategies]]", "[[../60 Distributed Systems/Defensible Eventual Consistency|Defensible Eventual Consistency]]"]
prerequisites: []
next: []
tags: [backend, architecture, performance, gov-context]
created: 2026-08-02
---

## The Situation

Sebuah endpoint yang menampilkan detail kasus memakai cache-aside dengan Redis untuk mengurangi beban database — data kasus di-cache selama 10 menit. Seorang petugas mengubah status kasus lewat form, langsung diarahkan ke halaman detail kasus yang sama, dan melihat status **lama** yang belum berubah — kebingungan yang wajar, karena dari sudut pandang petugas, perubahan yang baru saja mereka simpan seharusnya langsung terlihat.

## What Was Actually Asked For

Permintaan awal: "cache-nya harus langsung update setelah ada perubahan." Setelah didiskusikan, kebutuhan sebenarnya lebih spesifik: petugas yang **baru saja mengubah** data itu sendiri harus selalu melihat perubahannya (read-your-writes) — bukan berarti seluruh sistem butuh cache yang selalu 100% real-time untuk semua pengguna, karena pengguna lain yang tidak terlibat perubahan itu biasanya bisa menerima jeda singkat tanpa masalah.

## Options

### Option 1 — Perpendek TTL cache secara signifikan

Turunkan TTL dari 10 menit jadi 30 detik, mengurangi jendela waktu data basi bisa terlihat. Solusi termudah diimplementasikan, tapi tidak menghilangkan masalah sepenuhnya (masih ada jendela 30 detik di mana data basi bisa terlihat), dan menurunkan manfaat cache secara signifikan (lebih sering cache miss, lebih banyak beban ke database) untuk seluruh sistem, padahal masalahnya spesifik hanya untuk pengguna yang baru saja menulis.

### Option 2 — Invalidasi cache eksplisit saat menulis

Setiap kali data kasus diubah, kode aplikasi secara eksplisit menghapus (invalidate) entri cache yang relevan (lihat [[../50 Concurrency and Performance/Cache Invalidation Strategies|Cache Invalidation Strategies]]) — permintaan baca berikutnya (dari siapa pun) akan cache miss dan mengambil data segar dari database, memperbarui cache. Menyelesaikan masalah untuk semua pengguna, bukan hanya yang menulis, tapi butuh disiplin memastikan **setiap** titik yang mengubah data juga menghapus cache yang relevan — celah yang mudah terlewat kalau ada banyak jalur berbeda yang mengubah data yang sama.

### Option 3 — Write-through cache untuk operasi yang mengubah data

Alih-alih menulis ke database saja lalu invalidasi cache secara terpisah, tulis ke database **dan** perbarui cache dalam satu alur (write-through, lihat [[../50 Concurrency and Performance/Cache-Aside, Write-Through, and Write-Behind|Cache-Aside, Write-Through, and Write-Behind]]) — cache langsung berisi data terbaru setelah tulisan, tanpa jendela cache miss sama sekali. Lebih kompleks diimplementasikan dibanding invalidasi sederhana, dan menambah sedikit latency di jalur tulis (harus menunggu cache diperbarui juga).

## The Decision

Option 2 (invalidasi eksplisit) dipilih sebagai solusi utama — lebih sederhana diimplementasikan dibanding write-through penuh, dan cukup untuk menyelesaikan masalah read-your-writes yang jadi kebutuhan inti. Constraint penentunya: jumlah titik kode yang mengubah data kasus di sistem ini masih terbatas dan diketahui semua (bukan sistem besar dengan banyak jalur tulis tersembunyi), membuat disiplin menambahkan invalidasi di setiap titik itu realistis dijaga konsisten.

## What Breaks at 10x and 100x

Pada volume 10x (lebih banyak fitur yang mengubah data kasus ditambahkan seiring waktu), disiplin invalidasi manual di setiap titik mulai rawan terlewat — developer baru yang menambah fitur baru bisa lupa menambahkan invalidasi cache, memunculkan kembali masalah stale data dari jalur yang baru. Di titik ini, investasi write-through terpusat (Option 3) atau bahkan CDC-driven cache invalidation (mengikuti pola yang sama seperti sinkronisasi search index) jadi lebih sepadan, karena tidak bergantung pada disiplin manual di setiap titik kode baru.

## What You Would Do Differently

Membungkus akses tulis ke data kasus dalam satu layer terpusat (repository pattern) yang **selalu** menangani invalidasi cache di satu tempat — bukan tersebar di banyak handler berbeda yang masing-masing harus mengingat menambahkan invalidasi sendiri-sendiri.

## Concepts Exercised

- [[../50 Concurrency and Performance/Cache-Aside, Write-Through, and Write-Behind|Cache-Aside, Write-Through, and Write-Behind]] — pola caching yang jadi konteks masalah dan solusi alternatif (write-through) yang dipertimbangkan.
- [[../50 Concurrency and Performance/Cache Invalidation Strategies|Cache Invalidation Strategies]] — strategi invalidasi eksplisit yang dipilih sebagai solusi.
- [[../60 Distributed Systems/Defensible Eventual Consistency|Defensible Eventual Consistency]] — kerangka membedakan kebutuhan konsistensi ketat (read-your-writes untuk penulis) dari kebutuhan yang bisa menerima jeda (pengguna lain).

## Your Turn

Tim menerapkan invalidasi cache eksplisit, tapi menemukan race condition baru: kalau ada request baca yang sedang berjalan (mengambil data lama dari database sebelum tulisan terjadi) bersamaan dengan tulisan dan invalidasi, request baca itu bisa menyimpan hasil **lama** ke cache **setelah** invalidasi terjadi — cache jadi berisi data basi lagi meski invalidasi sudah dijalankan. Bagaimana ini bisa terjadi, dan apa yang bisa dilakukan?

> [!success]- Kunci jawaban
> Ini adalah race condition klasik antara operasi baca (cache-aside: baca database, simpan ke cache) dan operasi tulis-invalidasi yang terjadi hampir bersamaan — urutan kejadian yang bermasalah: (1) request A membaca database, mendapat data lama; (2) request B menulis data baru ke database dan menghapus cache; (3) request A, yang belum tahu ada perubahan, menyimpan data lama yang sudah dibacanya ke cache — cache sekarang berisi data lama lagi, meski invalidasi sudah terjadi sebelumnya. Mitigasi yang umum dipakai: menyertakan versi atau timestamp saat menyimpan ke cache, dan menolak menyimpan kalau versi yang akan disimpan lebih lama dari versi yang sudah ada di cache (mencegah data lama menimpa data yang lebih baru). Ini adalah masalah yang inheren pada pola cache-aside dan tidak sepenuhnya bisa dihilangkan tanpa kompleksitas tambahan — untuk sistem yang benar-benar tidak bisa menerima kemungkinan kecil ini, write-through (Option 3) menghilangkan race condition ini secara struktural karena cache selalu diperbarui dalam alur tulis yang sama, bukan lewat invalidasi terpisah.

## Catatan Saya

*Kalau kamu punya kejadian nyata dari kerjaan yang mirip tema ini, tulis sebagai case study baru dengan `source: personal`.*
