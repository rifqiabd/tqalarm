# TQAlarm

TQAlarm adalah aplikasi bel sekolah otomatis untuk sekolah Indonesia. Versi awal berjalan sebagai **Progressive Web App (PWA)** di browser dan dirancang untuk bekerja secara offline-first.

TQAlarm membantu operator sekolah atau TU menyiapkan jadwal bel, mengatur suara bel, menjalankan bel harian, dan menangani pengecualian seperti ujian, libur, atau acara khusus.

## Status Proyek

Proyek berada pada fase dokumentasi produk dan arsitektur. Kode aplikasi belum dibuat.

## Scope v1

Versi v1 adalah PWA browser-only yang tidak membutuhkan backend. Scope utamanya meliputi:

- Manajemen jadwal bel berdasarkan hari atau beberapa hari.
- Override per tanggal untuk ujian, libur, acara, dan kebutuhan kustom.
- Upload dan kustomisasi suara bel.
- Audio unlock flow untuk mengaktifkan audio browser.
- Web Worker timer dan leader election untuk mencegah bel ganda.
- Export dan import JSON.
- Import jadwal dari Excel menggunakan SheetJS.
- Onboarding tiga langkah dengan tombol **Test Bell**.
- Struktur i18n dengan Bahasa Indonesia lengkap dan Bahasa Inggris siap dikembangkan.

Versi v1 menggunakan IndexedDB dan tetap dapat digunakan tanpa koneksi cloud.

## Roadmap Singkat

- **v1:** PWA browser-only untuk jadwal dan bel sekolah.
- **v1.5:** Bridge desktop Tauri dengan native audio, native timer, SQLite, filesystem lokal, auto-start, system tray, kiosk mode, watchdog, dan validasi lisensi offline.
- **v2:** Display board pada route `/display`, sinkronisasi dan backup opsional melalui Supabase, serta branding per sekolah.
- **v2.5:** Multi-bridge, site license, dan iOS melalui Capacitor.

Fitur yang belum masuk scope v1 tetap berada di backlog dan tidak boleh ditambahkan tanpa perubahan scope yang disetujui.

## Arsitektur Utama

```text
+---------------------------------------------+
| Browser PWA                                 |
| Dashboard, jadwal, audio, IndexedDB        |
+----------------------+----------------------+
                       |
                       | WebSocket localhost
                       | + token authentication
                       v
+---------------------------------------------+
| Bridge Tauri (v1.5)                          |
| Native audio, native timer, SQLite, MP3     |
| filesystem, system tray, kiosk, watchdog   |
+----------------------+----------------------+
                       |
                       | Optional later capability
                       v
+---------------------------------------------+
| Supabase (v2)                                |
| Sync, backup, dan manajemen lisensi         |
+---------------------------------------------+
```

Saat bridge terhubung, bridge menjadi **source of truth**. Saat bridge tidak terhubung, browser menggunakan IndexedDB lokal sebagai fallback. Kiosk dan display tetap merupakan route PWA, bukan aplikasi terpisah.

## Struktur Dokumentasi

### Produk

- [PRD](docs/01-product/PRD.md) — kebutuhan dan scope produk.
- [ROADMAP](docs/01-product/ROADMAP.md) — fase pengembangan dan kriteria selesai.
- [BACKLOG](docs/01-product/BACKLOG.md) — fitur v2+ dan ide yang sengaja ditunda.
- [USER-FLOW](docs/01-product/USER-FLOW.md) — alur pengguna dan koneksi bridge.

### Arsitektur

- [SAD](docs/02-architecture/SAD.md) — software architecture description.
- [DATA-MODEL](docs/02-architecture/DATA-MODEL.md) — model data browser dan bridge.
- [API-CONTRACT](docs/02-architecture/API-CONTRACT.md) — kontrak PWA dan bridge.
- [ADR](docs/02-architecture/ADR/) — keputusan arsitektur yang disetujui.

### Desain dan teknis

- [WIREFRAMES](docs/03-design/WIREFRAMES.md) — sketsa UI berbasis ASCII.
- [SETUP](docs/04-technical/SETUP.md) — batasan setup teknis.
- [TESTING](docs/04-technical/TESTING.md) — strategi dan acceptance checks.

### Bisnis dan riset

- [PRICING](docs/05-business/PRICING.md) — prinsip penawaran dan status harga.
- [BETA-PLAN](docs/06-research/BETA-PLAN.md) — rencana validasi beta.

## Prinsip Produk

1. **Client-side first:** versi gratis tidak membutuhkan backend.
2. **Offline-first:** cloud adalah kemampuan tambahan, bukan syarat penggunaan v1.
3. **Fail-safe:** audio kustom yang bermasalah menggunakan default tone.
4. **Keandalan melalui bridge:** bridge membedakan versi berbayar melalui native audio, native timer, dan operasi saat browser ditutup.
5. **Tidak ada feature creep:** ide baru masuk ke backlog, bukan langsung ke scope v1.

## Teknologi yang Dikunci

Svelte 5, SvelteKit, TypeScript, Tauri, IndexedDB, SQLite, SheetJS, i18next, Web Audio API, Web Worker, WebSocket localhost, dan Supabase untuk kemampuan cloud pada tahap berikutnya.

## Lisensi dan Nilai TBD

Lisensi repository, harga Pro, harga site license, domain, dan branding resmi belum ditetapkan. Nilai tersebut tetap **TBD** sampai ada persetujuan.
