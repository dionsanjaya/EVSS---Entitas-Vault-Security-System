# EVSS - Entitas-Vault-Security-System
Sistem keamanan terintegrasi untuk kluis Pegadaian, menggabungkan EAS, RFID, Computer Vision, dan IoT untuk mengamankan barang gadai seperti perhiasan dan elektronik.

# README: Entitas Vault Security System - Pegadaian (EVSS-Pegadaian)

## 1. Nama Sistem
**Entitas Vault Security System - Pegadaian (EVSS-Pegadaian)**  
Sistem keamanan terintegrasi untuk kluis Pegadaian, menggabungkan EAS, RFID, Computer Vision, dan IoT untuk mengamankan barang gadai seperti perhiasan dan elektronik. Nama "Entitas" mencerminkan sistem yang kuat, terpercaya, dan spesifik untuk kebutuhan Pegadaian sebagai entitas keuangan terkemuka.

## 2. Tujuan Sistem Dibangun
EVSS-Pegadaian dirancang untuk meningkatkan keamanan dan efisiensi pengelolaan barang gadai di kluis, dengan tujuan:
- **Verifikasi Barang**: Memastikan barang gadai (misal, cincin emas) sesuai deskripsi saat diterima dan dikembalikan.
- **Pelacakan Lokasi**: Mendeteksi pergerakan barang di dalam kluis atau saat keluar-masuk.
- **Keamanan**: Mencegah pengambilan barang tanpa izin dengan alarm dan notifikasi.
- **Verifikasi Pengambilan**: Memastikan barang yang diambil nasabah sesuai catatan.
- **Penanganan *False Negatives***: Mengatasi kasus barang disembunyikan (misal, di wadah logam atau air).
- **Integrasi IoT**: Menyediakan dashboard real-time dan notifikasi untuk pemantauan.

## 3. Prerequisites
Untuk mengembangkan, menginstal, dan menjalankan sistem ini, pastikan kebutuhan berikut terpenuhi:
### Hardware
- **EAS**: Antena AM-based (misal, Sensormatic Synergy, ~Rp20 juta), tag AM (~Rp2.000 per tag).
- **RFID**: Reader UHF (misal, Zebra FX9600, ~Rp30 juta), tag UHF (misal, Impinj Monza R6-P, ~Rp5.000 per tag).
- **Computer Vision**: Kamera HD (misal, Hikvision, ~Rp3 juta), server dengan GPU untuk model AI.
- **Sensor Tambahan**: Load cell (HX711, ~Rp150.000 per kompartemen), metal detector (~Rp15 juta-Rp75 juta).
- **IoT**: Raspberry Pi (~Rp1 juta) atau server cloud (AWS IoT).
### Software
- **Sistem Operasi**: Linux (Ubuntu 20.04+) atau Raspbian untuk Raspberry Pi.
- **Bahasa Pemrograman**: Python 3.8+, JavaScript (untuk dashboard).
- **Library Python**:
  - `opencv-python`: Untuk computer vision.
  - `tensorflow` atau `pytorch`: Untuk model AI (misal, YOLOv8).
  - `pycryptodome`: Untuk enkripsi AES-256.
  - `twilio`: Untuk notifikasi SMS/email.
  - `flask`: Untuk dashboard web sederhana.
- **Database**: PostgreSQL untuk menyimpan data barang dan log.
- **Web Framework**: React (opsional) untuk dashboard interaktif.
- **Cloud**: AWS IoT atau Azure IoT untuk integrasi.
### Lainnya
- Koneksi internet stabil untuk notifikasi dan cloud.
- Dataset gambar barang gadai untuk pelatihan model AI.

## 4. Pembangunan
Sistem dikembangkan dengan pendekatan modular:
1. **Modul EAS**:
   - Antena AM dipasang di pintu kluis untuk deteksi tag aktif.
   - Tag AM ditempelkan pada barang gadai sebagai cadangan keamanan.
2. **Modul RFID**:
   - Tag UHF menyimpan data barang (ID, deskripsi) dengan enkripsi AES-256.
   - Reader UHF di kompartemen dan pintu melacak lokasi barang.
