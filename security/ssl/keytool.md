# Keytool


## 1. Import a CA cert to an exsting keystore

```sh
keytool -import -trustcacerts\
    -alias <alias>\
    -file <crt-file-path>\
    -keystore  <keystore-file-path> # Eg: /usr/lib/jvm/java-11-openjdk/lib/security/cacerts
```
