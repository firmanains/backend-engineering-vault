---
title: Project 1 - Authenticated REST API
type: project
level: junior
domain: architecture
status: unread
difficulty: 3
est_minutes: 0
concepts: ["[[../30 APIs and Web/REST Principles|REST Principles]]", "[[../80 Security/Password Hashing - bcrypt and argon2|Password Hashing - bcrypt and argon2]]", "[[../40 Databases/Database Migrations|Database Migrations]]", "[[../90 Architecture and Design/Handler-Service-Repository Layering|Handler-Service-Repository Layering]]"]
prerequisites: []
next: ["[[Project 2 - Worker Pool, Cache, and Streaming Service]]"]
tags: [backend, architecture, go, apis, databases, security]
created: 2026-08-02
---

## Learning Objectives

Membangun REST API produksi-realistis dari nol, menerapkan seluruh fondasi level junior sekaligus dalam satu sistem yang saling berkaitan — bukan latihan terisolasi per konsep. Setelah menyelesaikan proyek ini, kamu harus bisa: merancang resource REST yang idiomatic, mengimplementasikan autentikasi yang aman, menjalankan migration database secara terkontrol, menerapkan layering yang membuat kode bisa diuji dan dipelihara, dan menangani upload/download file dengan benar — bukan sekadar tahu masing-masing konsep secara teoretis, tapi tahu bagaimana mereka berinteraksi dan di mana keputusan satu bagian memengaruhi bagian lain.

## The Brief

Bangun API untuk sistem pengajuan permohonan sederhana — pengguna bisa mendaftar, login, mengajukan permohonan baru (dengan lampiran dokumen), melihat daftar dan detail permohonan miliknya sendiri, dan (untuk peran admin) melihat serta memperbarui status semua permohonan. Sistem ini kecil, tapi setiap bagiannya harus dikerjakan sampai standar yang layak dipakai sungguhan — bukan prototipe yang "kelihatannya jalan".

## Requirements

**Autentikasi dan otorisasi**
- Registrasi dan login dengan password yang di-hash memakai bcrypt atau argon2 (lihat [[../80 Security/Password Hashing - bcrypt and argon2|Password Hashing - bcrypt and argon2]]) — tidak pernah menyimpan password plain text.
- Autentikasi berbasis token (JWT atau session, pilih salah satu dan jelaskan alasannya di dokumentasi proyek).
- Dua peran: `user` (hanya bisa mengakses permohonan miliknya) dan `admin` (bisa melihat dan mengubah status semua permohonan) — terapkan RBAC sederhana yang menegakkan ini di level handler/middleware, bukan diperiksa manual di setiap fungsi secara tidak konsisten.

**Resource REST**
- `POST /permohonan` — buat permohonan baru.
- `GET /permohonan` — daftar permohonan (user: miliknya sendiri; admin: semua, dengan filter status).
- `GET /permohonan/{id}` — detail satu permohonan.
- `PATCH /permohonan/{id}/status` — ubah status (admin saja).
- Pagination pada endpoint daftar (cursor-based atau offset-based, jelaskan pilihanmu).
- Format error response yang konsisten di seluruh endpoint.

**Database**
- Skema relasional dengan migration terversi (bukan `AutoMigrate` atau skema yang dibuat manual sekali lalu tidak pernah diubah terkontrol) — lihat [[../40 Databases/Database Migrations|Database Migrations]].
- Constraint yang benar (foreign key, `NOT NULL` di kolom yang memang wajib, unique constraint pada email pengguna).
- Connection pooling dikonfigurasi eksplisit, bukan dibiarkan default tanpa dipikirkan.

**File upload/download**
- Endpoint upload dokumen pendukung untuk permohonan, dengan validasi tipe dan ukuran file.
- Endpoint download yang memverifikasi otorisasi sebelum memberi akses file (pengguna tidak boleh mengunduh dokumen milik permohonan orang lain).

