# Problem Framing: manow-v3-web (Pemecah Kebuntuan Makan Bersama)

- **Tanggal**: 2026-09-03
- **Status**: Tervalidasi (Validated)
- **Author / Lead**: Pero & Architect
- **Decision Record**: [docs/decisions/PFDR-202609031433.md](decisions/PFDR-202609031433.md)

---

## 1. Executive Problem Statement
Bagi pasangan dan rekan kerja, memutuskan menu makan siang atau malam sering kali bukan karena ketiadaan pilihan, melainkan karena **kebuntuan memilih dari daftar yang sudah ada (shortlist paralysis)** yang diperparah oleh rasa sungkan sosial: semua orang takut disalahkan jika makanannya mengecewakan sehingga berlindung di balik kata *"terserah"*. Tanpa adanya wasit netral yang adil, cepat, dan peka situasi harian, waktu terbuang 20–30 menit hingga kelaparan, dan kelompok akhirnya pasrah kembali memesan makanan yang itu-itu lagi yang membosankan (*default bias*).

---

## 2. Target Persona & Pain Points
- **Target Pengguna**: 
  - Pasangan muda yang lelah berdebat setiap jam makan malam (*"kamu mau makan apa?"* -> *"terserah"*).
  - Rekan kerja kantoran (kelompok 2–4 orang) yang memiliki waktu istirahat terbatas di jam makan siang.
- **Cara Lama yang Melelahkan (Workarounds)**:
  - Saling bertanya secara pasif di ruang obrolan atau di meja makan, saling menunggu siapa yang pertama kali menyebut nama tempat.
  - Membuka aplikasi pesan-antar ojek online lalu *scrolling* katalog panjang tanpa henti, membandingkan puluhan menu saat perut sudah keroncongan.
  - Memutar aplikasi roda putar (*wheel*) yang hasilnya sering ditolak dan diputar ulang (*infinite re-spin*) karena buta konteks (misal: cuaca panas tapi keluar kuah panas).
- **Dampak Kerugian**:
  - Kerugian Waktu: Terbuang 20–35 menit setiap hari (setara >130 jam per tahun) hanya untuk berputar-putar dalam ketidakpastian.
  - Kerugian Emosional: Penurunan suasana hati (*mood erosion*), stres sebelum makan (*hangry*), dan gesekan sosial antar individu.
  - Kerugian Kualitas: Mengorbankan selera dengan pasrah memakan menu yang membosankan dan monoton.

---

## 3. Root Cause Analysis (5-Whys)
1. *Mengapa kelompok bingung menentukan makan apa dari pilihan yang ada?*  
   -> Karena kandidat tempat makan sebenarnya sudah ada (3–8 tempat langganan di sekitar), namun kelompok kesulitan menyepakati 1 opsi akhir secara tegas.
2. *Mengapa kelompok sulit menyepakati 1 opsi dari shortlist tersebut?*  
   -> Karena masing-masing individu memiliki kecemasan sosial (*fear of responsibility* / takut disalahkan) jika pilihannya ternyata mahal, tidak higienis, atau tidak disukai oleh anggota lainnya, sehingga kata *"terserah"* dipakai sebagai mekanisme pertahanan diri.
3. *Mengapa cara acak sederhana (seperti lempar koin atau roda putar) tidak menyelesaikan masalah?*  
   -> Karena roda acak biasa bersifat "buta situasi" (tidak memperhitungkan bujet hari ini, kondisi cuaca, atau pantangan makan) dan tidak membangun komitmen bersama, sehingga hasilnya sering kali diveto atau diputar ulang.
4. *Mengapa situasi ini berdampak buruk bagi pengguna?*  
   -> Karena setelah waktu terbuang lama dalam keraguan dan rasa lapar kian memuncak, kelompok akhirnya menyerah pada opsi *default* yang membosankan hanya demi menyudahi kelelahan mental (*decision fatigue*).
5. *Apa akar masalah fundamentalnya?*  
   -> **Ketiadaan mekanisme wasit netral yang interaktif, adil, peka konteks harian, dan sangat cepat (<60 detik) untuk memfasilitasi eliminasi bersama di satu tempat tanpa membebankan rasa bersalah kepada salah satu individu.**

---

## 4. Boundaries & Scope Constraints

