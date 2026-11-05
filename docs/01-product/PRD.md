# Product Requirements Document — TQAlarm

**Status:** Draft untuk review  
**Target:** Sekolah Indonesia  
**Rilis:** v1 (MVP, browser-only)

## 1. Ringkasan Produk

TQAlarm adalah aplikasi bel sekolah otomatis berbasis **Progressive Web App (PWA)**. Aplikasi membantu operator sekolah mengatur jadwal bel, menjalankan bel harian, dan menangani pengecualian seperti ujian, libur, atau acara khusus. TQAlarm juga menyediakan noise generator dasar untuk fokus dan relaksasi.

Versi v1 berjalan sepenuhnya di browser, offline-first, dan tetap berguna tanpa backend. Pada tahap berikutnya, aplikasi dapat di-upgrade dengan **bridge desktop Tauri** berbayar untuk audio dan timer native, penyimpanan filesystem, serta keandalan saat browser ditutup.

Prinsip produk: **map, not manual** — detail implementasi dijelaskan di dokumen teknis, bukan di PRD ini.

## 2. Masalah yang Diselesaikan

Sekolah memerlukan cara yang sederhana dan dapat diandalkan untuk:

- Menyusun jadwal bel berdasarkan hari dan beberapa hari sekaligus.
- Menggunakan suara bel sendiri tanpa konfigurasi yang rumit.
- Menangani perubahan jadwal pada tanggal tertentu, misalnya ujian, libur, atau acara.
- Menjalankan bel secara konsisten dalam kondisi offline.
- Memastikan operator memahami status aplikasi dan dapat menguji bel sebelum digunakan.

Solusi v1 harus memberikan manfaat nyata tanpa memaksa sekolah membeli bridge. Keunggulan bridge nantinya adalah **keandalan**, bukan penghilangan fitur dasar dari versi gratis.

## 3. Target User dan Use Case

| Persona | Kebutuhan utama |
|---|---|
| **Operator sekolah / TU** | Menyiapkan jadwal, mengelola suara, menjalankan bel, dan membuat override. |
| **Kepala sekolah** | Memantau penggunaan dan memahami opsi lisensi pada tahap berikutnya. |
| **Guru** | Melihat jadwal dan menggunakan noise generator untuk fokus. |
| **Individu** | Menggunakan noise generator untuk fokus atau relaksasi. |

### Skenario utama

1. Operator membuka TQAlarm, menyelesaikan onboarding tiga langkah, lalu menekan **Test Bell**.
2. Operator membuat jadwal bel untuk satu atau beberapa hari.
3. Operator mengatur waktu, label, suara, durasi, dan opsi pra-bel.
4. Operator mengaktifkan jadwal dan membiarkan aplikasi menjalankan bel.
5. Untuk tanggal khusus, operator membuat override bertipe libur, ujian, acara, atau kustom.
6. Operator mengimpor jadwal dari Excel atau memindahkan konfigurasi melalui JSON.
7. Guru atau individu membuka noise generator dan memilih lapisan white, pink, atau brown noise.

## 4. Tujuan dan Indikator Keberhasilan

### Tujuan v1

- Operator dapat menyiapkan jadwal dan mendengar bel tanpa bantuan teknis.
- Aplikasi dapat digunakan offline dengan penyimpanan lokal.
- Audio browser dapat diaktifkan secara eksplisit dan statusnya mudah dipahami.
- Bel tidak berbunyi ganda ketika beberapa tab terbuka.
- Pengguna dapat memindahkan data melalui JSON dan Excel.
- Onboarding mengarahkan pengguna sampai berhasil menjalankan **Test Bell**.

### Indikator keberhasilan

- Pengguna baru menyelesaikan onboarding dan berhasil menjalankan **Test Bell**.
- Pengguna dapat membuat, mengaktifkan, dan mengubah jadwal bel.
- Jadwal tetap tersedia setelah aplikasi digunakan kembali secara offline.
- Satu jadwal tidak menghasilkan bel ganda karena tab browser yang bersamaan.
- Pengguna dapat mengekspor lalu mengimpor kembali konfigurasi tanpa kehilangan data yang didukung.

## 5. Ruang Lingkup v1 — Gratis, Browser-only

### Manajemen jadwal dan bel

- Jadwal berdasarkan hari dan multi-hari.
- Aktivasi dan penonaktifan jadwal.
- Data bel: waktu, label, suara, durasi, pra-bel, dan status aktif.
- Override berdasarkan tanggal untuk `holiday`, `exam`, `event`, dan `custom`.
- Catatan pada override.

### Audio

- Upload dan kustomisasi suara bel.
- Audio unlock flow berupa banner dan resume `AudioContext`.
- Default tone sebagai fallback apabila audio kustom bermasalah.
- Noise generator dasar: white, pink, dan brown noise.

