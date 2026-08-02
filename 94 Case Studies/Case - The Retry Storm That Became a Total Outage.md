---
title: Case - The Retry Storm That Became a Total Outage
type: case
level: intermediate
domain: architecture
status: unread
difficulty: 4
est_minutes: 15
source: synthesized
concepts: ["[[../30 APIs and Web/Circuit Breakers|Circuit Breakers]]", "[[../30 APIs and Web/Retries with Exponential Backoff and Jitter|Retries with Exponential Backoff and Jitter]]", "[[../30 APIs and Web/Load Shedding|Load Shedding]]"]
prerequisites: []
next: []
tags: [backend, architecture, resilience, gov-context]
created: 2026-08-02
---

## The Situation

Salah satu dari 13 aplikasi mengalami gangguan database sesaat (beberapa detik, karena masalah jaringan internal sementara) — gangguan kecil yang dalam kondisi normal seharusnya sembuh sendiri dengan cepat. Tapi aplikasi ini, dan beberapa service lain yang bergantung padanya, semuanya menerapkan retry otomatis tanpa backoff (mencoba lagi segera setiap kali gagal, tanpa jeda yang membesar). Begitu database kembali normal, ia langsung dibanjiri lonjakan permintaan dari **seluruh** retry yang sudah menumpuk selama gangguan singkat itu — beban mendadak ini yang justru membuat database benar-benar down, mengubah gangguan singkat jadi outage total yang berlangsung lebih dari satu jam.

## What Was Actually Asked For

Permintaan awal setelah insiden: "pastikan sistem lebih tahan gangguan database." Yang sebenarnya dibutuhkan lebih spesifik: sistem harus **tidak memperparah** gangguan kecil jadi besar — retry yang dimaksudkan sebagai mekanisme resiliensi justru menjadi penyebab utama eskalasi dalam insiden ini, kontradiksi yang perlu diperbaiki di akar penyebabnya, bukan sekadar "tambah lebih banyak resource database".

## Options

### Option 1 — Tambah kapasitas database

Upgrade database ke kapasitas lebih besar supaya bisa menyerap lonjakan retry di masa depan. Solusi yang mengatasi gejala (kapasitas kurang), bukan akar masalah (retry storm) — lonjakan permintaan dari retry storm bisa terus tumbuh proporsional dengan jumlah service dan pengguna, sehingga kapasitas yang "cukup" hari ini bisa tidak cukup lagi nanti, dan tidak ada batas atas yang masuk akal untuk terus menaikkan kapasitas demi mengantisipasi lonjakan retry yang seharusnya bisa dicegah.

### Option 2 — Exponential backoff dengan jitter di semua retry

Terapkan [[../30 APIs and Web/Retries with Exponential Backoff and Jitter|Retries with Exponential Backoff and Jitter]] di seluruh service yang melakukan retry — interval antar percobaan membesar secara eksponensial, dan jitter acak mencegah banyak retry dari berbagai service menumpuk tepat di waktu yang sama. Mengurangi intensitas lonjakan secara signifikan, tapi tidak sepenuhnya menghilangkan risiko kalau jumlah service dan permintaan yang retry memang sangat besar.

### Option 3 — Circuit breaker di setiap titik panggilan ke database/service lain

Tambahkan [[../30 APIs and Web/Circuit Breakers|Circuit Breakers]] — setelah sejumlah kegagalan berturut-turut, circuit "terbuka" dan berhenti mencoba sama sekali untuk periode tertentu, memberi waktu sistem yang bermasalah pulih tanpa terus dibombardir permintaan (retry atau bukan). Kombinasi paling matang dengan Option 2 — backoff mengurangi intensitas retry individual, circuit breaker mencegah retry sama sekali saat jelas sistem target sedang bermasalah.

## The Decision