### In-Scope (Fokus Utama MVP):
- **Turnamen Sistem Gugur 1-lawan-1 (Bracket Duel)**: Mekanisme eliminasi berpasangan (A vs B) yang diselesaikan secara visual dan seru di peramban.
- **Mode 1 Ponsel Bersama (Hotseat / Pass-the-Phone)**: Dirancang untuk digunakan bersama di satu layar ponsel tanpa memerlukan kode ruangan (*room code*) atau koneksi internet antar-perangkat.
- **Koleksi Shortlist Mandiri & Paket Preset Cepat**: Pengguna dapat menyimpan 5–10 tempat langganan di memori peramban lokal (*LocalStorage*), atau memilih paket preset bawaan (misal: *"Kuliner Kantor Dekat"*, *"Makan Malam Hemat"*).
- **Saringan Kilat Opsional 1-Ketukan (1-Tap Quick Filter)**: Tombol pra-saring cepat sebelum duel dimulai (misal: *"Lagi Hemat"*, *"Mau Berkuah"*, *"Jarak Dekat"*) untuk menyingkirkan opsi yang tidak relevan dengan situasi hari ini.
- **Masa Istirahat Otomatis (Smart Cooldown Tag)**: Opsi pemenang secara otomatis diistirahatkan (diberi jeda 1–2 hari) dari babak turnamen berikutnya agar menu tidak membosankan dan variatif.
- **Tampilan Juara Minimalis & Bersih (Pure Decider)**: Pengumuman pemenang mutlak dengan selebrasi visual yang jelas tanpa beban tautan luar yang rumit.
- **Arsitektur Web Statis Ringan (Pure Client-Side PWA)**: Loading instan (<200 milidetik), tanpa backend database, tanpa server cost, dan siap diakses kapan saja.

### Non-Goals (Dilarang Dibuat / Batasan Ketat MVP):
- **DILARANG Integrasi API Peta / Ojek Online Eksternal**: Tidak ada pemanggilan berbayar ke Google Places API atau penarikan data dinamis dari GoFood/GrabFood demi mencegah lonjakan biaya, latensi, dan *choice overload* baru.
- **DILARANG Sistem Akun, Login, atau Database Server**: Tidak ada registrasi email, kata sandi, ataupun penyimpanan cloud; seluruh data shortlist disimpan murni di peramban pengguna (*LocalStorage*).
- **DILARANG Fitur Multiplayer Online / WebSockets**: Tidak membuat sistem sinkronisasi online antar-ponsel via server untuk MVP; fokus mutlak pada kemudahan bermain di 1 ponsel bersama (*hotseat*).
- **DILARANG Fitur Transaksi & Checkout**: Tidak memproses pembayaran, pemesanan kurir, atau transaksi finansial apa pun.
- **DILARANG Algoritma Rekomendasi Berat / Black-Box AI**: Tidak menggunakan model rekomendasi machine learning yang rumit; komputasi murni berbasis logika eliminasi matematika deterministik ($O(N)$).

---

## 5. Bukti Empiris & Referensi Industri Terverifikasi
*(Terkumpul dari 3 Agen Inti + 2 Agen Spesialis Dinamis)*