3. **Modul Computer Vision**:
   - Kamera HD dengan model AI (YOLOv8) untuk verifikasi visual dan deteksi penyembunyian.
   - Pelatihan model menggunakan dataset barang gadai (misal, perhiasan, elektronik).
4. **Modul Sensor**:
   - Sensor berat (load cell) di kompartemen untuk deteksi pengambilan.
   - Metal detector di pintu untuk wadah logam.
5. **Modul IoT**:
   - Raspberry Pi menghubungkan semua komponen ke server cloud.
   - Dashboard web (Flask/React) menampilkan status dan notifikasi.
6. **Keamanan Data**:
   - Enkripsi AES-256 untuk data RFID dan rekaman visual.
   - Key management dengan AWS KMS atau HashiCorp Vault.

## 5. Instalasi
### Langkah-langkah Instalasi
1. **Siapkan Hardware**:
   - Pasang antena EAS di pintu kluis.
   - Pasang reader RFID di kompartemen dan pintu.
   - Pasang kamera HD di pintu dan stasiun verifikasi.
   - Pasang load cell di kompartemen dan metal detector di pintu.
   - Siapkan Raspberry Pi atau server cloud.
2. **Instal Software**:
   ```bash
   # Instal Python dan dependensi
   sudo apt update
   sudo apt install python3 python3-pip
   pip3 install opencv-python tensorflow pycryptodome twilio flask

   # Instal PostgreSQL
   sudo apt install postgresql postgresql-contrib

   # (Opsional) Instal Node.js untuk React dashboard
   curl -fsSL https://deb.nodesource.com/setup_16.x | sudo -E bash -
   sudo apt install -y nodejs
   ```
3. **Konfigurasi Database**:
   - Buat database PostgreSQL:
     ```bash
     sudo -u postgres psql -c "CREATE DATABASE evss_pegadaian;"
     ```
   - Buat tabel untuk data barang dan log:
     ```sql
     CREATE TABLE items (
         item_id VARCHAR(50) PRIMARY KEY,
         description TEXT,
         status VARCHAR(20),
         location VARCHAR(50),
         timestamp TIMESTAMP
     );
     ```
4. **Siapkan Model AI**:
   - Unduh atau latih model YOLOv8 dengan dataset barang gadai.
   - Simpan model di direktori proyek: `models/yolo_pawn_items.h5`.
5. **Konfigurasi IoT**:
   - Hubungkan Raspberry Pi ke EAS, RFID, kamera, dan sensor.
   - Konfigurasi AWS IoT untuk komunikasi cloud.

## 6. Penggunaan
1. **Penerimaan Barang**:
   - Tempelkan tag EAS dan RFID pada barang gadai.
   - Gunakan kamera untuk verifikasi visual dengan AI.
   - Catat berat barang dengan sensor load cell.
   - Simpan data di database (ID, deskripsi, lokasi).
2. **Penyimpanan**:
   - RFID reader di kompartemen lacak lokasi barang.
   - Sensor berat pantau keberadaan barang.
3. **Keluar Kluis**:
   - RFID dan EAS deteksi barang di pintu kluis.
   - Kamera AI verifikasi barang dan cek aktivitas mencurigakan.
   - Metal detector deteksi wadah logam.
   - Notifikasi dikirim via SMS/email jika tanpa izin.
4. **Pengambilan Nasabah**:
   - Verifikasi data RFID dan visual AI cocok dengan catatan.
   - Nonaktifkan tag EAS dengan alat demagnetisasi.
5. **Dashboard**:
   - Akses dashboard web (misal, `http://localhost:5000`) untuk lihat status barang, log, dan rekaman kamera.

