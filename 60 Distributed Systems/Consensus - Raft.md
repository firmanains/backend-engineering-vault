---
title: Consensus - Raft
type: concept
level: senior
domain: distributed
status: unread
difficulty: 5
est_minutes: 22
prerequisites: ["[[Quorums]]", "[[Failure Detectors]]"]
next: ["[[Consensus - Paxos in Overview]]"]
tags: [backend, distributed]
created: 2026-08-02
---

## TL;DR

Consensus adalah masalah membuat sekumpulan node yang bisa crash atau lambat (tapi tidak berbohong; Raft tidak menangani node jahat atau *Byzantine*) **sepakat** pada satu nilai atau urutan kejadian, meski jaringan di antara mereka tidak selalu andal. Raft menyelesaikan ini dengan cara yang secara sengaja dirancang **mudah dipahami** (kontras eksplisit dengan Paxos yang terkenal sulit, lihat [[Consensus - Paxos in Overview]]): satu node dipilih sebagai **leader** lewat mekanisme election berbasis timeout dan quorum, leader itu satu-satunya yang menerima tulisan baru dan menyusunnya jadi log berurutan, lalu mereplikasi log itu ke node lain (**follower**) — begitu mayoritas follower mengonfirmasi menerima satu entri log, entri itu dianggap **committed** dan tidak akan pernah hilang lagi, apa pun yang terjadi setelahnya.

## The Problem

Sebuah tim ingin membangun sistem konfigurasi terdistribusi untuk 13 aplikasi — satu sumber kebenaran untuk pengaturan yang harus selalu sama persis di semua node yang membacanya, bahkan saat sebagian node mengalami gangguan jaringan sesaat. Pendekatan naif: setiap node menyimpan salinan konfigurasi sendiri, dan saat ada perubahan, perubahan itu dikirim ke semua node yang bisa dihubungi saat itu. Masalahnya segera terlihat: kalau satu node sedang terputus saat perubahan dikirim, node itu ketinggalan perubahan tanpa ada mekanisme yang menjamin ia akan mengejar ketertinggalan itu dengan benar begitu terhubung kembali — dan kalau dua perubahan berbeda dikirim hampir bersamaan oleh dua sumber berbeda, tidak ada aturan jelas urutan mana yang "menang", berpotensi menghasilkan node yang punya versi konfigurasi berbeda-beda tanpa cara rekonsiliasi yang pasti.

Consensus adalah nama formal untuk masalah ini: bagaimana memastikan **semua** node yang berpartisipasi, meski sebagian sempat terputus atau lambat, pada akhirnya sepakat pada urutan perubahan yang **sama persis** — bukan sekadar "mengejar ketertinggalan" secara informal, tapi dengan jaminan matematis bahwa tidak ada dua node yang punya urutan berbeda untuk data yang sama.

## Intuition

Cara paling mudah memahaminya: Raft seperti **rapat dengan satu pemimpin sidang yang jelas**, dibanding rapat tanpa pemimpin di mana semua orang mengusulkan dan menyepakati sesuatu secara bersamaan (lebih dekat ke Paxos). Dalam rapat dengan pemimpin: semua usulan **harus** lewat pemimpin lebih dulu, pemimpin menuliskannya di papan tulis dalam urutan yang jelas, dan sebuah usulan baru dianggap "resmi disepakati" begitu **mayoritas** peserta rapat mengangguk setuju. Kalau pemimpin sidang itu sendiri tiba-tiba tidak bisa melanjutkan (sakit mendadak), peserta lain mengadakan pemilihan cepat untuk pemimpin baru — dan begitu pemimpin baru terpilih, semua orang kembali mengacu ke satu papan tulis yang sama, bukan mencatat sendiri-sendiri secara paralel.

