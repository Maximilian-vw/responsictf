# Writeup CTF Responsi

### Reconnaisance
Scan subnet untuk host discovery menggunakan nmap
![Screenshot 2024-12-19 010710](https://github.com/user-attachments/assets/3a96a87a-406f-4842-8242-081e02e8c9db)

Scan open port atau running service menggunakan nmap
![Screenshot 2024-12-19 010824](https://github.com/user-attachments/assets/c46e4144-ce07-4c52-a217-9ce108925abe)


### Initial Access
terdapat open port 80 (http/web). akses web dan register ke web.

setelah register, kita login sebagai user. setelah berhasil login user akan mendapatkan cookie dari server agar dapat diautentikasi.

kita terautentikasi sebagai user biasa di web. ganti cookie menjadi admin untuk memperoleh hak akses admin. cookie bisa dibuat dengan mengencode string "admin" ke base64. ganti cookie user dengan cookie admin. reload halaman dan kita akan terautentikasi sebagai admin.

admin memiliki hak akses lebih dari user biasa. admin bisa melakukan upload produk di halaman /product. di halaman product terdapat flag pertama.

### Foothold
coba upload shell melalui fitur upload gambar produk. fitur upload akan membatasi upload gambar hanya dengan ekstensi tertentu.

jika halaman tersebut diinspect maka kita bisa melihat bahwa terdapat fiter untuk membatasi ekstensi file yang dapat di upload.

namun karena filter berada di frontend kita bisa melakukan bypass dengan cara intercept trafik menggunakan burp dan modifikasi request sebelum dikirim ke server.

buat php reverse shell, sesuaikan ip dan port dengan listener.

set listener menggunakan nc.

ubah ekstensi php reverse shell ke png agar dapat diupload di web.

intercept trafik dengan burpsuite lalu upload php reverse shell yang sebelumnya sudah diubah ekstensinya.

modifikasi request dengan mengubah kembali ekstensi php reverse shell dari .png ke .php lalu forward request tersebut.

akses shell yang sudah diupload. shell akan terupload di direktori /assets/img

cek nc listener, kita mendapat shell sebagai user www-data

### Privilege Escalation ke User
terdapat 2 cara untuk memperoleh privilege user :
- terdapat hint di direktori web (hint.txt : enumerate user & rockyou)
- terdapat file database di direktori web (backup.db)

untuk cara pertama kita bisa melakukan bruteforce ssh. pada tahap reconnaisance di awal diketaui bahwa terdapat port 22 (ssh) yang terbuka. berdasarkan hint yang diperoleh, kita dapat melakukan enumerasi user untuk mengetahui username dan kita bisa menggunakan wordlist rockyou sebagai password untuk melakukan bruteforce.

enumerasi user dengan mengecek file /etc/passwd. terdapat user tk22eh.

lakukan bruteforce ssh dengan hydra

untuk cara kedua kita bisa menggunakan password admin yang ada di database. kita bisa berasumsi bahwa user admin di web adalah user yang sama dengan user di server. enumerasi user di server dengan cara yang sama dengan cara pertama (/etc/passwd).

login ke ssh dengan username (tk22eh) dan password ( - password di db - )


### Privilege Escalation ke Root
terdapat cron job yang berjalan. cek cron job dengan perintah cat /etc/crontab.

berdasarkan crontab kita bisa mengetahui bahwa ada cron job yang berjalan. cron job menjalankan file .config/priv_esc.sh di direktori home user.

cek hak akses file tersebut dengan perintah ls -la. kita memiliki akses untuk memodifikasi file tersebut.

modifkasi isi file. ubah file tersebut menjadi reverse shell. isi file dengan payload reverse shell sehingga ketika cron job dieksekusi maka reverse shell akan tereksekusi.

setup listener dengan nc. tungu hingga cron job tereksekusi dan kita mendapat shell sebagai root.
