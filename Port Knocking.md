# Port Knocking
Port Knocking adalah metode yang dilakukan untuk membuka akses ke port tertentu yang telah diblock oleh Firewall pada perangkat jaringan dengan cara mengirimkan paket atau koneksi tertentu. Koneksi bisa berupa protocol TCP, UDP maupuan ICMP

Ketika kamu melakukan pentesting misalnya pada HTB,Vulnhub, oscp dan terdapat beberapa port dengan status `filtered`. Maka  kemungkinan kita bisa melakukan port knocking

Tantangannya sekarang adalah bagaimana cara menemukan path yang benar agar port bisa diakses. 

Misal port 21 di `filtered` maka cara untuk port knocking nya adalah dengan cara melakukan ping ke server. Baru setelah itu port 21 bisa diakses. 

Ada lagi kasus pada mesin vulnhub **Pinky Places V2** dimana untuk bisa mengakses port yang filtered, maka kita harus melakukan port knocking pada tiga port yaitu `666,7000,8890` sehingga perintah untuk melakukannya adalah :

```
nmap -Pn -sT -r -p666,7000,8890 192.168.1.11
```

Namun masalahnya adalah port `filtered` masih belum bisa terbuka, maka kita harus melakukan kombinasi antara port `666,7000,8890`

```knock.sh

#!/bin/bash

TARGET=$1

for ports in $(cat permutation.txt); do
    echo "[*] Trying sequence $ports..."
    for p in $(echo $ports | tr ',' ' '); do
        nmap -n -v0 -Pn --max-retries 0 -p $p $TARGET
    done
    sleep 3
    nmap -n -v -Pn -p- -A --reason $TARGET -oN ${ports}.txt
done
```

File permutations.txt berisi kombinasi dari port `666,7000,8890`, cara untuk membuatnya :

```
python -c 'import itertools; print list(itertools.permutations([8890,7000,666]))' | sed 's/), /\n/g' | tr -cd '0-9,\n' | sort | uniq > permutation.txt
```

Contoh hasil permutation:
```
└─$ cat permutation.txt     
666,7000,8890
666,8890,7000
7000,666,8890
7000,8890,666
8890,666,7000
8890,7000,666

```

Hasil dari permutasi inilah yang akan digunakan oleh NMAP untuk port knocking
```
knock.sh 192.168.1.11
```

Misalnya untuk melakukan port knocking ditemukan bahwa kombinasinya adalah port `7000,666,8890`, maka perintah untuk langsung membuka port adalah
```
for p in 7000 666 8890; do nmap -n -v0 -Pn --max-retries 0 -p $p 192.168.1.11; done
```