Analogi ini bocor pada soal keandalan komunikasi. Rapat fisik mengasumsikan semua orang mendengar pemimpin dengan jelas. Dalam sistem terdistribusi nyata, pesan bisa hilang atau tertunda — dan Raft harus secara eksplisit menangani skenario di mana sebagian follower tidak menerima pesan leader tepat waktu, sesuatu yang tidak jadi masalah utama dalam rapat manusia biasa yang berlangsung dalam satu ruangan yang sama.

## How It Works

```mermaid
stateDiagram-v2
    [*] --> Follower
    Follower --> Candidate: Timeout tanpa heartbeat dari leader
    Candidate --> Leader: Menang mayoritas suara
    Candidate --> Candidate: Timeout tanpa pemenang (suara terbelah), election ulang
    Candidate --> Follower: Menemukan leader sah / term lebih baru
    Leader --> Follower: Menemukan term lebih baru
```
Setiap node punya salah satu dari tiga peran ini. **Follower** pasif, hanya menerima entri log dari leader dan heartbeat berkala. Kalau follower tidak menerima heartbeat dalam jangka waktu tertentu (mencurigai leader mati, mirip [[Failure Detectors]]), ia berubah jadi **candidate** dan meminta suara dari node lain untuk jadi leader baru. Timeout election di setiap node diacak (misalnya dalam rentang 150–300 ms di paper aslinya) supaya jarang ada dua candidate yang mulai bersamaan dan membelah suara. Kalau ia mendapat suara dari mayoritas (quorum, lihat [[Quorums]]), ia jadi **leader** — satu-satunya node yang boleh menerima tulisan baru dan mengatur urutan log untuk seluruh cluster.

Proses replikasi log: klien mengirim perintah ke leader, leader menambahkannya ke log lokalnya, lalu mengirim entri itu ke semua follower. Begitu **mayoritas** follower mengonfirmasi menerima entri itu, leader menganggapnya **committed** dan menerapkannya (misalnya menulis ke state aplikasi), lalu memberi tahu follower bahwa entri itu sudah commit di replikasi berikutnya. Follower yang tertinggal (sempat terputus) akan mengejar ketertinggalan begitu terhubung kembali. Setiap pesan `AppendEntries` membawa indeks dan term dari entri tepat **sebelum** entri baru (`prevLogIndex`, `prevLogTerm`). Follower menolak pesan itu kalau log-nya tidak punya entri yang cocok di posisi tersebut; leader lalu mundur satu posisi dan mencoba lagi sampai menemukan titik di mana kedua log sama. Semua entri follower sesudah titik itu yang bertentangan dengan log leader dihapus dan diganti. Pemeriksaan kecil ini (*log matching*) yang menjamin: kalau dua log punya entri dengan indeks dan term yang sama, seluruh isi sebelum entri itu juga identik.

## Under The Hood

**Term** adalah konsep krusial yang mencegah kebingungan antar leader lama dan baru — setiap kali ada election baru, term (nomor urut "generasi kepemimpinan") bertambah, dan setiap pesan yang dikirim menyertakan term pengirimnya. Node yang menerima pesan dengan term **lebih lama** dari yang ia ketahui langsung menolaknya — mekanisme ini yang mencegah leader lama (yang mungkin sempat terputus dan tidak sadar sudah digantikan) terus mengklaim otoritas setelah cluster sudah memilih leader baru, mencegah split brain (lihat [[Leader Election and Split Brain]]).

**Aturan election yang membuat entri committed tidak pernah hilang.** "Mayoritas punya salinannya" saja belum cukup: tanpa aturan tambahan, node yang log-nya tertinggal bisa terpilih jadi leader lalu menimpa entri yang sudah committed. Raft mencegahnya dengan *election restriction*: node hanya memberikan suaranya kepada candidate yang log-nya **setidaknya sama mutakhirnya** dengan log node itu sendiri (dibandingkan dari term entri terakhir, lalu panjang log). Karena entri committed ada di mayoritas node, dan candidate juga butuh suara mayoritas, setidaknya satu pemilih pasti memegang entri itu dan akan menolak candidate yang tidak memilikinya. Ada satu kehalusan lagi: leader hanya menghitung replika untuk menentukan commit atas entri dari **term-nya sendiri**; entri dari term sebelumnya ikut ter-commit secara tidak langsung saat entri term baru di atasnya ter-commit. Paper Raft menunjukkan kenapa aturan ini perlu lewat skenario di Figure 8-nya.

