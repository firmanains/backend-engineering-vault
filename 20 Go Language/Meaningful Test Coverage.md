---
title: Meaningful Test Coverage
type: concept
level: junior
domain: go
status: unread
difficulty: 2
est_minutes: 12
prerequisites: ["[[Stdlib Testing vs Testify]]", "[[Table-Driven Tests]]"]
next: ["[[../90 Architecture and Design/Handler-Service-Repository Layering|Handler-Service-Repository Layering]]"]
tags: [backend, go]
created: 2026-07-26
---

## TL;DR

Angka coverage (`go test -cover`) hanya mengukur baris kode mana yang **dieksekusi** selama test berjalan — ia sama sekali tidak mengukur apakah perilaku yang dieksekusi itu benar-benar **diverifikasi** dengan assertion yang bermakna. Coverage 100% bisa dicapai dengan test yang memanggil function tanpa memeriksa hasilnya sama sekali; coverage 70% dengan assertion yang tepat di jalur bisnis kritis dan error path bisa jauh lebih berharga. Memperlakukan angka coverage sebagai target itu sendiri (bukan sebagai alat diagnosis untuk menemukan kode yang sama sekali belum tersentuh) adalah contoh klasik Goodhart's law: begitu sebuah ukuran dijadikan target, ia berhenti jadi ukuran yang baik.

## The Problem

Bayangkan manajemen menetapkan target "coverage minimal 80%" untuk semua service, dan tim di bawah tekanan menulis test yang memanggil setiap function tanpa benar-benar memeriksa hasilnya secara berarti — cukup memastikan function itu "dipanggil" (kadang hanya memeriksa "tidak panic") demi angka coverage naik. Dashboard coverage menunjukkan 85%, semua terlihat baik di atas kertas.

Beberapa minggu kemudian, sebuah bug di logic perhitungan biaya layanan lolos ke production — bug yang sebenarnya ada di function yang *sudah* "tercover" 100% menurut dashboard, tapi test yang mengeksekusinya tidak pernah benar-benar memeriksa **nilai hasil perhitungannya**, hanya memastikan function itu tidak melempar error. Angka coverage yang tinggi memberi rasa aman yang salah — ia mengukur eksekusi, bukan verifikasi.

## Intuition

Bayangkan coverage seperti **menghitung berapa banyak ruangan yang dimasuki inspektur rumah**, tanpa memeriksa apakah inspektur itu benar-benar menguji sesuatu di setiap ruangan (memeriksa kabel, menyalakan keran) atau sekadar berjalan masuk lalu keluar lagi. Memasuki 100% ruangan (coverage 100%) tidak membuktikan apa pun kalau inspektur tidak benar-benar memverifikasi apa pun di dalamnya.

Analogi ini bocor pada soal siapa yang bisa menilai "pemeriksaan yang bermakna". Inspektur rumah sungguhan punya keahlian domain untuk tahu persis apa yang perlu diperiksa di setiap ruangan. Di kode, "bermakna" itu sangat bergantung konteks bisnis — tidak ada tool otomatis yang bisa memberi tahu apakah sebuah test benar-benar menangkap bug yang tepat. Coverage tool hanya bisa mengukur dimensi "sudah dieksekusi atau belum", sepenuhnya buta terhadap dimensi "sudah diverifikasi dengan benar atau belum" — dimensi kedua ini selalu butuh penilaian manusia.

## How It Works

```sh
go test -cover ./...
# ok  github.com/org/legal-service/biaya  0.012s  coverage: 85.7% of statements
```

Angka ini murni statement coverage — persentase baris kode yang tereksekusi minimal sekali selama test berjalan. Ia tidak membedakan antara baris yang dieksekusi dengan assertion ketat dan baris yang dieksekusi tanpa diperiksa sama sekali.

```go
// "Tercover" 100% menurut alat, tapi TIDAK BERMAKNA — hasil
// perhitungan tidak pernah diverifikasi sama sekali.
func TestHitungBiayaTidakBermakna(t *testing.T) {
    _, err := HitungBiaya("premium", 5)
    if err != nil {
        t.Fatal(err)
    }
    // selesai — tidak ada pemeriksaan terhadap NILAI biayanya!
}

// Bermakna: memverifikasi nilai hasil yang sebenarnya, bukan
// sekadar "tidak error".
func TestHitungBiayaBermakna(t *testing.T) {
    biaya, err := HitungBiaya("premium", 5)
    require.NoError(t, err)
    require.Equal(t, 250000, biaya) // memverifikasi HASIL sungguhan
}
```

Kedua test di atas menghasilkan angka coverage yang **identik** untuk function `HitungBiaya` — tapi hanya satu yang benar-benar bisa menangkap bug kalau logic perhitungannya salah.

## In His Stack

**PHPUnit** dengan Xdebug menghasilkan laporan coverage yang secara konsep sama persis — dan risiko Goodhart's law yang sama berlaku di sana. Kalau kamu pernah melihat dashboard coverage PHP yang "terlihat sehat" di kantor tapi bug tetap lolos ke production secara rutin, ini kemungkinan besar gejala yang sama: coverage tinggi yang dicapai lewat test yang mengeksekusi kode tanpa benar-benar memverifikasi hasilnya.

