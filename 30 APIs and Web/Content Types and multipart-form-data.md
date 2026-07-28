---
title: Content Types and multipart-form-data
type: concept
level: junior
domain: apis
status: unread
difficulty: 2
est_minutes: 14
prerequisites: ["[[../10 Foundations/HTTP 1.1 In Depth|HTTP 1.1 In Depth]]"]
next: ["[[Binary in JSON and the Base64 Tax]]", "[[Upload and Download Patterns]]"]
tags: [backend, apis]
created: 2026-07-26
---

## TL;DR

Header `Content-Type` memberi tahu server bagaimana mem-parsing body request. Tiga jenis yang paling relevan untuk kerja backend: `application/json` (seluruh body satu dokumen JSON), `application/x-www-form-urlencoded` (pasangan key-value teks sederhana), dan `multipart/form-data` (campuran field teks dan file biner, dipisahkan lewat penanda *boundary*). `multipart/form-data` adalah satu-satunya dari ketiganya yang bisa membawa data biner mentah **tanpa encoding tambahan apa pun** — inilah kenapa upload file selalu memakainya, bukan JSON.

## The Problem

Bayangkan seorang engineer mencoba mengirim file upload dengan menyisipkan byte biner mentah langsung ke dalam salah satu field string di payload JSON, berpikir "toh JSON juga cuma teks". Untuk file yang isinya kebetulan berupa karakter ASCII yang aman, ini terlihat bekerja di testing. Tapi begitu file sungguhan (misalnya PDF hasil scan yang isinya benar-benar biner, bukan teks) dikirim, payload JSON itu **rusak** — string JSON harus berupa teks Unicode yang valid dan di-escape dengan benar, sementara byte biner mentah bisa berisi urutan byte yang sama sekali bukan teks valid sama sekali.

Bug ini terlihat "acak" — bekerja untuk sebagian file, gagal untuk file lain — padahal sebenarnya sepenuhnya deterministik: tergantung apakah isi file itu kebetulan berupa urutan byte yang valid sebagai teks JSON atau tidak. Ini bukan bug yang bisa ditambal dengan escaping yang lebih hati-hati; ini pertanda format yang salah dipakai sejak awal untuk jenis data ini.

## Intuition

Bayangkan `multipart/form-data` seperti **satu kotak berisi beberapa paket terpisah, masing-masing dengan label dan pembatas jelas** — paket berisi teks dan paket berisi barang fisik (biner) bisa berdampingan dalam satu kotak yang sama tanpa saling campur, karena masing-masing dipisahkan pembatas fisik (boundary) dan diberi label sendiri (header `Content-Disposition` menyatakan field/nama file apa, `Content-Type` menyatakan jenis kontennya).

Analogi "pembatas fisik" ini bocor pada detail teknis penting: boundary di `multipart/form-data` bukan pembatas fisik sungguhan — ia hanyalah sebuah string yang **dipilih sedemikian rupa** (biasanya string acak yang cukup panjang) supaya **tidak pernah** kebetulan muncul di dalam isi file/data yang sedang dikirim. Kalau boundary yang dipilih terlalu pendek atau tidak cukup acak, ada kemungkinan (meski sangat kecil dalam praktik) ia bertabrakan dengan isi data sungguhan — inilah kenapa encoder multipart yang baik selalu memilih boundary yang cukup panjang dan acak, bukan string tetap yang sama setiap kali.

## How It Works

```
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxk

------WebKitFormBoundary7MA4YWxk
Content-Disposition: form-data; name="status"

diverifikasi
------WebKitFormBoundary7MA4YWxk
Content-Disposition: form-data; name="file"; filename="dokumen.pdf"
Content-Type: application/pdf

%PDF-1.4 ... (byte biner mentah PDF, TANPA encoding tambahan) ...
------WebKitFormBoundary7MA4YWxk--
```

