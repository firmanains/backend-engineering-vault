---
title: Stdlib Testing vs Testify
type: concept
level: junior
domain: go
status: unread
difficulty: 2
est_minutes: 13
prerequisites: ["[[Table-Driven Tests]]"]
next: ["[[Mocking Through Interfaces]]", "[[Meaningful Test Coverage]]"]
tags: [backend, go]
created: 2026-07-26
---

## TL;DR

Package `testing` bawaan Go hanya menyediakan primitif minimal — `t.Error`/`t.Errorf` (catat kegagalan, lanjutkan test), `t.Fatal`/`t.Fatalf` (catat kegagalan, **hentikan** test itu juga) — memaksa kamu menulis sendiri logic perbandingan. Library `testify` (paket `assert` dan `require`) menambah helper assertion yang lebih ringkas dan pesan kegagalan yang lebih terbaca. Perbedaan paling penting yang sering diabaikan: `assert.X` **melanjutkan** eksekusi test setelah gagal (seperti `t.Error`), sementara `require.X` **menghentikan** test seketika (seperti `t.Fatal`) — memilih yang salah di antara keduanya bisa membuat test lanjut berjalan setelah sebuah pemeriksaan penting gagal, lalu panic membingungkan di baris berikutnya karena melanjutkan dengan data yang seharusnya sudah dianggap tidak valid.

## The Problem

Bayangkan sebuah test memanggil function yang mengembalikan `(*Dokumen, error)`, memeriksa error-nya dengan `assert.NoError(t, err)`, lalu langsung mengakses field dari hasilnya (`doc.Status`) di baris berikutnya. Kalau ternyata `err` memang tidak nil (function-nya gagal), `assert.NoError` mencatat kegagalan itu **tapi test tetap lanjut berjalan** — baris berikutnya yang mengakses `doc.Status` akan panic karena `doc` bernilai `nil`, menghasilkan output test yang membingungkan: dua kegagalan tercatat sekaligus (assertion gagal, lalu panic), padahal akar masalahnya cuma satu.

Kalau baris itu ditulis `require.NoError(t, err)` alih-alih `assert.NoError`, test akan **berhenti seketika** begitu error terdeteksi tidak nil — tidak pernah sampai ke baris yang mengakses `doc.Status`, dan output kegagalan test jadi jelas menunjuk akar masalah yang sebenarnya, bukan efek berantai yang membingungkan.

## Intuition

Bayangkan `testing` bawaan seperti **perkakas ukur manual** — kamu menulis sendiri setiap langkah perbandingan dan keputusan lanjut/berhenti. `testify` seperti **alat ukur dengan pembacaan otomatis** yang langsung menunjukkan selisih yang diharapkan vs yang didapat, lebih nyaman dibaca.

Analogi "upgrade alat" ini bocor pada satu hal: menambah `testify` bukan sekadar upgrade tanpa konsekuensi seperti mengganti perkakas fisik dengan versi lebih canggih — ia menambah **dependency eksternal** ke codebase-mu, dan menambah dua konsep (`assert` vs `require`) yang harus dipahami benar oleh semua orang di tim, dengan konsekuensi nyata (lanjut vs berhenti) kalau salah pilih. ini adalah keputusan desain dengan trade-off sungguhan, bukan sekadar "lebih modern jadi lebih baik".

## How It Works

```go
// testing bawaan: manual, verbose, tapi zero dependency.
func TestAmbilDokumenStdlib(t *testing.T) {
    doc, err := AmbilDokumen("A-001")
    if err != nil {
        t.Fatalf("unexpected error: %v", err) // t.Fatal = berhenti seketika
    }
    if doc.Status != "draft" {
        t.Errorf("Status = %q, want %q", doc.Status, "draft") // t.Error = lanjut
    }
}

// testify: lebih ringkas, tapi butuh disiplin memilih assert vs require.
func TestAmbilDokumenTestify(t *testing.T) {
    doc, err := AmbilDokumen("A-001")
    require.NoError(t, err) // WAJIB require di sini — kalau err != nil,
                             // baris berikutnya (doc.Status) akan panic
                             // kalau test tetap lanjut jalan.
    assert.Equal(t, "draft", doc.Status) // assert cukup di sini — kegagalan
                                          // ini tidak menyebabkan efek berantai.
}
```

Aturan praktis: pakai `require` untuk pemeriksaan yang, kalau gagal, membuat baris-baris berikutnya di test itu tidak valid untuk dilanjutkan (biasanya pemeriksaan `err == nil` sebelum mengakses hasil). Pakai `assert` untuk pemeriksaan independen yang aman dilanjutkan meski gagal (misalnya memeriksa beberapa field berbeda dari satu struct yang sama, di mana kamu ingin tahu **semua** field yang salah sekaligus, bukan berhenti di field pertama).

## In His Stack

**PHPUnit** menyatukan seluruh assertion (`assertEquals`, `assertTrue`, dst.) dalam satu model: setiap assertion yang gagal melempar exception yang menghentikan method test itu seketika — tidak ada pembedaan "lanjut vs berhenti" seperti `assert` vs `require` di testify. Perbedaan filosofi Go ini (dua paket terpisah dengan semantik berbeda) memberi fleksibilitas lebih (bisa memilih mengumpulkan banyak kegagalan sekaligus dalam satu test), tapi menuntut pemahaman eksplisit dari setiap anggota tim tentang kapan memakai yang mana.

