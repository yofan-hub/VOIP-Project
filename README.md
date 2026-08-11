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
