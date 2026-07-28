---
title: Error Wrapping
type: concept
level: junior
domain: go
status: unread
difficulty: 2
est_minutes: 13
prerequisites: ["[[Errors as Values]]"]
next: ["[[Sentinel Errors vs Error Types]]", "[[../30 APIs and Web/Consistent Error Responses|Consistent Error Responses]]"]
tags: [backend, go]
created: 2026-07-26
---

## TL;DR

`fmt.Errorf("...: %w", err)` membungkus sebuah error dengan konteks tambahan **sambil tetap menyimpan error aslinya** dalam sebuah rantai yang bisa diperiksa lagi nanti lewat `errors.Is` (apakah rantai ini mengandung error tertentu) dan `errors.As` (ambil error bertipe tertentu dari rantai itu). Memakai `%v` alih-alih `%w` terlihat hampir identik di output teks, tapi **memutus rantai itu sepenuhnya** — error aslinya berubah jadi sekadar teks, dan `errors.Is`/`errors.As` tidak akan pernah bisa menemukannya lagi. Perbedaan satu karakter (`%w` vs `%v`) ini adalah salah satu bug paling halus dan paling sering lolos code review di Go.

## The Problem

Bayangkan sebuah repository yang memanggil database dan mengembalikan `sql.ErrNoRows` saat dokumen tidak ditemukan. Di layer service, error ini dibungkus dengan konteks tambahan — tapi ditulis dengan `fmt.Errorf("ambil dokumen %s: %v", id, err)`, memakai `%v` bukan `%w`. Di layer handler HTTP, kode memeriksa `errors.Is(err, sql.ErrNoRows)` untuk memutuskan apakah harus mengembalikan `404 Not Found` — pemeriksaan ini **selalu gagal**, meski error aslinya memang `sql.ErrNoRows`, karena `%v` sudah mengubah error itu jadi string biasa di layer service, memutus rantai yang dibutuhkan `errors.Is` untuk menelusurinya.

Akibatnya, setiap kasus "dokumen tidak ditemukan" — yang seharusnya jadi `404` yang jelas dan wajar — malah selalu jatuh ke jalur `500 Internal Server Error` generik. Bug ini nyaris tidak terlihat di code review karena `%v` dan `%w` menghasilkan **teks output yang identik** — perbedaannya hanya terlihat saat kode lain mencoba memeriksa rantai error itu secara programatik, jauh dari baris tempat bug sebenarnya ditulis.

## Intuition

Bayangkan `%w` seperti **memasukkan surat asli ke dalam amplop baru**, menambahkan catatan di luar amplop tapi tetap menyimpan suratnya utuh di dalam — siapa pun di kemudian hari bisa membuka amplop itu dan membaca surat aslinya. `%v` lebih seperti **menyalin ulang isi surat itu jadi kalimat baru**, lalu membuang surat aslinya — hasilnya terlihat memberi informasi yang sama, tapi surat asli yang mungkin dibutuhkan untuk verifikasi (misalnya tanda tangan resminya) sudah tidak ada lagi.

Analogi ini bocor pada soal mekanismenya: rantai error di Go bukan pembungkusan fisik literal, melainkan protokol yang didefinisikan lewat method `Unwrap() error` — sebuah error yang dibuat dengan `%w` secara otomatis mengimplementasikan method ini, dan `errors.Is`/`errors.As` menelusuri rantai `Unwrap()` ini selapis demi selapis sampai menemukan yang dicari atau rantainya habis.

## How It Works

```mermaid
flowchart LR
    subgraph WrapBenar["fmt.Errorf(\"...: %w\", err)"]
        E1["error baru"] -->|"Unwrap()"| E2["error asli tetap ada"]
    end
    subgraph WrapSalah["fmt.Errorf(\"...: %v\", err)"]
        E3["error baru\n(hanya string)"] -.->|"tidak ada Unwrap()"| X["rantai terputus"]
    end
```

