# VOIP-Project
dokumentasi voip menggunakan GSM


## 1. instalisasi server asterisk
sudo apt install asterisk -y

lakukan apakah asterisk sudah berjalan
sudo systemctl status asterisk

Digunakan untuk edit file konfigurasi PJSIP dan Dialplan:

sudo nano /etc/asterisk/pjsip.conf

sudo nano /etc/asterisk/extensions.conf

udo nano /etc/asterisk/voicemai.conf

untuk me restart perubahan di atas

sudo systemctl restart asterisk

instal modul voicemail
sudo apt install asterisk-sounds-main asterisk-sounds-core -y

restart
sudo asterisk -rx "reload"

di dalam modul asterisk ada beberapa modul yang tidak kita pakai contohnya ialah chan_sip
bila muncul di  -rvvvvvv bisa kita matikan

sudo nano /etc/asterisk/modules.conf

tambahkan ini
noload => chan_sip.so



untuk melakukan GSM diperlukan panggilan dial dengan 908123456789

## 2. konfigurasi sip

sudo nano /etc/asterisk/pjsip.conf

sudo nano /etc/asterisk/extensions.conf

restart setelah melakukan tersebut 

sudo asterisk -rx "core reload"

## 3. konfigurasi client
