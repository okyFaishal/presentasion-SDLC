# Task Backlog & Phased Execution Plan: manow-v3-web (Pemecah Kebuntuan Makan Bersama)

- **Versi**: 1.0
- **Status**: Disetujui (Approved)
- **Tanggal**: 2026-09-04
- **Dokumen Induk**:
  - [docs/PRD.md](PRD.md)
  - [docs/SystemSpec.md](SystemSpec.md)
  - [docs/Architecture.md](Architecture.md)
  - [docs/Governance.md](Governance.md)
- **Decision Record**: [docs/decisions/TDR-202609040851.md](decisions/TDR-202609040851.md)

---

## 1. Executive Summary & Strategy

Dokumen ini memecah seluruh spesifikasi produk, cetak biru arsitektur, dan aturan tata kelola kualitas `manow-v3-web` ke dalam rencana kerja bertahap (*Phased Execution Plan*) yang terdiri dari **5 Fase Pengerjaan** dan **29 Butir Kartu Tugas Modular**.

Ibarat mendirikan bangunan bertingkat, kita menerapkan strategi **Pondasi Bertahap (*Phased Pipeline*)**:
1. Menyiapkan fondasi kokoh, konfigurasi compiler strict, linter Biome, dan kontrak tipe data bersama (Fase 1).
2. Membangun lapisan penyimpanan lokal yang fail-safe, kebal layar putih, dan kaya data preset (Fase 2).
3. Meracik dapur logika murni matematika bagan turnamen, mesin status FSM, masa istirahat 24 jam, dan pertahanan sentuhan ganda (Fase 3).
4. Merakit antarmuka ramah jempol, kartu duel responsif, panggung juara, dan alur layar (Fase 4).
5. Menguji keselamatan menyeluruh (E2E), menguji ketajaman logika via Stryker mutation testing, dan menyegel bundel rilis PWA berukuran ringan (Fase 5).

Seluruh kartu tugas secara ketat berukuran **S (1–2 berkas, ~100 baris)** atau **M (2–3 berkas, ~200–250 baris)** untuk mencegah kehabisan memori konteks AI (*zero context starvation*). Setiap kartu memisahkan secara eksplisit antara berkas pengujian (`Target Files (Test)`) dan berkas kode (`Target Files (Implementation)`) demi menegakkan hukum besi TDD (*Red-Green-Refactor*).

### Domain Breakdown Tracks:
- 🌐 **Web**: Antarmuka browser mobile-first, kartu duel, bilah filter, panggung juara, custom hooks, dan alur layar.
- 🗄️ **Database**: Adapter LocalStorage fail-safe, migrasi skema migrate-on-read, auto-repair, dan loader paket preset.
- 🛡️ **Security**: Pagar pengunci sentuhan sinkron 150ms (`useRef` lock & CSS `pointer-events: none`) dan audit celah statis.
- 🧩 **Core / Cross-Cutting**: Scaffolding proyek, aturan linter/compiler, kontrak entitas domain, FSM pure reducer, matematika bagan turnamen, dan gerbang rilis produksi.
- ⚙️ **Backend**: *Declared Early-Exit N/A* (Tidak ada server Node.js; sistem murni peramban statis).
- 📱 **Mobile**: *Integrated in Web Track* (Dijalankan sebagai Responsive Mobile-First PWA).

---

## Phase 1: Foundation, Infrastructure & Shared Types
*Tujuan: Menyiapkan pondasi kokoh, perkakas linter Biome v1.9, compiler TypeScript strict flags, test runner Vitest, dan definisi kontrak tipe domain bersama sebelum komponen lain dibangun.*

- [x] **Task 1.1: Project Scaffolding, Package Manifest & Environment Guard** (Domain: Core)
  - **Complexity / Size**: `S (3 files, ~80 lines)`
  - **Depends On**: `None`
  - **Parallel Safe?**: `No (fondasi repositori utama)`
  - **Target Files**:
    - `package.json` (Dependencies & Scripts)
    - `.gitignore` (Security & Ignore Rules)
    - `.env.example` (Environment Guard Template)
  - **Technical Requirements**: Inisialisasi manifest `package.json` berbasis Node.js ESM (`"type": "module"`). Pasang dependensi inti: `react` (^19.0.0), `react-dom` (^19.0.0), `lucide-react`. Pasang dependensi pengembang: `vite` (^6.0.0), `@vitejs/plugin-react`, `tailwindcss` (^4.0.0), `@tailwindcss/vite`, `typescript` (^5.7.0), `@biomejs/biome` (^1.9.0), `vitest`, `jsdom`, `@testing-library/react`, `lefthook`. Daftarkan skrip CLI: `dev`, `build`, `preview`, `lint`, `typecheck`, `test`, `test:coverage`. Pasang pagar keamanan `.gitignore` untuk mencegah kebocoran file credential sesuai prinsip `env-guard`.
  - **Acceptance Criteria & Verification**:
    - [x] `package.json` terkonfigurasi dengan dependensi semver definitif tanpa wildcard `*`.
    - [x] Berkas `.env.example` tersedia tanpa rahasia kredensial nyata.
    - [x] Aturan `.gitignore` melindungi `.env*`, `dist/`, dan `coverage/`.
    - **Verification Command**: `node -e "console.log(require('./package.json').name)"` (Exit Code 0)

- [x] **Task 1.2: Strict Compiler Guardrails, Biome Linter & Git Hooks Setup** (Domain: Core)
  - **Complexity / Size**: `M (3 files, ~120 lines)`
  - **Depends On**: `Task 1.1`
  - **Parallel Safe?**: `No (fondasi pagar kualitas)`
  - **Target Files**:
    - `tsconfig.json` (Compiler Strict Settings)
    - `biome.json` (Linter & Formatter Rules)
    - `lefthook.yml` (Pre-commit & Pre-push Automated Hooks)
  - **Technical Requirements**: Konfigurasi `tsconfig.json` dengan standar strict mutlak sesuai `docs/Governance.md`: `strict: true`, `noImplicitAny: true`, `strictNullChecks: true`, `noUncheckedIndexedAccess: true`, `noUnusedLocals: true`, `noUnusedParameters: true`, `exactOptionalPropertyTypes: true`, `noEmit: true`, `isolatedModules: true`. Konfigurasi `biome.json` v1.9 untuk format dan linter TypeScript/CSS dengan kecepatan <100ms. Konfigurasi `lefthook.yml` untuk pre-commit (Biome staged check) dan pre-push (`tsc --noEmit` & `vitest run`).
  - **Acceptance Criteria & Verification**:
    - [x] `npx tsc --noEmit` mengevaluasi tanpa error konfigurasi.
    - [x] `npx @biomejs/biome check .` lulus dengan 0 error dan 0 warning.
    - [x] Lefthook terkonfigurasi untuk memblokir commit jika linter gagal.
    - **Verification Command**: `npx @biomejs/biome check . && npx tsc --noEmit` (Exit Code 0)