`errors.Is(err, target)` menelusuri rantai `Unwrap()` dari `err`, membandingkan setiap lapisan dengan `target`, mengembalikan `true` kalau ditemukan kecocokan di mana pun di rantai itu. `errors.As(err, &target)` melakukan hal serupa tapi mencari lapisan yang **bertipe** tertentu (bukan sama persis dengan satu instance), lalu meng-assign-nya ke `target`.

## In Go

```go
var ErrDokumenTidakDitemukan = errors.New("dokumen tidak ditemukan")

func ambilDariDatabase(ctx context.Context, id string) (*Dokumen, error) {
    row := db.QueryRowContext(ctx, "SELECT ... WHERE id = ?", id)
    var doc Dokumen
    if err := row.Scan(&doc.ID, &doc.Status); err != nil {
        if errors.Is(err, sql.ErrNoRows) {
            // %w membungkus ErrDokumenTidakDitemukan, rantai tetap utuh.
            return nil, fmt.Errorf("ambil dokumen %s: %w", id, ErrDokumenTidakDitemukan)
        }
        return nil, fmt.Errorf("query dokumen %s: %w", id, err)
    }
    return &doc, nil
}

func handleAmbilDokumen(w http.ResponseWriter, r *http.Request) {
    id := r.PathValue("id")
    doc, err := ambilDariDatabase(r.Context(), id)
    if err != nil {
        // Karena repository memakai %w, errors.Is berhasil menelusuri
        // rantai ini sampai menemukan ErrDokumenTidakDitemukan,
        // MESKI pesan errornya sudah dibungkus beberapa lapis konteks.
        if errors.Is(err, ErrDokumenTidakDitemukan) {
            http.Error(w, "dokumen tidak ditemukan", http.StatusNotFound)
            return
        }
        log.Printf("ambil dokumen %s: %v", id, err) // log detail internal
        http.Error(w, "kesalahan internal", http.StatusInternalServerError)
        return
    }
    json.NewEncoder(w).Encode(doc)
}
```

Kalau baris di repository ditulis `fmt.Errorf("ambil dokumen %s: %v", id, ErrDokumenTidakDitemukan)` (memakai `%v`), `errors.Is(err, ErrDokumenTidakDitemukan)` di handler akan **selalu** mengembalikan `false` — persis bug di "The Problem", meski pesan log yang dicetak terlihat sama persis.

## In His Stack

**PHP** punya mekanisme *exception chaining* yang secara konsep mirip — constructor `Exception` menerima parameter `$previous` untuk menyimpan exception asli saat sebuah exception baru dilempar sebagai gantinya. Bedanya, di PHP mekanisme ini eksplisit lewat parameter constructor yang jelas terlihat; di Go, perbedaannya bergantung pada **satu karakter verb** (`%w` vs `%v`) di dalam string format — jauh lebih mudah salah ketik tanpa disadari, dan lolos code review karena outputnya terlihat identik.

## Trade-offs and When Not To Use It

Membungkus error di setiap layer memberi jejak konteks yang berguna untuk debugging (`"proses verifikasi dokumen A-001: ambil dokumen A-001: dokumen tidak ditemukan"`), tapi membungkus **berlebihan** di setiap lapisan dengan teks yang nyaris identik menghasilkan pesan error yang panjang dan berulang tanpa menambah informasi baru. Aturan praktis: bungkus dengan `%w` di titik-titik yang benar-benar menambah konteks bermakna (nama resource, operasi yang sedang dilakukan), bukan di **setiap** pemanggilan function tanpa pertimbangan. Perlu diingat juga: pesan error yang dibungkus lengkap (termasuk detail teknis seperti query SQL) sebaiknya dicatat di log internal, **bukan** dikirim mentah-mentah ke response API — lihat [[../30 APIs and Web/Consistent Error Responses|Consistent Error Responses]] soal memisahkan detail internal dari pesan yang aman ditunjukkan ke klien.

## Common Mistakes

> [!warning] Jebakan
> Memakai `%v` alih-alih `%w` saat membungkus error yang nantinya perlu diperiksa lagi lewat `errors.Is`/`errors.As`. Outputnya terlihat identik di log, tapi rantai errornya putus sepenuhnya — bug ini sangat mudah lolos code review.

