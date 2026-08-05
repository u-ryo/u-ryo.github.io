---
layout: post
title: "How Secure Is My Password"
date: "2018-09-04 14:25"
author: 'u-ryo'
categories: [security]
comments: true
published: true
---
[いますぐ実践! Linux システム管理 / Vol.186](http://www.usupi.org/sysad/186.html)に載ってたものなので、何を今更、ですが。

[How Secure Is My Password?](http://howsecureismypassword.net/)、
自分の常用passwordは1 Dayでした。
色々試してみると、
「長いことはいいことだ(10文字以上はないと)」
「最後に大文字アルファベットでも足すと劇的に良くなる」
みたいです。
でも多分これ、localにhashか何かがあってbrute forceを試せる、
という前提での解読時間かなと思います。
今はjs fileはugrifyされてて具体的にどう計算しているのかlogic(簡単には)追えないです。
