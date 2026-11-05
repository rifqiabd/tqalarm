# User Flow — TQAlarm

**Status:** Draft untuk review  
**Scope:** v1 (PWA browser-only) dan alur koneksi bridge v1.5

## 1. Prinsip alur

TQAlarm adalah PWA offline-first. Pada v1, data disimpan di IndexedDB dan bel dijalankan oleh browser. Pengguna harus menyelesaikan onboarding sampai berhasil menjalankan **Test Bell** sebelum mengandalkan jadwal harian.

Pada v1.5, bridge menjadi **source of truth** saat terhubung. PWA tetap menjadi antarmuka konfigurasi dan remote control; IndexedDB browser menjadi fallback ketika bridge tidak terhubung. Komunikasi PWA dan bridge menggunakan WebSocket `localhost` dengan token auth, bukan extension browser.

## 2. Alur utama pengguna baru

```text
Buka PWA /
   |
   v
[Onboarding — Langkah 1: Kenali TQAlarm]
   |  Lanjut
   v
[Langkah 2: Audio]
   |  Audio belum aktif -> banner Audio unlock -> pengguna menekan Aktifkan Audio
   |  Resume AudioContext berhasil
   |  Test Bell wajib dijalankan
   v
[Langkah 3: Siapkan jadwal]
   |  Buat jadwal manual
   |  atau import Excel/JSON bila sudah punya data
   v
Dashboard /
   |
   v
Pilih jadwal aktif + cek status audio
   |
   v
Timer Web Worker + leader election
   |
   v
Waktu bel tercapai -> bel berbunyi
```

### Langkah 1 — Kenali TQAlarm

1. Pengguna membuka TQAlarm untuk pertama kali.
2. PWA menampilkan tujuan singkat: mengatur jadwal bel dan menjalankan bel otomatis.
3. Pengguna memilih **Lanjut**.
4. PWA menyimpan progres onboarding secara lokal agar pengguna dapat melanjutkan tanpa cloud.

### Langkah 2 — Aktifkan dan uji audio

1. PWA memeriksa status `audioUnlocked`.
2. Jika audio belum aktif, PWA menampilkan banner audio unlock dengan tindakan **Aktifkan Audio**.
3. Pengguna menekan tindakan tersebut sebagai interaksi eksplisit.
4. PWA me-resume `AudioContext` melalui Web Audio API.
5. Setelah berhasil, status audio berubah menjadi aktif dan banner tidak lagi meminta unlock.
6. Pengguna menekan **Test Bell**. TQAlarm memutar suara uji menggunakan suara yang dipilih.
7. Jika suara kustom belum tersedia atau gagal diputar, TQAlarm menjalankan alur fallback default tone.
8. Jika pengguna mendengar bel, pengguna memilih **Lanjut** ke langkah 3. Jika tidak, pengguna mencoba **Aktifkan Audio** atau memilih/mengunggah suara lain, lalu mengulangi **Test Bell**.

**Kondisi selesai langkah 2:** `AudioContext` sudah di-resume dan pengguna berhasil atau dapat mendengar hasil **Test Bell**.

### Langkah 3 — Siapkan jadwal

Pengguna memilih salah satu jalur berikut:

- **Buat manual:** mengisi nama jadwal, memilih satu atau beberapa hari, lalu membuat data bel.
- **Import Excel:** memilih file Excel. TQAlarm membaca data jadwal melalui SheetJS, menampilkan hasil yang dapat ditinjau, lalu menyimpan data yang valid secara lokal.
- **Import JSON:** memilih konfigurasi JSON yang kompatibel, meninjau hasil import, lalu menyimpan data yang didukung secara lokal.

Setelah data tersedia, pengguna kembali ke dashboard. Onboarding dianggap selesai setelah jadwal dapat dilihat dan pengguna memahami bahwa jadwal harus diaktifkan agar timer menjalankannya.

## 3. Membuat dan mengaktifkan jadwal