> [!warning] Jebakan
> Membungkus error di setiap lapisan pemanggilan dengan teks yang nyaris sama, menghasilkan pesan error yang panjang dan berulang (`"gagal proses: gagal ambil: gagal query: ..."`) tanpa menambah informasi yang benar-benar baru di setiap lapisannya.

> [!warning] Jebakan
> Mengirim pesan error yang sudah dibungkus lengkap (berisi detail teknis internal seperti query SQL atau path file) langsung ke response API tanpa disaring. Ini bisa membocorkan informasi internal yang seharusnya tidak pernah terlihat klien.

## Exercises

1. Apa perbedaan teknis antara `fmt.Errorf("...: %w", err)` dan `fmt.Errorf("...: %v", err)`?
2. Kenapa `errors.Is` bisa gagal menemukan error target meski error itu "ada" di suatu tempat dalam rantai pemanggilan?
3. Kapan `errors.Is` lebih tepat dipakai dibanding `errors.As`, dan sebaliknya?
4. Desain terbuka: sebuah tim menemukan bahwa endpoint mereka selalu mengembalikan `500` untuk kasus yang seharusnya `404`, meski kode terlihat sudah memakai `errors.Is` dengan benar di handler. Rancang langkah investigasi sistematis untuk menemukan lapisan mana yang memutus rantai error, dan usulkan aturan tim (bisa lewat code review checklist atau linter) untuk mencegah bug ini terulang di masa depan.

> [!success]- Kunci jawaban
> Investigasi: telusuri setiap lapisan pemanggilan dari handler sampai ke titik error pertama kali dibuat, periksa satu per satu apakah setiap `fmt.Errorf` yang membungkus error di sepanjang jalan memakai `%w` atau `%v` — cukup satu lapisan yang salah memakai `%v` untuk memutus seluruh rantai di titik itu. Cara cepat memverifikasi: tulis unit test yang secara eksplisit memanggil `errors.Is` pada error yang dikembalikan dari titik masuk paling luar (misalnya langsung dari function repository/service, tanpa perlu HTTP), memverifikasi rantai tetap utuh sebelum menyentuh handler. Untuk mencegah ini terulang, tambahkan linter (`go vet` dengan check errors, atau linter komunitas seperti `wrapcheck`) di CI yang mendeteksi pemakaian `%v` pada value bertipe `error`, dan jadikan aturan eksplisit di code review checklist: "setiap error yang mungkin diperiksa lagi lewat errors.Is/As di lapisan atas WAJIB dibungkus dengan %w, tidak %v".

## Self-Check

- Apa yang membedakan `%w` dari `%v` saat membungkus error dengan `fmt.Errorf`?
- Kenapa perbedaan `%w` vs `%v` sulit terlihat lewat code review biasa?
- Apa fungsi `errors.Is` dan `errors.As` masing-masing?
- Kenapa membungkus error berlebihan di setiap lapisan bisa jadi masalah tersendiri?

## Connected Notes

- [[Errors as Values]] — prasyarat: filosofi error sebagai nilai biasa yang mendasari kenapa pembungkusan eksplisit ini diperlukan.
- [[Sentinel Errors vs Error Types]] — dua gaya mendefinisikan error yang bisa diperiksa lewat `errors.Is`/`errors.As`, dibahas penuh di note itu.
- [[../30 APIs and Web/Consistent Error Responses|Consistent Error Responses]] — batas antara detail error internal (aman dibungkus lengkap di log) dan pesan yang aman dikirim ke klien API.
- [[../90 Architecture and Design/Handler-Service-Repository Layering|Handler-Service-Repository Layering]] — layering yang menjadi tempat rantai error ini biasanya melintas, dari repository sampai handler.

## Further Reading

- Artikel resmi *"Working with Errors in Go 1.13"* di blog resmi Go (go.dev/blog) — pengantar resmi `%w`, `errors.Is`, dan `errors.As`.

## Catatan Saya

*Tulis di sini kalau kamu pernah menemukan bug `errors.Is` yang gagal karena salah pakai `%v`, dan bagaimana akhirnya ditemukan.*
