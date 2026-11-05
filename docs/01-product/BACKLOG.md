# Backlog Produk — TQAlarm

**Status:** Draft untuk review  
**Aturan:** Jangan memasukkan item backlog ke scope v1 tanpa persetujuan dan perubahan roadmap.

Backlog ini memisahkan fitur yang sudah direncanakan pada **Feature Map v2+** dari ide yang sengaja ditunda. v1 tetap browser-only, offline-first, dan tanpa cloud atau sistem pembayaran otomatis.

## 1. Feature Map v2+

Item berikut sudah direncanakan untuk fase setelah v1. Jangan mengerjakannya sebagai bagian v1.

| Fitur | Status / target fase | Arah dan alasan penundaan |
|---|---|---|
| **Display board** pada route `/display` | Direncanakan, **v2** | Perluasan untuk tampilan sekolah. Route tetap berada di PWA, bukan aplikasi terpisah. Ditunda agar v1 fokus pada jadwal dan bel inti. |
| **Cloud sync dan backup melalui Supabase** | Direncanakan, **v2** | Cloud adalah enhancement opsional. v1 harus tetap zero backend dan dapat digunakan offline. |
| **Branding per sekolah** | Direncanakan, **v2** | Mendukung kebutuhan sekolah setelah fitur inti stabil. Field `schoolLogo` dan `displayMode` sudah disiapkan dalam model data. Nama, domain, dan branding resmi tetap **TBD**. |
| **Multi-bridge** | Direncanakan, **v2.5** | Mendukung penggunaan beberapa bridge dalam satu sekolah. Rancangan detail belum ditetapkan. |
| **Site license** | Direncanakan, **v2.5** | Mendukung skala penggunaan sekolah. Harga site license tetap **TBD** dan tidak boleh diasumsikan. |
| **iOS melalui Capacitor** | Direncanakan, **v2.5** | Platform tambahan setelah kebutuhan multi-bridge dan skala sekolah ditangani. |
| **Focus Mode (White/Pink/Brown Noise)** | Direncanakan, **v2+** | Ditunda untuk guru yang butuh fokus di ruang TU. |
| **Integrasi absensi QR** | Direncanakan, **v2+** | Tercantum sebagai fitur Tier 3. Fase rinci dan prioritas implementasinya belum ditetapkan. |

### Urutan fase yang menjadi acuan

- **v2:** Kiosk, display board, cloud sync/backup melalui Supabase, dan branding per sekolah.
- **v2.5:** Multi-bridge, site license, dan iOS melalui Capacitor.
- **Integrasi absensi QR:** tetap fitur v2+, tetapi target fase rinci belum ditetapkan.

## 2. Backlog yang Sengaja Ditunda

Item berikut **tidak boleh masuk scope v1**. Jangan mengubahnya menjadi fitur v1 hanya karena kebutuhan implementasi atau permintaan sementara.

| Item | Status / target | Alasan penundaan |
|---|---|---|
| **Detail UI display board**: jam besar, running text, dan slideshow | Backlog; setelah display board v2 | Display board sebagai kemampuan sudah direncanakan pada v2, tetapi rincian UI ini tetap di luar scope v1. |
| **Integrasi kalender akademik** | Backlog; target belum ditetapkan | Tidak termasuk Feature Map v2+ yang direncanakan. Tunda untuk menjaga fokus pada jadwal dan override manual. |
| **Bahasa Inggris lengkap** | Backlog; setelah struktur i18n v1 siap | v1 menyediakan Bahasa Indonesia lengkap dan struktur Bahasa Inggris yang siap dikembangkan; terjemahan lengkap bukan scope v1. |
| **Branding UI per sekolah** | Backlog implementasi; terkait branding per sekolah v2 | Branding per sekolah direncanakan pada v2, tetapi UI branding tidak boleh masuk v1. Nama, domain, dan branding resmi tetap **TBD**. |
| **Otomasi pembayaran penuh** melalui Midtrans atau Xendit | Backlog; target belum ditetapkan | Pembayaran harus manual melalui WhatsApp terlebih dahulu. Sistem pembayaran tidak dibangun di v1. |
| **Backup otomatis ke Google Drive** | Backlog; target belum ditetapkan | Jalur cloud yang direncanakan adalah sync/backup opsional melalui Supabase pada v2. Jangan menambah integrasi cloud lain ke v1. |
| **Fitur sosial atau berbagi konfigurasi Focus Mode** | Backlog; target belum ditetapkan | Tidak termasuk scope produk yang telah direncanakan. Tunda untuk mencegah feature creep dan menjaga fokus pada aplikasi bel sekolah. |

## 3. Batasan Backlog

- Pertahankan istilah **bridge**. Jangan gunakan istilah daemon, agent, atau service.
- Jangan menghapus atau menganggap selesai field masa depan `announcements`, `schoolLogo`, dan `displayMode`.
- Jangan mengisi lisensi repo, harga Pro, harga site license, domain, atau branding resmi. Semua tetap **TBD** sampai ada persetujuan user.
- Ide baru yang tidak tercantum di `AGENTS.md`, PRD, atau ROADMAP harus ditanyakan terlebih dahulu; jangan menambahkannya sebagai fitur produk.