Kombinasi Option 2 dan 3 dipilih sebagai solusi struktural, diterapkan di **semua** titik panggilan lintas 13 aplikasi yang saling bergantung (bukan hanya aplikasi yang mengalami insiden langsung). Constraint penentunya: akar masalah insiden ini bukan kapasitas database yang kurang (Option 1 tidak menjawab akar masalah), tapi pola retry yang secara kolektif menciptakan lonjakan self-inflicted — memperbaiki pola retry di semua service yang terlibat adalah satu-satunya cara mencegah pola yang sama terulang, terlepas dari seberapa besar kapasitas database yang tersedia.

## What Breaks at 10x and 100x

Pada volume 10x (lebih banyak service saling terhubung), circuit breaker per-koneksi individual mulai perlu dikoordinasikan lebih baik — kalau banyak circuit breaker terbuka bersamaan karena satu titik gagal di tengah (misalnya satu service hub yang dipanggil banyak service lain), efek domino kegagalan tetap bisa terjadi meski masing-masing circuit breaker bekerja benar secara individual. Pada 100x, load shedding di level lebih tinggi (lihat [[../30 APIs and Web/Load Shedding|Load Shedding]]) — menolak permintaan baru secara proaktif saat sistem sudah dekat kapasitas, bukan menunggu sampai benar-benar down — jadi lapisan pertahanan tambahan yang perlu ditambahkan di atas circuit breaker dan backoff yang sudah ada.

## What You Would Do Differently

Menetapkan standar retry (backoff eksponensial dengan jitter, plus circuit breaker) sebagai kebijakan wajib lintas 13 aplikasi sejak awal — bukan setiap tim mengimplementasikan retry sendiri-sendiri dengan cara berbeda-beda, yang justru jadi penyebab pola retry yang tidak terkoordinasi seperti terlihat dalam insiden ini. Ini adalah contoh konkret kenapa [[../90 Architecture and Design/API Governance|API Governance]] — standar teknis yang disepakati dan ditegakkan lintas tim — relevan langsung untuk mencegah insiden sistemik semacam ini.

## Concepts Exercised

- [[../30 APIs and Web/Circuit Breakers|Circuit Breakers]] — mekanisme inti yang mencegah retry storm terjadi berulang.
- [[../30 APIs and Web/Retries with Exponential Backoff and Jitter|Retries with Exponential Backoff and Jitter]] — perbaikan langsung terhadap pola retry yang jadi penyebab insiden.
- [[../30 APIs and Web/Load Shedding|Load Shedding]] — lapisan pertahanan tambahan untuk skala yang lebih besar.

## Your Turn

Setelah circuit breaker diterapkan, sebuah service mengalami situasi di mana circuit breaker-nya terbuka (berhenti mencoba database) tepat saat ada permintaan kritis dari petugas yang sedang menyelesaikan proses tenggat hukum mendesak. Apakah circuit breaker seharusnya tetap menolak permintaan ini tanpa pengecualian?

> [!success]- Kunci jawaban
> Ini adalah trade-off yang sudah inheren pada desain circuit breaker — menerima sebagian permintaan gagal cepat (fail fast) demi mencegah kerusakan yang lebih luas, bukan mencoba melayani setiap permintaan individual apa pun risikonya bagi sistem secara keseluruhan. Untuk kasus yang benar-benar kritis, solusi yang lebih baik bukan melewati circuit breaker begitu saja (yang berisiko mengembalikan masalah retry storm untuk permintaan "istimewa" ini), melainkan merancang **degradasi terkontrol** — misalnya menampilkan pesan jelas ke petugas bahwa sistem sedang mengalami gangguan sementara dengan estimasi waktu pulih, dan menyediakan jalur eskalasi manual (menghubungi tim teknis langsung) untuk kasus yang benar-benar mendesak — solusi organisasi dan komunikasi, bukan solusi teknis yang mengorbankan kembali stabilitas sistem demi satu permintaan.

## Catatan Saya

*Kalau kamu punya kejadian nyata dari kerjaan yang mirip tema ini, tulis sebagai case study baru dengan `source: personal`.*
