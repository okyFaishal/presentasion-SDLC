# Product Requirements Document (PRD): manow-v3-web (Pemecah Kebuntuan Makan Bersama)

- **Versi**: 1.0 (MVP)
- **Status**: Disetujui (Approved)
- **Tanggal**: 2026-09-04
- **Dokumen Induk**: [docs/ProblemFraming.md](ProblemFraming.md)
- **Decision Record**: [docs/decisions/PDR-202609040818.md](decisions/PDR-202609040818.md)

---

## 1. Executive Summary & Visi Produk
**manow-v3-web** adalah aplikasi web instan yang bertindak sebagai "wasit netral" pemutus kebuntuan saat pasangan atau rekan kerja bingung memilih tempat makan siang/malam. Mengubah daftar pilihan yang membingungkan menjadi pertandingan seru 1-lawan-1 (turnamen sistem gugur), aplikasi ini memungkinkan dua orang atau kelompok kecil mengambil keputusan adil di satu ponsel bersama (*hotseat*) dalam waktu kurang dari 60 detik tanpa drama saling lempar kata *"terserah"*, tanpa perlu membuat akun, dan tanpa membebankan rasa bersalah kepada siapa pun.

---

## 2. Latar Belakang Masalah & Target Pengguna
- **Masalah Utama**: Kebuntuan memilih dari daftar yang sudah ada (*shortlist paralysis*) pada kelompok 2 orang atau lebih akibat kecemasan sosial (takut disalahkan jika makanan tidak memuaskan). Cara lama menggunakan roda putar (*wheel*) gagal karena buta situasi (bujet dan cuaca) serta memicu *infinite re-spin*, sehingga waktu terbuang 20–35 menit dan berakhir pasrah di menu yang itu-itu lagi (*default bias*).
- **Target Persona**:
  - Pasangan muda yang lelah berdebat menu makan malam.
  - Rekan kerja kantoran (2–4 orang) yang memiliki jam istirahat makan siang terbatas.
- **Kondisi Penggunaan Utama**:
  - Menggunakan 1 layar ponsel bersama secara bergantian (*pass-the-phone*) saat duduk berdekatan di meja makan atau ruang kerja.

---

## 3. Matriks Cakupan Fitur (Feature Scope Matrix)

