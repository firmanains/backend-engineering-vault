---
title: Consistent Error Responses
type: concept
level: junior
domain: apis
status: unread
difficulty: 2
est_minutes: 13
prerequisites: ["[[Choosing Status Codes]]", "[[../20 Go Language/Error Wrapping|Error Wrapping]]"]
next: ["[[OpenAPI]]", "[[../70 Infrastructure and Delivery/Correlation IDs|Correlation IDs]]"]
tags: [backend, apis]
created: 2026-07-26
---

## TL;DR

Status code menyatakan **kelas** kegagalan (lihat [[Choosing Status Codes]]); format body error menyatakan **detailnya**, dan bentuknya harus **identik di seluruh endpoint** — biasanya sebuah amplop (envelope) berisi kode error yang bisa dibaca mesin (`"code": "DOKUMEN_TIDAK_DITEMUKAN"`), pesan yang aman ditampilkan ke pengguna, dan ID request untuk korelasi log. Detail internal (query SQL mentah, stack trace, path file) **tidak pernah** boleh ikut terkirim ke konsumen API — itu tempatnya di log server, bukan di response.

## The Problem

Bayangkan sebuah API di mana satu endpoint mengembalikan error sebagai `{"msg": "tidak ditemukan"}`, endpoint lain sebagai `{"error": "...", "detail": "..."}`, dan endpoint ketiga sebagai `{"errors": [{"field": "...", "message": "..."}]}`. Tim integrasi partner yang ingin menampilkan pesan error yang informatif ke pengguna akhir mereka (misalnya "kenapa permohonan Anda ditolak") harus menulis logic parsing berbeda untuk **setiap** endpoint — dan begitu ada endpoint baru dengan bentuk error yang sedikit berbeda lagi, logic itu harus disesuaikan lagi.

Masalah kedua yang lebih serius: salah satu endpoint, saat terjadi error database, mengembalikan pesan error mentah dari driver database langsung ke response — termasuk detail query SQL dan kadang nama tabel atau kolom internal. Ini bukan cuma tidak rapi; ini kebocoran informasi internal yang bisa dimanfaatkan pihak yang berniat buruk untuk memahami struktur database di baliknya, jauh melampaui apa yang seharusnya pernah terlihat dari luar sistem.

## Intuition

Bayangkan format error yang konsisten seperti **formulir laporan insiden standar yang dipakai semua departemen** dalam satu organisasi — apa pun departemen yang mengisinya (endpoint mana pun yang gagal), pembaca (client) tahu persis di mana menemukan kode insiden, deskripsi, dan nomor referensi, tanpa perlu belajar format baru setiap kali.

Analogi ini bocor pada soal toleransi kesalahan bentuk. Formulir kertas yang sedikit tidak rapi masih bisa dibaca manusia secara longgar. Client otomatis yang mem-parsing JSON tidak seluwes itu — kalau field `error` kadang berupa string dan kadang berupa object di endpoint berbeda, parsing otomatis langsung gagal secara diam-diam, bukan sekadar "agak merepotkan" seperti formulir kertas yang sedikit berantakan bagi pembaca manusia.

## How It Works

Amplop error yang konsisten, dipakai **semua** endpoint tanpa kecuali:

```json
{
  "error": {
    "code": "DOKUMEN_TIDAK_DITEMUKAN",
    "message": "Dokumen dengan ID tersebut tidak ditemukan.",
    "request_id": "req_8f3a2b1c"
  }
}
```

- **`code`** — string stabil yang bisa dijadikan dasar percabangan logic client (`if code == "DOKUMEN_TIDAK_DITEMUKAN"`), tidak pernah berubah meski kata-kata pesan berubah (misalnya karena perbaikan bahasa atau terjemahan).
- **`message`** — pesan yang **aman** ditampilkan ke pengguna akhir, tidak pernah berisi detail teknis internal.
- **`request_id`** — ID unik untuk korelasi antara laporan client dan log internal server (lihat [[../70 Infrastructure and Delivery/Correlation IDs|Correlation IDs]]), memudahkan tim support/ops menelusuri insiden spesifik tanpa harus menebak-nebak dari waktu kejadian saja.

```mermaid
flowchart TB
    Err["error internal (mis. sql.ErrNoRows,\nlengkap dengan detail teknis)"] --> Split{"Pisahkan di boundary"}
    Split -->|"Log internal (lengkap, boleh detail)"| Log["Server log:\nquery SQL, stack trace, dll."]
    Split -->|"Response ke client (aman, terbatas)"| Resp["{\"error\": {\"code\": ..., \"message\": ..., \"request_id\": ...}}"]
```

## In Go

```go
type ErrorResponse struct {
    Error ErrorDetail `json:"error"`
}

type ErrorDetail struct {
    Code      string `json:"code"`
    Message   string `json:"message"`
    RequestID string `json:"request_id"`
}

// Satu function terpusat, dipakai SEMUA handler — mencegah setiap
// handler menciptakan bentuk error sendiri-sendiri.
func writeError(w http.ResponseWriter, r *http.Request, err error) {
    requestID := middleware.GetReqID(r.Context()) // atau mekanisme serupa

    code, statusHTTP, pesanAman := klasifikasikanError(err)

    // Detail lengkap (termasuk error asli) HANYA masuk log server,
    // TIDAK PERNAH ke response.
    log.Printf("request_id=%s error=%v", requestID, err)

    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(statusHTTP)
    json.NewEncoder(w).Encode(ErrorResponse{
        Error: ErrorDetail{
            Code:      code,
            Message:   pesanAman,
            RequestID: requestID,
        },
    })
}

func klasifikasikanError(err error) (code string, statusHTTP int, pesanAman string) {
    switch {
    case errors.Is(err, ErrNotFound):
        return "DOKUMEN_TIDAK_DITEMUKAN", http.StatusNotFound, "Dokumen dengan ID tersebut tidak ditemukan."
    case errors.Is(err, ErrValidasiGagal):
        return "VALIDASI_GAGAL", http.StatusUnprocessableEntity, "Data yang dikirim tidak valid."
    default:
        // Pesan generik untuk error tak terduga — TIDAK menyertakan
        // detail teknis error asli sama sekali.
        return "KESALAHAN_INTERNAL", http.StatusInternalServerError, "Terjadi kesalahan internal. Silakan hubungi dukungan dengan menyertakan request ID."
    }
}
```