- [x] **Task 1.3: Vitest Test Harness & Tailwind CSS v4 Engine Setup** (Domain: Core)
  - **Complexity / Size**: `M (4 files, ~100 lines)`
  - **Depends On**: `Task 1.1, Task 1.2`
  - **Parallel Safe?**: `No (fondasi pengujian TDD)`
  - **Target Files**:
    - `tests/setup.ts` (Test Setup)
    - `tests/smoke.test.ts` (Test Runner Verification)
    - `vite.config.ts` (Bundler & Test Configuration)
    - `src/index.css` (Tailwind v4 Entrypoint)
  - **Technical Requirements**: Konfigurasi `vite.config.ts` menyertakan plugin `@vitejs/plugin-react` dan `@tailwindcss/vite`. Integrasikan blok pengujian `test` pada `vite.config.ts` dengan `globals: true`, `environment: 'jsdom'`, `setupFiles: ['./tests/setup.ts']`, dan target cakupan 100% untuk domain dan adapter. Di `src/index.css`, cantumkan `@import "tailwindcss";` murni. Buat `tests/smoke.test.ts` untuk memastikan test runner siap mengeksekusi TDD.
  - **Acceptance Criteria & Verification**:
    - [x] Smoke test berjalan sukses di lingkungan jsdom.
    - [x] Tailwind v4 terimpor tanpa membutuhkan berkas konfigurasi lawas.
    - [x] Laporan coverage Vitest aktif mendeteksi pengujian berikutnya.
    - **Verification Command**: `npx vitest run tests/smoke.test.ts` (Exit Code 0)

- [x] **Task 1.4: Standardized Application Error Hierarchy & Result Envelope** (Domain: Core)
  - **Complexity / Size**: `M (3 files, ~180 lines)`
  - **Depends On**: `Task 1.2, Task 1.3`
  - **Parallel Safe?**: `Yes (dapat berjalan bersamaan dengan Task 1.5)`
  - **Target Files**:
    - `tests/domain/errors/app-error.test.ts` (Unit Test)
    - `src/domain/errors/app-error.ts` (Implementation: AppError, DomainError, StorageError)
    - `src/domain/entities/result-envelope.ts` (Implementation: ResultEnvelope, SuccessEnvelope, ErrorEnvelope)
  - **Technical Requirements**: Terapkan TDD: tulis unit test terlebih dahulu. Definisikan kelas dasar `AppError extends Error` dengan atribut `code`, `message`, `statusCode`, dan `details`. Implementasikan `DomainError` dan `StorageError`. Ekspor konstanta error resmi sesuai `docs/SystemSpec.md` Seksi 6 (`ERR_EMPTY_NAME`, `ERR_DUPLICATE_NAME`, `ERR_INSUFFICIENT_CANDIDATES`, dll). Implementasikan modul `result-envelope.ts` sebagai *discriminated union* berbasis `status: "success" | "error"`.
  - **Acceptance Criteria & Verification**:
    - [x] 100% unit test lulus untuk serialisasi error dan pembuatan envelope.
    - [x] Amplop hasil mengembalikan format terstandarisasi tanpa tipe data `any`.
    - **Verification Command**: `npx vitest run tests/domain/errors/app-error.test.ts && npx tsc --noEmit` (Exit Code 0)

- [x] **Task 1.5: Core Candidate, CategoryTag & FilterState Data Contracts** (Domain: Core)
  - **Complexity / Size**: `M (3 files, ~220 lines)`
  - **Depends On**: `Task 1.2, Task 1.3`
  - **Parallel Safe?**: `Yes (dapat berjalan bersamaan dengan Task 1.4)`
  - **Target Files**:
    - `tests/domain/entities/candidate.test.ts` (Unit Test)
    - `src/domain/entities/candidate.ts` (Implementation: Candidate, CandidateStatus, PresetPackage)
    - `src/domain/entities/filter-state.ts` (Implementation: CategoryTag, FilterState)
  - **Technical Requirements**: Terapkan TDD. Definisikan antarmuka `Candidate` sesuai `docs/SystemSpec.md` Seksi 4.2.1 (`id`, `name`, `status`, `isCooldown`, `cooldownUntil`, `categoryTags`, `createdAt`, `updatedAt`). Definisikan antarmuka `PresetPackage` dan `FilterState`. Sediakan helper factory murni `createCandidate` yang memvalidasi nama 1–60 karakter dan batas maksimal 5 tag unik lowercase (`ERR_MAX_TAGS_EXCEEDED`).
  - **Acceptance Criteria & Verification**:
    - [x] Pembuatan kandidat dengan nama kosong atau >60 karakter ditolak validasi.
    - [x] Penambahan tag ke-6 ditolak sesuai batas aturan domain.
    - [x] Tag otomatis dikonversi ke format huruf kecil (*lowercase trimmed*).
    - **Verification Command**: `npx vitest run tests/domain/entities/candidate.test.ts && npx tsc --noEmit` (Exit Code 0)

- [x] **Task 1.6: Tournament Match, Round & Session Domain Contracts** (Domain: Core)
  - **Complexity / Size**: `M (3 files, ~240 lines)`
  - **Depends On**: `Task 1.4, Task 1.5`
  - **Parallel Safe?**: `No (membutuhkan Candidate dan ResultEnvelope)`
  - **Target Files**:
    - `tests/domain/entities/tournament.test.ts` (Unit Test)
    - `src/domain/entities/tournament-match.ts` (Implementation: TournamentMatch, TournamentRound)
    - `src/domain/entities/tournament-session.ts` (Implementation: TournamentSession, CooldownHistory)
  - **Technical Requirements**: Terapkan TDD. Definisikan antarmuka `TournamentMatch` (`id`, `sessionId`, `round`, `matchOrder`, `candidate1Id`, `candidate2Id`, `winnerId`, `nextMatchId`, `isBye`, `status`). Definisikan tipe `TournamentRound`, `TournamentSession`, dan `CooldownHistory`. Sediakan helper type-guard murni `isByeMatch`.
  - **Acceptance Criteria & Verification**:
    - [x] Seluruh atribut entitas turnamen memiliki tipe data konkret dan batasan nullability ketat.
    - [x] Type check `tsc --noEmit` sukses tanpa galat index access.
    - [x] Unit test memvalidasi integritas struktur ronde dan sesi.
    - **Verification Command**: `npx vitest run tests/domain/entities/tournament.test.ts && npx tsc --noEmit` (Exit Code 0)

---

## Phase 2: Core Domain Entities, Schemas & Data Layer
*Tujuan: Membangun fondasi penyimpanan lokal yang fail-safe, pemulihan mandiri saat memori rusak/penuh, rantai migrasi skema migrate-on-read, paket preset makanan bawaan, dan implementasi repository port.*

