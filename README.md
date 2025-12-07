# 🛡️ Penetration Testing: Smart Blind Stick Web Server (ESP32)
Analisis keamanan Web Server pada perangkat IoT Smart Blind Stick berbasis ESP32 terhadap serangan Brute Force dan Denial of Service (DoS).

---

## 👥 Anggota Kelompok 4
Berikut adalah tim yang terlibat dalam penyusunan laporan dan pengujian ini:

| No | Nama | NIM |
|----|------|-----|
| 1. | **Muhammad Afitfuddin** | 09030182327002 |
| 2. | **Muhammad Aren Saputra** | 09030282327026 |
| 3. | **Daniel Juan Karel P S** | 09030582327092 |
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

### 1. Cara Kerja
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

Pada antarmuka dashboard, terdapat fitur pemantauan utama berupa:<br>
**Status Sistem & GPS:** Menampilkan waktu sinkronisasi WiFi, status penguncian sinyal GPS (*Locked/Searching*), dan jumlah satelit yang terhubung. <br>
**Log Darurat:** Mencatat riwayat penekanan tombol darurat beserta koordinat lokasi kejadian yang terintegrasi langsung dengan **Google Maps** untuk mempermudah pelacakan pengguna.

---

## 🛠️ Tools & Lingkungan Pengujian
Pengujian dilakukan menggunakan sistem operasi **Kali Linux** dengan *tools* berikut:

1. **Nmap:** Untuk pemindaian jaringan dan port (*Information Gathering*).<br>
2. **CUPP (Common User Password Profiler):** Untuk membuat wordlist password berbasis *social engineering*.<br>
3. **Hydra:** Untuk melakukan serangan *Brute Force* pada form login.<br>
4. **Pentmenu (Hping3):** Untuk melakukan serangan DoS (TCP SYN Flood).<br>
5. **Tcpdump:** Untuk monitoring paket jaringan yang masuk.<br>

---

## ⚔️ Skenario Serangan 1: Brute Force Attack

### Apa itu Brute Force?
Secara sederhana, **Brute Force** adalah metode serangan "coba-coba" yang agresif. Bayangkan kita menemukan sebuah gembok angka 4 digit, tapi tidak tahu kodenya. Kamu kemudian mencoba memutar angka mulai dari `0000`, `0001`, `0002`, terus-menerus hingga gembok terbuka. Dalam konteks keamanan siber, penyerang menggunakan komputer untuk menebak kombinasi *username* dan *password* secara otomatis dan sangat cepat hingga menemukan pasangan yang cocok. Serangan ini memanfaatkan lemahnya pembuatan password dan tidak adanya pembatasan percobaan login (*rate limiting*) pada sistem.

---

### 🚀 Langkah-Langkah Pengujian

#### 1. Information Gathering (Pengumpulan Informasi)
Sebelum menyerang, kami memastikan target hidup dan mencari port yang terbuka. Kami menggunakan `ping` untuk cek koneksi dan `nmap` untuk melihat layanan yang berjalan. Dari hasil scan, ditemukan **Port 80 (HTTP)** terbuka.

| Tes Koneksi (Ping & Nmap) | Memvalidasi port terbuka di arduino ide  |
| :---: | :---: |
| <img src="https://github.com/user-attachments/assets/222d3bb2-80fd-4d50-8bed-592a3f0c89b7" width="100%"> | <img src="https://github.com/user-attachments/assets/6fcd2a40-47dd-4c1d-9155-2eb04e7814d6" width="100%"> |
| *Memastikan ESP32 aktif* | *Menemukan Port 80 Open* |

#### 2. Analisis Struktur HTML (Web Reconnaissance)
Setelah memastikan layanan web aktif, kami melakukan kunjungan langsung ke halaman web service untuk mengetahui struktur sederhana dari HTML-nya. Melalui fitur **Inspect Element**, kami menemukan arsitektur *body* HTML yang menunjukkan bahwa form login mengirimkan data menggunakan metode **POST** dengan nama parameter input `username` dan `password`. Informasi ini krusial untuk menyusun perintah serangan Hydra nantinya.

| Arsitektur HTML Body (Inspect Element) |
| :---: |
| <img src="https://github.com/user-attachments/assets/33d19abb-767b-4077-9f00-3c772ec52839" width="100%"> |
| *Identifikasi parameter login & metode POST* |

#### 3. Persiapan Wordlist
Kami menggunakan tools **CUPP** (*Common User Password Profiler*) untuk membuat "kamus"  username dan password. Tools ini memprediksi  username dan password  berdasarkan data target seperti nama, tanggal lahir, dan nama hewan peliharaan dll. Setelah proses input selesai, tools akan menghasilkan file wordlist yang berisi ribuan kombinasi username dan password.

