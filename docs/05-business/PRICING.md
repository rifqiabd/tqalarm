# Pricing TQAlarm

**Status:** Draft; nilai komersial menunggu persetujuan.

## Prinsip penawaran

TQAlarm dimulai dari versi gratis yang benar-benar berguna. Versi gratis adalah PWA browser-only, offline-first, dan tidak memerlukan backend. Fitur inti jadwal, audio, override, import/export, dan onboarding tidak boleh dibuat sengaja terbatas hanya untuk mendorong pembelian.

Pembeda versi berbayar adalah keandalan operasional melalui **bridge**, bukan penghapusan fitur dasar dari versi gratis. Bridge direncanakan untuk tahap v1.5 dan mencakup native audio, native timer, penyimpanan lokal, serta operasi ketika browser ditutup.

## Penawaran yang direncanakan

| Penawaran | Cakupan | Harga | Status |
|---|---|---|---|
| Gratis / PWA browser-only | Scope v1; berjalan di browser dan menyimpan data secara lokal | Tidak ditetapkan di dokumen ini | Model dasar produk |
| Pro / bridge desktop | Scope v1.5; bridge Tauri dan kemampuan keandalan yang disetujui | **TBD** | Belum boleh diisi |
| Site license | Direncanakan pada fase yang lebih lanjut; detail cakupan belum ditetapkan | **TBD** | Bukan scope v1 |

Site license tidak boleh diperlakukan sebagai fitur v1. Detail multi-bridge dan lisensi sekolah mengikuti roadmap setelah v1 dan v1.5, sesuai keputusan produk yang disetujui.

## Proses pembayaran saat ini

Pembayaran tetap **manual melalui WhatsApp**. Calon pengguna menghubungi kanal WhatsApp yang ditetapkan, menerima informasi penawaran yang sudah disetujui, dan menyelesaikan konfirmasi secara manual. TQAlarm tidak membangun checkout, integrasi Midtrans, integrasi Xendit, atau otomasi pembayaran pada v1.

Nomor WhatsApp, format instruksi pembayaran, bukti pembayaran, waktu pemrosesan, dan prosedur aktivasi belum ditetapkan dalam dokumen sumber. Jangan mengarang detail tersebut di sini.

## Nilai yang masih TBD

- **Harga Pro:** TBD.
- **Harga site license:** TBD.
- **Lisensi repo:** TBD — pilihan yang tercatat adalah MIT, Proprietary, atau MIT-core + Proprietary-bridge.
- **Domain:** TBD.
- **Nama domain dan branding resmi:** TBD.

TBD berarti belum diputuskan. Jangan mengganti TBD dengan kisaran, asumsi, nama, angka, atau pilihan final tanpa persetujuan.

## Batas produk

Versi gratis tetap client-side first, zero backend, dan offline-first. Cloud melalui Supabase merupakan kemampuan opsional pada tahap berikutnya, bukan syarat pembayaran atau syarat penggunaan v1. Branding per sekolah dan site license tetap berada di roadmap setelah v1, bukan fitur yang boleh ditambahkan melalui dokumen pricing ini.
