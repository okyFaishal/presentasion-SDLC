# Quality & Security Governance: manow-v3-web (Pemecah Kebuntuan Makan Bersama)

- **Versi**: 1.0
- **Status**: Disetujui (Approved)
- **Tanggal**: 2026-09-04
- **Dokumen Induk**: [docs/PRD.md](PRD.md), [docs/SystemSpec.md](SystemSpec.md), & [docs/Architecture.md](Architecture.md)
- **Decision Record**: [docs/decisions/GDR-202609040842.md](decisions/GDR-202609040842.md)

---

## 1. Concurrency, State Safety & Thread Isolation Rules

### 1.1. Penjelasan Konsep Inti (Analogi Sederhana)
Aplikasi ini beroperasi sepenuhnya di dalam satu peramban web ponsel (*single-threaded browser JavaScript event loop*). Bayangkan kasir tunggal di sebuah loket makanan: kasir hanya dapat melayani satu pelanggan pada satu waktu. Masalah timbul bukan karena ada banyak kasir yang saling berebut brankas (seperti pada server multi-core), melainkan ketika dua pelanggan di depan loket yang sama (dua pengguna yang memegang satu ponsel bersamaan) saling berebut memencet tombol menu dalam selisih sekejap mata (*multi-tap race condition*).

### 1.2. Deklarasi Pintu Keluar Dini (Early-Exit N/A)
Berdasarkan arsitektur *pure client-side static PWA* tanpa server backend, parameter konkurensi tingkat server berikut dinyatakan **N/A (Not Applicable)** secara tegas:
- **OS-level Multi-Threading & Mutex/RwLock**: N/A (JavaScript browser tidak mengelola thread kernel CPU secara manual).
- **Distributed Locks**: N/A (Tidak ada kluster server atau sinkronisasi antar mesin).
- **Actor Model Antar-Worker**: N/A (Seluruh komputasi berjalan di tab utama tanpa Web Workers).
- **Database Transaction Isolation Levels**: N/A (Tidak ada SQL database relasional; penyimpanan murni Web Storage lokal).

### 1.3. Aturan Isolasi & Pertahanan Sentuhan Klien
1. **Pertahanan Sentuhan Sinkron Ganda (Multi-Tap Barrier Rule)**:
   - Setiap kartu duel dilindungi oleh kunci logika seketika berbasis referensi:
     ```typescript
     if (isLockedRef.current) return;
     isLockedRef.current = true;
     ```
   - Wadah antarmuka kartu duel wajib membekukan interaksi fisik melalui kelas CSS `pointer-events: none` selama jendela transisi animasi (150 milidetik).
2. **Ketidakterubahan Status Mutlak (Strict State Immutability)**:
   - Seluruh mutasi status turnamen, filter kategori, dan shortlist wajib melalui fungsi reduksi murni (*FSM pure reducer*):
     $$\text{State}_{\text{baru}} = f(\text{State}_{\text{lama}}, \text{Action})$$
   - Dilarang keras melakukan modifikasi di tempat (*in-place mutation*) seperti `state.matches[0].winner = id` atau metode mutasi array langsung (`.push()`, `.splice()`, `.sort()`). Seluruh pembaruan wajib menghasilkan objek baru via *shallow copy* atau operator transformasi murni (`.map()`, `.filter()`).
3. **Protokol Pembersihan Sumber Daya Efek (Mandatory Effect Cleanup Protocol)**:
   - Setiap fungsi efek samping (`useEffect`) yang mendaftarkan pewaktu (`setTimeout`, `setInterval`) atau pendengar peristiwa peramban (`window.addEventListener`) **wajib mengembalikan fungsi pembersih (*cleanup function*)**:
     ```typescript
     useEffect(() => {
       const timerId = window.setTimeout(() => {
         setIsToastVisible(false);
       }, 3500);
       return () => window.clearTimeout(timerId);
     }, []);
     ```
   - Setiap langganan sumber daya wajib lulus uji siklus pasang-bongkar (*mount-unmount*) tanpa meninggalkan kebocoran memori (*zero memory leak*).
4. **Isolasi Memori Penyangga Pembatalan (Undo Buffer Isolation)**:
   - Penyangga pembatalan dibatasi tepat 1 langkah riwayat duel terakhir (`lastMatchSnapshot: TournamentMatch | null`) dengan beban memori terukur $O(1)$.
   - Begitu turnamen masuk ke babak penobatan juara (`CHAMPION`), tombol pembatalan dikunci permanen dan snapshot dihapus menjadi `null` demi kepastian konsensus bersama.

