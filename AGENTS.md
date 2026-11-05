# AGENTS.md — TQAlarm

> **Cara menggunakan dokumen ini:**
> Baca sampai selesai sebelum mengeksekusi apa pun.
> Bagian **🔒 LOCKED** tidak boleh diubah tanpa persetujuan user.
> Bagian **TBD** — tanya user, jangan berasumsi.
> Bagian **Deliverables** — buat satu per satu, review di antara dokumen.
> Jika ambigu, **TANYA dulu**. Jangan menebak.

---

## 1. Ringkasan Produk

**TQAlarm** adalah aplikasi bel sekolah otomatis berbasis **PWA (Progressive Web App)** yang berjalan sepenuhnya di browser sebagai versi gratis, dan dapat di-upgrade menjadi **aplikasi desktop berbayar** (bridge) untuk keandalan tingkat sistem. Target utama: **sekolah Indonesia**.

**Diferensiasi:**
- Versi gratis benar-benar berguna (bukan *crippled*).
- Versi berbayar menawarkan **keandalan** (bel tetap berbunyi walau browser ditutup, presisi native timer, penyimpanan filesystem).
- Offline-first, cloud opsional.

---

## 2. Target User & Use Case

| Persona | Use Case Utama |
|---|---|
| **Operator sekolah / TU** | Setup jadwal bel, jalankan bel harian, override untuk ujian/libur |
| **Kepala sekolah** | Monitor, site license, branding |
| **Guru** | Lihat jadwal dan gunakan aplikasi untuk mendukung kegiatan mengajar |

---

## 3. 🔒 Locked Decisions (JANGAN DIUBAH)

1. **Arsitektur:** client-side first. Zero backend untuk versi gratis.
2. **Bridge = source of truth** saat terhubung. Browser fallback ke IndexedDB lokal.
3. **Kiosk & display adalah *route*** (`/kiosk`, `/display`), bukan aplikasi terpisah.
4. **Tech stack dikunci:** Svelte 5 + SvelteKit, TypeScript, Tauri, Supabase, IndexedDB, SQLite, SheetJS, i18next.
5. **Komunikasi PWA ↔ Bridge:** WebSocket `localhost` + token auth (bukan extension browser).
6. **Skema data sudah menyertakan field masa depan** (`announcements`, `schoolLogo`, `displayMode`) — jangan hapus meski UI belum ada.
7. **Pembayaran manual dulu** (WA), otomasi menyusul. Jangan bangun sistem pembayaran di v1.
8. **Prinsip dokumen: "map, not manual."** Detail di `docs/`, `AGENTS.md` hanya peta.
9. **Bahasa:** ID untuk dokumen user-facing, EN untuk dokumen teknis (SAD, ADR, API Contract).

---

## 4. Feature Map per Tier

### Tier 1 — Gratis (Browser-only) — **Scope v1**
- Manajemen jadwal bel (per hari, multi-hari)
- Override per tanggal (ujian, libur, acara)
- Upload/kustomisasi suara bel
- Audio unlock flow (banner + resume AudioContext)
- Timer Web Worker + Leader Election (anti bel ganda)
- Export/Import JSON
- Import dari Excel (SheetJS)
- Onboarding 3 langkah + tombol "Test Bell"
- i18n structure (ID lengkap, EN siap)

### Tier 2 — Berbayar (Bridge Desktop) — **Scope v1.5**
- Semua fitur Tier 1
- Native audio playback (tanpa autoplay restriction)
- Native timer (presisi, background, tanpa throttle)
- Penyimpanan MP3 di filesystem lokal
- Setting di SQLite (source of truth)
- Bel tetap berbunyi walau browser ditutup
- Auto-start saat boot, system tray
- Kiosk mode (`/kiosk`)
- Watchdog + fail-safe
- Validasi lisensi offline
- Auto-update

