# Kerberos
Kerberos merupakan protokol autentikasi untuk aplikasi client/server yang biasa digunakan pada AD. Ketika user ingin mengakses suatu service, misalnya ingin mengakses FTP maka user ini harus mendapatkan izin dulu dari KDC (Key Distribution Center). KDC berisi 2 server :
* Authentication Server (AS)
* Ticket Granting Server (TGS)

Nah jadi untuk bisa mengakses FTP tadi, si user harus mendaptkan Ticket yang digenerate dari TGS. Untuk flownya

1. User mengirimkan request ke AS dengan mengirimkan data-data dari user, misalnya userid and so on. "Halo saya butuh tiket untuk mengakses server FTP". Pengiriman request ini terenkripsi dengan menggunakan password user. Jadi AS juga mengetahui password dari user tersebut dimana akan digunakan untuk melakukan decrypt request dari user. Nah hal ini biasanya ada celah Asreproast
2. Server AS kemudian akan mengirimkan ticket yang disebut Ticket Granting Ticket (TGT)
3. TGT yang didapatkan oleh user kemudian akan dikirimkan ke TGS
4. Dari TGS kemudian akan memberikan token ke user, token inilah yang digunakan untuk mengakses FTP