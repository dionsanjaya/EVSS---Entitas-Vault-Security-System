# README: Entitas Vault Security System - Pegadaian (EVSS-Pegadaian)

## 1. Nama Sistem
**Entitas Vault Security System - Pegadaian (EVSS-Pegadaian)**  
Sistem keamanan terintegrasi untuk kluis Pegadaian, menggabungkan EAS, RFID, Computer Vision, dan IoT untuk mengamankan barang gadai seperti perhiasan dan elektronik. Nama "Entitas" mencerminkan sistem yang kuat, terpercaya, dan dirancang khusus untuk Pegadaian.

## 2. Tujuan Sistem Dibangun
EVSS-Pegadaian bertujuan meningkatkan keamanan dan efisiensi pengelolaan barang gadai di kluis, dengan:
- **Verifikasi Barang**: Memastikan barang (misal, cincin emas) sesuai deskripsi saat diterima dan dikembalikan.
- **Pelacakan Lokasi**: Mendeteksi pergerakan barang di kluis atau saat keluar-masuk.
- **Keamanan**: Mencegah pengambilan tanpa izin dengan alarm dan notifikasi.
- **Verifikasi Pengambilan**: Memastikan barang yang diambil nasabah sesuai catatan.
- **Penanganan *False Negatives***: Mengatasi kasus barang disembunyikan (misal, di wadah logam atau air).
- **Integrasi IoT**: Menyediakan dashboard real-time dan notifikasi untuk pemantauan.

Sistem ini mendukung dua opsi implementasi: **berbasis cloud** (untuk skalabilitas) dan **server lokal** (untuk kontrol penuh tanpa internet).

---

## Opsi 1: Implementasi Berbasis Cloud

### 3. Prerequisites (Cloud)
#### Hardware
- **EAS**: Antena AM-based (misal, Sensormatic Synergy, ~Rp20 juta), tag AM (~Rp2.000 per tag).
- **RFID**: Reader UHF (misal, Zebra FX9600, ~Rp30 juta), tag UHF (misal, Impinj Monza R6-P, ~Rp5.000 per tag).
- **Computer Vision**: Kamera HD (misal, Hikvision, ~Rp3 juta), server cloud dengan GPU (misal, AWS EC2 g4dn.xlarge).
- **Sensor Tambahan**: Load cell (HX711, ~Rp150.000 per kompartemen), metal detector (~Rp15 juta-Rp75 juta).
- **IoT**: Raspberry Pi (~Rp1 juta) sebagai gateway IoT.
#### Software
- **Sistem Operasi**: Linux (Ubuntu 20.04+) untuk Raspberry Pi atau server cloud.
- **Bahasa Pemrograman**: Python 3.8+, JavaScript (untuk dashboard).
- **Library Python**:
  - `opencv-python`: Untuk computer vision.
  - `tensorflow` atau `pytorch`: Untuk model AI (misal, YOLOv8).
  - `pycryptodome`: Untuk enkripsi AES-256.
  - `twilio`: Untuk notifikasi SMS/email.
  - `flask`: Untuk dashboard web sederhana.
- **Database**: PostgreSQL (dihosting di AWS RDS).
- **Web Framework**: React (opsional) untuk dashboard interaktif.
- **Cloud Services**: AWS IoT Core, AWS KMS (key management), S3 (penyimpanan rekaman).
#### Lainnya
- Koneksi internet stabil (minimal 10 Mbps upload).
- Dataset gambar barang gadai untuk pelatihan model AI.

### 4. Pembangunan (Cloud)
Sistem dibangun modular:
1. **Modul EAS**: Antena di pintu kluis deteksi tag AM untuk alarm cepat.
2. **Modul RFID**: Tag UHF simpan data barang (enkripsi AES-256), reader lacak lokasi.
3. **Modul Computer Vision**: Kamera dengan YOLOv8 verifikasi barang dan deteksi penyembunyian.
4. **Modul Sensor**: Load cell deteksi pengambilan, metal detector cek wadah logam.
5. **Modul IoT**: Raspberry Pi kirim data ke AWS IoT Core, dashboard Flask/React tampilkan status.
6. **Keamanan**: Enkripsi AES-256, kunci dikelola via AWS KMS.