### Tier 3 — Lanjutan (v2+)
- Display board ala bank/stasiun (`/display`)
- Multi-bridge (master-slave)
- Cloud sync (Supabase)
- Branding per sekolah
- Site license
- Integrasi absensi QR
- iOS (Capacitor)
- Focus Mode (Noise Generator) untuk guru yang membutuhkan fokus di ruang TU

### 🚫 Backlog — JANGAN dimasukkan ke scope v1
- Display board UI (jam besar, running text, slideshow)
- Integrasi kalender akademik
- Multi-bahasa lengkap (EN)
- Branding UI per sekolah
- Otomasi pembayaran penuh (Midtrans/Xendit)
- Google Drive backup otomatis
- Fitur sosial / berbagi konfigurasi Focus Mode
- Focus Mode (Noise Generator) untuk guru yang membutuhkan fokus di ruang TU

---

## 5. Arsitektur Sistem

```
┌──────────────────────────────────────────────┐
│ BROWSER (PWA) — Dashboard / Remote Control   │
│ · UI konfigurasi jadwal dan upload MP3       │
│ · IndexedDB (mode gratis, fallback)          │
│ · Web Audio API (perlu audio unlock)         │
│ · Web Worker timer (perlu leader election)   │
└────────────┬─────────────────────────────────┘
             │ WebSocket (localhost + token auth)
             │ + Binary file transfer
┌────────────▼─────────────────────────────────┐
│ BRIDGE (Tauri) — Engine                      │
│ · Native audio (tanpa autoplay restriction)  │
│ · Native timer (presisi, background)         │
│ · SQLite (source of truth)                   │
│ · Filesystem untuk MP3                       │
│ · System tray, auto-start, kiosk mode        │
│ · License validation (offline-capable)       │
└────────────┬─────────────────────────────────┘
             │ (opsional, fitur berbayar)
┌────────────▼─────────────────────────────────┐
│ CLOUD (Supabase) — Sync & Backup             │
│ · Sync antar perangkat                       │
│ · Backup otomatis                            │
│ · Manajemen lisensi                          │
└──────────────────────────────────────────────┘
```

**Route di PWA:** `/` (dashboard), `/kiosk`, `/display` (v2), `/settings`, `/onboarding`.

---

## 6. Data Model Draft

> Skema lengkap ada di `docs/02-architecture/DATA-MODEL.md`. Ini ringkasannya.

**Tabel inti (IndexedDB di browser, SQLite di bridge):**

| Tabel | Field Kunci |
|---|---|
| `schedules` | id, name, isActive, days[], timestamps |
| `bells` | id, scheduleId, time, label, soundId, durationSec, preBellMin, isActive |
| `overrides` | id, date, type (holiday\|exam\|event\|custom), bellIds[], note |
| `sounds` | id, name, source (builtin\|user), fileBlob/filePath, durationSec |
| `settings` | schoolName, schoolLogo, themeColor, locale, displayMode, announcements[], audioUnlocked, bridgeConnected, bridgeToken |
| `logs` | id, timestamp, bellId, status, errorMessage |

**Catatan:** Field `announcements`, `schoolLogo`, `displayMode` disiapkan untuk v2 — jangan hapus.

---

## 7. Tech Stack

| Lapisan | Teknologi |
|---|---|
| Frontend | **Svelte 5 + SvelteKit** |
| Bahasa | **TypeScript** |
| PWA | Manifest + Service Worker (SvelteKit built-in) |
| Audio (browser) | Web Audio API |
| Timer (browser) | Web Worker + Leader Election |
| Storage (browser) | IndexedDB (`idb`/Dexie) |
| Excel | SheetJS (`xlsx`) |
| i18n | `i18next` |
| Bridge | **Tauri** |
| Storage (bridge) | SQLite |
| Transport | WebSocket (`ws` / tauri-plugin-websocket) |
| Cloud | Supabase |
| Licensing | Keyforge / Freemius |
| Hosting | GitHub Pages / Netlify / Vercel |
| Testing | Vitest + Playwright + `svelte-check` |

