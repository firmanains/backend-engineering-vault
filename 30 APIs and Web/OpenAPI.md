---
title: OpenAPI
type: concept
level: junior
domain: apis
status: unread
difficulty: 2
est_minutes: 14
prerequisites: ["[[REST Principles]]", "[[Consistent Error Responses]]"]
next: ["[[Contract Negotiation and Versioning]]", "[[../92 Tools/_Overview|Tools Overview]]"]
tags: [backend, apis, integration]
created: 2026-07-26
---

## TL;DR

OpenAPI (dulu disebut Swagger) adalah format spesifikasi yang bisa dibaca mesin (YAML/JSON) untuk mendeskripsikan endpoint, skema request/response, dan mekanisme autentikasi sebuah API — dijadikan **satu sumber kebenaran** yang bisa menghasilkan dokumentasi, client SDK, server stub, dan bahkan mock server secara otomatis. Berbeda dari dokumentasi API yang ditulis manual di wiki atau Google Doc terpisah (yang cepat basi begitu implementasi berubah), spesifikasi OpenAPI bisa **divalidasi otomatis** terhadap implementasi sungguhan — menangkap ketidaksesuaian antara janji kontrak dan kenyataan kode secara otomatis di CI, bukan ditemukan partner saat integrasi mereka tiba-tiba gagal.

## The Problem

Bayangkan sebuah API didokumentasikan di halaman wiki internal yang ditulis manual saat proyek dimulai, lalu tidak pernah benar-benar dijaga sinkron seiring endpoint berkembang. Beberapa bulan kemudian, sebuah field di response diganti namanya di kode, tapi wiki tidak pernah diperbarui. Partner instansi yang mengintegrasikan API ini berdasarkan dokumentasi wiki yang sudah basi tiba-tiba mendapati field yang mereka baca selalu kosong — tidak ada error yang jelas, hanya data yang salah secara diam-diam, karena field yang mereka cari di response memang sudah tidak ada lagi dengan nama itu.

Masalah ini murni soal **tidak ada mekanisme** yang memastikan dokumentasi tetap sinkron dengan implementasi sungguhan. Wiki yang ditulis manusia dan diperbarui manual selalu berisiko basi — tidak ada yang "memberi tahu" penulis wiki begitu kode berubah. Spesifikasi OpenAPI, kalau dijadikan bagian dari proses development (baik ditulis lebih dulu sebagai kontrak, atau di-generate dari kode, dan divalidasi di CI), memberi jaminan yang jauh lebih kuat: penyimpangan antara janji API dan kenyataan implementasi tertangkap otomatis, bukan ditemukan lewat keluhan partner.

## Intuition

Bayangkan spesifikasi OpenAPI seperti **cetak biru arsitektur terperinci yang dibagikan ke setiap kontraktor** (partner) yang mengerjakan sebuah gedung — semua pihak bekerja dari cetak biru yang sama persis, bukan dari deskripsi lisan yang berubah-ubah setiap kali diceritakan ulang. Perubahan pada cetak biru terlihat jelas dan tercatat, bukan sekadar "tanya arsitek apa yang berubah" secara informal.

Analogi "cetak biru kertas" ini bocor pada satu keunggulan penting: kekuatan sesungguhnya OpenAPI datang dari sifatnya yang bisa dibaca **mesin** dan divalidasi terus-menerus — spesifikasi ini bisa dipakai untuk otomatis menghasilkan kode client yang berfungsi, dan bahkan menguji secara otomatis apakah response server sungguhan benar-benar sesuai skema yang dijanjikan (contract testing). Cetak biru kertas tidak pernah punya kemampuan ini — tidak ada yang secara otomatis memverifikasi ulang gedung yang sudah jadi terhadap cetak birunya setiap hari.

## How It Works

Struktur inti dokumen OpenAPI: `paths` (setiap endpoint dan method HTTP-nya), `parameters` (path/query param), `requestBody` dan `responses` (masing-masing dengan skema JSON), `components/schemas` (definisi tipe yang dipakai ulang), dan `securitySchemes` (mekanisme autentikasi).