## In His Stack

**Yii2** dengan REST controller bawaannya sebenarnya sudah menyediakan formatter error JSON yang cukup konsisten secara default — tapi begitu tim mulai meng-override behavior ini per controller secara ad hoc (misalnya menangkap exception tertentu dan mengembalikan bentuk response custom sendiri tanpa mengikuti formatter bawaan), konsistensi itu perlahan terkikis persis seperti yang dijelaskan di "The Problem". Memiliki default framework yang baik tidak menjamin konsistensi bertahan — butuh disiplin tim yang disengaja (code review, atau bahkan linter) untuk mempertahankannya seiring codebase tumbuh.

## Trade-offs and When Not To Use It

Menegakkan satu bentuk error envelope di seluruh API butuh disiplin desain di awal (biasanya satu function/middleware terpusat seperti `writeError` di atas) — investasi kecil yang jauh lebih murah dilakukan sejak awal dibanding memperbaikinya setelah puluhan endpoint sudah terlanjur punya bentuk error yang berbeda-beda. Tidak ada skenario realistis di mana melonggarkan konsistensi ini menguntungkan — bahkan untuk API internal kecil, kebiasaan ini murah untuk diterapkan sejak awal dan mahal untuk ditambal belakangan.

## Common Mistakes

> [!warning] Jebakan
> Membiarkan setiap handler menciptakan bentuk response error sendiri-sendiri tanpa konvensi terpusat, memaksa setiap konsumen API menulis logic parsing berbeda untuk setiap endpoint.

> [!warning] Jebakan
> Mengirim detail error internal (pesan error database mentah, stack trace, path file) langsung ke response API. Ini kebocoran informasi yang bisa dimanfaatkan pihak luar untuk memahami struktur internal sistem.

> [!warning] Jebakan
> Membuat logic client bergantung pada teks pesan error (`message`) untuk percabangan logika, alih-alih kode error yang stabil (`code`). Perubahan kata-kata pesan (perbaikan bahasa, terjemahan) bisa diam-diam mematahkan logic client yang bergantung pada string tertentu.

## Exercises

1. Kenapa format body error perlu konsisten lintas semua endpoint, terpisah dari status code?
2. Kenapa `code` yang stabil lebih aman dijadikan dasar logic client dibanding `message`?
3. Kenapa detail error internal (query SQL, stack trace) tidak boleh masuk ke response API?
4. Desain terbuka: sebuah tim ingin mulai menegakkan format error konsisten di API yang sudah punya puluhan endpoint dengan bentuk error yang berbeda-beda selama bertahun-tahun, dan partner yang sudah ada bergantung pada bentuk lama masing-masing. Rancang strategi migrasi bertahap tanpa mematahkan integrasi partner yang sudah berjalan.

> [!success]- Kunci jawaban
> Perkenalkan format error baru secara aditif dulu — tambahkan field `error.code` dan `error.request_id` yang baru **berdampingan** dengan bentuk error lama yang sudah ada di setiap endpoint (bukan menggantikannya sekaligus), sehingga partner lama tidak langsung rusak. Dokumentasikan format baru ini sebagai standar yang direkomendasikan untuk integrasi baru, dan komunikasikan ke partner lama secara bertahap bahwa bentuk lama akan di-deprecate mengikuti kebijakan [[API Versioning]] dengan tenggat waktu yang realistis. Setelah cukup waktu (dan konfirmasi partner sudah bermigrasi ke pembacaan `error.code`), baru pertimbangkan menyederhanakan response sepenuhnya ke format baru saja, mengikuti proses deprecation yang sama seperti perubahan breaking lainnya.

## Self-Check

- Apa perbedaan peran status code dan body error dalam menyampaikan informasi kegagalan?
- Kenapa `code` yang stabil lebih baik untuk logic client dibanding teks pesan?
- Kenapa detail teknis internal tidak boleh masuk response API?
- Apa fungsi `request_id` dalam format error yang konsisten?

## Connected Notes

- [[Choosing Status Codes]] — prasyarat: status code menyatakan kelas kegagalan, body error di note ini menyatakan detailnya.
- [[../20 Go Language/Error Wrapping|Error Wrapping]] — batas antara error internal yang lengkap (log) dan error yang aman dikirim ke client (response), yang dijelaskan penuh di note ini.
- [[../70 Infrastructure and Delivery/Correlation IDs|Correlation IDs]] — `request_id` di envelope error ini adalah salah satu bentuk penerapan correlation ID.
- [[OpenAPI]] — format error envelope yang konsisten harus didokumentasikan sebagai bagian dari kontrak API di OpenAPI.
- [[../80 Security/The OWASP Top 10|The OWASP Top 10]] — kebocoran informasi lewat pesan error adalah salah satu kategori kerentanan umum yang relevan di sana.

## Further Reading

- RFC 9457 (*Problem Details for HTTP APIs*) — standar format error JSON yang diusulkan secara resmi untuk API HTTP, referensi yang baik untuk struktur envelope error yang matang.

## Catatan Saya

*Tulis di sini API di kerjaanmu yang bentuk error-nya tidak konsisten antar endpoint, dan bagaimana itu menyulitkan integrasi.*