**Membaca dari leader belum otomatis linearizable.** Leader yang sudah digantikan tapi belum tahu (misalnya sedang terisolasi oleh partition) masih bisa menjawab pembacaan dengan data usang. Implementasi nyata menambah langkah untuk pembacaan: leader memastikan dirinya masih leader dengan bertukar heartbeat dengan mayoritas sebelum menjawab (*ReadIndex*), atau memakai *lease* berbasis waktu yang bergantung pada asumsi batas clock drift.

Poin yang sering disalahpahami: Raft **tidak** menjamin setiap tulisan langsung diproses secepat mungkin — ada biaya nyata (leader harus menunggu konfirmasi mayoritas sebelum commit) demi jaminan bahwa entri yang sudah committed **tidak pernah** hilang, bahkan kalau leader yang menulisnya langsung mati sesaat setelahnya (karena mayoritas node lain sudah punya salinannya). Log yang belum committed (baru diterima sebagian kecil follower) **bisa** hilang atau ditimpa kalau leader berganti sebelum sempat commit — properti yang disengaja, bukan bug, karena entri yang belum diketahui mayoritas memang belum bisa dijamin bertahan.

## In Go

```go
package raft

// Kode ini menunjukkan tiga aturan inti Raft sebagai fungsi murni.
// Ini bukan implementasi lengkap: tidak ada jaringan, timer, persistensi,
// maupun penerapan entri ke state machine.

type Role int

const (
	Follower Role = iota
	Candidate
	Leader
)

type LogEntry struct {
	Term    int
	Command string
}

type Node struct {
	Role        Role
	CurrentTerm int
	VotedFor    string // kosong kalau belum memberi suara di term ini
	Log         []LogEntry
}

// lastLog mengembalikan indeks (berbasis 1, 0 = log kosong) dan term entri
// terakhir.
func (n *Node) lastLog() (index, term int) {
	if len(n.Log) == 0 {
		return 0, 0
	}
	return len(n.Log), n.Log[len(n.Log)-1].Term
}

// HandleRequestVote: aturan 1, term usang ditolak; aturan 2, suara hanya
// diberikan kepada candidate yang log-nya setidaknya sama mutakhir.
func (n *Node) HandleRequestVote(term int, candidateID string, lastIndex, lastTerm int) (granted bool, currentTerm int) {
	if term < n.CurrentTerm {
		return false, n.CurrentTerm
	}
	if term > n.CurrentTerm {
		n.CurrentTerm, n.Role, n.VotedFor = term, Follower, ""
	}
	if n.VotedFor != "" && n.VotedFor != candidateID {
		return false, n.CurrentTerm // sudah memilih orang lain di term ini
	}
	myIndex, myTerm := n.lastLog()
	upToDate := lastTerm > myTerm || (lastTerm == myTerm && lastIndex >= myIndex)
	if !upToDate {
		return false, n.CurrentTerm // election restriction
	}
	n.VotedFor = candidateID
	return true, n.CurrentTerm
}

// HandleAppendEntries: aturan 3, log matching. prevIndex/prevTerm menunjuk
// entri tepat sebelum entries (berbasis 1; 0 berarti awal log).
func (n *Node) HandleAppendEntries(term, prevIndex, prevTerm int, entries []LogEntry) (success bool, currentTerm int) {
	if term < n.CurrentTerm {
		return false, n.CurrentTerm // pengirim adalah leader dengan term usang
	}
	n.CurrentTerm, n.Role = term, Follower

	if prevIndex > len(n.Log) || (prevIndex > 0 && n.Log[prevIndex-1].Term != prevTerm) {
		return false, n.CurrentTerm // log tidak cocok; leader akan mundur dan mencoba lagi
	}

	for i, e := range entries {
		pos := prevIndex + i // posisi berbasis 0 untuk entri ini
		if pos < len(n.Log) {
			if n.Log[pos].Term == e.Term {
				continue // entri sudah ada dan sama
			}
			n.Log = n.Log[:pos] // konflik: buang entri ini dan semua sesudahnya
		}
		n.Log = append(n.Log, e)
	}
	return true, n.CurrentTerm
}

// Majority: jumlah node (termasuk leader) yang harus memiliki sebuah entri
// sebelum leader boleh menganggapnya committed.
func Majority(totalNodes int) int {
	return totalNodes/2 + 1
}
```