**Contoh Kode Pemantauan Pintu**:
```python
import time
from datetime import datetime
import sqlite3
from twilio.rest import Client

def detect_eas_tag(): return True  # Simulasi tag AM aktif
def read_rfid_tag(): return "ITEM123"  # Simulasi ID RFID
def verify_item_visual(image_path): return True  # Simulasi AI

client = Client('your_account_sid', 'your_auth_token')

def check_authorization(item_id):
    conn = sqlite3.connect('evss_pegadaian.db')
    cursor = conn.cursor()
    cursor.execute("SELECT status FROM items WHERE item_id = ?", (item_id,))
    status = cursor.fetchone()
    conn.close()
    return status and status[0] == 'Lunas'

def send_alert(item_id, authorized):
    message = f"Barang {item_id} terdeteksi di pintu kluis pada {datetime.now()}. Izin: {authorized}"
    client.messages.create(body=message, from_='+1234567890', to='+0987654321')
    print(f"Notifikasi: {message}")

while True:
    item_id = read_rfid_tag()
    eas_detected = detect_eas_tag()
    visual_verified = verify_item_visual('gate_image.jpg')
    if item_id or eas_detected:
        authorized = check_authorization(item_id) if item_id else False
        if authorized and visual_verified:
            print(f"Barang {item_id} diizinkan keluar.")
        else:
            send_alert(item_id or "Unknown", False)
            print("ALARM: Barang tanpa izin!")
    time.sleep(1)
```

## 7. Penyesuaian Pengguna
Untuk kelancaran sistem, karyawan Pegadaian perlu pelatihan dan penyesuaian:
- **Pelatihan Karyawan**:
  - **Verifikasi Barang**: Latih penggunaan kamera AI dan alat RFID saat menerima barang.
  - **Penggunaan Dashboard**: Ajarkan cara baca status barang, log, dan tangani notifikasi.
  - **Penanganan Alarm**: Latih prosedur saat alarm EAS/RFID berbunyi (misal, periksa tas dengan metal detector).
  - Durasi: 2-3 hari pelatihan intensif, plus sesi mingguan selama 1 bulan.
- **Penyesuaian Kebiasaan**:
  - Alur kerja baru: Semua barang wajib diberi tag EAS dan RFID sebelum masuk kluis.
  - Standar operasional: Verifikasi visual wajib untuk barang bernilai tinggi (misal, emas).
  - Kebiasaan keamanan: Karyawan wajib gunakan kartu RFID + PIN untuk akses kluis.
- **Dukungan**: Sediakan manual pengguna (PDF) dan tim IT untuk bantuan 24/7 selama 3 bulan pertama.

## 8. Maintenance dan Pengembangan
### Maintenance
- **Hardware**:
  - Periksa antena EAS, reader RFID, dan kamera setiap 3 bulan untuk fungsi normal.
  - Kalibrasi sensor berat setiap 6 bulan untuk akurasi.
  - Ganti tag EAS/RFID yang rusak (misal, akibat air/logam).
- **Software**:
  - Update library Python (`pip install --upgrade`) setiap 3 bulan.
  - Backup database PostgreSQL mingguan.
  - Monitor server cloud (AWS IoT) untuk performa dan keamanan.
- **Keamanan**:
  - Perbarui kunci AES-256 setiap 6 bulan via AWS KMS.
  - Audit log akses kluis bulanan untuk deteksi anomali.

### Pengembangan
- **Fitur Baru**:
  - Tambah autentikasi biometrik (sidik jari) untuk akses kluis.
  - Integrasikan X-ray scanner untuk deteksi barang tersembunyi.
  - Kembangkan model AI untuk kenali lebih banyak jenis barang.
- **Skalabilitas**:
  - Perluas sistem ke cabang lain dengan server cloud tambahan.
  - Tambah reader RFID untuk kluis besar.
- **Optimasi**:
  - Tingkatkan model AI untuk kurangi *false positives* dalam deteksi visual.
  - Optimasi dashboard untuk akses mobile.

## Kontribusi
Silakan fork repositori ini di GitHub, ajukan pull request untuk perbaikan, atau laporkan isu di tab Issues. Kontak tim IT Pegadaian untuk akses server atau dataset pelatihan AI.

## Lisensi
MIT License. Lihat file `LICENSE` untuk detail.
