# 🛡️ Penetration Testing: Smart Blind Stick Web Server (ESP32)
Analisis kerentanan keamanan Web Server pada perangkat IoT Tongkat Pintar (Smart Blind Stick) berbasis ESP32 terhadap serangan Brute Force dan Denial of Service (DoS).

---

## 👥 Anggota Kelompok 4
Berikut adalah tim yang terlibat dalam penyusunan laporan dan pengujian ini:

| No | Nama | NIM |
|----|------|-----|
| 1. | **Muhammad Afitfuddin** | 09030182327002 |
| 2. | **Muhammad Aren Saputra** | 09030282327026 |
| 3. | **Daniel** | [Masukkan NIM] |
| 4. | **Tri Pramisari** | 0903028327035 |
| 5. | **Fatimah Syafa Aulia** | 09030182327065 |
| 6. | **Safitri** | 09030282327041 |

---

## 📌 Latar Belakang & Tujuan
Proyek ini bertujuan untuk menguji keamanan antarmuka *Web Service* yang berfungsi sebagai dashboard monitoring untuk perangkat **Smart Blind Stick**. Web service ini memungkinkan komunikasi data telemetri secara *real-time* (status GPS, log tombol darurat).

**Tujuan Pengujian:**
1.  Melakukan **Information Gathering** untuk menemukan celah pada server.
2.  Menguji ketahanan sistem autentikasi terhadap serangan **Brute Force**.
3.  Menguji ketersediaan layanan (*availability*) saat menghadapi serangan **Denial of Service (DoS)**.

---

## 🤖 Deskripsi Perangkat (Smart Blind Stick)
Sebelum masuk ke pengujian keamanan, berikut adalah gambaran umum perangkat keras yang digunakan.

### 1. Kegunaan & Cara Kerja
Alat ini dirancang untuk membantu tunanetra dalam navigasi. Sistem bekerja dengan cara:
* **Deteksi Halangan:** Menggunakan sensor ultrasonik untuk mendeteksi objek di depan pengguna. Jika jarak terlalu dekat <= 50 cm buzzer akan aktif sebagai peringatan.
* **Monitoring Lokasi:** Perangkat dilengkapi modul GPS yang mengirimkan koordinat lokasi pengguna secara *real-time* ke Web Server ESP32.
* **Tombol Darurat:** Jika ditekan, alat akan mencatat log kejadian dan lokasi terakhir ke dashboard admin.

### 2. Skematik & Foto Alat
Berikut adalah rancangan perangkat keras dan implementasi fisiknya:

| Skematik Sederhana | Foto Alat (Implementasi) |
| :---: | :---: |
| <img src="https://github.com/user-attachments/assets/a423605a-6873-4e7d-8c3a-f58e59b4c846" width="300"> | <img src="https://github.com/user-attachments/assets/d98e4d7b-c0b1-48c3-a816-8062d7e4d8f8" width="300"> |

**Komponen Utama:**
* **ESP32 Development Board:** Sebagai otak utama (mikrokontroler) dan penyedia layanan Web Server.
* **3x Sensor Ultrasonik (HC-SR04):** Untuk mendeteksi halangan di tiga sisi (kiri, depan, kanan).
* **Modul GPS (U-blox NEO-8M):** Untuk mendapatkan koordinat lokasi pengguna secara real-time.
* **Water Level Sensor:** Untuk mendeteksi adanya genangan air di jalan.
* **Active Buzzer:** Sebagai output peringatan suara (alarm).
* **Tactile Push Button:** Berfungsi sebagai tombol darurat (Emergency Button).

---
## 🎯 Target Pengujian
Bagian ini berfokus pada sisi *software* yang berjalan di atas perangkat keras tersebut.

* **Perangkat:** ESP32 Web Server (Smart Blind Stick).
* **Fitur Web:** Login Page, Dashboard Monitoring (GPS Status, Emergency Log).
* **IP Target:** `10.172.145.82` (Berdasarkan pengujian lokal).
* **Port Terbuka:** 80/tcp (HTTP).

**Deskripsi Web Service:**
Web Service ini dirancang sebagai pusat kendali dan monitoring (*dashboard*) yang menampilkan data telemetri secara *real-time*. Sistem ini dilindungi oleh halaman login administrator untuk menjamin keamanan akses.

| Tampilan Halaman Login | Tampilan Dashboard Monitoring |
| :---: | :---: |
| <img src="https://github.com/user-attachments/assets/92c00702-3194-4041-bc8e-089907a239ad" width="100%"> | <img src="https://github.com/user-attachments/assets/42a7403f-97e4-4fe8-8034-70a0baa12d62" width="100%"> |

Pada antarmuka dashboard, terdapat fitur pemantauan utama berupa:
**Status Sistem & GPS:** Menampilkan waktu sinkronisasi WiFi, status penguncian sinyal GPS (*Locked/Searching*), dan jumlah satelit yang terhubung.
**Log Darurat:** Mencatat riwayat penekanan tombol darurat beserta koordinat lokasi kejadian yang terintegrasi langsung dengan **Google Maps** untuk mempermudah pelacakan pengguna.

---

## 🛠️ Tools & Lingkungan Pengujian
Pengujian dilakukan menggunakan sistem operasi **Kali Linux** dengan *tools* berikut:

**Nmap:** Untuk pemindaian jaringan dan port (*Information Gathering*).
**CUPP (Common User Password Profiler):** Untuk membuat wordlist password berbasis *social engineering*.
**Hydra:** Untuk melakukan serangan *Brute Force* pada form login.
**Pentmenu (Hping3):** Untuk melakukan serangan DoS (TCP SYN Flood).
**Tcpdump:** Untuk monitoring paket jaringan yang masuk.

---

## ⚔️ Skenario Serangan 1: Brute Force Attack

