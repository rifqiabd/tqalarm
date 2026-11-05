# Wireframes ASCII — TQAlarm

**Status:** Draft untuk review  
**Bahasa UI:** Bahasa Indonesia pada v1  
**Catatan:** Sketsa ini memetakan struktur dan alur, bukan spesifikasi visual final.

## 1. Dashboard — route `/`

```text
+----------------------------------------------------------------+
| TQAlarm                                      [Settings]         |
| Status: Audio aktif   Penyimpanan: IndexedDB   Bridge: --      |
+----------------------------------------------------------------+
| Jadwal aktif                                                   |
| [Pilih jadwal v]                         [Buat jadwal]         |
|                                                                |
| Senin, 07:00  Bel masuk       Default tone       Aktif         |
| Senin, 09:30  Istirahat       Bel sekolah        Aktif         |
| [Edit jadwal] [Nonaktifkan]                                      |
|                                                                |
| Hari ini: tidak ada override / [Kelola override tanggal]       |
+----------------------------------------------------------------+
| [Import Excel] [Import JSON] [Export JSON]                     |
| [Test Bell]                                                    |
+----------------------------------------------------------------+
| Noise generator: [White] [Pink] [Brown]                        |
+----------------------------------------------------------------+
```

Dashboard menampilkan status audio, sumber penyimpanan, status bridge bila relevan, jadwal, override tanggal, tindakan import/export, **Test Bell**, dan noise generator dasar. Tidak ada jam besar, running text, atau slideshow display board pada v1.

## 2. Onboarding — route `/onboarding`

### Langkah 1 dari 3

```text
+------------------------------------------------+
| Selamat datang di TQAlarm          [1/3]       |
+------------------------------------------------+
| Atur jadwal bel sekolah dan jalankan bel       |
| otomatis langsung dari browser.                |
|                                                |
| Data tersimpan lokal dan dapat digunakan       |
| tanpa cloud.                                   |
|                                                |
|                         [Lanjut]               |
+------------------------------------------------+
```

### Langkah 2 dari 3

```text
+------------------------------------------------+
| Aktifkan dan uji audio              [2/3]       |
+------------------------------------------------+
| [!] Audio browser belum aktif                  |
|     [Aktifkan Audio]                           |
|                                                |
| Setelah audio aktif, uji bel sebelum membuat   |
| jadwal harian.                                 |
|                                                |
| [Test Bell]       Hasil: Belum diuji           |
|                                                |
| [Kembali]                         [Lanjut]     |
+------------------------------------------------+
```

Tombol **Lanjut** pada langkah 2 sebaiknya efektif setelah audio unlock dan **Test Bell** dijalankan. Jika suara kustom bermasalah, hasil uji menggunakan default tone.

### Langkah 3 dari 3

```text
+------------------------------------------------+
| Siapkan jadwal                       [3/3]      |
+------------------------------------------------+
| Pilih cara mulai:                              |
| [Buat jadwal manual]                           |
| [Import dari Excel]                            |
| [Import JSON]                                  |
|                                                |
| Jadwal dapat diubah lagi dari dashboard.       |
|                                                |
| [Kembali]                  [Selesai ke Dashboard]|
+------------------------------------------------+
```

## 3. Audio unlock

Banner ini muncul di dashboard atau onboarding selama `audioUnlocked` belum aktif.

```text
+----------------------------------------------------------------+
| [!] Audio perlu diaktifkan agar bel dapat berbunyi              |
| Web Audio API menunggu interaksi pengguna.                     |
| [Aktifkan Audio]                                                |
+----------------------------------------------------------------+

Setelah ditekan:
+----------------------------------------------------------------+
| [OK] Audio aktif                                                |
| [Test Bell]       Suara: [Default tone v]                      |
+----------------------------------------------------------------+
```

Jika pemutaran suara kustom gagal, pesan tetap ringkas dan langsung menyediakan fallback:

```text
+----------------------------------------------------------------+
| [!] Suara kustom tidak dapat diputar                            |
| Default tone digunakan agar bel tetap berbunyi.                |
| [Test Bell] [Pilih suara lain]                                 |
+----------------------------------------------------------------+
```

## 4. Editor jadwal

```text
+----------------------------------------------------------------+
| < Dashboard   Editor jadwal                                   |
+----------------------------------------------------------------+
| Nama jadwal: [Jadwal sekolah pagi________________]              |
| Hari: [x] Senin [x] Selasa [x] Rabu [ ] Kamis [ ] Jumat        |
+----------------------------------------------------------------+
| Daftar bel                                                       |
| Waktu  Label             Suara             Durasi  Pra  Aktif  |
| 07:00  Bel masuk         Default tone      10 dtk  --   [x]    |
| 09:30  Istirahat          Bel sekolah       10 dtk  1m   [x]    |
|                                                                |
| [Tambah bel]                                                    |
+----------------------------------------------------------------+
| [Batal]                         [Simpan] [Simpan & Aktifkan]    |
+----------------------------------------------------------------+
```