---

## 2. Coding Standards, Anti-Slop Protocols & Linters

### 2.1. Standar Gaya Koding & Protokol Anti-Slop (`anti-slop`)
- **Prinsip Kesederhanaan (KISS & YAGNI)**: Tulis kode paling ringkas yang memenuhi spesifikasi pengujian. Dilarang membuat lapisan abstraksi buatan (seperti *generic factory pattern* atau *abstract base repository*) untuk modul yang hanya memiliki satu implementasi konkret.
- **Eliminasi Komentar Basa-Basi (Zero Trivial Comments)**: Dilarang menulis komentar yang hanya mengulangi apa yang sudah terbaca dari nama fungsi atau variabel (contoh dilarang: `// mengembalikan nama kandidat` di atas baris `return candidate.name;`). Komentar HANYA diizinkan untuk mendokumentasikan alasan non-intuitif (alasan algoritma pembagian *bye* pada angka ganjil atau penanganan batasan peramban ponsel tertentu).
- **Larangan Mock Tiruan Palsu (No Fake Mock Slop)**: Dilarang membuat data tiruan yang selalu sukses tanpa menguji batas kegagalan nyata (seperti pengujian kuota LocalStorage penuh atau format JSON korup).
- **Batasan Kompleksitas**: Fungsi atau metode maksimal 40 baris kode dan berpegang teguh pada prinsip tanggung jawab tunggal (*Single Responsibility Principle*).

### 2.2. Konvensi Penamaan Simbol
| Elemen | Konvensi | Contoh Baku | Keterangan |
|:---|:---|:---|:---|
| **File Komponen React** | `PascalCase.tsx` | `DuelCard.tsx`, `ChampionStage.tsx` | Merepresentasikan elemen visual antarmuka |
| **File Logika / Utilitas** | `kebab-case.ts` | `bracket-engine.ts`, `local-storage-adapter.ts` | Modul domain murni dan adapter |
| **File Pengujian** | `[nama].test.ts(x)` | `bracket-engine.test.ts` | Berdampingan di direktori yang sama atau `tests/` |
| **Interface / Tipe** | `PascalCase` | `Candidate`, `TournamentMatch`, `AppState` | Kontrak tipe data eksplisit TypeScript |
| **Fungsi / Custom Hooks** | `camelCase` | `useTournamentSession()`, `calculateByes()` | Berupa kata kerja aktif |
| **Konstanta / Kunci Enum** | `UPPER_SNAKE_CASE` | `STORAGE_KEY_CANDIDATES`, `COOLDOWN_MS` | Nilai tetap yang tidak dapat diubah |

### 2.3. Matriks Linter, Formatter & Compiler Strict Flags (Terverifikasi Context7)

| Lapisan / Komponen | Perkakas Terpilih | Perintah Eksekusi | Tindakan Pelanggaran |
|:---|:---|:---|:---|
| **Kode Sumber (TS / TSX / CSS)** | **Biome v1.9** | `npx @biomejs/biome check --write .` | **Blokir commit seketika** jika terdapat error linting atau format tidak rapi |
| **Pemeriksaan Tipe Kompilasi** | **TypeScript 5.7+ (`tsc`)** | `npx tsc --noEmit` | **Blokir push seketika** jika ditemukan galat tipe atau pelanggaran mode strict |
| **Audit Pasokan Pustaka** | **npm audit** | `npm audit --omit=dev --audit-level=high` | **Blokir build CI seketika** jika terdeteksi kerentanan High atau Critical |

#### Konfigurasi Pagar Kompiler Ketat (`tsconfig.json`)
Untuk mematikan potensi galat browser klasik `"Cannot read properties of undefined"` saat mengakses bagan turnamen:
```json
{
  "compilerOptions": {
    "target": "ES2022",
    "lib": ["DOM", "DOM.Iterable", "ES2022"],
    "module": "ESNext",
    "moduleResolution": "bundler",
    "jsx": "react-jsx",
    
    /* Strict Type-Checking Guardrails */
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "noUncheckedIndexedAccess": true,
    
    /* Linter & Clean Code Rules */
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "exactOptionalPropertyTypes": true,
    "noFallthroughCasesInSwitch": true,
    "noImplicitReturns": true,
    
    /* Module Integrity */
    "noEmit": true,
    "isolatedModules": true,
    "skipLibCheck": true
  }
}
```