## In His Stack

Raft (atau turunannya) adalah mesin di balik banyak tool yang sudah atau mungkin dipakai di ekosistem 13 aplikasi — [[../92 Tools/Consul|Consul]] memakai Raft untuk menjaga konsistensi data registry-nya sendiri, dan sistem seperti etcd (yang mendasari state Kubernetes) juga memakai Raft. Memahami Raft membantu menjelaskan perilaku operasional yang sering membingungkan: kenapa cluster Consul atau etcd butuh **jumlah node ganjil** (memudahkan mayoritas yang jelas tanpa hasil seri), dan kenapa cluster dengan node yang terlalu sedikit hidup (di bawah mayoritas) berhenti menerima tulisan sama sekali — bukan bug, tapi properti keamanan yang disengaja untuk mencegah split brain.

## Trade-offs and When Not To Use It

Raft (dan consensus formal secara umum) menambah latency nyata untuk setiap tulisan — menunggu konfirmasi mayoritas lebih lambat dari sekadar menulis ke satu node dan berharap yang terbaik. Untuk data yang tidak butuh jaminan konsistensi kuat lintas node (cache yang boleh sedikit tidak sinkron, data yang bisa direkonstruksi dari sumber lain kalau hilang), overhead consensus penuh tidak sepadan — quorum sederhana atau bahkan eventual consistency (lihat [[Consistency Models]]) sudah cukup dan jauh lebih murah. Consensus formal bernilai jelas untuk data yang benar-benar kritis dan tidak boleh hilang atau tidak konsisten — metadata cluster, konfigurasi kritis, log yang jadi sumber kebenaran tunggal.

## Common Mistakes

> [!warning] Jebakan
> Menganggap entri log yang baru diterima leader (belum dikonfirmasi mayoritas follower) sudah aman dan tidak akan hilang — hanya entri yang sudah **committed** (dikonfirmasi mayoritas) yang dijamin bertahan; entri yang belum committed bisa hilang kalau leader mati sebelum sempat mereplikasinya.

> [!warning] Jebakan
> Menjalankan cluster Raft dengan jumlah node genap. Cluster 4 node butuh mayoritas 3, sehingga hanya mentoleransi 1 node mati, sama seperti cluster 3 node; untuk mentoleransi 2 node mati dibutuhkan 5 node. Node keempat hanya menambah biaya dan satu lagi titik yang bisa gagal, dan partition yang membelah cluster jadi 2–2 membuat kedua sisi tidak punya mayoritas sama sekali.

> [!warning] Jebakan
> Mengabaikan konsep term dan menganggap leader lama otomatis "tahu" dirinya sudah digantikan — leader lama yang sempat terputus terus percaya dirinya leader sampai ia menerima pesan dengan term lebih baru; term-lah yang jadi mekanisme eksplisit membuatnya sadar dan mundur.

## Exercises

