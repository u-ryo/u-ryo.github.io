---
layout: post
title: "SoftEther Client memo"
date: "2016-09-21 13:24"
author: 'u-ryo'
categories: [softether]
comments: true
published: true
---
SoftEtherのClientを起動する際の自分用のcommand log。
自動化しちゃえばいいんでしょうが...

```
$ sudo /usr/local/vpnclient/vpnclient start
$ /usr/local/vpnclient/vpncmd /client localhost /cmd accountconnect mickey
$ sudo ip addr add 192.168.120.10/24 dev vpn_nic0
```