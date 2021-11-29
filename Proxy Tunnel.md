# Proxy Tunnel
terkadang ada port yang di blok karena hanya bisa diakses oleh local machine. Namun jika machine tersebut memilki open proxy, maka kita tunnel, misalnya terdapat port SSH yang hanya bisa diakses dengan machine lokal, namun ada proxy squid di port 3128 maka untuk melakukannya :
```
proxytunnel -p 192.168.1.24:3128 -d 127.0.0.1:22 -a 1234
```
Hasilnya dari command diatas proxytunnel akan membuat jalur khusus dimana port 22 pada machine bisa diakses oleh attacker machine pada port 1234. 

Selanjutnya untuk login SSH dengan cara
```
ssh john@localhost -p 1234
```
Latihan : SkyTower VulnHub