```text
Dashboard -> Buat jadwal
   |
   v
Isi nama + pilih hari (satu atau beberapa hari)
   |
   v
Tambah bel: waktu, label, suara, durasi, pra-bel, status aktif
   |
   v
Simpan
   |
   v
Tinjau daftar bel
   |-- Ubah / nonaktifkan bel bila perlu
   v
Aktifkan jadwal
   |
   v
Jadwal masuk ke timer browser
```

1. Operator memilih **Buat jadwal** di dashboard.
2. Operator mengisi nama jadwal dan memilih hari yang berlaku.
3. Operator menambahkan satu atau beberapa bel. Setiap bel dapat berisi waktu, label, suara, durasi, opsi pra-bel, dan status aktif.
4. Operator menyimpan jadwal.
5. Operator meninjau daftar bel, lalu mengaktifkan jadwal. Jadwal yang tidak aktif tidak dijalankan.
6. Operator dapat kembali untuk mengubah atau menonaktifkan jadwal dan bel.

## 4. Import dan perpindahan konfigurasi

### Import Excel

1. Operator membuka tindakan import pada dashboard atau editor jadwal.
2. Operator memilih file Excel.
3. TQAlarm memetakan baris yang didukung ke jadwal dan data bel melalui SheetJS.
4. PWA menampilkan ringkasan hasil import untuk ditinjau.
5. Operator mengonfirmasi penyimpanan. Data valid masuk ke IndexedDB; data yang tidak didukung tidak dijadikan asumsi baru.
6. Operator meninjau jadwal hasil import sebelum mengaktifkannya.

### Import JSON

1. Operator memilih **Import JSON**.
2. Operator memilih file konfigurasi JSON.
3. PWA memvalidasi struktur dan menampilkan ringkasan data yang akan dipindahkan.
4. Operator mengonfirmasi import.
5. Data yang didukung disimpan di IndexedDB pada v1, lalu operator meninjau dan mengaktifkan jadwal bila diperlukan.

Export JSON merupakan pasangan perpindahan konfigurasi: operator mengekspor konfigurasi yang didukung, lalu dapat mengimpornya kembali tanpa kehilangan data yang didukung.

## 5. Override tanggal

```text
Dashboard -> Override tanggal -> Tambah override
   |
   v
Pilih tanggal + tipe: holiday / exam / event / custom
   |
   v
Pilih bel yang berlaku (bila relevan) + isi catatan
   |
   v
Simpan
   |
   v
Pada tanggal tersebut, timer memakai override, bukan penerapan jadwal biasa
```

1. Operator membuka **Override tanggal**.
2. Operator memilih tanggal.
3. Operator memilih tipe `holiday`, `exam`, `event`, atau `custom`.
4. Untuk override yang memerlukan daftar bel, operator memilih `bellIds` yang berlaku. Operator dapat menambahkan catatan.
5. Operator menyimpan override.
6. Saat timer mengevaluasi tanggal tersebut, TQAlarm memakai override tanggal yang tersimpan untuk menentukan bel yang aktif. Pada hari libur, tidak ada bel yang dijalankan sesuai data override.
7. Operator dapat mengubah atau menghapus override melalui daftar override.

## 6. Menunggu sampai bel berbunyi di browser (v1)

1. PWA menyimpan jadwal, bel, override, suara, dan pengaturan di IndexedDB.
2. PWA menjalankan timer melalui Web Worker agar pemeriksaan waktu tidak bergantung pada pembaruan tampilan.
3. Jika beberapa tab TQAlarm terbuka, **leader election** memilih satu tab sebagai pemimpin timer.
4. Hanya tab pemimpin yang boleh mengeksekusi bel. Tab lain mengikuti status dan tidak memutar bel kedua.
5. Tab pemimpin mengevaluasi jadwal aktif, hari, tanggal, override, waktu bel, status aktif, dan opsi pra-bel.
6. Ketika waktu tercapai, PWA mencoba memutar suara bel melalui Web Audio API.
7. Jika audio kustom tidak tersedia atau gagal diputar, PWA menggunakan default tone sebagai fallback agar alur bel tetap fail-safe.
8. Hasil eksekusi dicatat sebagai log berhasil atau gagal sesuai kemampuan penyimpanan lokal.
9. Jika tab pemimpin ditutup atau tidak lagi tersedia, leader election memilih tab lain. Tab baru melanjutkan pemeriksaan tanpa membuat bel ganda.