### 5. Instalasi (Cloud)
1. **Siapkan Hardware**:
   - Pasang antena EAS, reader RFID, kamera, load cell, dan metal detector.
   - Konfigurasi Raspberry Pi sebagai gateway IoT.
2. **Setup Cloud**:
   - Buat instance AWS EC2 (g4dn.xlarge untuk AI) dan RDS (PostgreSQL).
   - Konfigurasi AWS IoT Core untuk komunikasi IoT.
3. **Instal Software**:
   ```bash
   sudo apt update
   sudo apt install python3 python3-pip
   pip3 install opencv-python tensorflow pycryptodome twilio flask boto3
   sudo apt install postgresql-client
   ```
4. **Konfigurasi Database**:
   - Hubungkan ke AWS RDS:
     ```bash
     psql -h <rds_endpoint> -U <user> -d evss_pegadaian
     ```
   - Buat tabel:
     ```sql
     CREATE TABLE items (
         item_id VARCHAR(50) PRIMARY KEY,
         description TEXT,
         status VARCHAR(20),
         location VARCHAR(50),
         timestamp TIMESTAMP
     );
     ```
5. **Siapkan Model AI**:
   - Latih YOLOv8 dengan dataset barang gadai, simpan di `models/yolo_pawn_items.h5`.
6. **Konfigurasi IoT**:
   - Hubungkan Raspberry Pi ke AWS IoT Core dengan sertifikat MQTT.

### 6. Penggunaan (Cloud)
1. **Penerimaan Barang**:
   - Tempel tag EAS dan RFID, verifikasi visual dengan kamera AI.
   - Catat berat barang, simpan data di AWS RDS.
2. **Penyimpanan**: RFID lacak kompartemen, sensor berat pantau barang.
3. **Keluar Kluis**: RFID/EAS deteksi di pintu, kamera AI cek, metal detector scan logam.
4. **Pengambilan Nasabah**: Verifikasi RFID/AI, nonaktifkan tag EAS.
5. **Dashboard**:
   - Akses via browser di komputer/smartphone (misal, `https://dashboard.evss-pegadaian.com`).
   - Lihat status barang, log, rekaman kamera, dan notifikasi.

---

## Opsi 2: Implementasi Tanpa Cloud (Server Lokal)

### 3. Prerequisites (Lokal)
#### Hardware
- **EAS**: Antena AM-based (misal, Sensormatic Synergy, ~Rp20 juta), tag AM (~Rp2.000 per tag).
- **RFID**: Reader UHF (misal, Zebra FX9600, ~Rp30 juta), tag UHF (~Rp5.000 per tag).
- **Computer Vision**: Kamera HD (misal, Hikvision, ~Rp3 juta), server lokal (PC dengan RAM 16GB, SSD 500GB, GPU NVIDIA GTX 1660).
- **Sensor Tambahan**: Load cell (HX711, ~Rp150.000 per kompartemen), metal detector (~Rp15 juta-Rp75 juta).
- **IoT**: Raspberry Pi (~Rp1 juta) sebagai pusat kontrol.
#### Software
- **Sistem Operasi**: Linux (Ubuntu 20.04+) untuk server lokal dan Raspberry Pi.
- **Bahasa Pemrograman**: Python 3.8+, JavaScript (untuk dashboard).
- **Library Python**:
  - `opencv-python`, `tensorflow`/`pytorch`, `pycryptodome`, `twilio`, `flask`.
- **Database**: PostgreSQL (lokal).
- **Web Framework**: React (opsional) untuk dashboard.
#### Lainnya
- Jaringan lokal (LAN/Wi-Fi) untuk akses dashboard.
- Dataset gambar barang gadai untuk pelatihan model AI.

### 4. Pembangunan (Lokal)
Sistem dibangun modular:
1. **Modul EAS**: Antena di pintu kluis deteksi tag AM.
2. **Modul RFID**: Tag UHF simpan data (enkripsi AES-256), reader lacak lokasi.
3. **Modul Computer Vision**: Kamera dengan YOLOv8 verifikasi barang dan deteksi penyembunyian.
4. **Modul Sensor**: Load cell deteksi pengambilan, metal detector cek logam.
5. **Modul IoT**: Raspberry Pi kelola data lokal, dashboard Flask/React di server lokal.
6. **Keamanan**: Enkripsi AES-256, kunci dikelola via file lokal terenkripsi.

