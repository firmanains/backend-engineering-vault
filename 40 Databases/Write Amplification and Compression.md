---
title: Write Amplification and Compression
type: concept
level: intermediate
domain: databases
status: unread
difficulty: 3
est_minutes: 14
prerequisites: ["[[LSM-Trees vs B-Trees]]"]
next: ["[[Beyond Relational - Document, Key-Value, Wide-Column, Graph, and Time-Series Stores]]"]
tags: [backend, databases, performance]
created: 2026-07-29
---

## TL;DR

[[LSM-Trees vs B-Trees]] menyinggung bahwa keduanya membayar "pajak" tulis tambahan lewat mekanisme berbeda — note ini memberi nama dan angka pada pajak itu: **write amplification**, rasio antara berapa banyak data yang benar-benar ditulis ke disk dibanding berapa banyak data yang secara logis diminta aplikasi untuk ditulis. Menulis satu baris kecil bisa, di baliknya, memicu penulisan ulang halaman berukuran jauh lebih besar (B-Tree) atau memicu compaction yang menulis ulang data yang sama berkali-kali seiring waktu (LSM-Tree) — angka amplifikasi 10x atau lebih bukan hal aneh. **Compression** adalah salah satu alat mengurangi dampak fisiknya (data yang ditulis ulang lebih kecil), tapi menambah beban CPU sebagai gantinya — trade-off yang sekali lagi tidak gratis.

## The Problem

Sebuah tim mengukur bahwa aplikasinya menulis sekitar 50 MB data logis ke database per jam, tapi monitoring disk menunjukkan volume tulis fisik ke SSD jauh lebih besar — mendekati 500 MB per jam. Selisih 10x ini bukan bug atau kebocoran — ini write amplification yang normal terjadi di balik layar, dan penting dipahami khususnya untuk sistem yang berjalan di infrastruktur dengan biaya I/O yang dihitung eksplisit (cloud storage dengan billing per operasi I/O) atau di storage dengan keterbatasan fisik (SSD yang punya batas siklus tulis/erase sebelum mulai rusak).

Tim ini awalnya mengira menambah SSD berkapasitas besar akan menyelesaikan masalah pertumbuhan data — tapi tanpa memahami write amplification, mereka tidak menyadari bahwa **umur pakai** SSD (dibatasi jumlah siklus tulis, bukan hanya kapasitas) terkikis 10x lebih cepat dari yang mereka kira berdasarkan volume data logis aplikasi, sebuah biaya operasional tersembunyi yang baru terlihat saat SSD mulai menunjukkan tanda keausan jauh lebih cepat dari perkiraan garansi vendor.

## Intuition

Bayangkan write amplification seperti **merevisi satu kalimat di tengah dokumen cetak yang sudah dijilid** — mengubah satu kalimat kecil di halaman 50 dari dokumen setebal 200 halaman, dalam praktiknya berarti mencetak ulang **seluruh** halaman itu (kadang beberapa halaman sekaligus kalau perubahan panjangnya menggeser teks), bukan hanya mengganti kalimat itu sendiri. Rasio "berapa banyak kertas dicetak ulang" dibanding "berapa banyak teks sebenarnya berubah" adalah write amplification versi kertas — perubahan sekecil apa pun tetap memicu pencetakan ulang dalam unit halaman penuh, karena itulah unit terkecil yang bisa ditulis ulang di sistem percetakan itu.

Analogi ini bocor pada satu hal: kertas yang dicetak ulang tidak "aus" secara fisik dari proses percetakan itu sendiri (kertas baru dipakai setiap kali). SSD justru **fisiknya sendiri** yang terkikis setiap kali ditulis — setiap sel flash memory punya jumlah siklus tulis/hapus terbatas sebelum mulai gagal menyimpan data dengan andal, sehingga write amplification bukan sekadar soal "kerja lebih banyak" tapi juga "mendekati akhir umur hardware lebih cepat".

## How It Works

**Write amplification di B-Tree** terjadi lewat beberapa jalur: setiap perubahan pada satu baris memicu penulisan ulang **seluruh halaman disk** (biasanya 4KB-16KB) yang menampung baris itu, meski perubahannya hanya beberapa byte — dan kalau perubahan itu memicu node split (dibahas di [[B+Tree Structure]]), lebih dari satu halaman ditulis ulang sekaligus. Write-ahead log yang mencatat setiap perubahan sebelum diterapkan ke data utama (untuk durability dan crash recovery) juga menggandakan penulisan fisik untuk setiap perubahan logis.