| No | Domain Riset (Agen) | Sumber / Publikasi | URL Referensi | Temuan Kunci / Fakta Empiris |
|:---|:---|:---|:---|:---|
| 1 | Persona & User Pain | Wakefield Research & Factor Meal Survey | [Business Wire – Factor Survey](https://www.businesswire.com/news/home/20230919246101/en/New-Survey-from-Factor-Reveals-Dinner-Decision-Fatigue-is-Real) | 68% orang dewasa menganggap memutuskan makanan adalah hal paling membuat stres; 44% pasangan kesulitan sepakat tiap malam; rata-rata 21–35 menit/hari terbuang sia-sia. |
| 2 | Market & Competitor | Adjust & AppsFlyer Industry Benchmarks | [Adjust Mobile App Trends](https://www.adjust.com) & [AppsFlyer Food & Drink Reports](https://www.appsflyer.com) | Retensi 30 hari (D30) aplikasi pengacak makanan mandiri anjlok di bawah 3% akibat terjebak sifat mainan sesaat (*toy gap*); latensi antarmuka >2 detik memicu 87% pentalan pengguna (*abandonment*). |
| 3 | Tech Feasibility | Single-Elimination Bracket Math (arXiv) | [arXiv:1402.1384 – Tournament Elimination](https://arxiv.org/abs/1402.1384) | Algoritma eliminasi turnamen menjamin 1 pemenang dalam tepat $N - 1$ perbandingan ($O(N)$) dan $\lceil \log_2 N \rceil$ putaran; komputasi selesai di JavaScript browser dalam tempo <0.5 ms tanpa beban server. |
| 4 | Inersia Kebiasaan & UX (Spesialis 1) | The Decision Lab – Decision Fatigue & Friction | [The Decision Lab – Decision Fatigue](https://thedecisionlab.com/biases/decision-fatigue) | Beban pilihan (*choice overload*) melumpuhkan daya kendali otak dan memicu bias status-quo; membatasi menu kurasi ke 3–5 pilihan mempercepat keputusan >30% dan meredam gesekan adopsi. |
| 5 | Benteng Pertahanan / Moat (Spesialis 2) | NFX Defensibility Framework & 7 Powers | [NFX – The Four Types of Defensibility](https://www.nfx.com/post/the-four-types-of-defensibility) | Pengacak acak murni memiliki biaya beralih nol (*zero switching cost*); nilai berkelanjutan (*moat*) diciptakan melalui integrasi riwayat lokal pengguna, saringan konteks situasi, dan fitur anti-kebosanan (*cooldown*). |

---

## 6. Ukuran Keberhasilan (Success Metrics)
- **Metrik Utama (Kecepatan)**: Waktu dari aplikasi pertama kali dibuka hingga 1 pemenang mutlak terpilih adalah **< 60 detik**.
- **Kualitas Interaksi**: Meniadakan dialog pasif saling lempar *"terserah"* dan mencapai konsensus bersama tanpa ketegangan sosial.
- **Kondisi Selesai (Technical Definition of Done)**:
  - Aplikasi web dapat dijalankan 100% secara instan di peramban ponsel tanpa perlu instalasi aplikasi native atau registrasi akun.
  - Saringan 1-ketukan dan turnamen eliminasi berjalan mulus dengan latensi transisi 0 milidetik (*client-side state*).
  - Riwayat menu pemenang tercatat di *LocalStorage* dan otomatis masuk masa tenang (*cooldown*) 1–2 hari.

---

## 7. Hasil Musyawarah Dewan AI & Keputusan Pengguna

### Konsensus Definisi Masalah:
Seluruh 5 penasihat dewan sepakat bahwa produk ini bukan sebuah direktori restoran, melainkan sebuah **"mesin wasit eliminasi cepat"** (*disposable arbitration tool*) yang menghilangkan beban psikologis saling menyalahkan dalam kelompok.

### Titik Buta & Kritik Tajam Dewan:
- *The Skeptic*: Menentang fitur multiplayer online berbasis server karena waktu setup room code akan menghabiskan lebih dari 60 detik.
- *The Tech Feasibility*: Memperingatkan bahaya integrasi API peta pihak ketiga yang mahal, memicu *choice overload*, dan berpotensi diblokir.
- *The Domain Specialist*: Mengingatkan bahwa pengacak acak murni selalu gagal karena buta situasi harian (bujet dan cuaca).

### Keputusan Strategis Pengguna (Hasil Grilling R2):
1. **Mode Bermain**: Mengadopsi mode 1 ponsel bersama (*hotseat*) untuk menjamin kesederhanaan dan kecepatan tanpa dependensi koneksi.
2. **Koleksi Data**: Memilih kurasi mandiri (*shortlist*) dan paket preset lokal, menolak penarikan data otomatis dari peta.
3. **Mekanisme Pemenang**: Memilih turnamen sistem gugur berpasangan (A vs B) yang seru dan adil.
4. **Konteks Harian**: Mengintegrasikan saringan kilat opsional 1-ketukan (1-tap quick filter).
5. **Titik Akhir**: Tampilan juara minimalis dan bersih (*pure decider*) tanpa ketergantungan tautan eksternal.
6. **Variasi Menu**: Mengaktifkan masa istirahat otomatis (*smart cooldown tag*) agar menu pemenang tidak berulang.