- [x] **Task 2.1: Fail-Safe LocalStorage Adapter with In-Memory Fallback & Auto-Repair** (Domain: Database)
  - **Complexity / Size**: `S (2 files, ~140 lines)`
  - **Depends On**: `Task 1.4`
  - **Parallel Safe?**: `Yes (terisolasi di adapter storage fisik)`
  - **Target Files**:
    - `tests/infrastructure/storage/local-storage-adapter.test.ts` (Test)
    - `src/infrastructure/storage/local-storage-adapter.ts` (Implementation)
  - **Technical Requirements**: Bangun adapter pembungkus `window.localStorage` dengan proteksi blok `try...catch`. Tangkap galat `QuotaExceededError` (kode 22, 1014, atau nama `QuotaExceededError`) dan secara otomatis beralih ke penyimpanan memori RAM internal (`Map<string, string>`). Tangkap `SyntaxError` akibat JSON korup, kembalikan kode galat `ERR_CORRUPT_STORAGE`, dan bersihkan data rusak secara aman tanpa membuat peramban crash.
  - **Acceptance Criteria & Verification**:
    - [x] Operasi `getItem`, `setItem`, `removeItem`, dan `clear` mengembalikan `ResultEnvelope<T>`.
    - [x] Simulasi `QuotaExceededError` tidak melempar unhandled exception dan sukses beralih ke in-memory cache.
    - [x] Pembacaan JSON korup tertangkap senyap dan mengembalikan error envelope terstruktur.
    - **Verification Command**: `npx vitest run tests/infrastructure/storage/local-storage-adapter.test.ts` (Exit Code 0)

- [x] **Task 2.2: LocalStorage Schema Versioning & Migrate-on-Read Engine** (Domain: Database)
  - **Complexity / Size**: `S (3 files, ~150 lines)`
  - **Depends On**: `Task 2.1`
  - **Parallel Safe?**: `No (membutuhkan interface storage adapter)`
  - **Target Files**:
    - `tests/infrastructure/storage/schema-migrator.test.ts` (Test)
    - `src/infrastructure/storage/schema-migrator.ts` (Implementation)
    - `src/infrastructure/storage/storage-schema.ts` (Types & Migrations)
  - **Technical Requirements**: Implementasikan pembungkus skema data `{ version: number, updatedAt: number, data: T }`. Buat mesin rantai migrasi bertahap (*step-by-step migration chain*) yang mentransformasikan data versi lama ke versi terbaru saat operasi baca (*migrate-on-read*). Sediakan validasi struktur payload sebelum diserahkan ke lapisan repository.
  - **Acceptance Criteria & Verification**:
    - [x] Data tanpa metadata versi otomatis dibungkus menjadi Versi 1.
    - [x] Transformasi bertahap (v1 -> v2) tereksekusi berurutan dan menyimpan kembali versi terbaru ke storage.
    - [x] Data dengan versi masa depan tak dikenal ditolak dengan aman dan diarahkan ke pemulihan fallback.
    - **Verification Command**: `npx vitest run tests/infrastructure/storage/schema-migrator.test.ts` (Exit Code 0)

- [x] **Task 2.3: Candidate Presets Data Fixtures & Preset Loader** (Domain: Database)
  - **Complexity / Size**: `S (3 files, ~160 lines)`
  - **Depends On**: `Task 1.5`
  - **Parallel Safe?**: `Yes (berkas data statis independen)`
  - **Target Files**:
    - `tests/infrastructure/presets/preset-loader.test.ts` (Test)
    - `src/infrastructure/presets/preset-loader.ts` (Implementation)
    - `src/infrastructure/presets/default-presets.ts` (Data Fixtures)
  - **Technical Requirements**: Buat data fixture untuk minimal 3 paket preset makanan lokal:
    1. *"Kuliner Kantor Hemat"* (Default: 5 warung, misal Warung Padang Sederhana, Mie Ayam Jamur, Soto Lamongan Cak Min, Bakso Solo Mas Pur, Nasi Uduk Kebon Kacang bertag 'hemat', 'dekat', 'berkuah').
    2. *"Cepat Saji Populer"* (Burger King Express, Fried Chicken Renyah, Bento Box, Pizza Slice).
    3. *"Pilihan Sehat & Segar"* (Salad Bowl Bar, Sup Ayam Herbal, Gado-Gado Ibu Eni).
    Setiap tempat makan memiliki UUID valid, tag maksimal 5, lowercase, disanitasi, dan status `isCooldown: false`. Buat loader untuk memuat paket terpilih atau paket default.
  - **Acceptance Criteria & Verification**:
    - [x] Paket default "Kuliner Kantor Hemat" memuat tepat 5 kandidat valid.
    - [x] Seluruh data tempat makan lolos validasi aturan domain (nama 1-60 karakter, tag unik <= 5).
    - [x] Fungsi `getPresetById` dan `getDefaultPreset` mengembalikan data utuh dalam amplop `ResultEnvelope`.
    - **Verification Command**: `npx vitest run tests/infrastructure/presets/preset-loader.test.ts` (Exit Code 0)

- [x] **Task 2.4: Local Storage Candidate Repository Implementation (`IStorageRepository`)** (Domain: Database)
  - **Complexity / Size**: `M (2 files, ~260 lines)`
  - **Depends On**: `Task 2.1, Task 2.2, Task 2.3`
  - **Parallel Safe?**: `No (merangkai adapter, migrator, dan preset loader)`
  - **Target Files**:
    - `tests/infrastructure/repositories/local-storage-repository.test.ts` (Test)
    - `src/infrastructure/repositories/local-storage-repository.ts` (Implementation)
  - **Technical Requirements**: Realisasikan antarmuka `IStorageRepository` dari `docs/SystemSpec.md`. Terapkan operasi `getShortlist`, `addCandidate`, `updateCandidate`, `deleteCandidate` (soft delete dengan token undo 3.5s), `undoDeleteCandidate`, `applyPresetPackage`, `unlockCooldown`, dan `resetToDefault`. Terapkan validasi anti-duplikasi nama (case-insensitive & trimmed whitespace). Jika storage kosong atau rusak, otomatis pulihkan ke preset default tanpa throwing error.
  - **Acceptance Criteria & Verification**:
    - [x] `getShortlist` otomatis memuat preset bawaan jika storage kosong atau korup (F-07).
    - [x] `addCandidate` menolak nama duplikat dengan error `ERR_DUPLICATE_NAME`.
    - [x] `deleteCandidate` dan `undoDeleteCandidate` mengembalikan data sesuai kondisi semula.
    - [x] `unlockCooldown` berhasil mengubah status `isCooldown` menjadi `false`.
    - **Verification Command**: `npx vitest run tests/infrastructure/repositories/local-storage-repository.test.ts` (Exit Code 0)

