## Cek Apakah bisa login pakai anonymous access
```
smbclient -L //192.168.1.1
```
## Gunakan Enum4Linux untuk recon
```
enum4linux <host>
```
## Cek apakah ada NULL session
Dalam hal ini kita bisa login tanpa password
```
smbclient -U '%' -N \\\\<IP>\\<SHARE>
```