## Trade-offs and When Not To Use It

`testify` mengurangi boilerplate dan memberi pesan kegagalan yang lebih terbaca (menampilkan diff yang jelas antara expected dan actual), tapi menambah dependency eksternal dan kompleksitas konseptual (`assert` vs `require`). Untuk package library yang akan diimpor banyak proyek lain, sebagian penulis Go lebih memilih tetap memakai `testing` bawaan bahkan untuk test (menghindari dependency eksternal apa pun, bahkan yang hanya dipakai saat testing) — pertimbangan ini lebih relevan untuk package yang benar-benar dipakai luas sebagai dependency pihak lain, bukan untuk kode aplikasi internal seperti kebanyakan service backend. Untuk kode aplikasi (bukan library publik), memakai `testify` demi keterbacaan adalah pilihan yang wajar — yang penting konsisten dan dipahami seluruh tim.

## Common Mistakes

> [!warning] Jebakan
> Memakai `assert.NoError` di tempat yang seharusnya `require.NoError` — sebelum kode yang akan panic kalau hasilnya memang nil/tidak valid. Test lanjut berjalan setelah kegagalan tercatat, lalu panic di baris berikutnya, menghasilkan output yang membingungkan alih-alih menunjuk akar masalah dengan jelas.

> [!warning] Jebakan
> Mengandalkan `reflect.DeepEqual` (atau fungsi pembanding serupa di testify) untuk semua jenis perbandingan tanpa menyadari perilakunya bisa mengejutkan untuk tipe tertentu — misalnya `time.Time` yang punya monotonic clock reading tersembunyi bisa dianggap "tidak sama" meski secara logis merepresentasikan waktu yang identik.

> [!warning] Jebakan
> Menambahkan `testify` (atau dependency test lain) ke sebuah package library kecil yang dimaksudkan dipakai luas sebagai dependency proyek lain, tanpa mempertimbangkan bahwa ini menambah beban dependency bagi semua yang mengimpor package itu, meski hanya dipakai saat testing.

## Exercises

1. Apa perbedaan perilaku `assert.X` dan `require.X` di testify?
2. Kenapa memakai `assert.NoError` sebelum kode yang mengakses hasil (yang bisa nil) berisiko menghasilkan test yang membingungkan?
3. Sebutkan satu alasan valid untuk tetap memakai `testing` bawaan tanpa testify.
4. Desain terbuka: sebuah tim baru mengadopsi testify di seluruh test suite mereka, tapi beberapa developer memakai `assert` di tempat yang seharusnya `require` (dan sebaliknya), menghasilkan beberapa test yang panic membingungkan atau berhenti terlalu dini tanpa mengumpulkan semua informasi kegagalan yang berguna. Rancang panduan singkat (aturan praktis) untuk tim ini memutuskan kapan memakai `assert` dan kapan `require`, dan pertimbangkan apakah ada cara menegakkan aturan ini otomatis lewat tooling.

> [!success]- Kunci jawaban
> Panduan praktis: pakai `require` untuk **setiap** pemeriksaan yang hasilnya dipakai langsung di baris-baris berikutnya dalam test yang sama (terutama pemeriksaan `err == nil` sebelum dereference hasil, atau pemeriksaan panjang slice sebelum mengakses index tertentu) — kalau pemeriksaan itu gagal, tidak ada gunanya melanjutkan test dengan data yang sudah terbukti tidak valid. Pakai `assert` untuk pemeriksaan independen di akhir test yang tidak saling bergantung (misalnya memeriksa beberapa field berbeda dari satu hasil yang sama, di mana melihat semua field yang salah sekaligus lebih berguna daripada berhenti di field pertama). Untuk penegakan otomatis, beberapa linter komunitas (seperti bagian dari `golangci-lint`) punya check khusus yang mendeteksi pola `assert.NoError` yang diikuti akses langsung ke hasil yang berpotensi nil — tambahkan sebagai bagian dari [[The Go Toolchain]] CI gate tim.

## Self-Check

- Apa perbedaan efek `assert.X` dan `require.X` saat sebuah pemeriksaan gagal?
- Kapan sebaiknya memakai `require` alih-alih `assert`?
- Sebutkan satu alasan valid tetap memakai `testing` bawaan tanpa testify.
- Apa risiko mengandalkan `reflect.DeepEqual` untuk semua jenis perbandingan?

## Connected Notes

- [[Table-Driven Tests]] — prasyarat: pola tabel yang sering dikombinasikan dengan assertion, baik stdlib maupun testify.
- [[The Go Toolchain]] — `go test` sebagai perintah yang menjalankan kedua gaya testing ini.
- [[Mocking Through Interfaces]] — testing dengan mock sering memakai assertion library yang sama untuk memverifikasi interaksi mock.
- [[Meaningful Test Coverage]] — assertion yang tepat (memakai `require` di tempat yang benar) adalah bagian dari apa yang membuat coverage benar-benar bermakna, bukan sekadar tinggi angkanya.

## Further Reading

- Dokumentasi resmi package `testing` (pkg.go.dev/testing) dan dokumentasi `testify` (pkg.go.dev/github.com/stretchr/testify) untuk referensi lengkap API masing-masing.

## Catatan Saya

*Tulis di sini konvensi testing yang dipakai di tempat kerjamu — testify, stdlib polos, atau campuran — dan alasan di baliknya kalau kamu tahu.*
