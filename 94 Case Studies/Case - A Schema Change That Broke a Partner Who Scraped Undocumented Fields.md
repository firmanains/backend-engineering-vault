---
title: Case - A Schema Change That Broke a Partner Who Scraped Undocumented Fields
type: case
level: intermediate
domain: architecture
status: unread
difficulty: 3
est_minutes: 14
source: synthesized
concepts: ["[[../30 APIs and Web/Contract Negotiation and Versioning|Contract Negotiation and Versioning]]", "[[../30 APIs and Web/API Versioning|API Versioning]]", "[[../90 Architecture and Design/API Governance|API Governance]]"]
prerequisites: []
next: []
tags: [backend, architecture, apis, gov-context, integration]
created: 2026-08-02
---

## The Situation

Sebuah tim menghapus field `internal_notes` dari response API — field itu tidak pernah didokumentasikan secara resmi dalam kontrak API (dianggap detail implementasi internal yang kebetulan ikut terserialisasi ke JSON response), dan tim yakin perubahan ini aman karena "tidak ada di dokumentasi, jadi tidak ada yang seharusnya bergantung padanya." Begitu perubahan ini dirilis, sebuah partner instansi melapor sistem mereka error — investigasi mengungkap partner itu ternyata mem-parsing field `internal_notes` ini secara langsung dari response JSON, meski field itu tidak pernah didokumentasikan atau dijanjikan sebagai bagian dari kontrak resmi.

## What Was Actually Asked For

Tim awalnya berpikir mereka "sudah benar" karena mengikuti dokumentasi resmi API secara ketat. Tapi kenyataan menunjukkan kebutuhan yang lebih dalam: **kontrak API sesungguhnya adalah apa yang benar-benar dipakai konsumen**, bukan sekadar apa yang tertulis di dokumentasi — prinsip yang dikenal luas di industri sebagai "Hyrum's Law" (dengan cukup banyak pengguna sebuah API, semua perilaku yang dapat diamati dari sistem itu — didokumentasikan atau tidak — akan dijadikan sandaran oleh seseorang).

## Options

### Option 1 — Tegaskan bahwa perubahan ini "sesuai kontrak", minta partner memperbaiki sendiri

Berpegang pada argumen teknis bahwa field itu memang tidak pernah dijanjikan, dan partner seharusnya tidak bergantung padanya. Secara teknis "benar", tapi merusak hubungan kerja dengan partner dan tidak menyelesaikan masalah operasional mereka yang sekarang error — pendekatan yang defensif dan tidak produktif untuk integrasi lintas organisasi yang harus terus berjalan.

### Option 2 — Kembalikan field itu, umumkan periode deprecation formal

Batalkan penghapusan, kembalikan field `internal_notes` untuk sementara, sambil mengumumkan secara resmi (lewat dokumentasi dan komunikasi langsung ke semua partner yang diketahui terhubung) bahwa field ini akan benar-benar dihapus setelah periode tertentu (misalnya 3 bulan), memberi waktu partner menyesuaikan diri. Solusi yang mengakui kenyataan operasional (partner memang bergantung padanya, terlepas dari status "resmi" atau tidak), sambil tetap menuju tujuan awal (membersihkan field yang tidak seharusnya ada di response publik).

### Option 3 — Kembalikan field secara permanen, jadikan bagian resmi kontrak

Alih-alih menghapusnya, terima kenyataan bahwa field ini sudah jadi bagian dari kontrak de facto (dipakai partner), dokumentasikan secara resmi, dan pertahankan selamanya seperti field lain yang memang dimaksudkan publik. Menghindari proses deprecation sepenuhnya, tapi mungkin mempertahankan field yang sebenarnya tidak seharusnya pernah terekspos (detail internal yang mungkin mengandung informasi yang tidak dimaksudkan untuk dilihat pihak luar).

## The Decision