Boundary yang dideklarasikan di header `Content-Type` dipakai untuk memisahkan setiap "bagian" (part) di body — masing-masing bagian punya header sendiri (`Content-Disposition` menyatakan nama field dan nama file kalau ini file, `Content-Type` menyatakan jenis konten bagian itu), diikuti isi mentahnya. Karena setiap bagian dipisahkan boundary, bukan digabung jadi satu string JSON, byte biner mentah bisa ditaruh apa adanya tanpa perlu di-escape atau di-encode ulang sama sekali.

## In Go

Sisi server, mem-parsing multipart form upload:

```go
func uploadHandler(w http.ResponseWriter, r *http.Request) {
    const maxMemory = 10 << 20 // 10 MB disimpan di memori, sisanya ke disk sementara
    if err := r.ParseMultipartForm(maxMemory); err != nil {
        http.Error(w, "gagal mem-parsing form", http.StatusBadRequest)
        return
    }

    status := r.FormValue("status") // field teks biasa

    file, header, err := r.FormFile("file") // bagian file
    if err != nil {
        http.Error(w, "file tidak ditemukan", http.StatusBadRequest)
        return
    }
    defer file.Close()

    fmt.Printf("status=%s, nama file=%s, ukuran=%d byte\n", status, header.Filename, header.Size)
    // ... simpan file ...
}
```

Sisi klien, membangun request multipart untuk mengirim file ke partner:

```go
func kirimDokumen(ctx context.Context, url, status string, isiFile io.Reader, namaFile string) error {
    var buf bytes.Buffer
    writer := multipart.NewWriter(&buf)

    writer.WriteField("status", status)

    part, err := writer.CreateFormFile("file", namaFile)
    if err != nil {
        return fmt.Errorf("buat bagian file: %w", err)
    }
    if _, err := io.Copy(part, isiFile); err != nil {
        return fmt.Errorf("salin isi file: %w", err)
    }
    writer.Close() // WAJIB — menuliskan boundary penutup

    req, err := http.NewRequestWithContext(ctx, http.MethodPost, url, &buf)
    if err != nil {
        return fmt.Errorf("buat request: %w", err)
    }
    req.Header.Set("Content-Type", writer.FormDataContentType()) // boundary di-generate otomatis

    resp, err := http.DefaultClient.Do(req)
    if err != nil {
        return fmt.Errorf("kirim dokumen: %w", err)
    }
    defer resp.Body.Close()
    return nil
}
```

`writer.FormDataContentType()` secara otomatis menghasilkan boundary yang cukup acak — kamu tidak perlu (dan sebaiknya tidak) menuliskan boundary secara manual.

## In His Stack

**Yii2** dengan `UploadedFile::getInstance()` dan superglobal `$_FILES` PHP sebenarnya sudah bergantung sepenuhnya pada `multipart/form-data` di level bawahnya — mekanisme ini sudah ditangani otomatis oleh PHP runtime, sehingga banyak developer Yii2 memakainya bertahun-tahun tanpa pernah benar-benar melihat struktur boundary mentah di baliknya. Memahami mekanisme ini tetap penting saat membangun service Go yang perlu berkomunikasi (menerima atau mengirim) dengan sistem berbasis multipart, termasuk saat men-debug masalah upload yang gagal karena `Content-Type` yang tidak sesuai antara pengirim dan penerima.

## Trade-offs and When Not To Use It

`multipart/form-data` adalah mekanisme yang benar untuk upload file bercampur dengan field teks, tanpa biaya encoding tambahan apa pun untuk data biner. Untuk payload yang sepenuhnya berupa data terstruktur tanpa file (kebanyakan API CRUD biasa), JSON tetap lebih sederhana untuk dibangun dan diparsing. Memaksakan JSON untuk kasus yang melibatkan file memaksa workaround seperti base64 encoding (dibahas penuh di [[Binary in JSON and the Base64 Tax]]) yang punya biaya nyata dari segi ukuran payload dan performa.

