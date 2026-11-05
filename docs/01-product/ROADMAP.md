# Roadmap — TQAlarm

**Status:** Draft untuk review  
**Target produk:** Sekolah Indonesia  
**Prinsip:** Offline-first, client-side first, dan *map, not manual*.

## 1. Arah Produk

TQAlarm dikembangkan bertahap dari PWA browser-only yang berguna tanpa backend menjadi aplikasi dengan bridge desktop untuk keandalan tingkat sistem. Fitur inti jadwal dan audio tersedia pada versi gratis. Bridge berbayar membedakan produk melalui native audio, native timer, penyimpanan lokal, dan kemampuan menjalankan bel saat browser ditutup.

Setiap fase harus mempertahankan prinsip offline-first. Cloud merupakan enhancement, bukan requirement untuk versi gratis. Detail teknis berada di dokumen arsitektur dan teknis, bukan di roadmap ini.

## 2. Milestone

| Fase | Fokus | Cakupan utama | Kriteria selesai |
|---|---|---|---|
| **v1 — MVP** | PWA browser-only | Jadwal bel, override tanggal, suara bel, audio unlock, timer Web Worker, leader election, JSON, import Excel, onboarding, i18n structure, dan noise generator dasar | Pengguna dapat menyiapkan jadwal dan bel berbunyi tanpa bantuan. |
| **v1.5** | Bridge desktop berbayar | Semua fitur v1, native audio, native timer, filesystem untuk MP3, SQLite sebagai source of truth, bel saat browser ditutup, auto-start, system tray, `/kiosk`, watchdog, lisensi offline, dan auto-update | Bel tetap berbunyi walau browser ditutup. |
| **v2** | Kiosk, display, dan cloud opsional | Display board pada route `/display`, sinkronisasi dan backup melalui Supabase, serta branding per sekolah | Sinkronisasi multi-perangkat berjalan dan display board berfungsi. |
| **v2.5** | Skala sekolah dan platform tambahan | Multi-bridge, site license, dan iOS melalui Capacitor | Satu sekolah dapat menggunakan multi-bridge. |

## 3. v1 — MVP Browser-only

### Tujuan

Menyediakan aplikasi bel sekolah yang dapat digunakan langsung di browser, bekerja offline, dan membantu operator menyiapkan jadwal tanpa bantuan teknis.

### Pekerjaan utama

1. Menyediakan dashboard pada route `/` untuk melihat dan mengelola jadwal.
2. Mendukung jadwal berdasarkan satu hari atau beberapa hari.
3. Mendukung data bel berupa waktu, label, suara, durasi, pra-bel, dan status aktif.
4. Mendukung override per tanggal untuk libur, ujian, acara, dan kustom.
5. Menyediakan upload dan kustomisasi suara bel.
6. Menyediakan audio unlock flow melalui banner dan resume `AudioContext`.
7. Menjalankan timer melalui Web Worker dan mencegah bel ganda dengan leader election.
8. Menyimpan data secara lokal di IndexedDB.
9. Menyediakan export/import JSON dan import Excel menggunakan SheetJS.
10. Menyediakan onboarding tiga langkah dengan tombol **Test Bell**.
11. Menyediakan struktur i18n dengan Bahasa Indonesia lengkap dan Bahasa Inggris siap dikembangkan.
12. Menyediakan noise generator white, pink, dan brown noise.

### Kriteria penerimaan

- Pengguna baru dapat menyelesaikan onboarding dan menjalankan **Test Bell**.
- Operator dapat membuat, mengaktifkan, menonaktifkan, dan mengubah jadwal.
- Operator dapat membuat override untuk tanggal khusus.
- Aplikasi tetap dapat digunakan tanpa koneksi cloud.
- Audio bermasalah dapat menggunakan default tone sebagai fallback.
- Beberapa tab tidak menghasilkan bel ganda.
- Data dapat diekspor dan diimpor kembali melalui JSON.
- Data jadwal dapat diimpor dari Excel.

## 4. v1.5 — Bridge Desktop

### Tujuan

Meningkatkan keandalan operasi sekolah melalui bridge Tauri tanpa mengurangi fitur yang tersedia di v1.

### Pekerjaan utama

- Menjadikan bridge sebagai source of truth saat terhubung.
- Menambahkan native audio tanpa pembatasan autoplay browser.
- Menambahkan native timer yang presisi dan tidak terkena throttle background.
- Menyimpan pengaturan di SQLite dan file MP3 di filesystem lokal.
- Mempertahankan komunikasi PWA dan bridge melalui WebSocket localhost dengan token auth.
- Menjalankan bel saat browser ditutup.
- Menambahkan auto-start, system tray, dan route `/kiosk`.
- Menambahkan watchdog dan fail-safe.
- Menambahkan validasi lisensi offline dan auto-update.

### Kriteria penerimaan

- Bridge dapat menerima konfigurasi dari PWA melalui WebSocket yang terautentikasi.
- Bridge menjalankan audio dan timer secara native.
- Pengaturan bridge tersimpan di SQLite.
- File MP3 tersimpan di filesystem lokal.
- Bel tetap berjalan saat browser ditutup.
- Browser menggunakan IndexedDB sebagai fallback ketika bridge tidak terhubung.

## 5. v2 — Display dan Cloud Opsional

### Tujuan

Memperluas penggunaan TQAlarm di lingkungan sekolah yang memerlukan tampilan dan sinkronisasi antar-perangkat.

### Pekerjaan utama

- Menyediakan display board pada route `/display`.
- Menambahkan sinkronisasi antar-perangkat melalui Supabase.
- Menambahkan backup otomatis melalui Supabase.
- Menambahkan branding per sekolah.
- Mempertahankan field data masa depan `announcements`, `schoolLogo`, dan `displayMode`.

### Kriteria penerimaan

- Display board berfungsi sebagai route PWA, bukan aplikasi terpisah.
- Data yang didukung dapat disinkronkan antar-perangkat.
- Backup cloud bersifat opsional dan tidak menghilangkan kemampuan offline.

## 6. v2.5 — Skala dan Platform

Fase ini mencakup multi-bridge, site license, dan iOS melalui Capacitor. Detail harga, lisensi, domain, dan branding resmi belum ditetapkan dan tetap menjadi TBD sampai ada persetujuan.

Kriteria selesai fase ini adalah satu sekolah dapat memakai multi-bridge sesuai rancangan yang disetujui.

## 7. Backlog yang Tidak Masuk Scope v1

Item berikut tidak boleh dimasukkan ke scope v1:

- Display board UI dengan jam besar, running text, atau slideshow.
- Integrasi kalender akademik.
- Bahasa Inggris lengkap.
- Branding UI per sekolah.
- Otomasi pembayaran penuh melalui Midtrans atau Xendit.
- Backup otomatis ke Google Drive.
- Fitur sosial atau berbagi preset noise.

Ide baru yang tidak tercantum pada Feature Map harus masuk ke `BACKLOG.md`, bukan langsung dimasukkan ke milestone.

## 8. Dependensi dan Aturan Perubahan

Tech stack tetap menggunakan Svelte 5, SvelteKit, TypeScript, Tauri, IndexedDB, SQLite, SheetJS, i18next, dan Supabase sesuai tahap yang relevan. Pembayaran tetap manual melalui WhatsApp pada tahap awal; sistem pembayaran tidak dibangun di v1.

Locked Decisions dalam `AGENTS.md` tidak boleh diubah melalui roadmap. Jika ruang lingkup atau urutan fase menjadi ambigu, keputusan harus dimintakan kepada user sebelum roadmap diubah.