1. Jelaskan tiga peran node dalam Raft, dan kapan transisi antar peran terjadi.
2. Apa itu term, dan bagaimana ia mencegah leader lama terus mengklaim otoritas setelah cluster memilih leader baru?
3. Kenapa "sudah ada di mayoritas node" belum cukup untuk menjamin entri committed tidak hilang, dan aturan election apa yang melengkapinya?
4. Desain terbuka: kamu diminta merancang sistem penyimpanan konfigurasi terpusat untuk 13 aplikasi yang harus tetap konsisten meski beberapa node mengalami gangguan jaringan sesaat, dan tim mempertimbangkan apakah perlu membangun sistem berbasis Raft sendiri atau memakai tool yang sudah mengimplementasikannya (seperti Consul/etcd). Jelaskan pertimbangan yang menentukan pilihan ini.

> [!success]- Kunci jawaban
> **1.** Follower (pasif, menerima entri dari leader), Candidate (mencalonkan diri jadi leader setelah timeout tanpa heartbeat), Leader (satu-satunya yang menerima tulisan baru dan mengatur replikasi log). Follower jadi Candidate saat timeout; Candidate jadi Leader kalau menang mayoritas suara, atau kembali jadi Follower kalau kalah atau menemukan leader lain yang sah.
> **4.** Kecuali tim punya kebutuhan yang sangat spesifik dan tidak terpenuhi tool yang sudah ada (kasus yang jarang terjadi), memakai tool matang seperti Consul atau etcd yang sudah mengimplementasikan Raft dengan benar hampir selalu pilihan yang lebih baik — mengimplementasikan Raft dari nol dengan benar (menangani seluruh edge case: partition, leader yang mati di tengah replikasi, election yang bersamaan) adalah pekerjaan yang jauh lebih rumit dan berisiko dibanding yang terlihat dari deskripsi konseptualnya, dan kesalahan implementasi consensus bisa menghasilkan bug yang sangat sulit dideteksi (data yang diam-diam tidak konsisten, bukan crash yang jelas terlihat). Investasi tim lebih baik dialokasikan ke integrasi dan operasional tool yang sudah teruji, bukan membangun ulang algoritma consensus yang butuh keahlian sangat mendalam untuk diimplementasikan benar.

## Self-Check

- Sebutkan tiga peran node dalam Raft.
- Apa itu term, dan apa fungsinya?
- Apa itu election restriction, dan kenapa tanpanya entri committed bisa tertimpa?
- Kenapa cluster Raft sebaiknya punya jumlah node ganjil?

## Connected Notes

- [[Quorums]] — mayoritas yang jadi syarat commit di Raft adalah penerapan langsung konsep quorum yang dibahas di note sebelumnya.
- [[Failure Detectors]] — mekanisme timeout yang memicu election di Raft adalah bentuk konkret failure detector yang dibahas sebelumnya.
- [[Consensus - Paxos in Overview]] — kelanjutan langsung: pendahulu Raft yang menyelesaikan masalah sama dengan pendekatan berbeda dan reputasi lebih sulit dipahami.
- [[Leader Election and Split Brain]] — mekanisme term di Raft adalah salah satu cara konkret mencegah split brain yang dibahas mendalam di note berikutnya.
- [[../92 Tools/Consul|Consul]] — tool konkret yang mengimplementasikan Raft untuk menjaga konsistensi data registry-nya.

## Further Reading

- Diego Ongaro dan John Ousterhout, "In Search of an Understandable Consensus Algorithm" (2014) — paper asli Raft, ditulis secara eksplisit untuk lebih mudah dipahami dibanding Paxos, sangat layak dibaca langsung untuk ambisi studi master distributed systems.
- The Raft Consensus Algorithm (raft.github.io) — visualisasi interaktif yang membantu membangun intuisi mekanisme election dan replikasi log.

## Catatan Saya

*Tulis di sini tool berbasis Raft (Consul, etcd, atau lain) yang sudah dipakai di infrastruktur pekerjaanmu, dan apakah kamu pernah mengalami insiden yang berkaitan dengan leader election-nya.*