---

## Phase 3: Core Business Logic, IPC/API & Service Engines
*Tujuan: Mengembangkan mesin matematika bagan turnamen sistem gugur, logika istirahat pintar 24 jam, FSM pure reducer dengan undo 1-langkah, dan pertahanan sentuhan ganda 150ms.*

- [x] **Task 3.1: Single-Elimination Bracket Math & Bye Allocation Engine** (Domain: Core)
  - **Complexity / Size**: `S (2 files, ~120 lines)`
  - **Depends On**: `Task 1.6`
  - **Parallel Safe?**: `Yes (modul matematika mandiri)`
  - **Target Files**:
    - `tests/domain/math/bracket-engine.test.ts` (Test)
    - `src/domain/math/bracket-engine.ts` (Implementation)
  - **Technical Requirements**: Menerapkan perhitungan pangkat 2 terdekat: $P = 2^{\lceil \log_2 N \rceil}$, tiket lolos gratis (*byes*) = $P - N$, total ronde = $\lceil \log_2 N \rceil$, dan total pertandingan = $N - 1$. Seluruh tiket *bye* ditempatkan khusus pada Ronde 1 agar Ronde 2 dan seterusnya selalu genap sempurna. Menghasilkan struktur pohon duel (*bracket tree*) yang menghubungkan pemenang duel ke `nextMatchId`. Wajib murni TypeScript tanpa dependensi React/DOM, dan menolak kandidat $< 2$ dengan kode `ERR_INSUFFICIENT_CANDIDATES`.
  - **Acceptance Criteria & Verification**:
    - [x] Algoritma menghasilkan tepat $N - 1$ pertandingan untuk $N$ kandidat (misal: 5 kandidat menghasilkan 4 duel dan 3 tiket bye).
    - [x] Peserta penerima tiket bye otomatis melewati ronde pertama tanpa lawan.
    - [x] Input kurang dari 2 kandidat menghasilkan amplop galat `ERR_INSUFFICIENT_CANDIDATES`.
    - **Verification Command**: `npx vitest run tests/domain/math/bracket-engine.test.ts` (Exit Code 0)

- [x] **Task 3.2: 24-Hour Smart Cooldown & Category Filter Engine** (Domain: Core)
  - **Complexity / Size**: `S (2 files, ~130 lines)`
  - **Depends On**: `Task 1.5`
  - **Parallel Safe?**: `Yes (fungsi kurasi data terisolasi)`
  - **Target Files**:
    - `tests/domain/services/cooldown-filter-engine.test.ts` (Test)
    - `src/domain/services/cooldown-filter-engine.ts` (Implementation)
  - **Technical Requirements**: Mengevaluasi masa rehat: memeriksa apakah `isCooldown: true` dan apakah timestamp istirahat masih berlaku (`currentTime < cooldownUntil`). Mendukung injeksi parameter waktu deterministik (`currentTime: number = Date.now()`) untuk pengujian tanpa flakiness. Menyaring kandidat berdasarkan tag aktif (misal `["hemat", "berkuah"]`). Jika saringan menghasilkan 0 kandidat, kembalikan sinyal ramah `ERR_NO_FILTER_MATCH`. Mendukung pembatalan rehat manual (*manual unlock override*).
  - **Acceptance Criteria & Verification**:
    - [x] Tempat makan yang memenangkan sesi otomatis diistirahatkan selama 24 jam penuh dari turnamen berikutnya.
    - [x] Tempat makan dengan masa istirahat kadaluarsa ($\ge 24$ jam) otomatis dipulihkan menjadi aktif kembali.
    - [x] Penyaringan dengan multi-tag berjalan akurat; jika hasil kosong, mengembalikan kode `ERR_NO_FILTER_MATCH` tanpa crash.
    - [x] Pembukaan kunci manual berhasil mengembalikan kandidat ke status `ACTIVE`.
    - **Verification Command**: `npx vitest run tests/domain/services/cooldown-filter-engine.test.ts` (Exit Code 0)

- [x] **Task 3.3: Tournament FSM Pure Reducer & Deterministic 1-Step Undo Buffer** (Domain: Core)
  - **Complexity / Size**: `M (2 files, ~230 lines)`
  - **Depends On**: `Task 3.1`
  - **Parallel Safe?**: `No (membutuhkan generator bagan dari Task 3.1)`
  - **Target Files**:
    - `tests/domain/fsm/tournament-reducer.test.ts` (Test)
    - `src/domain/fsm/tournament-reducer.ts` (Implementation)
  - **Technical Requirements**: Menerapkan fungsi murni `(state: TournamentState, action: TournamentAction): TournamentState` dengan pendekatan *Discriminated Unions*. Menangani aksi: `START_TOURNAMENT`, `PICK_WINNER`, `UNDO_LAST_PICK`, `FINALIZE_SESSION`, dan `ABORT_TOURNAMENT`. Pada aksi `PICK_WINNER`, alokasikan pemenang ke slot `nextMatchId` dan simpan snapshot 1 langkah sebelumnya di memori buffer ($O(1)$). Tiket *bye* diselesaikan secara otomatis. Pada aksi `UNDO_LAST_PICK`, pulihkan status ke duel sebelum ketukan terakhir. Begitu masuk ke babak juara final (`CHAMPION`), tombol pembatalan dikunci permanen (`undoAvailable: false`) dan snapshot dihapus menjadi `null`. Dilarang keras melakukan in-place mutation.
  - **Acceptance Criteria & Verification**:
    - [x] Seluruh transisi status menghasilkan objek state baru (*pure immutable update*).
    - [x] Mengetuk tombol *undo* pada duel penyisihan berhasil mengembalikan status pertandingan sebelumnya.
    - [x] Pada babak final, penobatan juara mengubah status menjadi `COMPLETED` dan mengunci tombol *undo* permanen.
    - [x] Upaya undo pada sesi tanpa riwayat mengembalikan kode `ERR_UNDO_NOT_AVAILABLE` atau mengabaikan aksi secara aman.
    - **Verification Command**: `npx vitest run tests/domain/fsm/tournament-reducer.test.ts` (Exit Code 0)