### Data dan kompatibilitas

- Penyimpanan lokal menggunakan IndexedDB.
- Export/import JSON.
- Import Excel menggunakan SheetJS.
- Struktur i18n dengan Bahasa Indonesia lengkap dan Bahasa Inggris siap dikembangkan.

### Onboarding dan keandalan browser

- Onboarding tiga langkah.
- Tombol **Test Bell** wajib tersedia dalam onboarding.
- Timer menggunakan Web Worker.
- Leader election untuk mencegah bel ganda.
- Aplikasi offline-first; cloud bukan prasyarat v1.

## 6. Di Luar Ruang Lingkup v1

Fitur berikut tidak boleh dimasukkan ke scope v1 dan tetap menjadi backlog atau tahap berikutnya:

- Display board dengan jam besar, running text, atau slideshow.
- Integrasi kalender akademik.
- Bahasa Inggris lengkap.
- Branding UI per sekolah.
- Otomasi pembayaran melalui Midtrans atau Xendit.
- Backup otomatis ke Google Drive.
- Fitur sosial atau berbagi preset noise.
- Bridge desktop, native timer/audio, system tray, auto-start, kiosk mode, watchdog, validasi lisensi offline, dan auto-update — fitur-fitur ini termasuk **v1.5**.
- Cloud sync, site license, multi-bridge, integrasi absensi QR, dan iOS — fitur-fitur ini termasuk **v2 atau lebih baru**.

## 7. Struktur Produk dan Arsitektur yang Mengikat

- Frontend menggunakan **Svelte 5, SvelteKit, dan TypeScript**.
- PWA memakai manifest dan Service Worker.
- Audio browser menggunakan Web Audio API.
- Timer browser menggunakan Web Worker dan leader election.
- Penyimpanan browser menggunakan IndexedDB.
- Route utama: `/` untuk dashboard, `/settings` untuk pengaturan, dan `/onboarding` untuk onboarding. `/kiosk` disiapkan untuk bridge/v1.5 dan `/display` untuk v2; keduanya tetap route PWA, bukan aplikasi terpisah.
- Saat bridge tersedia pada tahap berikutnya, bridge menjadi **source of truth** dan browser menjadi fallback ke IndexedDB lokal.
- Komunikasi PWA dan bridge menggunakan WebSocket localhost dengan token auth, bukan extension browser.
- Field masa depan `announcements`, `schoolLogo`, dan `displayMode` harus dipertahankan dalam model data meski UI-nya belum ada.

## 8. Prinsip UX dan Operasional

1. **Berguna sejak versi gratis:** fitur inti jadwal dan audio tidak dibuat crippled.
2. **Offline-first:** cloud adalah enhancement, bukan requirement.
3. **Fail-safe:** audio bermasalah menggunakan default tone; mekanisme watchdog menjadi bagian bridge pada tahap berikutnya.
4. **Onboarding menentukan adopsi:** alur singkat dan **Test Bell** wajib.
5. **Tidak ada feature creep:** ide baru masuk backlog, bukan scope v1.
6. **Bahasa konsisten:** gunakan istilah `bridge`; jangan menggantinya dengan `daemon`, `agent`, atau `service`.

## 9. Batasan dan Keputusan Terkunci

- v1 harus zero backend.
- Pembayaran manual melalui WhatsApp terlebih dahulu; sistem pembayaran tidak dibangun di v1.
- Data inti disiapkan agar dapat direpresentasikan di IndexedDB browser dan SQLite bridge.
- Lisensi repo, harga Pro, harga site license, domain, serta nama domain dan branding resmi masih **TBD** dan harus ditanyakan sebelum diisi.

## 10. Roadmap Tingkat Tinggi

| Fase | Fokus | Kriteria selesai |
|---|---|---|
| **v1** | Browser-only: jadwal, audio, onboarding, dan struktur i18n | Pengguna dapat menyiapkan jadwal dan bel berbunyi tanpa bantuan. |
| **v1.5** | Bridge desktop, lisensi, dan native engine | Bel tetap berbunyi saat browser ditutup. |
| **v2** | Kiosk, display board, dan cloud sync | Sinkronisasi multi-perangkat dan display board berfungsi. |
| **v2.5** | Multi-bridge, site license, dan iOS | Satu sekolah dapat memakai multi-bridge. |

## 11. Pertanyaan Terbuka

Hal berikut belum boleh diputuskan tanpa persetujuan user:

- Lisensi repo: MIT, Proprietary, atau MIT-core + Proprietary-bridge.
- Harga Pro.
- Harga site license.
- Domain resmi.
- Nama domain dan branding resmi.
