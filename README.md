# VOIP-Project

Dokumentasi konfigurasi server VoIP Asterisk dengan PJSIP dan simulasi GSM Gateway.

## 1. Instalasi Server Asterisk

Perbarui sistem dan instal paket Asterisk beserta dependensi suara dan modul tambahan:

```bash
sudo apt update
sudo apt install asterisk asterisk-core-sounds-en-gsm asterisk-modules -y
instal modul voicemail
sudo apt install asterisk-sounds-main asterisk-sounds-core -y
```
Cek status service Asterisk untuk memastikan server sudah aktif:
```bash
sudo systemctl status asterisk
```

di dalam modul asterisk ada beberapa modul yang tidak kita pakai contohnya ialah chan_sip
bila muncul di  -rvvvvvv bisa kita matikan

```bash
sudo nano /etc/asterisk/modules.conf

tambahkan ini
noload => chan_sip.so
```
lalu lakukan restart
```bash
sudo systemctl restart asterisk
```

untuk melakukan GSM diperlukan panggilan dial dengan 908123456789

## 2. konfigurasi sip

Konfigurasi ekstensi pengguna (endpoint) dan otentikasi pada file PJSIP
```bash
sudo nano /etc/asterisk/pjsip.conf
```
Konfigurasi aturan alur panggilan (dialplan) pada file extensions
```bash
sudo nano /etc/asterisk/extensions.conf
```
Konfigurasi mailbox untuk pengguna (jika menggunakan fitur voicemail)
```bash
sudo nano /etc/asterisk/voicemail.conf
```
Setelah mengubah file konfigurasi di atas, muat ulang (reload) Asterisk
```bash
sudo asterisk -rx "reload"
```
## 3. konfigurasi client
Ekstensi 1000 & 2000: Digunakan untuk pendaftaran client (MicroSIP / Zoiper).

Ekstensi 800: Fitur Echo Test untuk pengujian latency dan audio stream.

Ekstensi _9X.: Simulasi panggilan keluar (outbound call via GSM Gateway), contoh dial: 908123456789.

## 4. Pengambilan & Analisis Data QoS di Wireshark

1. Mulai Merekam Trafik (Start Capture)
-Buka Wireshark, pilih antarmuka jaringan yang sedang aktif digunakan untuk komunikasi (misalnya koneksi Wi-Fi atau Ethernet). Klik dua kali pada antarmuka tersebut, atau klik tombol ikon Sirip Hiu Biru di pojok kiri atas. Segera lakukan aktivitas jaringan yang ingin diukur (misalnya melakukan panggilan antar ekstensi).

2. Hentikan Rekaman (Stop Capture):Setelah panggilan atau aktivitas jaringan selesai, kembali ke jendela Wireshark dan klik ikon Kotak Merah di pojok kiri atas untuk menghentikan perekaman trafik.

3. Filter Paket Spesifik:Di kotak teks berlabel Apply a display filter... di bagian atas, ketik protokol yang ingin dianalisis. Jika ingin menganalisis kualitas media/suara, ketik rtp. Jika ingin melihat proses inisiasinya, ketik sip. Tekan Enter.

4. Buka Alat Analisis QoS (Jitter, Loss, Delay):Klik menu Telephony di panel atas > arahkan ke RTP > pilih RTP Streams. Akan muncul jendela baru berisi daftar aliran komunikasi. Pilih aliran jaringan yang terdeteksi, lalu klik tombol Analyze (atau Find Reverse untuk melihat komunikasi dua arah). Wireshark akan otomatis menyajikan tabel berisi perhitungan Jitter, Max Delta (latensi/delay tertinggi antar paket), dan persentase Packet Loss.

## 5. perbedaan UDP dan TLS 
A. Konfigurasi PJSIP (pjsip.conf)
Di server Asterisk, dipastikan transport UDP diaktifkan pada port 5060:
```bash
[transport-udp]
type=transport
protocol=udp
bind=0.0.0.0:5060
```
B. Konfigurasi Client (MicroSIP / Zoiper)
SIP Server: 192.168.1.10 (tanpa port tambahan/port 5060)

Transport: UDP
C. Langkah Pengujian & Analisis di Wireshark
Buka Wireshark, aktifkan capture pada antarmuka jaringan yang terhubung ke server.
Gunakan filter: sip
Lakukan registrasi akun atau panggil nomor 800 (Echo Test).
Hasil Analisis Kerentanan (UDP):Protokol: Paket langsung terdeteksi sebagai SIP.Analisis Header: Diklik pada paket REGISTER atau INVITE $\rightarrow$ perhatikan bagian Session Initiation Protocol.
Header Eksplisit: Parameter From:, To:, User-Agent, dan nomor ekstensi (1000/2000) terbaca jelas dalam bentuk plaintext tanpa membutuhkan kunci keamanan apapun.


TLS 
Karena pada Ubuntu versi baru script ast_tls_cert tidak tersedia secara default, sertifikat dibuat manual menggunakan OpenSSL

Buat direktori penyimpanan kunc
```bash
sudo mkdir -p /etc/asterisk/keys
```

Generate Private Key (asterisk.key) dan Certificate (asterisk.crt)
```bash
sudo openssl req -new -x509 -days 365 -nodes -newkey rsa:2048 \
  -out /etc/asterisk/keys/asterisk.crt \
  -keyout /etc/asterisk/keys/asterisk.key \
  -subj "/C=ID/ST=Jawa Barat/L=Bandung/O=Telkom University/OU=VoIP/CN=192.168.1.10"
```
Atur hak akses file
```bash
sudo chown -R asterisk:asterisk /etc/asterisk/keys
sudo chmod 600 /etc/asterisk/keys/*
```

Konfigurasi PJSIP untuk TLS (pjsip.conf)
Tambahkan blok transport TLS berikut pada file /etc/asterisk/pjsip.conf
```bash
[transport-tls]
type=transport
protocol=tls
bind=0.0.0.0:5061
cert_file=/etc/asterisk/keys/asterisk.crt
priv_key_file=/etc/asterisk/keys/asterisk.key
method=tlsv1_2
cipher=AES128-SHA
```

lalu restart
```bash
sudo systemctl restart asterisk
```
Konfigurasi Client (MicroSIP / Zoiper)
SIP Server: 192.168.1.10:5061
Transport: TLS

kstraksi Private Key untuk Wireshark
Agar pihak penyadap/admin dapat melakukan dekripsi di Wireshark:

1. Tampilkan isi kunci di terminal server
```bash
   sudo cat /etc/asterisk/keys/asterisk.key
   ```
2. Salin seluruh teks (mulai dari -----BEGIN PRIVATE KEY----- hingga -----END PRIVATE KEY-----)
3. Simpan di komputer client/Windows dengan nama file asterisk.key (pastikan ekstensi murni .key, bukan .key.txt)

Kondisi Didekripsi (Menggunakan Private Key / Simulasi Kebocoran Kunci)
Buka Wireshark: Edit, Preferences, Protocols, TLS.
Di bagian RSA keys list, klik Edit... + (Tambah)
IP Address: 192.168.1.10
Port: 5061Protocol: sipKey File: Pilih file asterisk.key yang telah disimpan.
Klik OK
Hasilp
aket Application Data otomatis terbongkar menjadi paket SIP.
Pada paket INVITE, baris Message Header dapat dibaca kembali (From: 1000, To: 2000).
Terdapat penanda khas pada header berupa parameter transport=tls, membuktikan bahwa paket ini merupakan hasil dekripsi dari tunnel TLS.