| Install & Menjalankan CUPP | Input Data Target |
| :---: | :---: |
| <img src="https://github.com/user-attachments/assets/1a71b3c7-56ee-42ec-9785-1118e9aa5d50" width="100%"> | <img src="https://github.com/user-attachments/assets/fad91a04-9876-4bfe-8100-2261a2b6f729" width="100%"> |
| *Persiapan tools generator* | *Membuat wordlist untuk username* |

| Input Data Target | Lokasi Penyimpanan Wordlist |
| :---: | :---: |
| <img src="https://github.com/user-attachments/assets/8468d66a-1338-4224-95f1-7dcddc45eca7" width="100%"> | <img src="https://github.com/user-attachments/assets/1920be1e-311d-4a46-b274-09967d1e318d" width="100%"> |
| *Membuat wordlist untuk password* | *File `user.txt` tersimpan di direktori* |

#### 4. Instalasi & Eksekusi Serangan (Hydra)
Sebelum melakukan serangan, pastikan tools **Hydra** sudah terinstall. Jika belum, jalankan perintah instalasi berikut di terminal:

`bash
sudo apt install hydra`

Eksekusi Serangan (Exploitation) 
<br>**Command:**<br>
```bash
hydra -t 1 -L user.txt -P password.txt 10.172.145.82 http-post-form '/login:username=^USER^&password=^PASS^:Username/Password Salah!'`
```

Setelah terinstall, kami menjalankan serangan dengan memadukan wordlist yang sudah dibuat (`user.txt` dan `password.txt`) dengan parameter HTML yang ditemukan sebelumnya.<br> 
**Hasil:**<br><img width="1366" height="654" alt="Langkah 7  Menemukan user dan password yang valid" src="https://github.com/user-attachments/assets/e409132d-9585-49f0-944f-772c22033caa" />
*Hydra berhasil menemukan kredensial yang valid.* 


## 💥 Skenario Serangan 2: Denial of Service (DoS)

### Apa itu DoS?
**Denial of Service (DoS)** adalah serangan yang bertujuan melumpuhkan layanan agar tidak bisa diakses oleh pengguna yang sah. Bayangkan sebuah pintu masuk gedung yang hanya muat untuk satu orang. Jika kita mengirim ribuan "orang palsu" untuk berdesak-desakan di pintu tersebut secara bersamaan, maka orang asli (pengguna sah) tidak akan bisa masuk. Dalam pengujian ini, kami menggunakan metode **TCP SYN Flood**. Kami membanjiri ESP32 dengan permintaan koneksi palsu hingga memori/CPU-nya penuh (*Resource Exhaustion*), menyebabkan sistem melambat, *hang*, atau *reboot*.

---

### 🚀 Langkah-Langkah Pengujian

#### 1. Instalasi & Setup Tools (Pentmenu)
Kami menggunakan tools **Pentmenu** untuk mempermudah serangan. Pentmenu sebenarnya menjalankan tools `hping3` di latar belakang untuk mengirim paket data dalam jumlah masif.

**Perintah Instalasi & Izin Eksekusi:**
| Download & Eksekusi Pentmenu | Tampilan Utama Tools |
| :---: | :---: |
| <img src="https://github.com/user-attachments/assets/47288dc2-9144-4c5b-b883-d16b6a756115" width="100%"> | <img src="https://github.com/user-attachments/assets/cad8327c-ce01-415c-857e-5377cd0dd355" width="100%"> |
| *Mengunduh script & memberi izin eksekusi (`chmod +x`)* | *Antarmuka awal Pentmenu siap digunakan* |

#### 2. Konfigurasi & Eksekusi Serangan (TCP SYN Flood)
Setelah masuk ke menu utama, kami melakukan konfigurasi serangan. Kami memilih modul **DoS** > **TCP SYN Flood**. Metode ini dipilih karena sangat efektif untuk melumpuhkan perangkat IoT dengan sumber daya terbatas seperti ESP32. Kami memasukkan **IP Target** (`10.172.145.82`) dan **Port** (`80`). Setelah dikonfirmasi, *Pentmenu* (via hping3) mulai membanjiri target dengan ribuan paket SYN palsu per detik.

| Setup Target IP & Port | Proses Flooding Berjalan |
| :---: | :---: |
| <img src="https://github.com/user-attachments/assets/8bec5850-37a3-433b-938a-05e14577db54" width="100%"> | <img src="https://github.com/user-attachments/assets/cdb3f5be-a66c-41f5-8367-d4bb092c6264" width="100%"> |
| *Input IP ESP32 dan Port 80* | *Mengirim paket SYN secara masif* |

#### 3. Monitoring Lalu Lintas Jaringan (Tcpdump)
Untuk memvalidasi bahwa serangan benar-benar terkirim, kami melakukan pemantauan jaringan menggunakan **Tcpdump** di sisi penyerang. Dengan memfilter paket yang menuju ke IP target (`dst host 10.172.145.82`), terlihat adanya banjir data (*flooding*) yang masuk secara terus-menerus dengan kecepatan tinggi. Ini adalah bukti visual bahwa server sedang dihujani permintaan palsu.

**Perintah Monitoring:**
```bash
sudo tcpdump -i eth1 -n "dst host 10.172.145.82"
```
| Monitoring Traffic (Bagian 1) | Monitoring Traffic (Bagian 2) |
| :---: | :---: |
| <img src="https://github.com/user-attachments/assets/8719c09b-52fd-4044-8831-44c0f7cddf73" width="100%"> | <img src="https://github.com/user-attachments/assets/cdf6dbc8-fd9f-432e-80ce-44ee6f6f6c16" width="100%"> |
| *Paket TCP SYN yang terkirim* | *Visualisasi banjir paket data (Flood)* |


#### 4. Dampak Serangan (Impact Analysis)
Untuk memverifikasi keberhasilan serangan, kami membandingkan aksesibilitas Web Server sebelum dan sesudah serangan dilakukan. <br>
**Sebelum Serangan:** Halaman Login dan Dashboard dapat diakses dengan lancar. Data telemetri seperti Status GPS (*Locked*) dan jumlah satelit terbaca secara *real-time*.<br>
| Kondisi Normal (Login) | Kondisi Normal (Dashboard) |
| :---: | :---: |
| <img src="https://github.com/user-attachments/assets/27adc4b5-b30c-4d4d-8353-3b409cb46c13" width="100%"> | <img src="https://github.com/user-attachments/assets/b660e662-0631-44ae-b5cc-aa7cfcf319bc" width="100%"> |
| *Halaman Login* | *Dashboard menampilkan data GPS & Log* |

**Sesudah Serangan:** Web Server menjadi tidak responsif (*Unreachable*). Browser menampilkan pesan error karena ESP32 mengalami *Resource Exhaustion* (kehabisan memori) akibat kebanjiran paket data.<br>
| Situs Tidak Dapat Dijangkau | Loading Terus Menerus |
| :---: | :---: |
| <img src="https://github.com/user-attachments/assets/6b1835c3-260f-4422-91e3-14034432ff1d" width="100%"> | <img src="https://github.com/user-attachments/assets/a4dbc783-a417-48cc-9ac8-7bd71810c5b0" width="100%"> |
| *Connection Timed Out* | *Sistem Gagal Merespons (Hang)* |


#### 5. Penghentian Serangan
Setelah dampak serangan terdokumentasi, proses *flooding* dihentikan dengan menekan `Ctrl+C` pada terminal Pentmenu agar jaringan kembali normal. Namun, pada beberapa kasus, ESP32 memerlukan *restart* manual untuk memulihkan fungsinya secara total.

| Menghentikan Script Attack |
| :---: |
| <img src="https://github.com/user-attachments/assets/b0686004-a1aa-47c1-9e2e-e69fd4f94129" width="100%"> |
| *Menghentikan proses hping3* |


## 📝 Kesimpulan
Berdasarkan hasil pengujian penetrasi yang telah kami lakukan, dapat disimpulkan bahwa antarmuka Web Server pada Smart Blind Stick berbasis ESP32 memiliki kerentanan keamanan yang signifikan. Hal ini dibuktikan dengan keberhasilan skenario serangan Brute Force yang mampu menembus sistem autentikasi akibat lemahnya manajemen password, serta serangan Denial of Service (DoS) dengan metode TCP SYN Flood yang sukses melumpuhkan ketersediaan layanan. Banjir paket data dari serangan tersebut menyebabkan mikrokontroler mengalami kehabisan sumber daya (*resource exhaustion*) yang berujung pada kegagalan fungsi sistem (*downtime*) dan *reboot* otomatis.

## 🛡️ Saran & Mitigasi
Untuk pengembangan selanjutnya, kami sarankan sistem keamanan jaringan pada perangkat ini perlu ditingkatkan lagi guna menutup celah serangan yang telah ditemukan. Langkah antisipasi yang disarankan meliputi penggunaan kombinasi password yang kompleks (hindari password default) dan penerapan logika pembatasan percobaan login (*Rate Limiting*) pada kode program untuk mencegah Brute Force. Selain itu, optimasi manajemen memori dan pengaturan *timeout* koneksi pada ESP32 juga diperlukan agar perangkat lebih tahan terhadap tumpukan permintaan koneksi palsu yang menyebabkan *hang*.
