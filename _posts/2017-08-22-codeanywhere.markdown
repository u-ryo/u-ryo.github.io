---
layout: post
title: "codeanywhere"
date: "2017-08-22 12:01"
author: 'u-ryo'
categories: [codeanywhere]
comments: true
published: true
---
ちょっとした注意点
---

- Javaは入ってないので自分で入れる。`apt-cache`で見ると普通に`apt install`で入れられるのはJava7までっぽいが7で十分みたい。
- previewは、`./grainw`でcompile後、`http://...URL...:4000`で行けますね。→うーん、失敗する時の方が多いかも。
- `timezone`の変更をしておかないと。何故か`dpkg-reconfigure`が無いので、`sudo ln -sf /usr/share/zoneinfo/Asia/Tokyo /etc/localtime`で。
- `./grainw create-post '...'`の後、directory treeで`blog`を右click後Refreshしないと新しいfileが出て来ない。
- `screen`は`apt install`すればおk。
- ssh accessも、`id_rsa.pub`を`authorized_keys`に登録後、`ssh -p 21808 cabox@host9.codeanyhost.com`でおk。
- `./grainw gendeploy`は、`~/.ssh/id_rsa.pub`をgithubに登録しておかないと失敗する。