- [x] **Task 3.4: Multi-Tap 150ms Lock Barrier & Tap Defense Guard** (Domain: Security)
  - **Complexity / Size**: `S (2 files, ~110 lines)`
  - **Depends On**: `Task 1.2`
  - **Parallel Safe?**: `Yes (hook pertahanan interaksi terisolasi)`
  - **Target Files**:
    - `tests/presentation/hooks/use-touch-lock.test.ts` (Test)
    - `src/presentation/hooks/use-touch-lock.ts` (Implementation)
  - **Technical Requirements**: Membangun hook pertahanan sentuhan ganda menggunakan referensi sinkron `isLockedRef.current` yang langsung bernilai `true` saat sentuhan pertama diterima (latensi 0ms). Mengabaikan ketukan susulan dalam rentang jendela 150 milidetik. Menyediakan penanda status fisik untuk kelas CSS `pointer-events: none` selama animasi berjalan. Wajib menyertakan fungsi pembersih pewaktu (`clearTimeout`) saat unmount untuk mencegah kebocoran memori.
  - **Acceptance Criteria & Verification**:
    - [x] Ketukan kedua dan ketiga dalam selang waktu <150ms diabaikan secara mutlak (hanya 1 aksi dieksekusi).
    - [x] Kunci terbuka kembali secara otomatis setelah 150ms berlalu.
    - [x] Fungsi pembersih pewaktu terbukti dipanggil saat unmount tanpa peringatan memory leak.
    - **Verification Command**: `npx vitest run tests/presentation/hooks/use-touch-lock.test.ts` (Exit Code 0)

---

## Phase 4: Feature Modules, UI/Client Workflows & Integration
*Tujuan: Membangun custom hooks pengendali alur, komponen visual ramah jempol, arena duel 1-lawan-1 berpelindung sentuhan ganda, panggung juara, toast notifikasi, dan orkestrasi layar antarmuka.*

- [x] **Task 4.1: Custom Hook `useShortlist` & Candidate Management** (Domain: Web)
  - **Depends On**: `Task 2.4`
  - **Parallel Safe?**: `Yes (terisolasi di hook shortlist)`
  - **Target Files**:
    - `tests/presentation/hooks/useShortlist.test.ts` (Test)
    - `src/presentation/hooks/useShortlist.ts` (Implementation)
  - **Technical Requirements**: Bangun hook pengelolaan kandidat tempat makan yang menghubungkan UI ke `IStorageRepository`. Mendukung pembacaan daftar tempat makan, penambahan dengan validasi anti-duplikasi nama (case-insensitive) dan batas maksimal 5 tag, soft-delete dengan pengembalian token undo 3.5s, buka kunci istirahat manual (`unlockCooldown`), dan reset ke preset bawaan.
  - **Acceptance Criteria & Verification**:
    - [x] `addCandidate` menolak nama yang sudah terdaftar dengan kode `ERR_DUPLICATE_NAME`.
    - [x] `deleteCandidate` mengembalikan `undoToken` dan memicu timer undo 3.5 detik.
    - [x] `undoDeleteCandidate` memulihkan kandidat ke urutan semula sebelum timer habis.
    - [x] `unlockCooldown` mengubah status `isCooldown` menjadi `false`.
    - **Verification Command**: `npx vitest run tests/presentation/hooks/useShortlist.test.ts` (Exit Code 0)
  - **Parallel Safe?**: `No (bergantung pada FSM reducer dan useTouchLock)`
  - **Target Files**:
    - `tests/presentation/hooks/useTournamentSession.test.ts` (Test)
    - `src/presentation/hooks/useTournamentSession.ts` (Implementation)
  - **Technical Requirements**: Bangun hook orkestrasi sesi turnamen yang menghubungkan UI ke mesin turnamen domain. Mengelola inisialisasi turnamen dari kandidat aktif/tersaring, mengambil duel aktif saat ini (`currentMatch`), mencatat kemenangan dengan perlindungan kunci sentuhan `useTouchLock` 150ms, mengelola penyangga pembatalan satu langkah (`single-step undo`), dan mengambil detail juara mutlak saat babak final selesai. Tombol undo dinonaktifkan mutlak saat status mencapai `COMPLETED`.
  - **Acceptance Criteria & Verification**:
    - [x] `startTournament` gagal jika jumlah kandidat aktif kurang dari 2 (`ERR_INSUFFICIENT_CANDIDATES`).
    - [x] `pickWinner` memajukan ronde turnamen dan mengaktifkan status pembatalan (`canUndo: true`).
    - [x] `undoLastMatch` mengembalikan pertandingan duel sebelumnya.
    - [x] Saat babak final selesai, `status` menjadi `COMPLETED` dan `canUndo` terkunci menjadi `false`.
    - **Verification Command**: `npx vitest run tests/presentation/hooks/useTournamentSession.test.ts` (Exit Code 0)

- [x] **Task 4.3: 1-Tap Category Filter Bar Component** (Domain: Web)
  - **Complexity / Size**: `S (2 files, ~120 lines)`
  - **Depends On**: `Task 1.5`
  - **Parallel Safe?**: `Yes (komponen visual independen)`
  - **Target Files**:
    - `tests/presentation/components/FilterBar.test.tsx` (Test)
    - `src/presentation/components/FilterBar.tsx` (Implementation)
  - **Technical Requirements**: Bangun komponen bilah saringan horizontal 1-ketukan berbasis tombol pil ramah jempol (tinggi minimal 48dp). Mendukung tombol saringan kondisi harian (*"Lagi Hemat"*, *"Mau Berkuah"*, *"Jarak Dekat"*). Saat saringan menghasilkan 0 kandidat cocok, tampilkan kotak pemberitahuan ramah dengan tombol relaksasi instan *"Matikan Saringan & Mainkan Semua"*.
  - **Acceptance Criteria & Verification**:
    - [x] Mengetuk tombol filter mengubah status visual menjadi aktif dalam tempo <100ms.
    - [x] Mengetuk ulang filter yang aktif mematikan saringan (toggle).
    - [x] Jika kandidat lolos berjumlah 0, tombol relaksasi muncul dan dapat diketuk untuk mereset seluruh filter.
    - [x] Area target sentuh seluruh pil tombol terverifikasi $\ge 48\times 48\text{ dp}$.
    - **Verification Command**: `npx vitest run tests/presentation/components/FilterBar.test.tsx` (Exit Code 0)

- [x] **Task 4.4: Candidate Item & Shortlist View Component** (Domain: Web)
  - **Complexity / Size**: `M (2 files, ~260 lines)`
  - **Depends On**: `Task 4.1, Task 4.3`
  - **Parallel Safe?**: `No (mengintegrasikan useShortlist & FilterBar)`
  - **Target Files**:
    - `tests/presentation/components/ShortlistView.test.tsx` (Test)
    - `src/presentation/components/ShortlistView.tsx` (Implementation)
  - **Technical Requirements**: Bangun tampilan utama shortlist tempat makan. Menampilkan daftar tempat makan aktif, badge status istirahat (*cooldown badge*) dengan ikon gembok yang dapat diketuk untuk membuka kunci manual, input penambahan tempat makan baru (maksimal 60 karakter disanitasi), tombol hapus tiap baris, tombol muat ulang preset bawaan, dan tombol utama *"Mulai Duel"* yang nonaktif jika opsi aktif kurang dari 2.
  - **Acceptance Criteria & Verification**:
    - [x] Daftar tempat makan ter-render lengkap dengan tag kategori masing-masing.
    - [x] Kandidat dalam masa istirahat menampilkan ikon gembok dan waktu rehat; ketukan membuka kunci memicu fungsi unlock.
    - [x] Tombol *"Mulai Duel"* berstatus nonaktif saat kandidat aktif <2 dan aktif saat $\ge 2$.
    - [x] Teks masukan nama tempat makan terisolasi bebas dari celah DOM XSS (`textContent`).
    - **Verification Command**: `npx vitest run tests/presentation/components/ShortlistView.test.tsx` (Exit Code 0)

