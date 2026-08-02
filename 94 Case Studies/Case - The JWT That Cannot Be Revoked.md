---
title: Case - The JWT That Cannot Be Revoked
type: case
level: intermediate
domain: architecture
status: unread
difficulty: 3
est_minutes: 14
source: synthesized
concepts: ["[[../80 Security/JWT - Structure, Signature, and When It Is The Wrong Tool|JWT - Structure, Signature, and When It Is The Wrong Tool]]", "[[../80 Security/Sessions vs Tokens|Sessions vs Tokens]]", "[[../80 Security/Key Management and Rotation|Key Management and Rotation]]"]
prerequisites: []
next: []
tags: [backend, architecture, security, gov-context]
created: 2026-08-02
---

## The Situation

Sebuah sistem memakai JWT untuk autentikasi, dengan masa berlaku token 24 jam demi mengurangi beban validasi ke database di setiap request (server hanya perlu memverifikasi signature, tidak perlu query "apakah sesi ini masih valid"). Suatu hari, akun seorang petugas diketahui disusupi (kredensialnya bocor) — tim keamanan ingin segera mencabut akses akun itu. Mereka menemukan masalah: JWT yang sudah diterbitkan untuk akun itu **tetap valid** sampai 24 jam masa berlakunya habis, karena verifikasi JWT murni berbasis signature, tanpa pengecekan ke database yang bisa dipakai "mematikan" token tertentu secara langsung.

## What Was Actually Asked For

Permintaan tim keamanan: "cabut akses akun ini sekarang." Kebutuhan sebenarnya terungkap sebagai konflik desain yang lebih dalam: sistem JWT stateless (yang sengaja dipilih demi performa) secara inheren sulit didamaikan dengan kebutuhan revocation instan — dua tujuan desain yang saling bertentangan, dan sistem yang ada belum pernah secara eksplisit memutuskan mana yang lebih penting.

## Options

### Option 1 — Perpendek masa berlaku JWT secara signifikan

Turunkan masa berlaku dari 24 jam jadi 15 menit, mengurangi jendela waktu token yang bocor bisa disalahgunakan. Mengurangi risiko, tapi tidak menghilangkannya — masih ada jendela (meski pendek) di mana token yang seharusnya sudah dicabut tetap valid, dan token yang sangat pendek butuh mekanisme refresh yang lebih sering, menambah kompleksitas dan traffic.

### Option 2 — Denylist token yang dicabut

Simpan daftar token (atau ID token, `jti` claim) yang secara eksplisit dicabut sebelum masa berlaku aslinya habis — setiap request memeriksa apakah token itu ada di denylist sebelum dianggap valid. Menyelesaikan masalah revocation instan, tapi mengembalikan kebutuhan query per-request yang tadinya ingin dihindari dengan memakai JWT stateless — kontradiksi langsung dengan alasan awal memilih JWT.

### Option 3 — Hybrid: JWT pendek plus refresh token yang bisa dicabut

JWT (access token) berumur sangat pendek (misalnya 5 menit, hanya dipakai untuk verifikasi cepat tanpa query database), dipasangkan dengan refresh token (disimpan di database, bisa dicabut kapan saja) yang dipakai menerbitkan access token baru setiap kali yang lama kedaluwarsa (lihat [[../80 Security/Sessions vs Tokens|Sessions vs Tokens]] untuk trade-off dasar token vs sesi). Mencabut refresh token berarti access token baru tidak bisa diterbitkan lagi — dalam waktu maksimal 5 menit (masa berlaku access token terakhir yang mungkin masih beredar), akses benar-benar terputus.

## The Decision

Option 3 dipilih — pendekatan hybrid ini mempertahankan sebagian besar manfaat performa JWT stateless (mayoritas request tetap diverifikasi cepat tanpa query database), sambil memberi kemampuan revocation yang cukup cepat (dalam hitungan menit, bukan sampai 24 jam) lewat pencabutan refresh token. Constraint penentunya: revocation instan sempurna (Option 2) mengorbankan seluruh alasan awal memilih JWT; menerima jendela revocation singkat (5 menit, bukan instan sempurna) adalah kompromi yang defensible untuk kebanyakan skenario, sementara tetap jauh lebih baik dari status quo 24 jam.

## What Breaks at 10x and 100x

Pada volume 10x, refresh token yang disimpan di database (bukan lagi sepenuhnya stateless) mulai menambah beban query yang sebelumnya coba dihindari — meski jauh lebih jarang dibanding memeriksa setiap request (hanya saat refresh, bukan setiap panggilan API), beban ini perlu dipantau seiring pertumbuhan pengguna aktif. Pada 100x lintas banyak layanan yang masing-masing menerbitkan token sendiri, kebutuhan revocation terpusat (satu tempat mencabut akses ke **semua** layanan sekaligus, bukan satu per satu) menjadi kebutuhan arsitektural baru — mendekati kebutuhan sistem identity provider terpusat (seperti yang diimplementasikan penuh oleh OAuth2/OIDC provider).

## What You Would Do Differently

Mempertimbangkan trade-off revocation ini secara eksplisit **sebelum** memilih JWT sebagai mekanisme autentikasi, bukan menemukan konflik ini reaktif setelah insiden nyata terjadi — keputusan "JWT stateless demi performa" seharusnya sejak awal disandingkan dengan pertanyaan "bagaimana kita mencabut akses kalau perlu", bukan pertanyaan yang baru muncul saat sudah terlambat.

## Concepts Exercised

- [[../80 Security/JWT - Structure, Signature, and When It Is The Wrong Tool|JWT - Structure, Signature, and When It Is The Wrong Tool]] — batasan inheren JWT yang jadi akar masalah kasus ini.
- [[../80 Security/Sessions vs Tokens|Sessions vs Tokens]] — trade-off dasar yang relevan dipertimbangkan ulang dalam desain hybrid.
- [[../80 Security/Key Management and Rotation|Key Management and Rotation]] — pertimbangan tambahan: mencabut satu token individual berbeda dari mencabut seluruh kunci penandatanganan (yang membatalkan **semua** token sekaligus, langkah jauh lebih drastis).

## Your Turn

Tim mempertimbangkan solusi yang lebih ekstrem: mencabut **kunci penandatanganan** JWT sepenuhnya begitu ada indikasi akun disusupi, membatalkan semua token yang pernah diterbitkan sekaligus. Kenapa ini biasanya bukan solusi yang tepat untuk kasus satu akun yang disusupi?

> [!success]- Kunci jawaban
> Mencabut kunci penandatanganan membatalkan **seluruh** token yang pernah diterbitkan dengan kunci itu — bukan hanya token milik akun yang disusupi, tapi token milik **semua** pengguna yang sedang aktif menggunakan sistem. Ini setara memaksa logout massal seluruh pengguna untuk menangani masalah satu akun — dampak yang jauh tidak proporsional dengan masalah aslinya, dan justru bisa mengganggu operasional petugas lain yang sedang bekerja tanpa alasan yang berkaitan dengan mereka sama sekali. Mencabut kunci penandatanganan hanya masuk akal untuk skenario yang jauh lebih serius — kunci itu sendiri yang bocor atau dicurigai disusupi, bukan satu akun pengguna individual. Untuk kasus satu akun, solusi yang tepat tetap revocation spesifik akun itu (lewat refresh token yang dicabut di Option 3), bukan tindakan drastis yang berdampak ke semua orang.

## Catatan Saya

*Kalau kamu punya kejadian nyata dari kerjaan yang mirip tema ini, tulis sebagai case study baru dengan `source: personal`.*