**Write amplification di LSM-Tree** terjadi lewat compaction: data yang sama, secara logis ditulis **satu kali** oleh aplikasi, bisa ditulis ulang secara fisik **berkali-kali** seiring proses compaction menggabungkan SSTable kecil jadi SSTable lebih besar berulang kali sepanjang siklus hidup data itu di sistem — semakin banyak level compaction yang dilalui sebuah data sebelum akhirnya dihapus/digantikan, semakin tinggi write amplification totalnya.

```mermaid
flowchart LR
    A["1 baris logis diubah\n(beberapa byte)"] --> B["B-Tree: tulis ulang\n1 halaman penuh (4-16KB)\n+ WAL"]
    A --> C["LSM-Tree: tulis ke memtable\n+ WAL, lalu ditulis ulang\nsetiap kali compaction\nmenyentuh data ini"]
```

Diagram ini menunjukkan bahwa **tidak ada struktur yang bebas dari write amplification** — keduanya membayar pajak ini, hanya lewat mekanisme dan pola yang berbeda. Mengukur write amplification aktual (rasio bytes ditulis ke disk dibanding bytes yang diminta aplikasi, biasanya tersedia lewat metrik storage engine atau OS-level disk I/O monitoring) adalah langkah pertama memahami apakah sistem tertentu punya amplifikasi yang wajar atau tidak wajar tinggi untuk beban kerjanya.

## Under The Hood

**Compression** mengurangi write amplification secara tidak langsung — dengan mengompresi data sebelum ditulis ke disk, jumlah **byte fisik** yang ditulis untuk representasi logis yang sama menjadi lebih kecil, mengurangi dampak fisik dari amplifikasi yang tetap terjadi secara rasio. Trade-off-nya eksplisit: kompresi butuh siklus CPU untuk mengompresi saat menulis dan mendekompresi saat membaca — untuk beban kerja yang sudah CPU-bound, menambah kompresi bisa memindahkan bottleneck dari I/O ke CPU, bukan menghilangkan biaya sama sekali, hanya memindahkannya.

Algoritma kompresi yang dipakai database punya trade-off berbeda antara **rasio kompresi** dan **kecepatan**: algoritma seperti Snappy atau LZ4 mengutamakan kecepatan (kompresi/dekompresi sangat cepat) dengan rasio kompresi sedang; algoritma seperti Zstandard (zstd) atau gzip di level kompresi tinggi mengutamakan rasio kompresi lebih baik dengan kecepatan lebih rendah. Pilihan ini bukan "yang terbaik secara universal" — database yang menangani beban tulis sangat tinggi biasanya memilih algoritma cepat (rasio kompresi dikorbankan sedikit demi throughput), sementara sistem yang menyimpan data dingin (jarang diakses, misalnya arsip) bisa memilih rasio kompresi maksimal karena kecepatan bukan prioritas.

> [!question] Perlu diverifikasi
> Klaim: algoritma kompresi spesifik (Snappy, LZ4, Zstandard) dan mana yang dipakai default oleh database tertentu.
> Kenapa ragu: pilihan algoritma kompresi default bisa berbeda antar versi database dan sering dikonfigurasi ulang oleh vendor seiring waktu.
> Cara verifikasi: dokumentasi resmi masing-masing database (PostgreSQL TOAST compression, InnoDB page compression, RocksDB compression options) untuk versi yang relevan.

## In Go

Write amplification dan kompresi adalah keputusan konfigurasi database, bukan sesuatu yang diimplementasikan di kode aplikasi — tapi memahami dampaknya memengaruhi keputusan desain skema dan pola akses:

