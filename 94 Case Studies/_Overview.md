---
title: Case Studies Overview
type: concept
level: intermediate
domain: architecture
status: mastered
difficulty: 1
est_minutes: 10
prerequisites: []
next: ["[[Case - Sending PDFs Through a JSON API]]"]
tags: [backend, architecture]
created: 2026-07-26
---

## Apa Ini

Folder ini adalah tempat concept note berubah jadi judgement. Concept note mengajarkan apa yang ada; case study melatih **memilih di bawah constraint yang tidak kamu pilih sendiri** — deadline, partner yang tidak bisa berubah, aturan procurement, tim tanpa skill untuk mengoperasikan solusi yang elegan.

Setiap case study di sini punya `source: synthesized` — disusun untuk masuk akal secara teknis dan menyerupai situasi nyata, tapi bukan transkrip kejadian spesifik. **Kamu didorong untuk menulis case study milikmu sendiri** dengan `source: personal` begitu kamu punya pengalaman nyata yang sesuai polanya — itu akan jauh lebih berharga daripada versi sintetis manapun di sini, karena kamu tahu persis apa yang sungguhan terjadi dan apa yang tidak tertulis di laporan post-mortem resmi.

## Kenapa Ini Penting

Minimal 25 case study di folder ini sengaja diberi bobot berat pada skenario integrasi, data, dan kegagalan yang plausibel terjadi di sistem pemerintah dan enterprise Indonesia — karena itu adalah bentuk paling realistis dari pekerjaanmu sehari-hari. Setiap case study punya minimal tiga opsi nyata (bukan dua strawman dan satu jawaban jelas benar), karena di dunia nyata pilihan yang "jelas benar" hampir tidak pernah ada — hanya trade-off yang harus dipertanggungjawabkan.

## Reading Order

Case study tidak punya urutan wajib seperti concept note — baca yang paling relevan dengan masalah yang sedang kamu hadapi (*problem-driven*, lihat [[../00 Start Here/How To Read This Vault|How To Read This Vault]]). Daftar di bawah dikelompokkan menurut tema, bukan tingkat kesulitan.

### Integrasi dan partner eksternal

1. [[Case - Sending PDFs Through a JSON API]] — mengirim dokumen biner lewat API JSON saat partner tidak bisa menerima `multipart`.
2. [[Case - The Partner Who Calls Twice]] — partner memanggil endpoint-mu dua kali karena timeout mereka lebih pendek dari waktu prosesmu.
3. [[Case - The Webhook That Was Down For Six Hours]] — webhook partner mati enam jam; apa yang dilakukan retry policy-mu terhadap mereka dan dirimu sendiri.
4. [[Case - Daily CSV Over SFTP in 2026]] — partner bersikeras memakai CSV harian lewat SFTP, dan cara membuatnya tetap andal.
5. [[Case - A Government Partner's API That Only Accepts SOAP]] — mengintegrasikan sistem modern dengan kontrak lama yang tidak bisa diubah.
6. [[Case - The Rate-Limited External API and the Batch Job]] — API eksternal dibatasi jauh di bawah kebutuhan batch job-mu.
7. [[Case - An OAuth2 Integration With a Partner Who Has No Refresh Tokens]] — mendesain sesi yang tahan lama tanpa mekanisme refresh token standar.
8. [[Case - A Schema Change That Broke a Partner Who Scraped Undocumented Fields]] — perubahan yang "aman" menurut kontrak, tapi mematahkan partner yang bergantung pada field yang tidak pernah dijanjikan.

### Data, konsistensi, dan kegagalan konkurensi

9. [[Case - The Counter That Undercounts]] — counter yang salah karena dua worker melakukan read-modify-write bersamaan.
10. [[Case - The Search Index That Drifted Out of Sync]] — index pencarian diam-diam tidak lagi mencerminkan database.
11. [[Case - The Cache That Returned Stale Data After a Write]] — cache mengembalikan data basi tepat setelah penulisan, dan titik di mana eventual consistency berhenti bisa diterima.
12. [[Case - The Background Job That Ran Twice After a Deployment]] — job berjalan dua kali karena deployment tidak menangani proses yang sedang berjalan.
13. [[Case - The Upload Endpoint That Died Under Real Concurrency]] — endpoint upload yang lolos testing tapi rubuh di bawah concurrency sungguhan.
14. [[Case - The Timezone Bug in a Nationwide Report]] — bug batas hari dan zona waktu yang mengubah angka laporan nasional.

### Database dan performa

15. [[Case - The Report Query That Locks a Table During Business Hours]] — query laporan mengunci tabel di jam sibuk.
16. [[Case - Migrating a 200-Million-Row Table Without Downtime]] — migrasi skema di tabel raksasa tanpa mematikan layanan.
17. [[Case - The Just Add An Index Request]] — permintaan "tambah index saja" yang membuat penulisan jadi tidak bisa diterima.
18. [[Case - Slow Query vs Cache vs Read Replica]] — memilih di antara memperbaiki query lambat, menambah cache, atau menambah read replica — dengan biaya operasional masing-masing.
19. [[Case - Choosing Between Kafka and a Database Outbox Table for One Consumer]] — kapan infrastruktur messaging penuh adalah overkill dibanding outbox table sederhana.

### Arsitektur dan skala organisasi

20. [[Case - Splitting a Monolith Coupled at the Database]] — memisahkan monolith yang sebenarnya masih menyatu lewat satu database.
21. [[Case - A Multi-Tenant Batch Job That Starves The Rest of the Tenants]] — satu tenant menghabiskan resource bersama hingga tenant lain kelaparan.
22. [[Case - Log Volume That Costs More Than The Servers]] — volume log yang tagihannya melampaui biaya server yang menghasilkannya.

### Keamanan dan kepatuhan

23. [[Case - The JWT That Cannot Be Revoked]] — token yang tidak bisa dicabut saat akun diketahui disusupi.
24. [[Case - The Audit Requirement That Arrived Late]] — kebutuhan audit yang datang setelah sistem selesai dibangun.

### Kegagalan sistemik

25. [[Case - The Retry Storm That Became a Total Outage]] — retry yang mengubah gangguan sebagian jadi outage total.

## Bagaimana Domain Ini Terhubung Dengan Domain Lain

Setiap case study di atas menautkan minimal tiga concept note dari domain `10`–`90`, dan concept note itu balik menautkan case study yang relevan sebagai "Connected Notes" — lihat masing-masing `_Overview.md` domain untuk sisi lain hubungan ini. Case study juga sering menautkan tool note di [[../92 Tools/_Overview|Tools Overview]] saat keputusan melibatkan produk konkret.

## Progress

```dataview
TABLE concepts, status
FROM #backend
WHERE type = "case"
SORT file.name ASC
```

## Catatan Saya

*Kalau kamu punya kejadian nyata dari kerjaan yang mirip salah satu tema di atas, tulis sebagai case study baru dengan `source: personal` — itu akan jauh lebih berharga daripada versi sintetis di sini.*