---

## 8. Roadmap & Milestone

| Fase | Fokus | Estimasi | Kriteria Selesai |
|---|---|---|---|
| **v1 (MVP)** | Browser-only: jadwal, audio, onboarding, i18n structure | 4–6 minggu | User bisa setup jadwal & bel berbunyi tanpa bantuan |
| **v1.5** | Bridge desktop, lisensi, native engine | 4–6 minggu | Bel tetap berbunyi walau browser ditutup |
| **v2** | Kiosk, display board, cloud sync | 4–6 minggu | Multi-device sync berjalan, display board berfungsi |
| **v2.5** | Multi-bridge, site license, iOS | Fleksibel | Satu sekolah bisa pakai multi-bridge |

---

## 9. Prinsip Desain

1. **Versi gratis harus berguna** — bukan *crippled*. Diferensiasi utama: keandalan, bukan fitur dasar.
2. **Offline-first** — cloud adalah *enhancement*, bukan *requirement*.
3. **Zero backend untuk versi gratis** — hilangkan dependensi server.
4. **Fail-safe** — jika audio corrupt, fallback ke default tone. Jika bridge crash, ada watchdog.
5. **Onboarding menentukan adopsi** — 3 langkah, tombol "Test Bell" wajib.
6. **Anti feature creep** — ide baru masuk *Backlog*, bukan scope v1.

---

## 10. ⚠️ Instruksi untuk Agent (BACA BERULANG)

### Saat Menulis Dokumen
- **Format:** Markdown.
- **Bahasa:** ID (PRD, ROADMAP, BACKLOG, USER-FLOW, WIREFRAMES, PRICING, BETA-PLAN) / EN (SAD, DATA-MODEL, API-CONTRACT, ADR, SETUP, TESTING).
- **Gaya:** instruksional langsung, tidak bertele-tele.
- **Panjang:** PRD maks 3 halaman; dokumen lain sesuai kebutuhan, jangan *over-engineer*.
- **Konsisten:** gunakan istilah yang sama di semua dokumen (contoh: selalu "bridge", bukan "daemon" / "agent" / "service").
- **Diagram:** ASCII art saja. Jangan Mermaid (tidak semua agent render).

### Larangan Keras
- ❌ **JANGAN** menambah fitur di luar Feature Map. Ide baru → tulis di `BACKLOG.md`.
- ❌ **JANGAN** mengubah Locked Decisions (bagian 3).
- ❌ **JANGAN** mengisi TBD (bagian 14) tanpa persetujuan user.
- ❌ **JANGAN** langsung eksekusi semua deliverable sekaligus. **Satu per satu**, review di antara.
- ❌ **JANGAN** menulis kode di fase dokumentasi ini.

### Jika Ambigu
- **TANYA.** Jangan menebak. Contoh: *"Apakah multi-bridge masuk v1 atau v2? Spec tidak jelas."*

---

## 11. 🛠️ Tools untuk Fase Coding (Nanti, Bukan Sekarang)

> Detail konfigurasi ada di `docs/04-technical/SETUP.md` (akan dibuat agent). Ini pointer-nya.

**Wajib:**
- `@sveltejs/mcp` (Svelte MCP Server) + Skills resmi (`svelte-code-writer`, `svelte-core-bestpractices`)
- Context7 MCP (untuk library pihak ketiga: Supabase, SheetJS, dll.)

**Direkomendasikan:**
- `svelte-grab` atau `@clicktocode/svelte` (Alt+Click elemen → context untuk agent)

**Jangka Panjang:**
- Vitest + Playwright + `svelte-check`

---

## 12. 🎯 Quality Gates (Opsional, untuk Dokumentasi)