### 5. Instalasi (Lokal)
1. **Siapkan Hardware**:
   - Pasang antena EAS, reader RFID, kamera, load cell, dan metal detector.
   - Siapkan server lokal (PC Ubuntu) dan Raspberry Pi.
2. **Instal Software**:
   ```bash
   sudo apt update
   sudo apt install python3 python3-pip
   pip3 install opencv-python tensorflow pycryptodome twilio flask
   sudo apt install postgresql postgresql-contrib
   ```
3. **Konfigurasi Database**:
   - Buat database lokal:
     ```bash
     sudo -u postgres psql -c "CREATE DATABASE evss_pegadaian;"
     ```
   - Buat tabel:
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
   - Latih YOLOv8, simpan di `models/yolo_pawn_items.h5`.
5. **Konfigurasi IoT**:
   - Hubungkan Raspberry Pi ke EAS, RFID, kamera, sensor via LAN.

### 6. Penggunaan (Lokal)
1. **Penerimaan Barang**:
   - Tempel tag EAS dan RFID, verifikasi dengan kamera AI.
   - Catat berat, simpan data di database lokal.
2. **Penyimpanan**: RFID lacak kompartemen, sensor berat pantau.
3. **Keluar Kluis**: RFID/EAS deteksi di pintu, kamera AI cek, metal detector scan.
4. **Pengambilan Nasabah**: Verifikasi RFID/AI, nonaktifkan EAS.
5. **Dashboard**:
   - Akses via browser di komputer/smartphone di jaringan lokal (misal, `http://192.168.x.x:5000`).
   - Lihat status barang, log, rekaman, dan notifikasi.
   - (Masa depan) Kembangkan aplikasi mobile untuk akses dashboard.

**Contoh Kode Pemantauan Pintu** (Berlaku untuk Keduanya):
```python
import time
from datetime import datetime
import sqlite3
from twilio.rest import Client

def detect_eas_tag(): return True  # Simulasi tag AM
def read_rfid_tag(): return "ITEM123"  # Simulasi RFID
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
### Pelatihan Karyawan
- **Verifikasi Barang**: Latih penggunaan kamera AI dan alat RFID.
- **Dashboard**: Ajarkan baca status barang, log, dan tangani notifikasi.
- **Alarm**: Latih prosedur saat alarm EAS/RFID (misal, cek tas dengan metal detector).
- Durasi: 2-3 hari intensif, plus sesi mingguan selama 1 bulan.
### Penyesuaian Kebiasaan
- Alur baru: Semua barang diberi tag EAS/RFID sebelum masuk kluis.
- Standar: Verifikasi visual wajib untuk barang bernilai tinggi.
- Keamanan: Karyawan gunakan kartu RFID + PIN untuk akses kluis.
### Dukungan
- Manual pengguna (PDF).
- Tim IT bantu 24/7 selama 3 bulan pertama.

## 8. Maintenance dan Pengembangan
### Maintenance
- **Hardware**: Periksa EAS, RFID, kamera setiap 3 bulan; kalibrasi sensor berat setiap 6 bulan; ganti tag rusak.
- **Software**: Update library Python setiap 3 bulan; backup database mingguan.
- **Keamanan**:
  - Cloud: Perbarui kunci AES-256 via AWS KMS setiap 6 bulan.
  - Lokal: Perbarui kunci di file terenkripsi setiap 6 bulan.
  - Audit log akses bulanan.
### Pengembangan
- **Fitur Baru**: Autentikasi biometrik, X-ray scanner, model AI lebih canggih.
- **Skalabilitas**:
  - Cloud: Tambah instance EC2/RDS untuk cabang baru.
  - Lokal: Tambah server lokal dan reader RFID.
- **Optimasi**: Kurangi *false positives* AI, optimasi dashboard untuk mobile.

## Kontribusi
Fork repositori di GitHub, ajukan pull request, atau laporkan isu di tab Issues. Kontak tim IT Pegadaian untuk akses server atau dataset AI.

## Lisensi
MIT License. Lihat file `LICENSE` untuk detail.