- [x] **Task 4.5: Thumb-Friendly Duel Card Component** (Domain: Web)
  - **Complexity / Size**: `S (2 files, ~110 lines)`
  - **Depends On**: `Task 1.5`
  - **Parallel Safe?**: `Yes (komponen visual murni)`
  - **Target Files**:
    - `tests/presentation/components/DuelCard.test.tsx` (Test)
    - `src/presentation/components/DuelCard.tsx` (Implementation)
  - **Technical Requirements**: Bangun komponen kartu duel individual berorientasi vertikal ramah jempol. Kartu memiliki tinggi minimal 140px, target sentuh luas, teks nama tempat makan berukuran besar kontras tinggi (WCAG AAA), pil tag kategori di sudut kartu, dan umpan balik ketukan visual instan (`active:scale-98 transition-transform duration-100`). Bebas dari atribut teks berbahaya.
  - **Acceptance Criteria & Verification**:
    - [x] Kartu menampilkan nama warung dan daftar tag kategori dengan benar.
    - [x] Sentuhan/klik pada kartu memicu callback `onSelect` dengan ID kandidat yang benar.
    - [x] Memiliki kelas utilitas Tailwind target sentuh minimum $\ge 48\times 48\text{ dp}$.
    - **Verification Command**: `npx vitest run tests/presentation/components/DuelCard.test.tsx` (Exit Code 0)

- [x] **Task 4.6: Duel Arena & Fast-Tap Lock Barrier Component** (Domain: Web)
  - **Complexity / Size**: `M (2 files, ~230 lines)`
  - **Depends On**: `Task 4.2, Task 4.5`
  - **Parallel Safe?**: `No (mengintegrasikan DuelCard & useTournamentSession)`
  - **Target Files**:
    - `tests/presentation/components/DuelArena.test.tsx` (Test)
    - `src/presentation/components/DuelArena.tsx` (Implementation)
  - **Technical Requirements**: Bangun panggung pertandingan duel 1-lawan-1 yang merender dua `DuelCard` saling berhadapan (atas vs bawah). Dilengkapi bilah progres babak ("Ronde X - Pertandingan Y dari Z"), tombol pembatalan satu langkah (*Single-Step Undo*) di sudut layar, dan pembungkus fisik CSS `pointer-events: none` selama jendela transisi mikro 150ms untuk mematikan ketukan ganda fisik.
  - **Acceptance Criteria & Verification**:
    - [x] Menampilkan dua kandidat yang sedang bertanding di ronde aktif.
    - [x] Mengetuk salah satu kartu memicu pergantian duel dan mengunci pointer selama 150 milidetik.
    - [x] Tombol "Batal / Undo" berstatus nonaktif saat belum ada duel selesai dan aktif saat snapshot tersedia.
    - [x] Mengetuk tombol "Batal / Undo" membatalkan hasil duel terakhir dan mengembalikan lawan sebelumnya.
    - **Verification Command**: `npx vitest run tests/presentation/components/DuelArena.test.tsx` (Exit Code 0)

- [x] **Task 4.7: Champion Stage & Celebration View Component** (Domain: Web)
  - **Complexity / Size**: `S (2 files, ~130 lines)`
  - **Depends On**: `Task 1.5`
  - **Parallel Safe?**: `Yes (komponen tampilan mandiri)`
  - **Target Files**:
    - `tests/presentation/components/ChampionStage.test.tsx` (Test)
    - `src/presentation/components/ChampionStage.tsx` (Implementation)
  - **Technical Requirements**: Bangun panggung selebrasi pemenang mutlak turnamen. Menampilkan kartu juara berbingkai emas/aksen tegas, ikon mahkota juara dengan mikro-animasi denyut halus (*subtle pulse*), informasi masa istirahat 24 jam yang otomatis berlaku, tombol tindakan *"Bagikan Hasil"* (memicu Web Share / Clipboard), dan tombol *"Mulai Turnamen Baru"*. Tombol undo ditiadakan permanen di layar ini.
  - **Acceptance Criteria & Verification**:
    - [x] Nama pemenang juara dan tag kategorinya tampil secara dominan dan jelas.
    - [x] Informasi durasi masa istirahat 24 jam tertera dengan ramah.
    - [x] Tombol "Bagikan Hasil" memicu callback `onShare`.
    - [x] Tombol "Mulai Turnamen Baru" memicu callback `onReset` untuk kembali ke shortlist.
    - **Verification Command**: `npx vitest run tests/presentation/components/ChampionStage.test.tsx` (Exit Code 0)

- [x] **Task 4.8: Soft-Delete Undo Toast Component** (Domain: Web)
  - **Complexity / Size**: `S (2 files, ~90 lines)`
  - **Depends On**: `None`
  - **Parallel Safe?**: `Yes (komponen notifikasi mandiri)`
  - **Target Files**:
    - `tests/presentation/components/UndoToast.test.tsx` (Test)
    - `src/presentation/components/UndoToast.tsx` (Implementation)
  - **Technical Requirements**: Bangun komponen notifikasi mengambang (*floating toast*) saat pengguna menghapus tempat makan dari shortlist. Menampilkan teks *"Tempat makan dihapus"*, tombol *"Batalkan"* berarea sentuh jempol $\ge 48\times 48\text{ dp}$, bilah progres hitung mundur 3.5 detik, dan pembersihan timer otomatis saat waktu habis atau saat tombol batal ditekan.
  - **Acceptance Criteria & Verification**:
    - [x] Toast muncul di bagian bawah layar saat dipicu dan hilang setelah 3.5 detik.
    - [x] Menekan tombol "Batalkan" memicu callback `onUndo` dan langsung menutup toast.
    - [x] Fungsi pembersih timer teruji mencegah eksekusi state update setelah unmount.
    - **Verification Command**: `npx vitest run tests/presentation/components/UndoToast.test.tsx` (Exit Code 0)