Option 2 dipilih. Constraint penentunya: field ini, setelah ditinjau, memang berisi detail implementasi internal yang idealnya tidak terekspos jangka panjang (menjustifikasi tetap menghapusnya akhirnya, bukan Option 3), tapi partner yang sudah bergantung padanya berhak atas waktu transisi yang wajar (menjustifikasi tidak menghapusnya mendadak, bukan mempertahankan posisi keras di Option 1). Proses deprecation formal — mengumumkan, memberi tenggat jelas, berkomunikasi langsung dengan partner yang diketahui terdampak — adalah jalan tengah yang menghormati baik kebutuhan teknis (membersihkan API) maupun kenyataan operasional (partner sudah bergantung pada perilaku yang tidak didokumentasikan).

## What Breaks at 10x and 100x

Pada volume 10x (lebih banyak field "tidak resmi" serupa ditemukan bergantung oleh berbagai partner berbeda), menangani setiap kasus secara ad-hoc jadi tidak terkelola — titik ini membenarkan investasi proses formal [[../90 Architecture and Design/API Governance|API Governance]]: kontrak API didefinisikan eksplisit lewat skema (OpenAPI, misalnya) yang menjadi sumber kebenaran tunggal, dan response API secara struktural tidak boleh mengekspos field yang tidak ada di skema itu — mencegah masalah field "tidak resmi" muncul sejak awal, bukan ditangani reaktif setiap kali ditemukan. Pada 100x (API ini dipakai puluhan partner berbeda), setiap perubahan kontrak — bahkan yang terlihat kecil — butuh proses formal dengan visibility luas (siapa saja konsumen yang mungkin terdampak), bukan keputusan sepihak satu tim yang tidak tahu siapa saja yang benar-benar memakai API ini.

## What You Would Do Differently

Menegakkan skema response API secara struktural sejak awal (misalnya lewat serialisasi eksplisit yang hanya menyertakan field yang benar-benar dimaksudkan publik, bukan serialisasi otomatis dari seluruh struct internal) — mencegah field internal seperti `internal_notes` pernah terekspos ke response publik sama sekali, menghilangkan akar masalah ini secara struktural alih-alih menanganinya reaktif setelah partner terlanjur bergantung padanya.

## Concepts Exercised

- [[../30 APIs and Web/Contract Negotiation and Versioning|Contract Negotiation and Versioning]] — proses deprecation formal yang jadi solusi kasus ini.
- [[../30 APIs and Web/API Versioning|API Versioning]] — pertimbangan versi yang relevan untuk perubahan kontrak yang benar-benar breaking.
- [[../90 Architecture and Design/API Governance|API Governance]] — kerangka jangka panjang mencegah masalah field "tidak resmi" muncul berulang di masa depan.

## Your Turn

Setelah kejadian ini, seorang developer mengusulkan agar setiap response API selalu menyertakan **seluruh** field yang ada di database (pendekatan "biar aman, sertakan semua"), supaya tidak ada lagi kejutan field yang "seharusnya tidak ada tapi ternyata dipakai". Apakah ini pendekatan yang tepat?

> [!success]- Kunci jawaban
> Justru sebaliknya — pendekatan ini memperparah masalah, bukan menyelesaikannya. Menyertakan **seluruh** field database di response API berarti **setiap** field, tidak peduli seberapa internal sifatnya, berpotensi jadi kontrak de facto begitu ada satu konsumen yang mulai bergantung padanya (persis pola Hyrum's Law yang menyebabkan masalah ini sejak awal) — semakin banyak field terekspos, semakin besar permukaan yang bisa diam-diam jadi "kontrak" tanpa disadari. Pendekatan yang benar adalah kebalikannya: response API secara eksplisit dan sengaja hanya menyertakan field yang **dimaksudkan** untuk publik, didefinisikan lewat skema/struct response terpisah dari struct database internal — field baru hanya ditambahkan ke response setelah keputusan sadar bahwa field itu memang dimaksudkan jadi bagian kontrak publik, bukan kebetulan ikut terserialisasi karena struct internal dan struct response adalah satu struct yang sama.

## Catatan Saya

*Kalau kamu punya kejadian nyata dari kerjaan yang mirip tema ini, tulis sebagai case study baru dengan `source: personal`.*