```go
package skema

// KomentarPanjang menunjukkan pertimbangan desain skema yang MEMPERHATIKAN
// write amplification: kolom yang sering di-UPDATE terpisah dari kolom yang
// jarang berubah, mengurangi ukuran halaman yang perlu ditulis ulang untuk
// setiap perubahan kecil.
//
// Alih-alih satu tabel besar dengan kolom deskripsi_panjang (jarang berubah)
// bercampur dengan kolom status (sering berubah) dalam baris yang sama,
// memisahkan keduanya ke tabel berbeda mengurangi ukuran halaman yang
// perlu ditulis ulang setiap kali HANYA status yang berubah — halaman
// yang menyimpan deskripsi_panjang tidak pernah tersentuh oleh update status.
type contohSkemaDipisah struct {
	// Tabel permohonan_status: sering diubah, baris kecil
	// id, status, updated_at

	// Tabel permohonan_detail: jarang diubah, baris besar
	// id, deskripsi_panjang, metadata_json, dokumen_terlampir
}
```

Pola ini — memisahkan kolom yang sering berubah dari kolom yang jarang berubah ke tabel terpisah — secara langsung mengurangi write amplification di level B-Tree, karena halaman yang berisi data yang jarang berubah tidak pernah ikut ditulis ulang hanya karena satu kolom kecil di baris "logis" yang sama berubah.

## In His Stack

Untuk sistem dengan volume log dan audit trail besar (relevan untuk kepatuhan compliance pemerintah), memahami write amplification menjelaskan kenapa biaya storage seringkali jauh lebih tinggi dari yang diharapkan berdasarkan ukuran data mentah — baik dari sisi ruang disk yang terpakai (sebelum kompresi efektif diterapkan) maupun dari sisi keausan hardware fisik untuk deployment on-premise. Untuk deployment di Kubernetes dengan storage berbasis SSD cloud, biaya I/O yang ditagih berdasarkan jumlah operasi (bukan hanya volume data) membuat write amplification punya dampak finansial yang bisa dihitung langsung — sesuatu yang layak dipahami koordinator teknis saat mengevaluasi biaya infrastruktur lintas 13 aplikasi.

## Trade-offs and When Not To Use It

Mengaktifkan kompresi tidak selalu menguntungkan — untuk beban kerja yang sudah CPU-bound (banyak komputasi per request, bukan I/O-bound), menambah overhead kompresi/dekompresi bisa memperlambat keseluruhan sistem meski mengurangi I/O. Untuk data yang sudah terkompresi secara alami (gambar, file terenkripsi, data yang sudah dikompresi di level aplikasi sebelum disimpan), kompresi tambahan di level database memberi manfaat minimal (data yang sudah random/terkompresi tidak bisa dikompresi lebih jauh secara signifikan) sementara tetap membayar biaya CPU untuk mencobanya. Mengurangi write amplification lewat desain skema (memisahkan kolom sering-berubah dari jarang-berubah) juga menambah kompleksitas query (butuh `JOIN` untuk data yang sebelumnya ada dalam satu tabel) — trade-off yang hanya sepadan kalau volume tulis dan write amplification-nya benar-benar terukur signifikan, bukan diterapkan sebagai optimasi prematur di semua tempat.

## Common Mistakes

> [!warning] Jebakan
> Mengukur kebutuhan kapasitas storage/SSD hanya berdasarkan volume data logis aplikasi, tanpa memperhitungkan write amplification — bisa meremehkan kecepatan keausan SSD atau kebutuhan I/O throughput yang sesungguhnya jauh lebih tinggi dari volume data yang terlihat di aplikasi.

> [!warning] Jebakan
> Mengaktifkan kompresi tingkat tinggi secara serampangan tanpa mengukur dampaknya pada CPU — untuk sistem yang sudah CPU-bound, ini bisa memindahkan bottleneck dari I/O ke CPU, memperlambat sistem secara keseluruhan meski volume I/O berkurang.

> [!warning] Jebakan
> Mencampur kolom yang sangat sering berubah dengan kolom besar yang jarang berubah dalam satu baris tabel yang sama, tanpa menyadari setiap perubahan kecil memaksa penulisan ulang seluruh halaman yang berisi kolom besar itu juga.

## Exercises