- [x] **Task 4.9: Web Share & Clipboard Fallback Toast Component** (Domain: Web)
  - **Complexity / Size**: `S (2 files, ~100 lines)`
  - **Depends On**: `None`
  - **Parallel Safe?**: `Yes (komponen feedback mandiri)`
  - **Target Files**:
    - `tests/presentation/components/ShareToast.test.tsx` (Test)
    - `src/presentation/components/ShareToast.tsx` (Implementation)
  - **Technical Requirements**: Bangun komponen penyaji hasil juara ke media sosial atau clipboard. Menangani debouncing ketukan ganda pada tombol bagikan, memanggil Web Share API native (`navigator.share`), beralih elegan ke `navigator.clipboard.writeText` jika share tidak didukung, dan menampilkan toast konfirmasi ramah (*"Teks juara berhasil disalin ke papan klip!"*) selama 3 detik.
  - **Acceptance Criteria & Verification**:
    - [x] Ketukan cepat beruntun dalam 200ms hanya memicu operasi berbagi tepat 1 kali.
    - [x] Saat metode adalah clipboard, toast konfirmasi sukses muncul di layar selama 3 detik.
    - [x] Penanganan pembatalan berbagi (*share cancellation*) ditangani senyap tanpa eror di konsol.
    - **Verification Command**: `npx vitest run tests/presentation/components/ShareToast.test.tsx` (Exit Code 0)

- [x] **Task 4.10: Browser Native Share & Clipboard Adapter** (Domain: Web)
  - **Complexity / Size**: `S (2 files, ~90 lines)`
  - **Depends On**: `Task 1.4`
  - **Parallel Safe?**: `Yes (adapter peramban luar)`
  - **Target Files**:
    - `tests/infrastructure/share/browser-share-adapter.test.ts` (Test)
    - `src/infrastructure/share/browser-share-adapter.ts` (Implementation)
  - **Technical Requirements**: Realisasikan antarmuka `IShareService` dari `docs/SystemSpec.md`. Periksa dukungan `navigator.share` dan `navigator.canShare`. Jika tersedia, panggil share API dengan teks format baku turnamen. Jika tidak didukung atau berada dalam konteks non-HTTPS, alihkan ke `navigator.clipboard.writeText` dan kembalikan penanda fallback pada amplop hasil.
  - **Acceptance Criteria & Verification**:
    - [x] Mengembalikan status sukses saat `navigator.share` berhasil.
    - [x] Otomatis beralih ke Clipboard API saat `navigator.share` tidak tersedia.
    - [x] Menangani penolakan pengguna (*AbortError*) dengan status ramah pengguna.
    - **Verification Command**: `npx vitest run tests/infrastructure/share/browser-share-adapter.test.ts` (Exit Code 0)

- [x] **Task 4.11: App Shell & Screen Transition Orchestrator** (Domain: Web)
  - **Complexity / Size**: `M (2 files, ~240 lines)`
  - **Depends On**: `Task 4.4, Task 4.6, Task 4.7, Task 4.8, Task 4.9, Task 4.10`
  - **Parallel Safe?**: `No (perakit akhir lapisan antarmuka)`
  - **Target Files**:
    - `tests/presentation/App.test.tsx` (Test)
    - `src/presentation/App.tsx` (Implementation)
  - **Technical Requirements**: Bangun komponen induk `App` yang merakit seluruh alur layar berdasarkan status sesi FSM: layar shortlist (`DRAFT`), arena pertandingan (`RUNNING`), dan panggung selebrasi juara (`COMPLETED`). Menjamin transisi layar mulus tanpa pergeseran tata letak (*Cumulative Layout Shift = 0.00*), menyematkan wadah toast global, dan menerapkan batasan lebar kontainer mobile-first (`max-w-md mx-auto min-h-screen`).
  - **Acceptance Criteria & Verification**:
    - [x] Alur transisi dari shortlist ke arena duel dan ke panggung juara berjalan mulus sesuai state FSM.
    - [x] Seluruh komponen toast (Undo Toast dan Share Toast) dapat dirender tanpa merusak layout duel.
    - [x] Tata letak terpusat di layar ponsel dengan aspek rasio stabil (CLS = 0.00).
    - **Verification Command**: `npx vitest run tests/presentation/App.test.tsx` (Exit Code 0)

---

## Phase 5: E2E Verification, Security Audit & Release Polish
*Tujuan: Memvalidasi alur pengguna holistik (US-001 s/d US-007), menjamin ketangguhan logika domain dengan uji mutasi >=70%, memastikan nol celah keamanan dependensi, serta memvalidasi kesiapan rilis produksi PWA offline dengan ukuran bundel gzip <= 150 KB.*

- [x] **Task 5.1: End-to-End (E2E) Critical Flow Integration Suite (US-001 s/d US-007)** (Domain: Web)
  - **Complexity / Size**: `M (3 files, ~260 lines)`
  - **Depends On**: `Task 4.11`
  - **Parallel Safe?**: `No (pengujian alur holistik merangkai seluruh modul)`
  - **Target Files**:
    - `tests/e2e/tournament_critical_flow.e2e.test.tsx` (Test)
    - `tests/e2e/storage_recovery_cooldown.e2e.test.tsx` (Test)
    - `src/presentation/App.tsx` (Implementation / Wiring Integration)
  - **Technical Requirements**: Bangun rangkaian pengujian integrasi peramban hulu-ke-hilir yang mencakup 7 alur cerita pengguna secara terpadu:
    1. Inisialisasi shortlist dari preset bawaan dan operasi tambah/hapus dengan notifikasi pembatalan (*undo toast*) 3.5 detik (US-001).
    2. Penerapan filter 1-ketuk dan dialog pemulihan instan jika hasil filter kosong (US-002).
    3. Eksekusi generator bagan duel 1-lawan-1 untuk peserta genap maupun ganjil (*bye handling*) (US-003).
    4. Interaksi ketukan kartu duel dengan kunci sentuhan ganda 150ms (`pointer-events: none` & `useRef` lock) dan fungsionalitas pembatalan 1 langkah (*single-step undo*) (US-004).
    5. Layar selebrasi juara, penguncian permanen tombol pembatalan, pemanggilan Web Share API native dan beralih otomatis ke Clipboard API jika tidak didukung (US-005).
    6. Penandaan status istirahat otomatis 24 jam (*smart cooldown*) pada menu juara dan pengujian buka kunci manual (*unlock toggle*) (US-006).
    7. Ketahanan penyimpanan lokal: pemulihan otomatis saat data JSON korup dan penanganan kuota penyimpanan penuh (*QuotaExceededError*) (US-007).
  - **Acceptance Criteria & Verification**:
    - [x] 100% skenario US-001 s/d US-007 teruji lulus tanpa error.
    - [x] Pengujian menangani transisi state asinkron secara deterministik tanpa kebocoran pewaktu (*timer leak*).
    - [x] Bebas dari pesan galat peramban atau *unhandled rejection* di konsol pengujian.
    - **Verification Command**: `npx vitest run tests/e2e/` (Exit Code 0, 0 failures)

