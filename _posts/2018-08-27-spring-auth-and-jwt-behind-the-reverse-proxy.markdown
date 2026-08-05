---
layout: post
title: "Spring Auth and JWT behind the reverse proxy"
date: "2018-08-27 17:47"
author: 'u-ryo'
categories: [spring, spring auth, oauth2, jwt, reverse proxy, nginx]
comments: true
published: true
---
JHipsterのSpring Authなapplicationを
httpsのreverse proxy(nginx)の後ろに置いて、
GoogleのOAuth2でJWTな認証をしようとしました。
当然、backend serverからはGoogle APIに自分のhost名でaccessするような
URLを返してしまい、Google APIから戻ってきたところでJWT認証は弾かれます。
backend serverはfrontend serverの名前を知らないんですから、
そりゃあ当然です。
こういうreverse proxyの後ろにbackend server置いてOAuth2 + JWTなんて
そもそもダメなの? 何とかならないの?
と調べてみると、
[Spring Boot and OAuth2: redirect url over reverse proxy](https://stackoverflow.com/questions/31834278/spring-boot-and-oauth2-redirect-url-over-reverse-proxy)に、
reverse proxy側で`X-Forwarded-Port`とかのproxy用HTTP Response Headerを設定し、
Spring application側で`server.use-forward-headers=true`にすればいいよ、
とあったので、
じゃぁnginxではどうやるのだろうと調べると、
[Nginx のリバースプロキシ設定のメモ](https://qiita.com/HeRo/items/7063b86b5e8a2efde0f4)にありました。

```
server {
  listen 80;
  server_name hoge.com;

  location / {
    proxy_set_header X-Real-IP $remote_addr;
    index index.html index.htm;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $http_host;
    proxy_redirect off;
    proxy_pass http://127.0.0.1:3000;
  }
}
```

この通りやってみると、
何故かGoogle APIには`http://proxy-server/...`で渡っており、
じゃぁっていうんで`proxy_set_header X-Forwarded-Proto https;`
とベタ書きしてみてもダメで、
うーんとか思っていると、
[nginx でリバースプロキシするときの Tips](https://blog.akagi.jp/archives/3883.html)に`off`じゃなくて`proxy_redirect http:// https://;`という記述があったので、
試してみると、上手く行きました。
あーちなみに、`proxy_set_header X-Forwarded-Proto https;`も
ベタ書きじゃないとダメでした。
結局うちの場合は、以下の通りになりました。

```
server {
        listen 80 default_server;
        listen [::]:80 default_server;

        # Everything is a 404
        location / {
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-Proto https;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header Host $http_host;
                proxy_redirect http:// https://;
                proxy_pass http://walt.mydns.bz:10022/;
        }

        # You may need this to prevent return 404 recursion.
        location = /404.html {
                internal;
        }
}
```

で、`sudo /usr/sbin/nginx -s reload`です。

でも、これでGoogle APIから無事戻ってくるようにはなったものの、
その後「No-providerで登録」になってしまい、
まだ完成しません。
ただ、その問題は別のもののようで、一歩は進んだと思うので、記事にしました。

↑その「No-providerで登録」になってしまうのは、
backendで以下のようなerrorが出ていて。

```
javax.validation.ConstraintViolationException: Validation failed for classes [bz.mydns.walt.canmatch.domain.User] during persist time for groups [javax.validation.groups.Default, ]
List of constraint violations:[
        ConstraintViolationImpl{interpolatedMessage='must match "^[_'.@A-Za-z0-9-]*$"', propertyPath=login, rootBeanClass=class bz.mydns.walt.canmatch.domain.User, messageTemplate='{javax.validation.constraints.Pattern.message}'}
]
        at org.hibernate.cfg.beanvalidation.BeanValidationEventListener.validate(BeanValidationEventListener.java:140)
        at org.hibernate.cfg.beanvalidation.BeanValidationEventListener.onPreInsert(BeanValidationEventListener.java:80)
        at org.hibernate.action.internal.EntityIdentityInsertAction.preInsert(EntityIdentityInsertAction.java:197)
        at org.hibernate.action.internal.EntityIdentityInsertAction.execute(EntityIdentityInsertAction.java:75)
        at org.hibernate.engine.spi.ActionQueue.execute(ActionQueue.java:626)
   :
   :
```

何なんでしょうね。
これは、account mail addressが「w.disney@somecompany.co.jp」みたいな
「.」が入るものなんですが、それがいけないとかなのでしょうか。
というのも、フツーの「gepetto@gmail.com」みたいなmail accountなら
全く同じcodeで何の問題もなく入れるのです。
「must match」の対象が何なのか、よく分かりません。