---

## 3. Security, Privacy & Supply Chain Guardrails

### 3.1. Deklarasi Pintu Keluar Dini (Early-Exit N/A)
Karena aplikasi tidak memiliki sistem akun pengguna atau database terpusat:
- **Argon2 / bcrypt Password Hashing**: N/A (Aplikasi tidak mengumpulkan atau menyimpan kredensial/password pengguna).
- **Database Encryption at Rest**: N/A (Tidak ada database server disk).
- **Server Vaults & KMS**: N/A (Tidak ada runtime server yang memegang sertifikat privat).
- **OAuth2 / JWT Bearer Tokens**: N/A (Turnamen beroperasi tanpa autentikasi jaringan).

### 3.2. Pencegahan Suntikan Kode Peramban (DOM-Based XSS Hygiene)
- **Interpolasi Aman Bawaan**: Seluruh render teks masukan nama tempat makan dan tag kategori wajib mengandalkan interpolasi JSX bawaan React 19 (`{candidate.name}`) yang secara otomatis memetakan ke properti `textContent`.
- **Larangan Mutlak Celah Berbahaya**:
  - Dilarang keras menggunakan properti `dangerouslySetInnerHTML`.
  - Dilarang keras menggunakan fungsi eksekusi teks mentah `eval()` atau `new Function()`.
  - Dilarang membuat tautan dengan skema URL berbahaya seperti `javascript:alert(1)`. Seluruh navigasi tombol menggunakan penangan klik internal murni.
  - Aturan linter penegak: `"react/no-danger": "error"` dan `"no-eval": "error"`.

### 3.3. Pagar Pembatas Sumber Daya Ganda (Content Security Policy / CSP)
Untuk mencegah peramban memuat skrip asing tak dikenal, ditegakkan dua lapis perlindungan:
1. **Lapis Utama (HTTP Response Header di `vercel.json`)**:
   ```json
   {
     "headers": [
       {
         "source": "/(.*)",
         "headers": [
           {
             "key": "Content-Security-Policy",
             "value": "default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'; img-src 'self' data:; connect-src 'self'; object-src 'none'; base-uri 'self'; form-action 'self';"
           },
           {
             "key": "X-Content-Type-Options",
             "value": "nosniff"
           },
           {
             "key": "X-Frame-Options",
             "value": "DENY"
           },
           {
             "key": "Referrer-Policy",
             "value": "strict-origin-when-cross-origin"
           },
           {
             "key": "Permissions-Policy",
             "value": "camera=(), microphone=(), geolocation=()"
           }
         ]
       }
     ]
   }
   ```
2. **Lapis Cadangan (Meta Tag di `index.html`)**:
   ```html
   <meta http-equiv="Content-Security-Policy" content="default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'; img-src 'self' data:; connect-src 'self'; object-src 'none'; base-uri 'self';">
   ```

### 3.4. Kebersihan Penyimpanan Lokal & Perlindungan Rahasia (`env-guard`)
- **Kebersihan LocalStorage**: LocalStorage hanya boleh memuat daftar entitas fungsional (`Candidate[]`, `cooldownHistory`, `tournamentSnapshot`). Dilarang keras menaruh data pengenal pelacak pribadi (*zero tracking*).
- **Isolasi Nilai Rahasia**: Dilarang menyisipkan nilai token atau kunci privat pada berkas `.env` klien.
- **Pembersihan Log Produksi (Automated Log Scrubbing)**:
  - Pada lingkungan build produksi, seluruh pemanggilan `console.log` dan `console.debug` wajib dihapus secara otomatis via konfigurasi bundler Vite (`esbuild.drop: ['console', 'debugger']`).
  - Pencatatan kesalahan yang tersisa (`console.warn`, `console.error`) wajib disanitasi menggunakan kode galat terstruktur (`ERR_CODE`) dan token anonim sesi turnamen (`sessionId: UUIDv4`).

