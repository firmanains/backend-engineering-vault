---
title: Needs Verification
type: concept
level: junior
domain: foundations
status: mastered
difficulty: 1
est_minutes: 3
prerequisites: []
next: []
tags: [backend, meta]
created: 2026-07-26
---

Daftar setiap klaim di vault yang ditandai `> [!question] Perlu diverifikasi` di dalam note-nya. Setiap note yang dikerjakan menjalani self-review pass sebelum dikirim (lihat `CLAUDE.md` §16) — klaim yang tidak bisa diverifikasi dari memori tetap dipertahankan kalau ia membantu penjelasan, tapi ditandai di sini dan di dalam note itu sendiri, bukan dinyatakan dengan percaya diri palsu.

## Format Baris

| Note | Klaim | Kenapa ragu | Cara verifikasi |
|---|---|---|---|
| [[../10 Foundations/Syscalls and File Descriptors\|Syscalls and File Descriptors]] | Nilai default `ulimit -n` pada distribusi Linux modern. | Berbeda-beda antar distro, versi kernel, dan konfigurasi container runtime, dan berubah dari waktu ke waktu. | Jalankan `ulimit -n` langsung di environment production/staging yang relevan. |
| [[../10 Foundations/How An OS Handles Network Connections\|How An OS Handles Network Connections]] | Nilai default `net.core.somaxconn` dan bagaimana persis `net.Listen` di Go menentukan backlog effective-nya. | Nilai default berbeda antar versi kernel Linux/distro, dan perilaku `net.Listen` bisa berubah antar versi Go. | Periksa `sysctl net.core.somaxconn` di server relevan, dan changelog `net` package Go yang dipakai. |
| [[../10 Foundations/TCP Handshake and Connection Lifecycle\|TCP Handshake and Connection Lifecycle]] | Durasi standar `TIME_WAIT` sering disebut "2×MSL". | Nilai MSL dan durasi efektif `TIME_WAIT` bisa dikonfigurasi berbeda antar OS/kernel. | Periksa parameter kernel (`net.ipv4.tcp_fin_timeout` di Linux) langsung di server relevan. |
| [[../10 Foundations/Introduction to HTTP2\|Introduction to HTTP2]] | Detail dukungan HTTP/2 bawaan `net/http` (termasuk h2c/cleartext) berbeda antar versi Go. | Perilaku default `net/http` terhadap HTTP/2 pernah berubah antar rilis Go; h2c tidak selalu built-in. | Baca release notes Go untuk versi yang dipakai, dan dokumentasi `golang.org/x/net/http2`. |
| [[../10 Foundations/The TLS Handshake\|The TLS Handshake]] | Jumlah round-trip pasti TLS 1.2 vs TLS 1.3, dan perilaku spesifik 0-RTT resumption. | Bergantung mode resumption yang dipakai, bisa berbeda antar implementasi/konfigurasi library. | Baca RFC 8446 bagian handshake, atau packet capture (`tcpdump`/Wireshark) pada koneksi nyata. |
| [[../20 Go Language/Map Internals\|Map Internals]] | Konsistensi deteksi *concurrent map access* sebagai fatal error di semua kondisi timing dan versi Go. | Detektor ini best-effort, detail implementasi bisa berbeda antar versi Go. | Baca dokumentasi package `runtime` dan release notes terkait; jalankan `go test -race` untuk deteksi eksplisit. |
| [[../20 Go Language/Table-Driven Tests\|Table-Driven Tests]] | `tc := tc` di dalam loop hanya dibutuhkan untuk subtest paralel (`t.Parallel()`) pada Go sebelum 1.22. | Perilaku bergantung versi Go proyek — Go 1.22 mengubah semantik variable loop per-iterasi secara default. | Periksa versi Go di `go.mod`, dan baca release notes Go 1.22 soal semantik variable loop `for`. |
| [[../30 APIs and Web/REST Principles\|REST Principles]] | `http.ServeMux` bawaan mendukung pattern method HTTP (`"GET /path"`) dan wildcard path (`{id}`) sejak versi tertentu. | Fitur ini penambahan relatif baru di `net/http`; proyek Go versi lebih lama mungkin butuh router pihak ketiga. | Periksa versi Go di `go.mod`, dan release notes Go terkait `ServeMux` enhancements. |
| [[../30 APIs and Web/Timeouts in HTTP Servers\|Timeouts in HTTP Servers]] | Cakupan dan interaksi persis `ReadTimeout` vs `WriteTimeout` (total siklus vs independen per fase). | Dokumentasi resmi Go sendiri mencatat nuansa ini dengan hati-hati; kesalahpahaman umum di komunitas. | Baca dokumentasi resmi `net/http.Server` (pkg.go.dev) untuk versi Go yang dipakai. |
| [[../30 APIs and Web/Pre-signed URLs\|Pre-signed URLs]] | Signature dan call spesifik pre-signed URL berbeda antar vendor object storage dan versi SDK. | Tidak ada API universal — tiap vendor punya SDK dan parameter signing sendiri. | Baca dokumentasi resmi SDK object storage yang benar-benar dipakai proyek. |
| [[../30 APIs and Web/OpenAPI\|OpenAPI]] | Detail command dan konfigurasi spesifik `oapi-codegen` atau tool code-gen OpenAPI lain untuk Go. | Tooling code-generation berkembang cepat, opsi konfigurasi berubah antar versi. | Baca dokumentasi resmi repository tool yang dipilih tim, untuk versi yang dipakai. |
| [[../40 Databases/Subqueries vs CTEs\|Subqueries vs CTEs]] | Perilaku CTE sebagai "optimization fence" vs "inlined" bervariasi antar versi PostgreSQL dan MySQL/MariaDB. | Perilaku ini pernah berubah signifikan di riwayat PostgreSQL (materialization behavior); detail versi mudah basi. | Dokumentasi resmi PostgreSQL bagian "WITH Queries", cari kata kunci "MATERIALIZED". |
| [[../40 Databases/Window Functions\|Window Functions]] | MariaDB mendukung window function sejak versi 10.2. | Nomor versi mudah salah diingat, bisa tertukar dengan MySQL (cabang berbeda, mendukung sejak 8.0). | Changelog resmi MariaDB fitur "window functions", atau `SELECT VERSION();` lalu cek dokumentasi versi tersebut. |
| [[../40 Databases/Aggregation and GROUP BY Semantics\|Aggregation and GROUP BY Semantics]] | `ONLY_FULL_GROUP_BY` aktif secara default di instalasi MariaDB yang lebih baru. | Default `sql_mode` berbeda antara MySQL dan MariaDB (project bercabang terpisah), dan berubah antar versi. | `SELECT @@sql_mode;` di instance MariaDB relevan, atau release notes MariaDB untuk versi yang dipakai. |
| [[../40 Databases/Set Operations in SQL\|Set Operations in SQL]] | Versi MariaDB pertama yang mendukung `INTERSECT` dan `EXCEPT`. | Fitur penambahan relatif baru di riwayat MariaDB, nomor versi persis mudah salah diingat. | Changelog resmi MariaDB fitur "INTERSECT"/"EXCEPT", atau `SELECT VERSION();` lalu cek dokumentasi versi tersebut. |
| [[../40 Databases/Data Types and Constraints\|Data Types and Constraints]] | Dukungan `CHECK` constraint yang benar-benar ditegakkan (bukan hanya diterima secara sintaks) di MariaDB/MySQL bergantung versi. | MySQL sempat menerima sintaks `CHECK` tanpa menegakkannya di versi lama, sebelum penegakan sungguhan ditambahkan. | Dokumentasi resmi MySQL/MariaDB bagian "CHECK Constraints", periksa versi produksi yang dipakai. |
| [[../40 Databases/ACID\|ACID]] | InnoDB adalah storage engine default untuk tabel baru di instalasi MariaDB modern. | Default storage engine bisa dikonfigurasi berbeda di level server (`my.cnf`); sistem legacy mungkin tidak mengikuti default versi terbaru. | `SHOW TABLE STATUS` atau `SELECT ENGINE FROM information_schema.TABLES WHERE table_name = '...'` pada tabel relevan. |
| [[../40 Databases/Basic Isolation Levels\|Basic Isolation Levels]] | MySQL/MariaDB (InnoDB) default `REPEATABLE READ`, PostgreSQL default `READ COMMITTED`. | Default isolation level bisa dikonfigurasi berbeda di level server; sebaiknya tidak diasumsikan tanpa verifikasi untuk instance tertentu. | `SELECT @@transaction_isolation;` (MariaDB) atau `SHOW default_transaction_isolation;` (PostgreSQL) di instance relevan. |
| [[../80 Security/Encryption at Rest vs In Transit\|Encryption at Rest vs In Transit]] | Nama parameter konfigurasi enkripsi InnoDB (`innodb_encrypt_tables` dan sejenisnya) di MariaDB. | Nama dan perilaku parameter ini berbeda antar versi MariaDB/MySQL, dan berubah cukup sering. | Dokumentasi resmi MariaDB bagian "Data at Rest Encryption" untuk versi yang benar-benar dipakai. |
| [[../80 Security/Compliance Trails for Government Systems\|Compliance Trails for Government Systems]] | Kewajiban spesifik retensi audit trail, chain of custody, dan pelaporan untuk sistem legal-services pemerintah di bawah regulasi perlindungan data pribadi yang berlaku. | Detail kewajiban ini spesifik per regulasi dan bisa berubah lewat aturan turunan; note ini tidak boleh jadi rujukan hukum. | Konsultasi langsung dengan bagian legal/kepatuhan instansi tempat bekerja. |
| [[../70 Infrastructure and Delivery/Kubernetes Config, Secrets, Probes, and Autoscaling\|Kubernetes Config, Secrets, Probes, and Autoscaling]] | Perilaku default penyimpanan Secret di etcd (base64 tanpa enkripsi) dan opsi encryption-at-rest yang tersedia. | Perilaku ini bisa berbeda tergantung distribusi Kubernetes (managed service dari vendor cloud sering mengaktifkan encryption at rest secara default, instalasi self-managed sering tidak). | Dokumentasi resmi Kubernetes bagian "Encrypting Confidential Data at Rest", dan dokumentasi spesifik distribusi/vendor yang benar-benar dipakai. |

## Connected Notes

- [[Vault Manifest]] — konteks note mana saja yang sudah/akan ditulis dan berpotensi memuat klaim yang perlu diverifikasi.
- [[Progress Tracker]] — status note; note dengan entri terbuka di sini idealnya tidak naik ke `mastered` sampai klaim terverifikasi atau dihapus.

## Catatan Saya

*Kosong — file ini murni catatan sistem, bukan tempat refleksi pribadi.*
