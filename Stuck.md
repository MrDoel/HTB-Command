##  Stuck di halaman login? 
coba sql injection. `1'or'1--`

## Ada forbidden port dan juga ada LFI atau bug lain yang cuma bisa baca file

Misal ada port 8080 forbidden, namun di port 80 ada bug LFI. Kalau kita gak bisa ngapa-ngapain coba cari config apache seperti `httpd.conf`

1.  `/etc/apache2/apache2.conf` – Ini biasanya digunakan di keluarga distro Debian/Ubuntu
2.  `/etc/httpd/httpd.conf` – Kalau ini akan ditemukan di keluarga distro Linux RHEL/CentOS/Fedora
3.  `/etc/httpd/conf/httpd.conf` – Ada yang seperti ini juga di CentOS
4.  `C:\xampp\apache\conf\httpd.conf` – Letaknya kalau di XAMPP
5.  `usr/local/etc/apache22/httpd.conf` - FreeBSD

Kalau sudah ketemu, cek kenapa port 8080 forbidden, biasanya harus pakai `user-agent` tertentu

Latihan : Kioptrix 2014

## Cuma port 80 yang terbuka, gak ada apa-apa
Cek pada endpoint apakah ada method
```
OPTIONS /test/ HTTP/1.1
Host: 192.168.1.10
```
Latihan : SickOS