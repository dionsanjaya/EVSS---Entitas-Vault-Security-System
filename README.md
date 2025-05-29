# README: Entitas Vault Security System - Pegadaian (EVSS-Pegadaian)

## 1. Nama Sistem
**Entitas Vault Security System - Pegadaian (EVSS-Pegadaian)**  
Sistem keamanan terintegrasi untuk kluis Pegadaian, menggabungkan EAS, RFID, Computer Vision, dan IoT untuk mengamankan barang gadai seperti perhiasan dan elektronik di setiap cabang.

## 2. Tujuan Sistem Dibangun
EVSS-Pegadaian bertujuan meningkatkan keamanan dan efisiensi pengelolaan barang gadai, dengan:
- **Verifikasi Barang**: Memastikan barang (misal, cincin emas) sesuai deskripsi.
- **Pelacakan Lokasi**: Mendeteksi pergerakan barang di kluis atau keluar-masuk.
- **Keamanan**: Mencegah pengambilan tanpa izin dengan alarm dan notifikasi.
- **Verifikasi Pengambilan**: Memastikan barang yang diambil nasabah sesuai catatan.
- **Penanganan *False Negatives***: Mengatasi barang disembunyikan (misal, di wadah logam/air).
- **Integrasi IoT**: Dashboard real-time dan notifikasi, dengan akses terpusat untuk kantor pusat.

Sistem mendukung implementasi di banyak cabang, dengan opsi **cloud** (skalabel) atau **server lokal** (kontrol penuh).

---

## Opsi 1: Implementasi Berbasis Cloud

### 3. Prerequisites (Cloud)
#### Hardware
- **EAS**: Antena AM-based (misal, Sensormatic Synergy, ~Rp20 juta), tag AM (~Rp2.000 per tag).
- **RFID**: Reader UHF (misal, Zebra FX9600, ~Rp30 juta), tag UHF (misal, Impinj Monza R6-P, ~Rp5.000 per tag).
- **Computer Vision**: Kamera HD (misal, Hikvision, ~Rp3 juta), server cloud dengan GPU (misal, AWS EC2 g4dn.xlarge).
- **Sensor Tambahan**: Load cell (HX711, ~Rp150.000 per kompartemen), metal detector (~Rp15 juta-Rp75 juta).
- **IoT**: Raspberry Pi (~Rp1 juta) per cabang sebagai gateway IoT.
#### Software
- **Sistem Operasi**: Linux (Ubuntu 20.04+) untuk Raspberry Pi/server cloud.
- **Bahasa Pemrograman**: Python 3.8+, JavaScript (dashboard).
- **Library Python**:
  - `opencv-python`, `tensorflow`/`pytorch`, `pycryptodome`, `twilio`, `flask`, `boto3`, `psycopg2`.
- **Database**: PostgreSQL (AWS RDS) dengan replikasi streaming untuk cabang.
- **Web Framework**: React (opsional) untuk dashboard.
- **Cloud Services**: AWS IoT Core, AWS KMS, S3 (rekaman).
#### Lainnya
- Internet stabil (minimal 10 Mbps upload).
- Dataset barang gadai untuk pelatihan AI.

### 4. Pembangunan (Cloud)
Sistem modular:
1. **Modul EAS**: Antena deteksi tag AM di pintu kluis.
2. **Modul RFID**: Tag UHF simpan data (AES-256), reader lacak lokasi.
3. **Modul Computer Vision**: Kamera dengan YOLOv8 verifikasi barang.
4. **Modul Sensor**: Load cell dan metal detector deteksi anomali.
5. **Modul IoT**: Raspberry Pi kirim data ke AWS IoT Core, dashboard Flask/React.
6. **Database**: PostgreSQL (RDS) dengan primary instance di pusat, read replicas di cabang.
7. **Keamanan**: Enkripsi AES-256, kunci via AWS KMS, SSO untuk dashboard.

### 5. Instalasi (Cloud)
1. **Hardware**:
   - Pasang antena EAS, reader RFID, kamera, load cell, metal detector di cabang.
   - Konfigurasi Raspberry Pi sebagai gateway IoT.
2. **Cloud Setup**:
   - Buat EC2 (g4dn.xlarge), RDS (PostgreSQL), IoT Core di AWS.
3. **Software**:
   ```bash
   sudo apt update
   sudo apt install python3 python3-pip
   pip3 install opencv-python tensorflow pycryptodome twilio flask boto3 psycopg2
   ```