- **`asd-ste100-skill`** (mode Strict) — untuk `API-CONTRACT.md` dan `ADR/*.md`. Hilangkan ambiguitas.
- **`make-it-click`** — untuk `SAD.md` dan `DATA-MODEL.md`. Diagram ASCII / HTML interaktif.

**Gunakan sebagai *lapisan akhir***, bukan pengganti dokumen inti.

---

## 13. Deliverables

Buat **satu per satu**, review di antara. Urutan yang disarankan:

| # | Dokumen | Lokasi | Bahasa |
|---|---|---|---|
| 1 | PRD.md | `docs/01-product/` | ID |
| 2 | ROADMAP.md | `docs/01-product/` | ID |
| 3 | BACKLOG.md | `docs/01-product/` | ID |
| 4 | USER-FLOW.md | `docs/01-product/` | ID |
| 5 | SAD.md | `docs/02-architecture/` | EN |
| 6 | DATA-MODEL.md | `docs/02-architecture/` | EN |
| 7 | API-CONTRACT.md | `docs/02-architecture/` | EN |
| 8 | ADR/0001..0004.md | `docs/02-architecture/ADR/` | EN |
| 9 | WIREFRAMES.md | `docs/03-design/` | ID |
| 10 | SETUP.md | `docs/04-technical/` | EN |
| 11 | TESTING.md | `docs/04-technical/` | EN |
| 12 | PRICING.md | `docs/05-business/` | ID |
| 13 | BETA-PLAN.md | `docs/06-research/` | ID |

**ADR awal yang harus dibuat:**
- `0001-client-side-first.md`
- `0002-bridge-source-of-truth.md`
- `0003-kiosk-display-as-route.md`
- `0004-payment-manual-first.md`

---

## 14. Directory Tree

```
/
├── AGENTS.md                    ← file ini
├── README.md
├── CHANGELOG.md
├── LICENSE                       ← TBD
├── docs/
│   ├── 00-INDEX.md
│   ├── 01-product/              ← PRD, ROADMAP, BACKLOG, USER-FLOW
│   ├── 02-architecture/         ← SAD, DATA-MODEL, API-CONTRACT, ADR/
│   ├── 03-design/               ← WIREFRAMES
│   ├── 04-technical/            ← SETUP, TESTING, (nanti: DEPLOYMENT, SECURITY)
│   ├── 05-business/             ← PRICING, (nanti: EULA, PRIVACY)
│   └── 06-research/             ← BETA-PLAN, (nanti: FEEDBACK-LOG)
├── src/                         ← kode PWA (nanti)
│   ├── lib/{db,audio,scheduler,i18n}/
│   ├── components/
│   ├── routes/                  ← SvelteKit: /, /kiosk, /display, /settings, /onboarding
│   └── locales/
├── bridge/                      ← kode Tauri (nanti)
└── tests/
```

---

## 15. TBD (Tanya User Sebelum Mengisi)

- **Lisensi repo:** MIT / Proprietary / MIT-core + Proprietary-bridge?
- **Harga Pro:** Rp 50–100rb sekali bayar?
- **Harga Site License:** Rp 500rb–1jt/tahun?
- **Domain:** `tqalarm.com` / `.id` / subdomain GitHub Pages dulu?
- **Nama domain & branding resmi:** belum final.

---

## 16. Alur Kerja yang Diharapkan

1. Baca `AGENTS.md` ini sampai selesai.
2. Konfirmasi pemahaman dengan ringkasan: (a) produknya apa, (b) arsitektur utama, (c) apa yang **tidak boleh** dilakukan.
3. Tunggu instruksi user — **jangan mulai menulis** sebelum diminta.
4. Saat diminta, buat **satu dokumen**, tunjukkan, tunggu review.
5. Setelah disetujui, lanjut dokumen berikutnya.
6. Simpan hasil ke path yang sesuai di `docs/`.

**Selamat bekerja. Ingat: map, not manual. Jangan menebak. Jangan menambah fitur.**