- [ ] **Task 5.2: Stryker Mutation Testing Gate for Domain Core (`src/domain/`)** (Domain: Core)
  - **Complexity / Size**: `S (2 files, ~90 lines)`
  - **Depends On**: `Task 3.1, Task 3.2, Task 3.3`
  - **Parallel Safe?**: `Yes (fokus terisolasi pada folder src/domain/)`
  - **Target Files**:
    - `stryker.config.json` (Configuration)
    - `tests/domain/mutation_guard.test.ts` (Test / Boundary Assertions)
  - **Technical Requirements**: Konfigurasikan perkakas uji mutasi `@stryker-mutator/core` dengan pelari `@stryker-mutator/vitest-runner`. Batasi target mutan secara ketat pada modul logika murni `src/domain/**/*.ts` (`math/`, `fsm/`, `entities/`, `errors/`). Tetapkan ambang batas kelulusan `thresholds: { high: 80, low: 70, break: 70 }`. Jika mutan yang disuntikkan pada pembagian bagan atau reduksi status bertahan hidup (*survived*), lengkapi kasus uji batas pada `mutation_guard.test.ts` hingga skor mutasi $\ge 70\%$.
  - **Acceptance Criteria & Verification**:
    - [ ] Berkas konfigurasi `stryker.config.json` terintegrasi mulus dengan konfigurasi Vitest proyek.
    - [ ] Skor pengujian mutasi Stryker pada direktori `src/domain/` mencapai $\ge 70\%$.
    - [ ] Seluruh mutan kritis pada algoritma alokasi *bye* dan mesin status berhasil dieliminasi (*killed*).
    - **Verification Command**: `npx stryker run` (Exit Code 0, Mutation score >= 70%)

- [x] **Task 5.3: Dependency Vulnerability, Secret Leak & Static Security Audit** (Domain: Security)
  - **Complexity / Size**: `S (2 files, ~80 lines)`
  - **Depends On**: `Task 1.1`
  - **Parallel Safe?**: `Yes (audit statis independen)`
  - **Target Files**:
    - `scripts/security-audit.mjs` (Implementation / Audit Script)
    - `tests/security/security_audit.test.ts` (Test)
  - **Technical Requirements**: Buat skrip audit keamanan terpusat untuk memverifikasi tiga pagar pembatas:
    1. Audit rantai pasok dependensi: Jalankan pemeriksaan paket produksi tanpa celah berlabel *High* atau *Critical*.
    2. Audit kebocoran rahasia (`env-guard`): Pindai seluruh repositori untuk memastikan tidak ada kunci privat, token otentikasi, atau kredensial yang bocor di kode sumber maupun `.env`.
    3. Sanitasi celah DOM XSS: Pindai seluruh kode antarmuka `src/` untuk memastikan tidak ada pemanggilan fungsi berbahaya seperti `dangerouslySetInnerHTML`, `eval()`, atau skema tautan `javascript:`.
  - **Acceptance Criteria & Verification**:
    - [x] Laporan `npm audit` menunjukkan 0 kerentanan High atau Critical.
    - [x] Skrip audit memastikan nol kebocoran secret pada berkas lingkungan.
    - [x] Skrip memastikan sanitasi XSS terpenuhi 100% pada kode komponen.
    - **Verification Command**: `npm audit --omit=dev --audit-level=high && node scripts/security-audit.mjs` (Exit Code 0)

- [x] **Task 5.4: PWA Offline Service Worker, CSP Verification & Production Bundle Sizing Gate** (Domain: Core)
  - **Complexity / Size**: `M (3 files, ~180 lines)`
  - **Depends On**: `Task 5.1, Task 5.2, Task 5.3`
  - **Parallel Safe?**: `No (gerbang kompilasi rilis final)`
  - **Target Files**:
    - `scripts/verify-bundle-size.mjs` (Implementation / Bundle Gate Script)
    - `vercel.json` (Configuration / CSP Headers)
    - `vite.config.ts` (Configuration / PWA & Rollup Optimization)
  - **Technical Requirements**: Siapkan konfigurasi rilis produksi optimal:
    1. Konfigurasikan Service Worker PWA dengan strategi *Network-First* menggunakan `vite-plugin-pwa` agar pembaruan cepat terserap dan aplikasi tetap dapat dimainkan saat tanpa jaringan internet (luring).
    2. Terapkan header keamanan peramban ketat di `vercel.json` dan meta tag `index.html` (Content Security Policy, X-Content-Type-Options: nosniff, X-Frame-Options: DENY).
    3. Buat skrip verifikasi otomatis `scripts/verify-bundle-size.mjs` yang memeriksa hasil keluaran `dist/` untuk memastikan ukuran total bundel JavaScript dan CSS terkompresi gzip $\le 150\text{ KB}$ sesuai batasan NFR PRD dan arsitektur.
  - **Acceptance Criteria & Verification**:
    - [x] Kompilasi rilis produksi `npm run build` sukses dengan exit code 0 tanpa error tipe TypeScript maupun linter Biome.
    - [x] Total ukuran bundel produksi terkompresi gzip terbukti $\le 150\text{ KB}$.
    - [x] Aset Service Worker dan Web App Manifest terpasang lengkap untuk fungsionalitas PWA.
    - [x] Header keamanan CSP terkonfigurasi identik pada `vercel.json` dan `index.html`.
    - **Verification Command**: `npm run build && node scripts/verify-bundle-size.mjs` (Exit Code 0)

---

## Progress Tracker & Domain Matrix

| Domain | Phase 1 (Infra) | Phase 2 (Data) | Phase 3 (Logic) | Phase 4 (UI/Client) | Phase 5 (E2E/Audit) | Total Tasks |
|:---|:---|:---|:---|:---|:---|:---:|
| **Core** | Task 1.1, 1.2, 1.3, 1.4, 1.5, 1.6 | - | Task 3.1, 3.2, 3.3 | - | Task 5.2, 5.4 | **11** |
| **Database** | - | Task 2.1, 2.2, 2.3, 2.4 | - | - | - | **4** |
| **Security** | - | - | Task 3.4 | - | Task 5.3 | **2** |
| **Web** | - | - | - | Task 4.1, 4.2, 4.3, 4.4, 4.5, 4.6, 4.7, 4.8, 4.9, 4.10, 4.11 | Task 5.1 | **12** |
| **Backend** | - (*N/A*) | - (*N/A*) | - (*N/A*) | - (*N/A*) | - (*N/A*) | **0** |
| **Mobile** | - (*Web PWA*) | - (*Web PWA*) | - (*Web PWA*) | - (*Web PWA*) | - (*Web PWA*) | **0** |
| **Total** | **6** | **4** | **4** | **11** | **4** | **29** |

