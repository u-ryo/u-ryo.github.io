---
layout: post
title: "daemonizing jhipster application"
date: "2018-08-19 23:51"
author: 'u-ryo'
categories: [jhipster, spring boot, ubuntu, java]
comments: true
published: true
---
[JHipster](https://www.jhipster.tech/)のapplicationで、
OS(Ubuntu 18.04)起動時にapplicationもdaemonとして自動起動するようにするには。

きっとSpring Bootでdaemonizeする方法を探ればいいと思って、
[61. Installing Spring Boot Applications](https://docs.spring.io/spring-boot/docs/current/reference/html/deployment-install.html)にあるように、

```
bootJar {
	launchScript()
}
```

でもこれ、Spring Boot 2での話で、
こちとらまだJHipster 4.14.4、Spring Bootは1.5です。
そんなものはない、と当然失敗します。
なのでもうちょっと古い記事を探しました。

[spring bootアプリの起動スクリプトを作る](https://ishiis.net/2016/09/27/spring-boot-init-script/)を見て、

```
apply plugin: 'spring-boot'

springBoot {
    executable = true
}
```

としたんですが、`plugin 'spring-boot'`はないと言われ、
`executable = true`は`build.gradle`に既に書いてありました。

そもそもそんなことしなくても、
[Using in Production](https://www.jhipster.tech/production/)
にあるように、

```
$ gradle bootRepackage -Pprod
```

でexecutable war作れるんですね。
で、それを`/etc/init.d/`にsymlinkすればいいだけという。
`-Pprod`を付けないとdevelopment versionになってしまいます。
あとは、`update-rc.d appname defaults`で登録すれば良いです。
