---
layout: post
title: "local IP with hostname without an inner DNS (Wildcard DNS)"
date: "2016-09-07 21:45"
author: 'u-ryo'
categories: [dns, ip]
comments: true
published: true
---
内部DNS建てなくても、hostname付けられるんだって。凄い。

```
$ host some-project.127.0.0.1.xip.io
some-project.127.0.0.1.xip.io has address 127.0.0.1
```

[ローカルDNSを立てずにホスト名をIPアドレスに解決する超絶簡単な方法](http://qiita.com/tady/items/b7b46486fb3175dac0b1)

nip.io でも全く同じことが出来るようです。

```
$ host some-project.192.168.33.176.nip.io
some-project.192.168.33.176.nip.io has address 192.168.33.176
```

更に更に、[sslip.io](https://sslip.io/)では正式なserver certも用意出来ちゃう、なんていう素晴らしいとこだった、ようですが、今はもうダメポです。自分でやるしかなさそうです。とはいっても、wildcard certificationはLet's Encryptでは扱ってないので、やるならお金かかっちゃいますかね...