## Common Mistakes

> [!warning] Jebakan
> Mencoba menyisipkan byte biner mentah langsung ke dalam field string JSON, merusak payload untuk file yang isinya benar-benar biner (bukan sekadar teks yang kebetulan aman).

> [!warning] Jebakan
> Tidak mengatur `Content-Type` yang sesuai (termasuk boundary yang benar) saat mengirim atau menerima request multipart, menyebabkan server gagal mem-parsing body meski datanya sebenarnya terstruktur dengan benar.

> [!warning] Jebakan
> Memanggil `ParseMultipartForm` tanpa mempertimbangkan batas ukuran (`maxMemory`) yang wajar, membuka celah upload yang sangat besar menghabiskan memori server — dibahas lebih dalam di [[Request Size Limits Along The Path]].

## Exercises

1. Kenapa data biner mentah tidak bisa disisipkan langsung ke dalam field string JSON?
2. Apa fungsi boundary di `multipart/form-data`, dan kenapa ia harus dipilih cukup acak?
3. Apa perbedaan `application/x-www-form-urlencoded` dan `multipart/form-data`, dan kapan masing-masing tepat dipakai?
4. Desain terbuka: sebuah partner instansi mengirim permintaan upload dokumen ke API-mu, tapi mereka bersikeras hanya bisa mengirim lewat payload JSON murni (sistem mereka tidak mendukung multipart) karena keterbatasan tooling internal mereka. Rancang solusi yang tetap memungkinkan integrasi ini bekerja, dan jelaskan trade-off apa yang harus disepakati bersama partner.

> [!success]- Kunci jawaban
> Kalau partner benar-benar tidak bisa mengirim `multipart/form-data`, satu-satunya opsi realistis adalah menerima file dalam bentuk base64-encoded string di dalam field JSON — solusi yang bekerja, tapi dengan biaya nyata yang harus disepakati eksplisit bersama partner: ukuran payload membengkak signifikan dibanding ukuran file asli (dibahas detail di [[Binary in JSON and the Base64 Tax]]), dan perlu dipastikan ada batas ukuran file yang jelas di kedua sisi supaya payload JSON yang membengkak ini tidak melampaui batas ukuran request yang wajar. Alternatif lain yang lebih baik kalau memungkinkan: minta partner mengunggah file lebih dulu ke [[Pre-signed URLs|pre-signed URL]] object storage, lalu hanya mengirim referensi/URL file itu lewat JSON — memisahkan transfer file besar dari payload JSON metadata sepenuhnya.

## Self-Check

- Kenapa data biner tidak bisa aman disisipkan langsung ke field string JSON?
- Apa fungsi boundary di `multipart/form-data`?
- Kapan `multipart/form-data` lebih tepat dipakai dibanding JSON?
- Apa risiko memanggil `ParseMultipartForm` tanpa batas ukuran yang wajar?

## Connected Notes

- [[../10 Foundations/HTTP 1.1 In Depth|HTTP 1.1 In Depth]] — prasyarat: header dan body HTTP yang menjadi dasar teknis Content-Type.
- [[Binary in JSON and the Base64 Tax]] — kelanjutan langsung: biaya nyata memaksakan data biner ke dalam JSON saat multipart tidak bisa dipakai.
- [[Upload and Download Patterns]] — pola production lengkap untuk menangani upload file besar.
- [[Pre-signed URLs]] — alternatif yang memisahkan transfer file dari payload API sepenuhnya.
- [[Request Size Limits Along The Path]] — batas ukuran yang wajib dipertimbangkan saat menangani multipart upload.

## Further Reading

- RFC 7578 (*Returning Values from Forms: multipart/form-data*) — spesifikasi resmi format ini.

## Catatan Saya

*Tulis di sini pengalaman menangani upload file di kerjaanmu — apakah lewat multipart, base64 di JSON, atau pre-signed URL, dan alasan pilihannya.*