### 3.1. P0 — MVP (Wajib Ada di Rilis Pertama)
| ID | Fitur P0 | Deskripsi Singkat | Masalah yang Dijawab (Traceability) | Kriteria Penerimaan Terukur (Definition of Done) | Batasan Fitur (Feature Non-Goals) |
|:---|:---|:---|:---|:---|:---|
| **F-01** | **Manajemen Shortlist Lokal & Paket Preset** | Menambah, mengubah, dan menghapus kandidat tempat makan (nama, tag kategori harga/suasana) yang disimpan di `localStorage`, dilengkapi 2–3 paket preset bawaan. | Menyediakan wadah instan untuk 3–8 tempat langganan tanpa perlu mengetik ulang setiap hari. | - Pengguna baru langsung disuguhi preset aktif.<br>- Operasi tambah/hapus berjalan seketika (<50 ms).<br>- Tersedia tombol reset ke preset bawaan jika data kosong. | Tidak ada sinkronisasi cloud atau akun pengguna. |
| **F-02** | **Saringan Kilat Opsional 1-Ketukan (1-Tap Filter)** | Tombol filter kondisi harian (*"Lagi Hemat"*, *"Mau Berkuah"*, *"Jarak Dekat"*) sebelum turnamen dimulai. | Menjawab kelemahan pengacak biasa yang buta situasi harian (Q5: B ProblemFraming). | - Dapat diabaikan dalam 0 detik jika tidak dibutuhkan.<br>- Jika 0 opsi cocok, muncul saran relaksasi filter instan tanpa memblokir alur turnamen. | Tidak ada form filter berlapis atau input angka manual. |
| **F-03** | **Generator Turnamen 1-lawan-1 & Bye Handling** | Algoritma penyusunan bagan sistem gugur otomatis adaptif untuk jumlah fleksibel ($\ge 2$ opsi, tanpa batas kuota kaku), dengan tiket lolos otomatis (*bye*) bagi jumlah kandidat ganjil. | Mengurangi beban kognitif ekstrem (*Hick's Law*) dengan hanya membandingkan 2 opsi sekaligus. | - Menghasilkan juara dalam tepat $N - 1$ duel.<br>- Kandidat ganjil (misal 3, 5, 7) mendapatkan penempatan *bye* otomatis tanpa merusak bagan. | Tidak ada sistem liga/round-robin atau perhitungan skor probabilitas. |
| **F-04** | **Antarmuka Duel Kartu Ramah Jempol** | Dua kartu vertikal dengan target sentuh besar (minimal 48×48 dp), teks kontras tinggi, mikro-animasi halus (<200 ms), dan tombol pembatalan 1 langkah (*single-step undo*). | Menghilangkan rasa takut salah pencet dan memberikan sensasi kontrol langsung yang memuaskan. | - Umpan balik visual ketukan muncul dalam <100 ms.<br>- Tombol *undo* mengembalikan keadaan duel sebelumnya dengan sempurna.<br>- Animasi dapat dipotong (*interruptible*) jika pengguna mengetuk cepat. | Tidak ada input teks di tengah duel. |
| **F-05** | **Tampilan Juara & Selebrasi Minimalis** | Layar pengumuman pemenang mutlak dengan kartu juara yang jelas, efek selebrasi visual, dan tombol salin/bagikan via *Web Share API*. | Menutup siklus kebingungan (*closure*) dan memudahkan mengabari teman tanpa server. | - Layar juara muncul seketika setelah duel final.<br>- Tombol salin teks atau WhatsApp Share berfungsi via API peramban native. | Tidak ada tombol checkout pemesanan atau pemanggilan kurir. |
| **F-06** | **Masa Istirahat Otomatis (Smart Cooldown Tag)** | Menu pemenang otomatis diberi status istirahat (1–2 hari) dari turnamen berikutnya, dengan tombol buka kunci manual (*unlock toggle*) di shortlist. | Mencegah kebosanan menu berulang hari berikutnya sekaligus memberi kebebasan jika ngidam. | - Menu juara otomatis ditandai `isCooldown: true` dengan timestamp.<br>- Menu yang sedang rehat dapat dibuka kembali dengan 1 ketukan gembok. | Tidak ada penguncian permanen yang melarang pengguna memilih. |
| **F-07** | **Ketahanan Penyimpanan Lokal (Fail-Safe Storage)** | Pembungkus aman `try...catch` untuk seluruh operasi `localStorage` dengan penanganan `QuotaExceededError` dan pemulihan otomatis data korup. | Menjamin aplikasi tidak pernah mengalami layar putih (*white screen of death*) pada peramban apa pun. | - Jika `JSON.parse` gagal, sistem otomatis memulihkan ke preset bawaan.<br>- Bebas eror penyimpanan pada mode penyamaran (*incognito*). | Tidak ada penyimpanan file binary atau gambar berukuran besar. |

### 3.2. P1 — Fase Berikutnya (Should-Have)
- [ ] **Mode Ekspor / Impor Tautan (URL Hash Sharing)**: Membagikan shortlist kandidat ke ponsel teman lain melalui tautan URL yang dikodekan (*base64 state*), tanpa butuh database server.
- [ ] **Kustomisasi Durasi Cooldown**: Pengaturan fleksibel durasi masa istirahat menu (misal: 1 hari, 3 hari, atau 7 hari).
- [ ] **Tema Visual Suasana (Dark / Light Theme)**: Penyesuaian tema visual gelap dan terang berbasis preferensi sistem perangkat.

### 3.3. P2 — Masa Depan (Nice-to-Have / Backlog Panjang)
- [ ] **Multiplayer Online Real-Time (WebSockets / Room Code)**: Sinkronisasi duel di dua ponsel terpisah untuk skenario jarak jauh (LDR / beda ruangan).
- [ ] **Statistik Tren Makanan Pribadi**: Grafik frekuensi kemenangan tempat makan selama 30 hari terakhir.
- [ ] **Integrasi Deep-link Peta / Ojek Online**: Tombol pencarian otomatis ke Google Maps atau aplikasi pengantaran lokal.

---

## 4. Matriks Alur Perjalanan Pengguna (Happy Path vs Edge Cases)

| Tahap Alur | Skenario Normal (Happy Path) | Skenario Gagal / Edge Case (Unhappy Path) | Strategi Pemulihan (Error Recovery) |
|:---|:---|:---|:---|
| **1. Buka Aplikasi & Shortlist** | Pengguna membuka web; daftar shortlist langsung terisi dari data tersimpan atau paket preset bawaan. | Memori peramban kosong, rusak, atau browser berada di mode pemblokiran ketat. | Sistem mendeteksi kegagalan baca dan otomatis memuat paket preset bawaan tanpa membuat aplikasi *crash*. |
| **2. Pengaturan Filter Opsional** | Pengguna mengetuk 1 filter (misal: *"Lagi Hemat"*); kandidat tersaring otomatis menjadi 4 opsi. | Kombinasi filter menghasilkan **0 opsi cocok** di daftar shortlist. | Sistem memunculkan pemberitahuan ramah: *"Belum ada warung yang cocok dengan filter ini"* dengan 1 tombol untuk merelaksasi saringan dan lanjut main. |
| **3. Babak Duel Turnamen** | Dua kartu menu diadu; pengguna bergantian mengetuk pilihan hingga ronde final selesai dalam 30–45 detik. | Pengguna salah memencet kartu lawan karena jempol terpeleset saat mengetuk cepat. | Pengguna dapat menekan tombol **"Batal / Undo"** di sudut layar untuk mengulang duel terakhir tanpa mengulang turnamen dari awal. |
| **4. Penetapan Juara & Selesai** | Pemenang final muncul di panggung juara dengan selebrasi; opsi pemenang otomatis masuk masa rehat 1–2 hari. | Perangkat tidak mendukung *Web Share API* untuk membagikan hasil ke WhatsApp. | Sistem secara elegan beralih ke penyalinan teks biasa (*clipboard copy*) dengan notifikasi: *"Teks hasil berhasil disalin!"*. |

---

## 5. Kebutuhan Non-Fungsional Multi-Dimensi (NFR)

| Dimensi NFR | Spesifikasi & Target Terukur | Strategi Pengujian / Mitigasi |
|:---|:---|:---|
| **Kecepatan Rendering (LCP)** | Largest Contentful Paint **≤ 1.2 detik** pada jaringan 4G seluler (Standar Google: ≤ 2.5s). | Bundel JavaScript statis minimal (<150 KB gzip), tanpa font eksternal berat, rendering murni client-side. |
| **Kelancaran Interaksi (INP)** | Interaction to Next Paint **≤ 80 milidetik** (Standar Google: ≤ 200 ms). | Umpan balik visual langsung via CSS class state tanpa pemblokiran antrean komputasi JS (*no main-thread blocking*). |
| **Kestabilan Tata Letak (CLS)** | Cumulative Layout Shift **= 0.00** (Standar Google: ≤ 0.1). | Seluruh kartu duel dan wadah tombol diberi dimensi ukuran tetap (*fixed aspect ratio/min-height*). |
| **Batas Penyimpanan Web** | Penggunaan memori `localStorage` **≤ 50 KB** (Jauh di bawah kuota peramban 5 MB). | Validasi skema data JSON ketat; data riwayat dibatasi maksimal 20 catatan terakhir. |
| **Keamanan Klien (Client Security)**| Sanitasi teks input pengguna bebas dari celah DOM-based XSS (OWASP). | Teks selalu dirender via API teks aman (`textContent`), dilarang keras menggunakan `innerHTML` atau `eval()`. |
| **Kompatibilitas Perangkat** | Berjalan sempurna di Safari iOS (WebKit), Chrome Android (Blink), dan Desktop modern. | Desain responsif berbasis prinsip *mobile-first* dengan area sentuh minimal 48×48 dp. |

---

## 6. Metrik Keberhasilan (KPIs) & Mitigasi Risiko

### 6.1. Metrik Kunci Produk (KPIs)
| Metrik / KPI | Target Kuantitatif | Cara Mengukur |
|:---|:---|:---|
| **Waktu Hingga Putusan (Time to Decision)** | **< 60 Detik** dari pertama buka aplikasi hingga juara terpilih. | Telemetri waktu lokal (start duel hingga champion screen). |
| **Tingkat Penyelesaian Turnamen (Completion Rate)** | **> 85%** turnamen yang dimulai diselesaikan hingga babak final. | Rasio event `tournament_completed` terhadap `tournament_started`. |
| **Kesiapan Instan (Zero Friction)** | **0 Detik** waktu tunggu registrasi (100% tanpa login). | Audit alur pengguna (tidak ada layar registrasi/pendaftaran). |

### 6.2. Analisis & Mitigasi Risiko
| Potensi Risiko | Tingkat Dampak | Rencana Mitigasi Konkret |
|:---|:---|:---|
| **Jumlah Opsi Terlalu Sedikit (<2 Opsi Aktif)** | Sedang | Tombol mulai duel dinonaktifkan jika opsi aktif kurang dari 2, disertai tombol instan *"Aktifkan Semua Menu"* atau *"Muat Preset"*. |
| **Data LocalStorage Terhapus oleh Sistem iOS** | Rendah | Menggunakan strategi inisialisasi default: jika data hilang, preset langsung dimuat ulang tanpa pesan eror yang membingungkan. |
| **Pengguna Bosan dengan Format Turnamen** | Sedang | Membatasi durasi duel maksimal 3–4 ronde (4–8 opsi) dan menjaga mikro-animasi tetap singkat (<200 ms) agar terasa gesit dan seru. |

---

## 7. Hasil Validasi Konsistensi Hulu & Kesepakatan Dewan (Context Alignment Gate)
- **Kesesuaian Masalah Inti**: [✓] 100% Fitur P0 menjawab langsung akar masalah di [ProblemFraming.md](ProblemFraming.md) (shortlist paralysis & kecemasan sosial).
- **Kepatuhan Batasan Non-Goals**: [✓] Tidak ada fitur yang melanggar daftar *Non-Goals* (tanpa akun/login, tanpa backend database, tanpa live scraping Maps/ojek online, tanpa WebSockets, tanpa checkout).
- **Konsensus Sidang Dewan AI**:
  - Memangkas fitur ekspor/impor ke P1 dan multi-user sync ke P2.
  - Memprioritaskan penanganan *bye* otomatis untuk jumlah opsi ganjil (F-03).
  - Mengintegrasikan tombol *undo* 1 langkah (F-04) dan pemulihan data lokal otomatis (F-07).
- **Keputusan Strategis Pengguna (Hasil Grilling PRD)**:
  1. *Tiket Lolos Otomatis*: Mendukung jumlah pilihan ganjil/fleksibel (Q1: A).
  2. *Smart Cooldown Override*: Istirahat cerdas dengan tombol buka kunci manual (Q2: A).
  3. *Penanganan Saringan Kosong*: Pelonggaran saringan instan 1-ketukan, tidak memblokir pengguna (Q3: A).
  4. *Pemulihan Data Rusak*: Otomatis memuat preset bawaan jika memori kosong/eror (Q4: A).
  5. *Penanganan Salah Pencet*: Tombol *single-step undo* selama turnamen berlangsung (Q5: A).
  6. *Berbagi Juara*: Berbagi ringan via Web Share API native tanpa server (Q6: A).