### 3.5. Keamanan Rantai Pasok Dependensi (Supply Chain Security)
- **Komitmen Mutlak Lockfile**: Berkas `package-lock.json` wajib selalu di-commit ke repositori Git.
- **Disiplin CI**: Pada pipeline CI/CD, wajib menggunakan `npm ci` (bukan `npm install`) untuk mencocokkan hash pohon dependensi secara identik.
- **Larangan Versi Liar**: Dilarang menggunakan wildcard (`*`) pada `package.json`. Seluruh paket eksternal wajib terkunci pada versi semver definitif.
- **Pintu Gerbang Kerentanan**: Perintah `npm audit --omit=dev --audit-level=high` dijalankan otomatis pada setiap push/build. Jika ditemukan kerentanan berlabel High atau Critical, proses build wajib dibatalkan seketika.

---

## 4. Testing Standards, Quality Gates & Automated Git Hooks

### 4.1. Kewajiban Mutlak TDD (`test-driven-development`)
Setiap pembuatan fitur atau perbaikan bug wajib menaati siklus ketat:
1. **Red**: Tulis berkas pengujian terlebih dahulu dan buktikan gagal di terminal.
2. **Green**: Tulis kode implementasi paling minimal hingga pengujian berubah menjadi lulus.
3. **Refactor**: Bersihkan struktur kode, hapus duplikasi, dan rapikan penamaan tanpa mengubah perilaku yang sudah hijau.

### 4.2. Matriks Piramida Pengujian & Uji Mutasi (Mutation Testing)

| Tingkatan Tes | Cakupan Modul & Sasaran Kode | Porsi Piramida | Target Cakupan (Coverage) | Uji Mutasi (Stryker Score) |
|:---|:---|:---|:---|:---|
| **Unit Tests** | `src/domain/math/`<br>`src/domain/fsm/`<br>`src/domain/errors/` | **~70%** | **100% Cakupan Mutlak** (Line, Branch, Function, Statement) | **$\ge 70\%$ mutan tertangkap** (*killed*) |
| **Integration Tests** | `src/infrastructure/storage/`<br>`src/presentation/hooks/` | **~20%** | **100% Cakupan Mutlak** (Adapter & Custom Hooks) | Evaluasi kontrak & efek samping |
| **End-to-End Tests** | Alur Kritis P0 (US-001 s/d US-007) | **~10%** | **100% Skenario P0 Lulus** | Pengujian alur holistik di peramban |

> **Pengecualian Cakupan Valid**: Berkas titik masuk peramban murni (`src/main.tsx`) dan berkas deklarasi tipe (`src/**/*.d.ts`) dikecualikan dari perhitungan coverage untuk mencegah penulisan tes tiruan palsu (*fake mock slop*).

#### Uji Mutasi Kode (Stryker Mutator Gate)
- Menggunakan `@stryker-mutator/core` dan `@stryker-mutator/vitest-runner` yang difokuskan khusus pada direktori `src/domain/**/*.ts`.
- Jika mutan sengaja disuntikkan ke dalam logika pembagian bagan turnamen namun test suite tetap hijau (*survived mutant*), pengembang wajib menambahkan kasus uji hingga ambang batas kelulusan **$\ge 70\%$** terpenuhi.

### 4.3. Pagar Otomatis Lokal (Pre-Commit & Pre-Push Git Hooks via Lefthook)
Konfigurasi terpusat pada berkas `lefthook.yml`:
- **Pre-Commit Hook** (Latensi <100ms):
  ```yaml
  pre-commit:
    parallel: true
    commands:
      biome-check:
        glob: "*.{js,ts,jsx,tsx,css,json}"
        run: npx @biomejs/biome check --write --staged {staged_files}
        stage_fixed: true
  ```
  *Jika linter menemukan error yang tidak dapat diperbaiki otomatis, proses commit dibatalkan seketika.*

- **Pre-Push Hook**:
  ```yaml
  pre-push:
    commands:
      type-check:
        run: npx tsc --noEmit
      unit-tests:
        run: npx vitest run --coverage
  ```
  *Jika ada 1 tes yang gagal atau pemeriksaan tipe kompilasi meleset, pengiriman branch ke server ditolak otomatis oleh terminal.*

### 4.4. Kriteria Gerbang Kelulusan (Quality Gates & Verification)
Sesuai standar `verification-before-completion`:
- [ ] **Test Suite**: 100% pengujian lulus dengan exit code `0` (`0 failures`, `0 errors`).
- [ ] **Domain & Hook Coverage**: 100% baris kode domain inti dan adapter teruji.
- [ ] **Mutation Score**: $\ge 70\%$ mutan Stryker tereliminasi di `src/domain/`.
- [ ] **Linter & Static Analysis**: Nol lint error atau warning Biome yang diabaikan.
- [ ] **TypeScript Check**: `tsc --noEmit` sukses dengan exit code `0`.
- [ ] **Dependency Audit**: 0 kerentanan High atau Critical.
- [ ] **Evidence**: Bukti eksekusi nyata dari terminal dilampirkan sebelum tugas dinyatakan selesai.