**Layering**
- Pisahkan handler (HTTP), service (logika bisnis), dan repository (akses data) secara eksplisit — lihat [[../90 Architecture and Design/Handler-Service-Repository Layering|Handler-Service-Repository Layering]]. Handler tidak boleh langsung memanggil database.

**Testing**
- Test untuk logika bisnis di layer service, memakai mock untuk dependency (database, dsb.) lewat interface — bukan test yang menyentuh database sungguhan untuk setiap kasus.
- Minimal satu test `httptest` yang menguji satu alur endpoint penuh (request masuk sampai response keluar).
- Test **harus** benar-benar menguji perilaku (termasuk kasus gagal — otorisasi ditolak, input tidak valid), bukan sekadar memastikan kode tidak crash.

## Staged Milestones

1. **Fondasi**: struktur project, koneksi database, migration awal (tabel `users`, `permohonan`, `dokumen`), layering kosong (handler/service/repository) yang saling terhubung tapi belum banyak logika.
2. **Autentikasi**: registrasi, login, hashing password, middleware yang memverifikasi token dan mengekstrak identitas pengguna ke context.
3. **CRUD Permohonan**: keempat endpoint inti berfungsi, dengan otorisasi RBAC yang benar (user vs admin), pagination pada endpoint daftar.
4. **File Handling**: upload dan download dokumen, dengan validasi dan otorisasi yang benar.
5. **Testing dan Polish**: test lengkap untuk layer service dan minimal satu test end-to-end, error handling konsisten di seluruh endpoint, dokumentasi API (README atau OpenAPI ringkas).

## Definition of Done

- Seluruh endpoint di atas berfungsi dan bisa diverifikasi manual (lewat `curl` atau Postman).
- `go test ./...` lulus tanpa error, mencakup kasus sukses **dan** kasus gagal (otorisasi ditolak, input tidak valid, resource tidak ditemukan).
- Tidak ada password atau secret ter-hardcode di kode — semua lewat environment variable.
- Migration bisa dijalankan dari database kosong sampai skema penuh tanpa langkah manual tambahan.
- README menjelaskan cara menjalankan proyek dari nol (setup database, environment variable, menjalankan migration, menjalankan server).

## Self-Review Checklist

- [ ] Apakah password di-hash dengan algoritma yang tepat, bukan disimpan atau dibandingkan sebagai plain text di titik mana pun?
- [ ] Apakah setiap endpoint yang butuh otorisasi benar-benar memverifikasinya, termasuk yang mudah terlewat (download file, misalnya)?
- [ ] Apakah service layer benar-benar tidak bergantung langsung pada detail HTTP (bisa diuji tanpa `net/http`)?
- [ ] Apakah setiap error dari database atau dependency lain ditangani secara eksplisit (dibungkus dengan `fmt.Errorf("...: %w", err)`), bukan diabaikan dengan `_`?
- [ ] Apakah migration bisa dijalankan berulang (idempoten) tanpa error kalau sudah pernah dijalankan sebelumnya?
- [ ] Kalau membaca kode ini enam bulan lagi tanpa ingatan apa pun tentang proyek ini, apakah struktur dan penamaannya cukup jelas untuk dipahami ulang dengan cepat?

## Concepts Exercised

- [[../30 APIs and Web/REST Principles|REST Principles]] — desain resource dan status code yang idiomatic.
- [[../80 Security/Password Hashing - bcrypt and argon2|Password Hashing - bcrypt and argon2]] — penerapan langsung hashing password yang aman.
- [[../40 Databases/Database Migrations|Database Migrations]] — migration terversi sebagai praktik wajib, bukan opsional.
- [[../90 Architecture and Design/Handler-Service-Repository Layering|Handler-Service-Repository Layering]] — struktur kode yang membuat proyek ini bisa diuji dan dipelihara.

## Catatan Saya

*Tulis di sini keputusan desain yang kamu ambil selama mengerjakan proyek ini (JWT vs session, cursor vs offset pagination, dsb.) dan alasannya.*