```text
IndexedDB -> Web Worker timer -> leader election
                                      |
                         hanya leader memeriksa dan mengeksekusi
                                      |
                  waktu tercapai -> suara kustom -> bel berbunyi
                                      |
                         gagal -> default tone -> bel berbunyi
```

### Catatan operasional v1

- Browser harus tetap menjalankan PWA agar timer browser dapat bekerja.
- Audio unlock harus sudah selesai sebelum operasi bel harian diandalkan.
- Tidak ada cloud sebagai prasyarat; PWA tetap dapat digunakan offline.
- UI display board bukan fitur v1. Route `/display` disiapkan untuk v2 dan tidak memiliki alur UI v1.

## 7. Alur fallback default tone

```text
Mulai Test Bell / bel terjadwal
   |
   v
Suara kustom tersedia dan dapat diputar?
   | Ya                         | Tidak
   v                            v
Putar suara kustom        Catat kegagalan yang relevan
                                |
                                v
                         Putar default tone
                                |
                                v
                         Bel tetap berbunyi
```

Fallback digunakan ketika file suara kustom bermasalah, tidak tersedia, atau gagal dimainkan setelah audio unlock. Pengguna tidak perlu menunggu suara kustom diperbaiki untuk mendapatkan bunyi bel; pengguna dapat kembali ke pengaturan suara setelah eksekusi selesai.

## 8. Alur koneksi PWA ke bridge (v1.5)

```text
PWA /settings atau indikator koneksi
   |
   v
Cari bridge pada WebSocket localhost
   |
   v
Bridge meminta token auth
   |
   v
Token valid?
   | Tidak                     | Ya
   v                           v
Tolak koneksi + tetap     Hubungkan PWA <-> bridge
IndexedDB sebagai fallback      |
                                v
                 Bridge menjadi source of truth
                                |
             sinkronkan konfigurasi yang didukung
                                |
                 bridge native timer + native audio
                                |
              bel tetap berjalan saat browser ditutup
```

1. Pada v1.5, PWA mencoba terhubung ke bridge melalui WebSocket `localhost`.
2. Koneksi menggunakan token auth. Koneksi tanpa token yang valid ditolak.
3. Setelah autentikasi berhasil, bridge menjadi **source of truth**. Pengaturan disimpan di SQLite bridge dan file MP3 dapat disimpan di filesystem lokal sesuai cakupan v1.5.
4. PWA mengirim konfigurasi yang didukung melalui koneksi WebSocket. Bridge mengonfirmasi penerimaan dan status koneksi ditampilkan secara jelas.
5. Bridge menggunakan native timer dan native audio. Karena engine berjalan di bridge, bel dapat tetap berbunyi walau browser ditutup.
6. Jika bridge tidak tersedia, token tidak valid, atau koneksi terputus, PWA kembali menggunakan IndexedDB sebagai fallback dan mekanisme v1: Web Worker, leader election, serta Web Audio API.
7. Route `/kiosk` tersedia pada v1.5 sebagai route PWA yang terhubung ke bridge. UI kiosk tidak mengubah `/display`; display board tetap ditunda ke v2.

## 9. Titik akhir utama

Alur berhasil apabila operator dapat menyelesaikan onboarding tiga langkah, mendengar **Test Bell**, memiliki jadwal aktif, memahami override tanggal bila diperlukan, dan membiarkan TQAlarm menjalankan bel. Pada v1.5, alur juga berhasil apabila PWA terautentikasi ke bridge dan bridge dapat menjalankan bel saat browser ditutup.
