---
layout: post
title: "Illegal keysize"
date: "2017-09-04 02:19"
author: 'u-ryo'
categories: [android, bouncycastle, crypt, ssl, tls, nanohttpd]
comments: true
published: true
---
Androidで[NanoHTTPD](https://github.com/NanoHttpd/nanohttpd)を
動かすprogramを開発しているんですが、
WebRTCにするのに、TLSが必要じゃないですか。
そのserver certを普通に作ると、`Illegal keysize`と言われて
key load時に落ちるのでハマりました。
[NanoHTTPDの解説](https://github.com/NanoHttpd/nanohttpd#generating-an-self-signed-ssl-certificate)にあるように、

```
$ keytool -genkey -keyalg RSA -alias selfsigned -keystore keystore.jks -storepass password -validity 360 -keysize 2048 -ext SAN=DNS:localhost,IP:127.0.0.1  -validity 9999
```

と作ってもダメでした。
色々調べると、AndroidにはBouncyCastle(BKS)でないとならないらしく、
証明書は面倒なのでsnakeoilを流用して、

```
$ sudo openssl pkcs12 -export -in /etc/ssl/certs/ssl-cert-snakeoil.pem -inkey /etc/ssl/private/ssl-cert-snakeoil.key -out ~/AndroidStudioProjects/SharedEye/ssl-cert-snakeoil.p12 -name ssl-cert-snakeoil
$ /usr/lib/jvm/java-8-oracle/bin/keytool -importkeystore -deststorepass password -destkeypass password -destkeystore snakeoil.jks -srckeystore ssl-cert-snakeoil.p12 -srcstoretype PKCS12 -srcstorepass password -storetype BKS -provider org.bouncycastle.jce.provider.BouncyCastleProvider -providerpath bcprov-jdk15-1.46.jar
```

とすると、

```
Problem importing entry for alias java.security.KeyStoreException: java.io.IOException: Error initialising store of key store: java.security.InvalidKeyException: Illegal key size.
```

と言われてimport出来ませんでした(→jksが作れませんでした)。
かなり悩んだのですが、結局[JavaでAES256を使用できるようにする](http://qiita.com/mizuki_takahashi/items/cc26a7fd51aa04396e92)にあるように、
JCE(Java Cryptography Extension)を落としてきて
`local_policy.jar`を上書きしたら、
jksも出来て、Android側でも何事もなくloadしてくれました。
