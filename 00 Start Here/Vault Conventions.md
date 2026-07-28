---
title: Vault Conventions
type: concept
level: junior
domain: foundations
status: mastered
difficulty: 1
est_minutes: 10
prerequisites: ["[[Read Me First]]"]
next: ["[[Backend Engineering Map]]"]
tags: [backend, meta]
created: 2026-07-26
---

Ini adalah referensi teknis vault ini: frontmatter, tag, dan aturan penamaan file. Note ini tidak perlu dibaca sampai hafal — cukup tahu bahwa ia ada, lalu kembali ke sini setiap kali menulis atau mengedit note dan lupa formatnya.

## Frontmatter

Setiap note dibuka dengan blok YAML ini. Semua query Dataview di vault bergantung padanya, jadi field yang salah ketik akan membuat note itu hilang dari semua tabel progress.

```yaml
---
title: Connection Pooling
type: concept            # concept | tool | case | project | term
level: junior             # junior | intermediate | senior
domain: databases         # foundations | go | apis | databases | concurrency | distributed | infrastructure | security | architecture | tools
status: unread             # unread | learning | understood | mastered
difficulty: 2              # 1-5
est_minutes: 12
prerequisites: ["[[TCP Connection Lifecycle]]", "[[Database Transactions]]"]
next: ["[[The N+1 Query Problem]]"]
tags: [backend, databases, performance, go]
created: 2026-07-26
---
```

Field tambahan untuk tool note:

```yaml
type: tool
depth: working             # deep | working | orientation
volatility: high            # high | medium | low — seberapa cepat pengetahuan ini basi
implements: ["[[Log-Based Messaging]]", "[[Consumer Groups and Rebalancing]]"]
```

Field tambahan untuk case study:

```yaml
type: case
source: synthesized         # personal | synthesized | industry
concepts: ["[[Idempotency]]", "[[API Payload Design]]"]
```

### Catatan tentang `domain`

Nilai `domain` memakai nama pendek, bukan nama folder penuh, supaya query Dataview tetap ringkas:

| `domain` value | Folder |
|---|---|
| `foundations` | `10 Foundations/` |
| `go` | `20 Go Language/` |
| `apis` | `30 APIs and Web/` |
| `databases` | `40 Databases/` |
| `concurrency` | `50 Concurrency and Performance/` |
| `distributed` | `60 Distributed Systems/` |
| `infrastructure` | `70 Infrastructure and Delivery/` |
| `security` | `80 Security/` |
| `architecture` | `90 Architecture and Design/` |
| `tools` | `92 Tools/` |

## Tag Registry

Tag dipakai untuk pengelompokan lintas domain yang tidak tertangkap oleh folder atau `domain`. Jangan membuat tag baru tanpa menambahkannya ke tabel ini.

| Tag | Dipakai untuk |
|---|---|
| `#backend` | Tag induk, ada di **setiap** note tanpa kecuali — ini yang membuat query `FROM #backend` di seluruh vault bekerja. |
| `#go` | Note yang membahas Go secara langsung (bukan hanya menyebut Go sebagai contoh). |
| `#meta` | Note tentang vault ini sendiri (folder `00 Start Here/`, `01 Maps/`, `02 Templates/`). |
| `#foundations`, `#apis`, `#databases`, `#concurrency`, `#distributed`, `#infrastructure`, `#security`, `#architecture` | Sama dengan `domain`, disediakan supaya bisa dipakai sebagai filter tag terpisah dari field `domain`. |
| `#performance` | Note yang secara spesifik membahas latency, throughput, atau resource usage. |
| `#integration` | Note yang membahas komunikasi sistem-ke-sistem, terutama dengan partner eksternal. |
| `#gov-context` | Case study atau catatan yang secara eksplisit relevan dengan konteks sistem pemerintah/legal-services. |

## Penamaan File

- **Concept note**: judul deskriptif dan alami, tanpa prefix angka. Contoh: `Connection Pooling.md`, `Goroutine Scheduler (GMP).md`, `Why Indexes Make Reads Fast.md`. Nama file *adalah* teks link — `[[Connection Pooling]]` enak dibaca di tengah kalimat, `[[40-03-conn-pool]]` merusaknya.
- **Glossary note**: `Term - Idempotency.md`.
- **Tool note**: nama produk itu sendiri — `PostgreSQL.md`, `Kafka.md`, `Grafana.md`. Sub-note memakai tanda hubung: `PostgreSQL - Locking and SELECT FOR UPDATE.md`.
- **Case study**: `Case - Sending PDFs Through a JSON API.md`.
- **Project**: `Project 1 - Authenticated REST API.md`.
- Tidak ada tanggal, tidak ada nomor versi, tidak ada singkatan yang belum jadi standar industri di nama file.

## Callout yang Dipakai

- `> [!info]` — konteks tambahan yang bukan peringatan.
- `> [!warning] Jebakan` — kesalahan umum, selalu memakai judul "Jebakan".
- `> [!tip]` — saran praktis.
- `> [!example]` — contoh konkret.
- `> [!success]- Kunci jawaban` — jawaban exercise, selalu foldable (`-` setelah tipe callout).
- `> [!question] Perlu diverifikasi` — klaim yang belum bisa diverifikasi penuh; lihat `Needs Verification.md`.

## Connected Notes

- [[Read Me First]] — konteks tujuan vault yang membuat konvensi ini masuk akal.
- [[How To Read This Vault]] — siklus `status` yang dijelaskan di sini secara teknis.
- [[Concept Note Template]], [[Tool Note Template]], [[Case Study Template]] — template yang memakai semua field dan aturan di atas secara langsung.
- [[Progress Tracker]] — konsumen utama dari frontmatter ini lewat query Dataview.

## Catatan Saya

*Tulis di sini kalau kamu menambahkan domain atau tag baru, dan alasannya.*