Form bel mendukung waktu, label, suara, durasi, opsi pra-bel, dan status aktif. Pemilihan suara dapat memakai default tone atau suara kustom yang tersedia. Import Excel/JSON masuk ke tinjauan data sebelum disimpan melalui editor atau alur import yang setara.

## 5. Override tanggal

```text
+----------------------------------------------------------------+
| < Dashboard   Override tanggal                                |
+----------------------------------------------------------------+
| [Tambah override]                                               |
|                                                                |
| Tanggal      Tipe       Bel berlaku       Catatan              |
| 2026-06-10   exam       Bel ujian          Ujian akhir          |
| 2026-06-17   holiday    --                 Libur                |
|                                                                |
| [Edit] [Hapus]                                                  |
+----------------------------------------------------------------+
```

Form tambah/edit:

```text
+---------------------------------------------------------------+
| Override tanggal                                               |
| Tanggal: [2026-06-10____]                                     |
| Tipe:    [exam v]  holiday | exam | event | custom             |
| Bel:     [Pilih bel yang berlaku________________]              |
| Catatan: [________________________________________]             |
|                                                               |
| [Batal]                                      [Simpan]          |
+---------------------------------------------------------------+
```

Untuk tipe `holiday`, daftar bel dapat kosong sesuai data override; pada tanggal tersebut jadwal biasa tidak dijalankan. Override `exam`, `event`, dan `custom` dapat menunjuk bel yang berlaku.

## 6. Settings — route `/settings`

```text
+----------------------------------------------------------------+
| Pengaturan                                                     |
+----------------------------------------------------------------+
| Audio                                                          |
| Status: Audio aktif                                           |
| Suara default: [Default tone v]                               |
| Suara kustom:  [Upload suara]                                 |
|                                                                |
| Penyimpanan                                                   |
| Mode v1: IndexedDB lokal                                      |
| [Export JSON] [Import JSON]                                   |
|                                                                |
| Bridge (v1.5)                                                  |
| Status: Tidak terhubung                                       |
| [Hubungkan ke bridge]                                         |
| Endpoint: WebSocket localhost                                 |
| Token auth: [••••••••••]                                      |
|                                                                |
| Bahasa: Bahasa Indonesia                                      |
|                                                                |
| [Simpan]                                                       |
+----------------------------------------------------------------+
```

Pada v1, bagian bridge dapat ditampilkan sebagai status/kesiapan koneksi tanpa mengubah mode browser-only. Pada v1.5, koneksi menggunakan WebSocket `localhost` dan token auth. Saat terhubung, label sumber data menunjukkan bridge sebagai **source of truth**; saat tidak terhubung, IndexedDB menjadi fallback.

Field masa depan seperti `announcements`, `schoolLogo`, dan `displayMode` tetap dipertahankan dalam data, tetapi tidak dibuat sebagai kontrol UI v1.

## 7. Route kiosk — v1.5

Route `/kiosk` relevan setelah bridge tersedia pada v1.5. Ini adalah route PWA, bukan aplikasi terpisah.

```text
+----------------------------------------------------------------+
| TQAlarm Kiosk                         Bridge: Terhubung        |
| Jadwal: Sekolah pagi                 Sumber: bridge            |
+----------------------------------------------------------------+
| Status operasi                                                     |
|                                                                |
| BEL BERIKUTNYA                                                  |
| 09:30  Istirahat                                               |
| Suara: Bel sekolah   Durasi: 10 dtk                            |
|                                                                |
| [Test Bell]                 [Kembali ke Dashboard]             |
+----------------------------------------------------------------+
| Native timer bridge aktif | Browser boleh ditutup              |
+----------------------------------------------------------------+
```

Route ini hanya memetakan operasi kiosk dan status bridge yang termasuk v1.5. Detail display board seperti jam besar, running text, dan slideshow tidak dibuat sebagai fitur v1; route `/display` ditunda ke v2.

## 8. Status timer dan leader election (indikator bersama)

Indikator kecil dapat muncul pada dashboard/editor tanpa menjadi layar baru:

```text
Timer browser: Aktif   Leader tab: Ya   Tab lain: Mengikuti
Jadwal berikutnya: 09:30   Override hari ini: Tidak ada
```

Hanya tab pemimpin yang mengeksekusi bel. Jika tab pemimpin tidak tersedia, tab lain dapat mengambil peran melalui leader election tanpa memutar bel ganda.
