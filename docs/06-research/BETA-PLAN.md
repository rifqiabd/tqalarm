# Rencana Beta TQAlarm

**Fokus:** validasi operator sekolah untuk v1 browser-only.

## Tujuan beta

Beta memvalidasi apakah operator sekolah atau TU dapat menggunakan TQAlarm untuk menyiapkan dan menjalankan jadwal bel tanpa bantuan teknis. Validasi berfokus pada manfaat dan kejelasan v1: onboarding, Test Bell, jadwal, override tanggal, suara, penyimpanan lokal, import/export, dan penggunaan offline.

Beta tidak digunakan untuk memperluas scope produk. Ide baru yang tidak tercantum dalam Feature Map tetap menjadi backlog dan tidak otomatis masuk v1.

## Peserta utama

Peserta utama adalah operator sekolah atau staf tata usaha yang bertanggung jawab atas jadwal dan operasional bel. Kepala sekolah dan guru dapat memberi konteks tambahan, tetapi penilaian utama berasal dari operator yang menjalankan alur kerja harian.

Jumlah peserta, nama sekolah, jadwal sesi, dan kanal rekrutmen belum ditetapkan dalam dokumen sumber. Jangan mengisi nilai tersebut sebagai keputusan produk.

## Skenario uji v1

Setiap peserta diminta menjalankan alur berikut pada PWA browser-only:

1. Membuka aplikasi dan menyelesaikan onboarding tiga langkah.
2. Menggunakan tombol **Test Bell** dan menjelaskan apakah hasilnya terdengar serta mudah dipahami.
3. Membuat jadwal untuk satu hari dan beberapa hari, lalu mengaktifkan dan menonaktifkannya.
4. Mengatur waktu, label, suara, durasi, dan pra-bel pada data bel.
5. Membuat override untuk tanggal libur, ujian, acara, dan kustom.
6. Menggunakan aplikasi tanpa koneksi cloud dan memeriksa bahwa data lokal tetap tersedia.
7. Mengimpor jadwal dari Excel menggunakan alur yang disediakan.
8. Mengekspor konfigurasi JSON dan mengimpor kembali hasilnya.
9. Membuka lebih dari satu tab dan mengamati bahwa satu jadwal tidak menghasilkan bel ganda.
10. Mencoba audio unlock dan menjelaskan status audio yang ditampilkan aplikasi.

Skenario ini menguji scope v1 yang telah disetujui. Beta tidak meminta peserta menguji fitur yang belum menjadi bagian v1.

## Pertanyaan validasi

Tanyakan apakah operator dapat menyelesaikan onboarding dan Test Bell tanpa bantuan. Catat bagian yang membuat operator berhenti, ragu, atau salah memahami status audio, jadwal, override, atau penyimpanan.

Tanyakan apakah operator dapat memindahkan konfigurasi melalui JSON dan Excel, serta apakah hasil import sesuai harapan. Tanyakan juga apakah penggunaan offline cukup untuk rutinitas sekolah dan apakah operator memahami kapan jadwal aktif atau tidak aktif.

Catat kebutuhan operator dengan bahasa netral. Bedakan masalah pemahaman atau keandalan pada fitur v1 dari permintaan fitur baru. Permintaan di luar Feature Map tidak boleh dianggap sebagai komitmen pengembangan.

## Sinyal keberhasilan v1

Beta memberikan bukti positif apabila operator dapat menyelesaikan onboarding, menjalankan Test Bell, membuat dan mengaktifkan jadwal, membuat override tanggal, dan menggunakan data lokal tanpa bantuan teknis yang tidak direncanakan. Bukti tambahan adalah data dapat diekspor lalu diimpor kembali tanpa kehilangan data yang didukung, serta beberapa tab tidak menghasilkan bel ganda.

Kriteria kuantitatif, ambang kelulusan, dan format skor belum ditetapkan dalam dokumen sumber. Jangan menetapkan angka baru di sini.

## Pemisahan v1 dan feedback v1.5

**Temuan v1** adalah masalah atau keberhasilan pada PWA browser-only: onboarding, audio unlock, Web Worker timer, leader election, IndexedDB, jadwal, override, import/export, penggunaan offline, dan noise generator dasar. Temuan ini menjadi dasar keputusan kesiapan v1.

**Feedback v1.5** hanya dicatat sebagai masukan terpisah tentang bridge desktop dan keandalan tingkat sistem, termasuk native audio, native timer, SQLite sebagai source of truth, filesystem MP3, operasi saat browser ditutup, auto-start, system tray, `/kiosk`, watchdog, validasi lisensi offline, dan auto-update. Feedback ini tidak menjadi acceptance criteria v1 dan tidak boleh digunakan untuk menambahkan fitur v1.

Komunikasi PWA dan bridge pada tahap v1.5 tetap menggunakan WebSocket `localhost` dengan token auth. Beta v1 tidak menguji implementasi bridge, karena bridge berada di luar scope v1.

## Pencatatan hasil

Untuk setiap sesi, catat skenario, langkah yang dilakukan, hasil yang diharapkan, hasil yang diamati, bantuan yang diberikan, dan klasifikasi temuan: `v1`, `v1.5`, atau `backlog/TBD`. Jangan mencatat informasi pribadi peserta yang tidak diperlukan untuk validasi produk.

Ringkasan beta harus menjawab tiga hal: apakah operator dapat menjalankan alur inti v1, hambatan mana yang menghalangi penggunaan tanpa bantuan, dan masukan mana yang benar-benar berkaitan dengan bridge v1.5. Rencana ini tidak menetapkan fitur baru, harga, lisensi, domain, atau branding.
