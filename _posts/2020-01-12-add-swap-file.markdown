---
layout: post
title: "add swap file"
date: "2020-01-12 01:19"
author: 'u-ryo'
categories: [linux, swap]
comments: true
published: true
---
年末にideapad 310のmemoryが壊れてしまい、2度買い直してもmemoryを認識しなかったので、これは本体側が壊れたのだろうと。これを直すには本体のmother board交換になる、くらいならまだ2年程しか使ってませんが新しいのを買う? と考え、物色していたところ、新しいものでもmemoryって搭載容量4GBとか程度で、何故? と思ったんですけど、今はSSDだからそれでいいってことなんですね! そっか、今はSSDが廉価になったからswapでいいのか! なら310もSSDに換装すれば、ということで、SSD 1TBにしました。1.11万円でSamsungの1TBをGET。ubuntu 19.10を入れて... 最初、折角だからZFSにしてみたのですが、SSDと相性悪いんですねこれ。swap partitionは、位置が固定されるためSSDには良くないと。XFSもSSDではjournalingを止めた方が良いとのことで、それだと何のためのXFSなの?! ということなのでつまらないですけどext4で再installします。しかし、partitioningをお任せにすると、今はdefaultでswap partitionではなく2GBのswap fileになるというのは驚きですね。
ということで、本題です。よく、`dd`でswap fileを作る例が載っていますけれども、[スワップ領域の追加方法|server-memo.net](https://www.server-memo.net/centos-settings/system/add-swap.html#ext4)には`fallocate`を使う方法があったので、メモです。

```sh
sudo fallocate -l 2G /swapfile1
sudo mkswap /swapfile1
Setting up swapspace version 1, size = 2097148 KiB
no label, UUID=89a19a54-42c9-4847-a482-1971c388fa95
sudo chmod 600 /swapfile1
sudo swapon /swapfile1
```