4. **Database**:
   - Hubungkan ke RDS:
     ```bash
     psql -h <rds_endpoint> -U <user> -d evss_pegadaian
     ```
   - Buat tabel:
     ```sql
     CREATE TABLE items (
         item_id VARCHAR(50) PRIMARY KEY,
         branch_id VARCHAR(20),
         description TEXT,
         status VARCHAR(20),
         location VARCHAR(50),
         timestamp TIMESTAMP
     );
     ```
5. **AI Model**: Latih YOLOv8, simpan di `models/yolo_pawn_items.h5`, upload ke S3.
6. **IoT**: Hubungkan Raspberry Pi ke AWS IoT Core.

### 6. Penggunaan (Cloud)
1. **Penerimaan Barang**: Tag EAS/RFID, verifikasi AI, catat berat, simpan di RDS.
2. **Penyimpanan**: RFID lacak kompartemen, sensor berat pantau.
3. **Keluar Kluis**: RFID/EAS deteksi, kamera AI cek, metal detector scan.
4. **Pengambilan Nasabah**: Verifikasi RFID/AI, nonaktifkan EAS.
5. **Dashboard**:
   - Akses via browser (misal, `https://dashboard.evss-pegadaian.com`) di komputer/smartphone.
   - Kantor pusat lihat data semua cabang, cabang lihat data lokal.
   - Notifikasi SMS/email untuk anomali.
6. **Handle Offline**:
   - Kalau internet mati, data disimpan lokal di Raspberry Pi, sinkron ke RDS pas online lagi.

---

## Opsi 2: Implementasi Tanpa Cloud (Server Lokal)

### 3. Prerequisites (Lokal)
#### Hardware
- **EAS**: Antena AM-based (~Rp20 juta), tag AM (~Rp2.000).
- **RFID**: Reader UHF (~Rp30 juta), tag UHF (~Rp5.000).
- **Computer Vision**: Kamera HD (~Rp3 juta), server lokal (PC Ubuntu, RAM 16GB, SSD 500GB, GPU GTX 1660).
- **Sensor Tambahan**: Load cell (~Rp150.000), metal detector (~Rp15 juta-Rp75 juta).
- **IoT**: Raspberry Pi (~Rp1 juta) per cabang.
- **Server Pusat**: PC Ubuntu (RAM 32GB, SSD 1TB) di kantor pusat.
#### Software
- **Sistem Operasi**: Linux (Ubuntu 20.04+).
- **Bahasa Pemrograman**: Python 3.8+, JavaScript.
- **Library Python**:
  - `opencv-python`, `tensorflow`/`pytorch`, `pycryptodome`, `twilio`, `flask`, `psycopg2`.
- **Database**: PostgreSQL lokal dengan replikasi streaming.
- **Web Framework**: React (opsional).
#### Lainnya
- Jaringan lokal/VPN (OpenVPN) untuk sinkronisasi.
- Dataset barang gadai untuk AI.

### 4. Pembangunan (Lokal)
Sistem modular:
1. **Modul EAS**: Antena deteksi tag AM.
2. **Modul RFID**: Tag UHF simpan data (AES-256), reader lacak lokasi.
3. **Modul Computer Vision**: Kamera dengan YOLOv8 verifikasi barang.
4. **Modul Sensor**: Load cell dan metal detector deteksi anomali.
5. **Modul IoT**: Raspberry Pi kelola data lokal, dashboard Flask/React.
6. **Database**: PostgreSQL di server cabang (slave), replikasi ke server pusat (master).
7. **Keamanan**: Enkripsi AES-256, kunci di file lokal, VPN untuk komunikasi.

### 5. Instalasi (Lokal)
1. **Hardware**:
   - Pasang antena EAS, reader RFID, kamera, load cell, metal detector.
   - Siapkan server lokal per cabang dan server pusat.
2. **Software**:
   ```bash
   sudo apt update
   sudo apt install python3 python3-pip
   pip3 install opencv-python tensorflow pycryptodome twilio flask psycopg2
   sudo apt install postgresql postgresql-contrib
   ```
3. **Database**:
   - Buat database di server cabang:
     ```bash
     sudo -u postgres psql -c "CREATE DATABASE evss_pegadaian;"
     ```
   - Buat tabel (sama seperti cloud).
   - Konfigurasi replikasi streaming ke server pusat via VPN.
4. **AI Model**: Latih YOLOv8, simpan di `models/yolo_pawn_items.h5`.
5. **IoT**: Hubungkan Raspberry Pi ke server cabang via LAN.