### 4.5. Gerbang Review 2 Lapis (`code-reviewer`)
- **Lapis 1: Kesesuaian Spesifikasi (Spec Compliance)**: Memastikan kode memenuhi 100% kriteria penerimaan Gherkin di `docs/SystemSpec.md` tanpa fitur siluman di luar cakupan (*anti-YAGNI*).
- **Lapis 2: Kualitas Kode & Keamanan (Quality & Security)**: Memeriksa pertahanan kunci sentuhan ganda 150ms, fungsi pembersih efek samping (`cleanup`), kemurnian fungsi FSM reducer, dan tidak adanya kebocoran data di konsol.

---

## 5. Git Workflow, Feature Flags & Rollback Protocol

### 5.1. Alur Percabangan (Branching Strategy)
- **Model Percabangan**: **Trunk-Based Development Hybrid**.
- Pengerjaan tugas dilakukan pada cabang berumur pendek (maksimal 1–2 hari):
  - Fitur: `feat/<nama-fitur>`
  - Perbaikan bug: `fix/<nama-bug>`
  - Pengujian: `test/<cakupan-test>`
  - Refaktor: `refactor/<nama-modul>`
- **Disiplin Penggabungan**: Seluruh cabang fitur wajib digabungkan ke cabang utama (`main`) menggunakan mode **Squash and Merge** agar riwayat rel utama selalu linier, bersih, dan mempermudah pelacakan.

### 5.2. Format Pesan Commit (Conventional Caveman Commits via `git-ops`)
Mengadopsi format padat, hemat ruang baca, dan berorientasi alasan teknis (*Why over What*):
- Format Baku: `<type>(<scope>): <ringkasan imperatif pendek>`
- Batasan: Subjek maksimal 50 karakter, menggunakan huruf kecil setelah titik dua, tanpa titik di akhir kalimat, dan tanpa narasi basa-basi AI.
- Contoh Standar:
  - `feat(domain): add single-elimination bye allocation math`
  - `fix(storage): fallback to memory on quota error`
  - `test(cooldown): verify 24h expiration timestamp`
  - `refactor(fsm): extract pure reducer transitions`
  - `chore(hooks): configure lefthook pre-commit biome`

### 5.3. Saklar Fitur (Feature Flags)
Fitur-fitur yang memiliki potensi risiko interaksi dikontrol melalui saklar konfigurasi lingkungan Vite:
- `VITE_ENABLE_SMART_COOLDOWN=true` (Dapat dimatikan instan jika logika masa istirahat bermasalah).
- `VITE_ENABLE_WEB_SHARE=true` (Dapat dinonaktifkan instan jika API native ponsel memicu kendala peramban).
- Pola konsumsi terisolasi di `src/infrastructure/config/features.ts`.

### 5.4. Prosedur Mundur Cepat (<5 Menit RTO)
1. **Level Infrastruktur CDN (Vercel Instant Rollback)**:
   - Karena aset rilis bersifat *immutable* di Vercel Global Edge, pembatalan rilis bermasalah dilakukan seketika via Dashboard Vercel (menu Deployments > Instant Rollback) atau perintah CLI:
     ```bash
     vercel rollback <deployment-id>
     ```
   - Lalu lintas produksi dialihkan ke versi stabil sebelumnya dalam tempo <5 detik (target RTO < 5 menit tercapai seketika).
2. **Level Source Control (Penyelarasan Repositori Git)**:
   - Segera setelah rollback Vercel aktif, cabang `main` diselaraskan menggunakan perintah pembalik otomatis:
     ```bash
     git revert --no-edit HEAD && git push origin main
     ```
   - Menjamin keselarasan mutlak antara kode di repositori dengan aset yang sedang aktif di peramban pengguna.

### 5.5. Tata Kelola Keputusan Proyek (`decision-recorder`)
Setiap perubahan standar kualitas kode, arsitektur, atau kebijakan keamanan wajib dibukukan ke dalam arsip resmi:
```
docs/decisions/GDR-[YYYYMMDDHHmm].md
```