```yaml
paths:
  /dokumen/{id}:
    get:
      summary: Ambil detail dokumen
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: string
      responses:
        '200':
          description: Dokumen ditemukan
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Dokumen'
        '404':
          description: Dokumen tidak ditemukan
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
components:
  schemas:
    Dokumen:
      type: object
      properties:
        id:
          type: string
        status:
          type: string
```

Ekosistem tooling di sekitar spesifikasi ini luas: generator dokumentasi (Swagger UI, Redoc) merender spesifikasi jadi dokumentasi yang bisa dijelajahi manusia; generator client SDK menghasilkan kode client siap pakai di berbagai bahasa langsung dari spesifikasi; dan tool contract testing memverifikasi response server sungguhan benar-benar sesuai skema yang dijanjikan, menangkap penyimpangan otomatis di CI.

## In Go

Untuk codebase Go, ada dua arah yang umum dipakai: **spec-first** (menulis YAML dulu sebagai kontrak, lalu generate server stub dan tipe Go dari situ) atau **code-first** (menulis handler dan struct Go seperti biasa, lalu generate spesifikasi dari anotasi komentar di kode). Tool seperti `oapi-codegen` mendukung alur spec-first — menghasilkan interface Go dan tipe struct langsung dari YAML, memastikan kode Go dan spesifikasi tidak pernah berbeda karena keduanya berasal dari sumber yang sama:

```go
// Tipe ini di-generate otomatis dari components/schemas/Dokumen
// di file OpenAPI — bukan ditulis manual, mencegah drift antara
// spesifikasi dan struct Go yang sebenarnya dipakai kode.
type Dokumen struct {
    Id     string `json:"id"`
    Status string `json:"status"`
}
```

> [!question] Perlu diverifikasi
> Klaim: detail command dan konfigurasi spesifik `oapi-codegen` atau tool code-generation OpenAPI lain untuk Go.
> Kenapa ragu: tooling code-generation berkembang cepat dan opsi konfigurasinya bisa berubah antar versi.
> Cara verifikasi: baca dokumentasi resmi repository tool yang dipilih tim (misalnya `oapi-codegen` di GitHub) untuk versi yang sedang dipakai.

## In His Stack

Untuk pekerjaan koordinasi integrasi lintas 13+ aplikasi dan partner instansi pemerintah, memberi partner spesifikasi OpenAPI formal (bukan sekadar dokumen Google Docs informal) punya manfaat praktis besar: tim teknis partner — yang sering kali tidak seberpengalaman tim software murni — bisa langsung mengimpor file OpenAPI itu ke tool seperti Postman atau Insomnia dan langsung mencoba endpoint tanpa perlu menulis kode apa pun lebih dulu, jauh mengurangi email bolak-balik klarifikasi "field ini formatnya apa" yang biasa terjadi dengan dokumentasi informal.

## Trade-offs and When Not To Use It

Menjaga spesifikasi OpenAPI (baik ditulis manual spec-first atau di-generate dari kode) menambah overhead proses dibanding sekadar menulis endpoint tanpa dokumentasi formal — untuk API internal yang hanya dipakai satu tim yang sama dan berubah sangat cepat, overhead ini kadang terasa berlebihan. Tapi untuk API mana pun yang dikonsumsi partner eksternal — konteks kerja yang sesungguhnya di sini — biaya ini terbayar cepat lewat berkurangnya beban dukungan integrasi. Pendekatan spec-first juga membuka kemungkinan partner mulai membangun integrasi mereka lebih awal lewat mock server yang di-generate dari spesifikasi, **sebelum** backend sungguhan selesai dibangun — keuntungan nyata untuk timeline proyek yang melibatkan banyak pihak.

## Common Mistakes

> [!warning] Jebakan
> Mendokumentasikan API di dokumen terpisah (wiki, Google Doc) yang diperbarui manual, tanpa mekanisme apa pun yang menjamin sinkron dengan implementasi sungguhan — basi seiring waktu tanpa ada yang menyadarinya sampai partner mengeluh.

