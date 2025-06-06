# Contoh Kasus Lapangan: Sistem Keamanan Pegadaian

Dokumen ini berisi contoh kasus nyata yang mungkin terjadi selama **Proof of Concept (PoC)** atau operasional penuh sistem keamanan pegadaian, yang mengintegrasikan **QR Code**, **RFID**, **autentikasi sidik jari**, **CCTV dengan pengenalan wajah**, dan **kontrol akses berbasis IoT**. Setiap kasus mencakup deskripsi situasi, potensi masalah, dan solusi praktis, untuk memastikan sistem berjalan lancar di lapangan. Dokumen ini dirancang sebagai panduan bagi petugas, supervisor, dan tim IT pegadaian.

## Daftar Isi
1. [Kasus 1: Pendaftaran Kantong Baru dengan QR Code Batch](#kasus-1-pendaftaran-kantong-baru-dengan-qr-code-batch)
2. [Kasus 2: Sidik Jari Petugas Gagal Terdeteksi](#kasus-2-sidik-jari-petugas-gagal-terdeteksi)
3. [Kasus 3: Pengenalan Wajah Tidak Sesuai Sidik Jari](#kasus-3-pengenalan-wajah-tidak-sesuai-sidik-jari)
4. [Kasus 4: Orang Tidak Dikenal Terdeteksi di Kluis](#kasus-4-orang-tidak-dikenal-terdeteksi-di-kluis)
5. [Kasus 5: RFID Kantong Tidak Sesuai Daftar](#kasus-5-rfid-kantong-tidak-sesuai-daftar)
6. [Kasus 6: Kamera RTSP Gagal Merekam](#kasus-6-kamera-rtsp-gagal-merekam)
7. [Kasus 7: Pelatihan AI untuk Karyawan Baru](#kasus-7-pelatihan-ai-untuk-karyawan-baru)
8. [Kasus 8: Laporan Analitik untuk Audit Bulanan](#kasus-8-laporan-analitik-untuk-audit-bulanan)

## Kasus 1: Pendaftaran Kantong Baru dengan QR Code Batch
**Skenario**:  
Selama PoC, petugas kasir harus mendaftarkan 8 kantong barang jaminan (misalnya, emas dari Pelanggan A) dalam satu transaksi untuk mempercepat proses. Petugas memilih opsi **QR Code Batch** di antarmuka Flask.

**Detail**:  
- Petugas memindai 8 QR Code (X1-X8) dan tag RFID (Y1-Y8) di meja kasir menggunakan webcam dan RFID reader RC522.
- Detail barang (misalnya, “Emas 10g, Pelanggan A, 06/06/2025”) dimasukkan via dashboard Flask di PC server.
- Sistem menghasilkan QR Code Batch untuk mewakili 8 kantong.

**Potensi Masalah**:  
- Salah satu QR Code tidak terbaca karena label buram.
- Data barang tidak lengkap di Flask (misalnya, lupa memasukkan nama pelanggan).

**Solusi**:  
- **QR Code Buram**: Petugas memindai ulang QR Code atau memasukkan nomor seri secara manual di Flask. Jika tetap gagal, ganti label QR Code menggunakan printer label di kasir.
- **Data Tidak Lengkap**: Dashboard Flask menampilkan peringatan “Data tidak lengkap” sebelum menyimpan. Petugas diminta melengkapi data (nama pelanggan, tanggal) sebelum lanjut.
- **Log Aktivitas**: Sistem mencatat pendaftaran di SQLite, termasuk waktu, petugas, dan QR Code Batch, untuk audit.

**Hasil**:  
Pendaftaran selesai dalam 2 menit, QR Code Batch mempermudah verifikasi di pintu kluis, dan log tersimpan di dashboard.

## Kasus 2: Sidik Jari Petugas Gagal Terdeteksi
**Skenario**:  
Petugas Budi mencoba masuk kluis untuk menyimpan kantong. Sensor sidik jari R307 di pintu kluis menampilkan “Peringatan: Sidik jari tidak dikenali” di dashboard Flask.

**Detail**:  
- Budi terdaftar di database sidik jari di Orange Pi 5 Ultra.
- Sidik jari gagal karena jari kotor atau sensor berdebu.

**Potensi Masalah**:  
- Sensor R307 kotor setelah penggunaan intensif selama PoC.
- Sidik jari Budi rusak karena luka kecil.

**Solusi**:  
- **Sensor Kotor**: Petugas membersihkan sensor R307 dengan kain mikrofiber dan alkohol 70%. Dashboard menampilkan panduan singkat untuk pembersihan.
- **Sidik Jari Rusak**: Budi mencuci tangan untuk menghilangkan kotoran. Jika gagal, supervisor dapat menyetujui akses sementara via dashboard Flask (dengan autentikasi berbasis peran) sambil merekam kejadian di log SQLite.
- **Pemeliharaan**: Tim IT menjadwalkan pembersihan sensor setiap 2 minggu selama PoC.
- **Log Aktivitas**: Peringatan “Sidik jari tidak dikenali” dicatat di SQLite, termasuk waktu dan ID petugas.

**Hasil**:  
Budi berhasil masuk setelah membersihkan jari, dan sistem mencatat percobaan gagal untuk analisis keamanan.

## Kasus 3: Pengenalan Wajah Tidak Sesuai Sidik Jari
**Skenario**:  
Petugas Ani memindai sidik jari di pintu kluis, tetapi CCTV mendeteksi wajah yang tidak sesuai. Dashboard Flask menampilkan “Peringatan: Wajah tidak sesuai sidik jari.”

**Detail**:  
- Sidik jari Ani valid di database SQLite.
- YOLOv8 Nano mendeteksi orang, tetapi *face_recognition* di Orange Pi gagal mencocokkan wajah Ani karena topi besar yang menutupi wajah.

**Potensi Masalah**:  
- Pencahayaan kluis kurang (<800 lumen) karena lampu LED mati.
- Wajah Ani tidak terlatih dengan baik di database (hanya 5 foto saat pendaftaran).

**Solusi**:  
- **Pencahayaan Buruk**: Petugas memeriksa lampu LED (5000K, 800 lumen) dan mengganti jika mati. Tim IT menjadwalkan pengecekan bulanan.
- **Wajah Terhalang**: Ani melepas topi dan memindai ulang wajah. Dashboard menampilkan panduan “Pastikan wajah jelas, tanpa penutup.”
- **Pelatihan Ulang**: Tim IT menggunakan skrip Flask di PC server (Ryzen 5, GTX 1650) untuk menambah 10 foto wajah Ani, meningkatkan akurasi *face_recognition*. Proses memakan waktu 5-30 menit per karyawan.
- **Log Aktivitas**: Peringatan dicatat di SQLite, dan supervisor diberi notifikasi di dashboard.

**Hasil**:  
Ani masuk kluis setelah melepas topi, dan pelatihan ulang meningkatkan akurasi wajah ke 75%.

## Kasus 4: Orang Tidak Dikenal Terdeteksi di Kluis
**Skenario**:  
Selama PoC, CCTV mendeteksi seseorang di kluis tanpa sidik jari yang valid. Dashboard Flask menampilkan “Peringatan: Orang tidak dikenal terdeteksi.”

**Detail**:  
- YOLOv8 Nano mendeteksi orang via kamera RTSP (120° FOV).
- *face_recognition* di Orange Pi menandai wajah sebagai *unknown*.
- Video dengan *bounding box* dan label “unknown” direkam via MEDIAMTX dan disimpan di NVR.

**Potensi Masalah**:  
- Orang tak dikenal adalah petugas baru yang belum terdaftar.
- Kemungkinan upaya akses tidak sah.

**Solusi**:  
- **Petugas Baru**: Supervisor memeriksa identitas orang via dashboard dan mendaftarkan wajah baru menggunakan skrip Flask (10–20 foto, 10-60 menit di PC server).
- **Akses Tidak Sah**: Supervisor mengunci kluis via dashboard Flask (override solenoid lock) dan melaporkan ke manajemen. Rekaman CCTV disimpan untuk investigasi.
- **Notifikasi**: Dashboard menampilkan peringatan merah untuk *unknown*, hanya dapat diakses supervisor.
- **Log Aktivitas**: Kejadian dicatat di SQLite dengan waktu, lokasi, dan status *unknown*.

**Hasil**:  
Petugas baru terdaftar dalam 10-60 menit, atau akses tidak sah dicegah dengan penguncian cepat.

## Kasus 5: RFID Kantong Tidak Sesuai Daftar
**Skenario**:  
Petugas mencoba membawa 10 kantong ke kluis, tetapi pembaca RFID RC522 mendeteksi tag Y9 tidak sesuai daftar. Dashboard menampilkan “Peringatan: RFID tidak sesuai.”

**Detail**:  
- Kantong Y1-Y8 valid, tetapi Y9 tidak terdaftar di SQLite.
- RFID reader memiliki jarak baca 1 meter.

**Potensi Masalah**:  
- Tag Y9 rusak atau salah pendaftaran di meja kasir.
- Interferensi sinyal RFID dari perangkat lain.

**Solusi**:  
- **Tag Rusak**: Petugas memindai ulang Y9. Jika gagal, ganti tag RFID baru dan perbarui di Flask.
- **Salah Pendaftaran**: Petugas memeriksa log pendaftaran di dashboard dan memperbaiki data Y9.
- **Interferensi**: Tim IT memeriksa perangkat di sekitar kluis dan memindahkan sumber interferensi.
- **Log Aktivitas**: Peringatan RFID dicatat di SQLite untuk analisis.

**Hasil**:  
Kantong Y9 diganti atau diperbaiki dalam 3 menit, dan petugas melanjutkan penyimpanan.

## Kasus 6: Kamera RTSP Gagal Merekam
**Skenario**:  
Selama PoC, CCTV gagal merekam saat petugas masuk kluis. Dashboard menampilkan “Peringatan: Koneksi RTSP terputus.”

**Detail**:  
- MEDIAMTX di Orange Pi tidak menerima stream dari kamera RTSP.
- MicroSD di Orange Pi penuh (256GB).

**Potensi Masalah**:  
- Koneksi jaringan terputus antara kamera dan Orange Pi.
- Kapasitas microSD habis karena rekaman PoC intensif.

**Solusi**:  
- **Koneksi Terputus**: Tim IT memeriksa kabel Ethernet atau Wi-Fi, restart MEDIAMTX via SSH di Orange Pi.
- **MicroSD Penuh**: Dashboard menampilkan peringatan penyimpanan. Tim IT menghapus rekaman lama (>30 hari) atau memindahkan ke NVR.
- **Pemeliharaan**: Jadwalkan pengecekan penyimpanan mingguan selama PoC.
- **Log Aktivitas**: Kegagalan rekaman dicatat di SQLite.

**Hasil**:  
Koneksi dipulihkan dalam 5 menit, dan rekaman dilanjutkan setelah membersihkan microSD.

## Kasus 7: Pelatihan AI untuk Karyawan Baru
**Skenario**:  
Pegadaian merekrut 10 karyawan baru selama PoC. Tim IT perlu melatih ulang model *face_recognition* untuk mengenali wajah mereka.

**Detail**:  
- Skrip Flask di PC server (Ryzen 5, 32GB RAM, GTX 1650) mendukung pelatihan batch.
- Dibutuhkan 10–20 foto per karyawan dari kamera RTSP.

**Potensi Masalah**:  
- Foto berkualitas rendah (buram atau pencahayaan buruk).
- Proses pelatihan memakan waktu lama jika foto banyak.

**Solusi**:  
- **Foto Buram**: Skrip Flask memvalidasi kualitas foto (resolusi, deteksi wajah) sebelum pemrosesan. Petugas diminta mengambil ulang foto di kluis dengan lampu LED.
- **Waktu Pelatihan**: GPU GTX 1650 memproses 200 foto (10 karyawan) dalam 10–60 menit. Dashboard menampilkan progres pelatihan.
- **Pembaruan Data**: Skrip juga mendukung penambahan foto untuk karyawan lama, meningkatkan akurasi.
- **Log Aktivitas**: Pelatihan dicatat di SQLite (waktu, jumlah karyawan, akurasi).

**Hasil**:  
10 karyawan terdaftar dalam 10 menit, dengan akurasi pengenalan wajah >75%.

## Kasus 8: Laporan Analitik untuk Audit Bulanan
**Skenario**:  
Supervisor cabang meminta laporan analitik bulanan untuk audit kepatuhan selama PoC. Laporan dihasilkan dari log SQLite via dashboard Flask.

**Detail**:  
- Laporan mencakup jumlah akses kluis, deteksi *unknown*, waktu rata-rata transaksi.
- Ekspor ke PDF/CSV untuk auditor.

**Potensi Masalah**:  
- Data log tidak lengkap karena kesalahan input petugas.
- Supervisor tidak terbiasa dengan antarmuka Flask.

**Solusi**:  
- **Data Tidak Lengkap**: Skrip Flask memvalidasi input (misalnya, detail barang wajib diisi). Log SQLite mencatat semua aktivitas untuk redundansi.
- **Pelatihan Pengguna**: Tim IT mengadakan sesi 30 menit untuk supervisor tentang penggunaan dashboard (filter laporan, ekspor PDF/CSV).
- **Laporan Otomatis**: Dashboard menyediakan template laporan (misalnya, “Akses Kluis Juni 2025: 150 transaksi, 2 deteksi unknown”).
- **Log Aktivitas**: Pembuatan laporan dicatat di SQLite untuk audit.

**Hasil**:  
Laporan bulanan diekspor dalam 5 menit, memenuhi kebutuhan auditor.

## Kontak
- **Pengelola**: Mr. Don
- **Asisten**: Bejo (bejo@donvirtus.net, WhatsApp: +62-xxx-xxx-xxxx)