### 6. Penggunaan (Lokal)
1. **Penerimaan Barang**: Tag EAS/RFID, verifikasi AI, catat berat, simpan di database lokal.
2. **Penyimpanan**: RFID lacak kompartemen, sensor berat pantau.
3. **Keluar Kluis**: RFID/EAS deteksi, kamera AI cek, metal detector scan.
4. **Pengambilan Nasabah**: Verifikasi RFID/AI, nonaktifkan EAS.
5. **Dashboard**:
   - Akses via browser di jaringan lokal/VPN (misal, `http://192.168.x.x:5000` untuk cabang, `http://pusat.pegadaian.local:5000` untuk pusat).
   - Kantor pusat lihat data semua cabang, cabang lihat data lokal.
   - Notifikasi SMS/email untuk anomali.
6. **Handle Offline**:
   - Kalau VPN mati, data disimpan lokal, sinkron ke pusat pas koneksi balik.

**Contoh Kode Pemantauan Pintu** (Berlaku untuk Keduanya):
```python
import time
from datetime import datetime
import psycopg2
from twilio.rest import Client

# Simulasi deteksi
def detect_eas_tag(): return True  # Tag AM aktif
def read_rfid_tag(): return "ITEM123"  # Tag RFID dibaca
def verify_item_visual(image_path): return True  # AI verifikasi

# Inisialisasi Twilio
client = Client('your_account_sid', 'your_auth_token')

# Cek status izin
def check_authorization(item_id):
    try:
        conn = psycopg2.connect(dbname="evss_pegadaian", user="user", password="pass", 
                                host="localhost" if not cloud else "<rds_endpoint>")
        cursor = conn.cursor()
        cursor.execute("SELECT status FROM items WHERE item_id = %s", (item_id,))
        status = cursor.fetchone()
        conn.close()
        return status and status[0] == 'Lunas'
    except Exception as e:
        print(f"Error koneksi DB: {e}")
        return False

# Kirim notifikasi
def send_alert(item_id, authorized):
    message = f"Barang {item_id} terdeteksi di pintu kluis pada {datetime.now()}. Izin: {authorized}"
    try:
        client.messages.create(body=message, from_='+1234567890', to='+0987654321')
        print(f"Notifikasi: {message}")
    except Exception as e:
        print(f"Gagal kirim SMS: {e}")

# Loop pemantauan
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
- **Verifikasi**: Latih penggunaan kamera AI dan RFID.
- **Dashboard**: Ajarkan baca status, log, dan tangani notifikasi.
- **Alarm**: Latih prosedur saat alarm EAS/RFID (cek tas dengan metal detector).
- Durasi: 2-3 hari intensif, sesi mingguan selama 1 bulan.

### SOP Karyawan (Ringkasan)
1. **Keluar-Masuk Kluis**:
   - Scan ID RFID + PIN, lewati metal detector, catat di dashboard.
2. **Penerimaan Barang**:
   - Verifikasi AI/RFID, input data, simpan di kompartemen.
3. **Penanganan Alarm**:
   - Cek dashboard, lihat footage, koordinasi dengan manajer.

### Penyesuaian Kebiasaan
- Alur baru: Tag EAS/RFID wajib sebelum masuk kluis.
- Standar: Verifikasi visual untuk barang bernilai tinggi.
- Keamanan: Karyawan gunakan RFID + PIN untuk akses kluis.

### Dukungan
- Manual pengguna (PDF).
- Tim IT bantu 24/7 selama 3 bulan pertama.

## 8. Maintenance dan Pengembangan
### Maintenance
- **Hardware**: Periksa EAS, RFID, kamera setiap 3 bulan; kalibrasi sensor berat setiap 6 bulan.
- **Software**: Update Python library setiap 3 bulan; backup database mingguan.
- **Keamanan**:
  - Cloud: Perbarui kunci AES-256 via AWS KMS setiap 6 bulan.
  - Lokal: Perbarui kunci lokal setiap 6 bulan.
  - Audit log bulanan.

### Pengembangan
- **Fitur**: Biometrik, X-ray scanner, AI lebih canggih.
- **Skalabilitas**:
  - Cloud: Tambah instance EC2/RDS.
  - Lokal: Tambah server dan reader RFID.
- **Optimasi**: Kurangi *false positives* AI, dashboard mobile.

## Kontribusi
Fork repositori, ajukan pull request, atau laporkan isu di GitHub. Kontak tim IT Pegadaian untuk akses server atau dataset.

## Lisensi
MIT License. Lihat file `LICENSE`.
