# Pivoting Port
Misal pada mesin yang mau kita Pivot terdapat 1 port yang terbuka yaitu 8009, maka perintah untuk port forwarding adalah
```
socat tcp-listen:8009,fork,reuseaddr tcp:192.168.40.129:8009 &
```
Jadi `192.168.40.129` adalah IP victim,