## Trade-offs and When Not To Use It

Coverage tetap berguna sebagai **alat diagnosis** — menemukan kode yang **sama sekali** belum tersentuh test sama sekali, terutama error path yang sering diabaikan karena "jalur bahagia" (happy path) sudah terlihat tercover. Yang perlu dihindari adalah menjadikannya **target** itu sendiri. Prioritaskan cakupan test berdasarkan risiko dan kompleksitas bisnis (logic perhitungan kritis, jalur error yang menentukan konsekuensi finansial atau hukum) di atas kode trivial (getter/setter sederhana, kode yang di-generate otomatis) — codebase dengan coverage 60% yang terkonsentrasi di logic kritis dan error path bisa jauh lebih aman daripada codebase dengan coverage 95% yang sebagian besar hanya menguji hal-hal sepele tanpa assertion berarti.

## Common Mistakes

> [!warning] Jebakan
> Menjadikan angka coverage sebagai target/KPI itu sendiri, mendorong tim menulis test yang mengeksekusi kode tanpa benar-benar memverifikasi hasilnya secara bermakna — persis Goodhart's law: begitu ukuran jadi target, ia berhenti jadi ukuran yang baik.

> [!warning] Jebakan
> Mengabaikan jalur error karena "coverage jalur bahagia sudah tinggi". Jalur error justru sering menjadi tempat bug paling mahal bersembunyi, sekaligus jalur yang paling jarang diuji secara sungguh-sungguh.

> [!warning] Jebakan
> Berasumsi coverage yang lebih tinggi selalu berarti lebih aman. Coverage yang tinggi tapi terkonsentrasi di kode trivial, dengan assertion yang lemah di logic bisnis kritis, bisa memberi rasa aman yang salah dibanding coverage lebih rendah yang terkonsentrasi tepat di tempat yang penting.

## Exercises

1. Apa yang sebenarnya diukur angka coverage, dan apa yang **tidak** diukurnya?
2. Kenapa test yang mencapai 100% coverage untuk sebuah function tidak menjamin function itu benar?
3. Kenapa menjadikan target coverage sebagai KPI bisa menghasilkan kualitas test yang lebih buruk, bukan lebih baik?
4. Desain terbuka: sebuah tim dari 13 aplikasi legal-services diminta manajemen mencapai coverage minimal 80% di semua service dalam waktu satu bulan, di bawah tekanan waktu yang ketat. Rancang pendekatan yang tetap memenuhi tekanan angka ini secara jujur, tanpa jatuh ke pola menulis test yang tidak bermakna hanya demi mengejar angka.

> [!success]- Kunci jawaban
> Alih-alih mengejar 80% secara merata tanpa pandang bulu, prioritaskan test bermakna dulu di area berisiko tertinggi: logic perhitungan bisnis, validasi input, dan jalur error yang menentukan konsekuensi hukum/finansial — area ini kemungkinan besar sudah menyumbang porsi signifikan dari total baris kode di layer service/domain. Kode trivial (getter/setter, DTO polos tanpa logic) secara alami akan ikut tercover sebagai efek samping menguji logic yang memakainya, tanpa perlu test terpisah untuknya. Komunikasikan ke manajemen bahwa angka 80% akan dicapai sebagai **hasil** dari testing yang benar diprioritaskan dengan tepat, bukan sebagai target yang dikejar langsung — dan usulkan melacak metrik pelengkap yang lebih sulit di-gaming, seperti jumlah bug production yang lolos per bulan, sebagai indikator kualitas testing yang sebenarnya, berdampingan dengan angka coverage.

## Self-Check

- Apa yang diukur angka coverage, dan apa yang tidak diukurnya?
- Kenapa 100% coverage tidak menjamin kode itu benar?
- Apa itu Goodhart's law, dan bagaimana ia berlaku pada target coverage?
- Kenapa jalur error sering jadi bagian paling sedikit diuji, padahal paling berisiko?

## Connected Notes

- [[Table-Driven Tests]] dan [[Stdlib Testing vs Testify]] — alat menulis assertion yang bermakna, bukan sekadar mengeksekusi kode.
- [[Mocking Through Interfaces]] — membuat kode yang sulit diuji (bergantung langsung pada dependency eksternal) jadi testable, prasyarat untuk coverage bermakna di area itu.
- [[Errors as Values]] — jalur error yang sering diabaikan test, padahal seringkali paling kritis untuk diverifikasi.
- [[../90 Architecture and Design/Handler-Service-Repository Layering|Handler-Service-Repository Layering]] — layering yang memudahkan memprioritaskan test bermakna di layer logic bisnis, terpisah dari layer trivial.

## Further Reading

- Dokumentasi resmi `go test -cover` dan `go tool cover` (pkg.go.dev/cmd/go) untuk detail teknis penghitungan statement coverage.

## Catatan Saya

*Tulis di sini kalau kamu pernah melihat (atau menulis sendiri) test yang "tercover" tinggi tapi ternyata tidak menangkap bug nyata.*
