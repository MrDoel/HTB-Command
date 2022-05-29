# Java Keystore
## List Entries
```
keytool -list -v -keystore .keystore
```

## Extract Private Key From Keystore (PKCS12)
```
keytool -v -importkeystore -srckeystore keystore -srcalias tomcat -destkeystore myp12file.p12 -deststoretype PKCS12
```