1. Jelaskan apa itu write amplification, dan kenapa ia terjadi di kedua struktur (B-Tree maupun LSM-Tree) meski lewat mekanisme berbeda.
2. Kenapa write amplification punya dampak finansial langsung untuk deployment di cloud dengan billing berbasis operasi I/O?
3. Kenapa kompresi tidak selalu menguntungkan, meski secara umum mengurangi write amplification fisik?
4. Desain terbuka: tabel `permohonan` di sistemmu punya kolom `status` (diubah puluhan kali per hari per baris seiring alur persetujuan) dan kolom `dokumen_lengkap_base64` (blob besar, disimpan sekali saat submit, hampir tidak pernah berubah). Rancang perubahan skema yang mengurangi write amplification akibat perubahan `status` yang sering, dan jelaskan trade-off yang muncul dari perubahan itu.

> [!success]- Kunci jawaban
> **1.** Write amplification adalah rasio antara byte yang benar-benar ditulis ke disk dibanding byte yang secara logis diminta aplikasi untuk ditulis. Di B-Tree, ia terjadi karena setiap perubahan kecil memaksa penulisan ulang seluruh halaman disk yang menampungnya (plus WAL), bukan hanya byte yang berubah. Di LSM-Tree, ia terjadi karena data yang sama ditulis ulang berkali-kali seiring proses compaction menggabungkan SSTable dari kecil ke besar sepanjang siklus hidupnya di sistem. Kedua mekanisme berbeda total, tapi keduanya menghasilkan fenomena yang sama: byte fisik yang ditulis jauh melebihi byte logis yang diminta.
> **4.** Pisahkan kolom `dokumen_lengkap_base64` (besar, jarang berubah) ke tabel terpisah dari kolom `status` (kecil, sering berubah) — misalnya tabel `permohonan` menyimpan `id`, `status`, `updated_at` (baris kecil), dan tabel `permohonan_dokumen` menyimpan `permohonan_id`, `dokumen_lengkap_base64` (baris besar, ditulis sekali saat submit). Setelah pemisahan ini, setiap perubahan `status` hanya menulis ulang halaman-halaman kecil di tabel `permohonan`, tidak pernah menyentuh halaman besar yang berisi dokumen di tabel terpisah — mengurangi write amplification untuk operasi yang paling sering terjadi (perubahan status) secara signifikan. Trade-off: mengambil data lengkap satu permohonan (status **dan** dokumen) sekarang butuh `JOIN` antara dua tabel alih-alih satu `SELECT` sederhana — kompleksitas query bertambah sedikit demi mengurangi biaya I/O yang jauh lebih sering terjadi (perubahan status berkali-kali per hari, dibanding pengambilan dokumen lengkap yang relatif jarang).

## Self-Check

- Apa itu write amplification, dan kenapa ia terjadi di B-Tree maupun LSM-Tree?
- Kenapa write amplification punya dampak langsung pada umur pakai SSD, bukan hanya soal kecepatan?
- Apa trade-off inti mengaktifkan kompresi di database?
- Bagaimana desain skema (memisahkan kolom sering-berubah dari jarang-berubah) bisa mengurangi write amplification?

## Connected Notes

- [[LSM-Trees vs B-Trees]] — write amplification adalah biaya konkret yang muncul dari kedua struktur yang dibahas di note itu, hanya lewat mekanisme berbeda.
- [[B+Tree Structure]] — node split yang dibahas di note itu adalah salah satu sumber write amplification di struktur B-Tree.
- [[Beyond Relational - Document, Key-Value, Wide-Column, Graph, and Time-Series Stores]] — pemilihan model data yang tepat untuk pola akses tertentu juga berdampak pada write amplification, dibahas di note berikutnya.
- [[../50 Concurrency and Performance/_Overview|Concurrency and Performance Overview]] — trade-off CPU vs I/O yang muncul dari kompresi adalah tema yang berulang dalam optimasi performa di domain itu.
- [[../92 Tools/PostgreSQL - Features Worth Switching For|PostgreSQL - Features Worth Switching For]] — TOAST compression di PostgreSQL adalah implementasi konkret dari trade-off kompresi yang dibahas di note ini.

## Further Reading

- Dokumentasi resmi RocksDB, bagian "Compression" dan penjelasan write amplification di konteks LSM-Tree.
- Dokumentasi resmi PostgreSQL, bagian "TOAST" untuk mekanisme kompresi bawaan pada kolom besar.

## Catatan Saya

*Tulis di sini apakah kamu pernah mengukur write amplification nyata di sistem kerjaanmu (volume I/O fisik dibanding volume data logis) — kalau belum, coba cek metrik storage-nya sekali.*