> [!warning] Jebakan
> Menulis spesifikasi OpenAPI sekali di awal proyek dan tidak pernah memperbaruinya lagi seiring API berkembang — tool ini tidak menjamin sinkronisasi otomatis kecuali memang dijadikan bagian proses (generate dari kode, atau divalidasi di CI).

> [!warning] Jebakan
> Memperlakukan spesifikasi OpenAPI murni sebagai output dokumentasi, tanpa memanfaatkan kemampuan validasi/contract testing-nya untuk menangkap penyimpangan response server sungguhan dari skema yang dijanjikan.

## Exercises

1. Apa perbedaan mendasar antara dokumentasi API manual (wiki) dan spesifikasi OpenAPI dari segi jaminan sinkronisasi dengan implementasi?
2. Apa perbedaan pendekatan spec-first dan code-first dalam menulis spesifikasi OpenAPI?
3. Sebutkan satu manfaat praktis memberi partner spesifikasi OpenAPI formal dibanding dokumentasi informal.
4. Desain terbuka: sebuah tim akan membangun API baru untuk diintegrasikan tiga instansi partner berbeda, dan pengembangan backend serta integrasi partner perlu berjalan paralel demi memenuhi tenggat waktu proyek. Rancang alur kerja yang memanfaatkan OpenAPI untuk memungkinkan ketiga partner mulai membangun integrasi mereka sebelum backend sungguhan selesai.

> [!success]- Kunci jawaban
> Terapkan pendekatan spec-first: tim menulis spesifikasi OpenAPI lengkap (path, skema request/response, format error) sebagai kontrak yang disepakati bersama ketiga partner di awal proyek, sebelum implementasi backend dimulai. Dari spesifikasi ini, generate mock server (banyak tool OpenAPI mendukung ini) yang mengembalikan response sesuai skema yang dijanjikan tanpa logika bisnis sungguhan di baliknya — ketiga partner bisa mulai membangun dan menguji integrasi mereka terhadap mock server ini secara paralel dengan tim backend yang sedang membangun implementasi sungguhan. Begitu backend asli selesai, jalankan contract test yang memverifikasi response backend sungguhan benar-benar sesuai spesifikasi yang sama — memastikan tidak ada penyimpangan antara apa yang dijanjikan ke partner di awal dan apa yang benar-benar dikirim backend sungguhan.

## Self-Check

- Apa perbedaan jaminan sinkronisasi antara dokumentasi wiki manual dan spesifikasi OpenAPI?
- Apa perbedaan pendekatan spec-first dan code-first?
- Sebutkan satu manfaat praktis OpenAPI untuk partner dengan tim teknis yang kurang berpengalaman.
- Bagaimana OpenAPI bisa dipakai untuk memungkinkan pengembangan backend dan integrasi partner berjalan paralel?

## Connected Notes

- [[REST Principles]] — prasyarat: OpenAPI mendokumentasikan API yang sudah didesain mengikuti prinsip REST.
- [[Consistent Error Responses]] — format error yang konsisten harus didokumentasikan formal di spesifikasi OpenAPI.
- [[Contract Negotiation and Versioning]] — OpenAPI sebagai alat konkret dalam negosiasi kontrak dengan partner.
- [[Sandbox Environments]] — mock server hasil generate dari OpenAPI adalah salah satu bentuk sandbox environment.
- [[../90 Architecture and Design/API Governance|API Governance]] — spesifikasi OpenAPI sebagai artefak konkret dalam tata kelola API lintas tim.

## Further Reading

- Spesifikasi resmi OpenAPI (spec.openapis.org) — sumber kebenaran definitif untuk struktur dan versi spesifikasi.

## Catatan Saya

*Tulis di sini apakah API di kerjaanmu sudah punya spesifikasi OpenAPI, dan seberapa sinkron ia dengan implementasi sungguhan saat ini.*
