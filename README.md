# Sistem Keamanan Pegadaian

Proyek ini mengimplementasikan sistem keamanan untuk ruang kluis pegadaian, mengintegrasikan **QR Code**, **RFID**, **autentikasi sidik jari**, **CCTV dengan pengenalan wajah**, dan **kontrol akses berbasis IoT**. Sistem dirancang untuk menangani hingga 10 kantong barang jaminan dalam satu transaksi, dengan anggaran Rp30 juta per kluis, dan terintegrasi dengan NVR untuk perekaman CCTV. Fokus utama adalah **Proof of Concept (PoC)** untuk memastikan keandalan sebelum ekspansi.

## Daftar Isi
1. [Gambaran Umum](#gambaran-umum)
2. [Kebutuhan Perangkat Keras](#kebutuhan-perangkat-keras)
3. [Alur Kerja Sistem](#alur-kerja-sistem)
4. [Input ke Sistem](#input-ke-sistem)
5. [Output Sistem](#output-sistem)
6. [Akses Output Sistem](#akses-output-sistem)
7. [Notifikasi Sistem](#notifikasi-sistem)
8. [Kemungkinan Error](#kemungkinan-error)
9. [Pemeliharaan](#pemeliharaan)
10. [Lisensi](#lisensi)
11. [Peningkatan di Masa Depan](#peningkatan-di-masa-depan)
12. [Model Bisnis](#model-bisnis)
13. [Kontak](#kontak)

## Gambaran Umum
Sistem Keamanan Pegadaian mengamankan penyimpanan dan pengambilan barang jaminan di kluis dengan teknologi mutakhir:
- **QR Code**: Identifikasi kantong, mendukung **QR Code Batch** untuk multi-kantong.
- **RFID**: Deteksi pergerakan kantong di pintu kluis untuk memastikan hanya barang yang diizinkan yang masuk/keluar, dengan notifikasi untuk anomali.
- **Sensor Sidik Jari**: Autentikasi petugas untuk masuk (sensor luar) dan keluar (sensor dalam) kluis.
- **CCTV dengan Pengenalan Wajah**: Identifikasi petugas real-time di depan pintu (1 kamera) dan di dalam kluis (2 kamera), terintegrasi dengan NVR via **MEDIAMTX**.
- **Orange Pi 5 Ultra**: Pusat kontrol lokal dengan NPU 6 TOPS untuk inferensi AI (YOLOv8 Nano, *face_recognition*), streaming RTSP, dan log SQLite.
- **PC Server**: Dashboard Flask, pelatihan AI batch, dan database wajah terpusat.
- **Solenoid Lock**: Mengamankan pintu kluis, dikontrol berdasarkan sidik jari, wajah, dan QR Code (untuk Deposit/Retrieve).

**Skenario Petugas (Skenario 1)**:
- Semua petugas terdaftar di awal (sidik jari via R307, wajah via *face_recognition* dengan 10–20 foto per petugas).
- Petugas dapat memilih tujuan: Deposit (memasukkan barang), Retrieve (mengambil barang), atau Inspect (inspeksi tanpa barang).
- Untuk Deposit/Retrieve, petugas memindai QR Code batch di pintu. Untuk Inspect, hanya sidik jari dan wajah diperlukan.
- Skenario ini dipilih untuk PoC karena efisiensi, kepraktisan, dan kesesuaian dengan pengujian autentikasi serta deteksi RFID.

**Fitur Tambahan**:
- **Mode Keluar Sementara**: Memungkinkan petugas keluar sementara selama Deposit dengan membawa barang, dengan verifikasi RFID saat kembali.
- **Multi-Petugas**: Sistem mendukung beberapa petugas di kluis bersamaan, dengan CCTV memverifikasi wajah masing-masing.

**Catatan PoC**:
- PoC menggunakan NVR terpisah untuk perekaman CCTV.
- Stream RTSP dikelola oleh MEDIAMTX di Orange Pi, perekaman hanya saat orang terdeteksi.
- Dashboard Flask menampilkan notifikasi real-time dan log untuk audit.

## Kebutuhan Perangkat Keras
Berikut daftar perangkat keras untuk PoC, total Rp30 juta:

| **Komponen**                     | **Biaya (Rp)** | **Fungsi**                                                                 |
|----------------------------------|----------------|---------------------------------------------------------------------------|
| Orange Pi 5 Ultra (16GB)         | 3.000.000      | Inferensi AI (YOLOv8 Nano, *face_recognition*), streaming RTSP via MEDIAMTX, integrasi sidik jari, solenoid lock, log SQLite. |
| Kamera RTSP (1080p, 120° FOV, 2 unit) | 3.000.000 | Perekaman di kluis, verifikasi wajah real-time. |
| Kamera RTSP Depan Pintu (1080p)  | 1.500.000      | Autentikasi wajah di pintu masuk. |
| NVR (untuk PoC)                  | 1.000.000      | Menyimpan rekaman CCTV, kompresi H.264. |
| NVMe SSD 256GB                   | 1.000.000      | Penyimpanan log SQLite, embedding wajah, rekaman sementara. |
| RFID Reader + 100 Tag (RC522)    | 1.000.000      | Deteksi kantong masuk/keluar di pintu kluis (jarak 1m). |
| Sensor Sidik Jari (R307, 2 unit) | 800.000        | Autentikasi petugas (1 luar untuk masuk, 1 dalam untuk keluar, <2 detik). |
| Solenoid Lock (12V, 5A)          | 500.000        | Mengamankan pintu kluis. |
| Segel Plastik (100 unit)         | 200.000        | Segel anti-rusak untuk kantong. |
| Lampu LED (5000K, 800 lumen)     | 200.000        | Pencahayaan untuk pengenalan wajah. |
| UPS (1000VA)                     | 1.000.000      | Keandalan saat listrik padam (2 jam). |
| Scanner QR Portabel              | 1.000.000      | Verifikasi QR Code di pintu kluis. |
| PC Server (32GB RAM, Ryzen 5, GTX 1650) | 9.000.000 | Dashboard Flask, pelatihan AI batch, database wajah. |
| Pengembangan AI & Antarmuka (termasuk laporan) | 5.000.000 | Model *face_recognition*, skrip Flask, laporan analitik. |
| Instalasi & Konfigurasi          | 2.000.000      | Setup MEDIAMTX, NVR, jaringan. |
| Cadangan Pengujian & *Fine-Tuning* | 1.300.000    | Pengujian PoC, simulasi 10 kantong. |
| **Total**                        | **30.000.000** |                                                                           |

**Catatan Anggaran**:
- Anggaran mencakup dua sensor sidik jari untuk keamanan maksimal.
- Cadangan pengujian dipangkas dari Rp1,7 juta ke Rp1,3 juta untuk mengakomodasi sensor tambahan.
- Laporan analitik termasuk dalam “Pengembangan AI & Antarmuka”.

## Alur Kerja Sistem
Flowchart Mermaid untuk alur kerja umum (Deposit, Retrieve, Inspect), dengan fokus pada proses keluar:

```mermaid
graph TD
    A[Start] --> B[Petugas pilih tujuan di dashboard Flask]
    B --> C{Tujuan}
    C -->|Deposit| D1[Pindai QR Code Batch X1-Xn]
    C -->|Retrieve| D2[Pilih kantong Y1-Yn di Flask]
    C -->|Inspect| D3[Tidak perlu QR Code]
    D1 --> E[Ke pintu kluis]
    D2 --> E
    D3 --> E
    E --> F[Pindai sidik jari luar]
    F --> G{Sidik jari valid?}
    G -->|Ya| H[Kamera depan deteksi wajah]
    G -->|Tidak| I[Peringatan: Akses ditolak]
    H --> J{Wajah terdaftar?}
    J -->|Ya| K[Deposit/Retrieve: Verifikasi QR Code]
    J -->|Tidak| I
    K --> L{QR Code valid?}
    L -->|Ya| M[Solenoid lock buka pintu]
    L -->|Tidak| I
    M --> N[Masuk kluis]
    N --> O[CCTV dalam kluis rekam & verifikasi wajah]
    O --> P{Wajah valid?}
    P -->|Ya| Q[Lakukan tindakan sesuai tujuan]
    P -->|Tidak| R[Peringatan: Orang tidak dikenal]
    Q -->|Deposit| S1[Tempatkan kantong di kabinet]
    Q -->|Retrieve| S2[Ambil kantong dari kabinet]
    Q -->|Inspect| S3[Lakukan inspeksi]
    S1 --> T{Opsi Keluar Sementara?}
    T -->|Ya| U[Pilih Keluar Sementara di Flask]
    T -->|Tidak| V[Pindai sidik jari dalam untuk keluar]
    U --> W[RFID deteksi kantong saat keluar]
    W --> X{Kantong sesuai?}
    X -->|Ya| Y[Pindai sidik jari dalam]
    X -->|Tidak| Z[Peringatan: Kantong tidak diizinkan]
    Y --> AA{Sidik jari valid?}
    AA -->|Ya| AB[Solenoid lock buka pintu]
    AA -->|Tidak| AC[Peringatan: Akses ditolak]
    AB --> AD[Petugas keluar]
    AD --> AE[RFID deteksi kantong saat keluar]
    AE --> AF{Kantong diizinkan?}
    AF -->|Ya| AG[Catat log: Petugas, waktu, RFID, tujuan]
    AF -->|Tidak| AH[Peringatan ke dashboard: Kantong tidak diizinkan]
    AH --> AG
    AG --> AI[Selesai]
    S2 --> V
    S3 --> V
    V --> AD
```

### Detail Alur Kerja
- **Pilih Tujuan**: Petugas memilih Deposit, Retrieve, atau Inspect di dashboard Flask.
- **Autentikasi Masuk**:
  - Sidik jari luar, wajah (kamera depan), dan QR Code (untuk Deposit/Retrieve).
  - Inspect hanya memerlukan sidik jari dan wajah.
- **Di Dalam Kluis**:
  - CCTV merekam dan memverifikasi wajah. Notifikasi untuk wajah *unknown*.
  - Deposit: Tempatkan kantong, opsi Keluar Sementara.
  - Retrieve: Ambil kantong.
  - Inspect: Inspeksi tanpa barang.
- **Autentikasi Keluar**:
  - Pindai sidik jari dalam (V). Jika valid, pintu terbuka (AB → AD).
  - RFID mendeteksi kantong **setelah** petugas keluar (AE → AF), memicu notifikasi jika tidak diizinkan (AH), lalu log dicatat (AG).

## Input ke Sistem
- **Pendaftaran Kantong**: QR Code, RFID, detail barang via Flask.
- **Autentikasi Petugas**: Sidik jari (luar/dalam), wajah (depan/dalam kluis).
- **Tujuan**: Deposit, Retrieve, Inspect, Keluar Sementara via Flask.
- **Verifikasi Kantong**: QR Code di pintu, RFID untuk masuk/keluar.

## Output Sistem
- **Kontrol Akses**: Pintu terbuka jika autentikasi valid.
- **Log**: SQLite mencatat petugas, waktu, tujuan, RFID kantong, status transaksi.
- **Rekaman CCTV**: Disimpan di NVR via MEDIAMTX saat orang terdeteksi.
- **Laporan Analitik**: Bulanan, dari log SQLite (akses kluis, anomali, waktu rata-rata), ekspor PDF/CSV.
- **Dashboard**: Notifikasi real-time, status perekaman, konfirmasi anomali.

## Akses Output Sistem
- **Petugas**: Lihat umpan balik di dashboard.
- **Supervisor**: Akses log, laporan, rekaman, override anomali.
- **Tim IT**: Pemeliharaan dengan izin.
- **Auditor**: Akses log dan rekaman untuk audit.
- **Pihak Berwenang**: Akses rekaman untuk investigasi.

## Notifikasi Sistem
Notifikasi di dashboard Flask:
- **Sidik Jari**: Valid/tidak valid.
- **Wajah**: Valid/tidak valid/unknown.
- **RFID**: Kantong tidak diizinkan keluar/masuk tidak sesuai grup (hanya notifikasi, tidak menghentikan akses).
- **Deteksi Orang**: Peringatan untuk orang tak dikenal.
- **Status Perekaman**: Aktif/berhenti.

## Kemungkinan Error
- Sidik jari gagal, wajah gagal, RFID gagal, QR Code buram, kamera/NVR gagal, solenoid lock gagal, Orange Pi overheating.

## Pemeliharaan
- **Perangkat Keras**: Bersihkan sensor, uji UPS, monitor suhu.
- **Perangkat Lunak**: Perbarui OS, AI, MEDIAMTX; cadangkan SQLite.
- **Pelatihan AI**: Batch 10 karyawan via skrip Flask di PC server (GPU GTX 1650).

## Lisensi
MIT License. Lihat `LICENSE`.

## Peningkatan di Masa Depan
- **Skenario 2: Penugasan Petugas**: Supervisor memilih petugas tertentu untuk akses kluis, meningkatkan akuntabilitas.
- Notifikasi eksternal (SMS/Telegram/WhatsApp) untuk anomali.
- Analisis CCTV dengan YOLOv8 untuk deteksi tindakan (meletakkan/mengambil kantong).
- Segel pintar berbasis NFC untuk pelacakan kantong.
- Cloud backup SQLite untuk multi-kluis.
- Sensor berat di kabinet untuk verifikasi penempatan kantong.

## Model Bisnis
**Security-as-a-Service (SaaS)**:
- Biaya awal: Rp30 juta per kluis.
- Langganan bulanan untuk pemeliharaan, pembaruan AI, laporan analitik.
- Mulai dengan PoC, ekspansi ke 5–10 cabang.

## Kontak
- **Pengelola**: Jo
- **Asisten**: Bejo (bejo@donvirtus.net, WhatsApp: +62-xxx-xxx-xxxx)
