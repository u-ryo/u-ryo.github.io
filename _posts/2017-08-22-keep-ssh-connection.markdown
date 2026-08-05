---
layout: post
title: "keep ssh connection"
date: "2017-08-22 11:55"
author: 'u-ryo'
categories: [ssh]
comments: true
published: true
---
sshがよく切れるサイトが多いのですが、基本的には諦めてました。
でも、こうすればよかったんですね。

```
ssh -L 8888:localhost:8080 -o 'TCPKeepAlive yes' -o 'ServerAliveInterval 10' r.umetsu@219.101.192.235
```