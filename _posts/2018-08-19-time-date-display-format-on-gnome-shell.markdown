---
layout: post
title: "Time/Date Display Format on GNOME Shell"
date: "2018-08-19 23:24"
author: 'u-ryo'
categories: [ubuntu, GNOME]
comments: true
published: true
---
Ubuntu 18.04使ってるんですが、
Unityから変わったGNOME Shellの真ん中に出てるのが
曜日と時分だけなので、不便でした。
日付を確認したくなることもあるし、
秒がないと動いているのか止まっているのかよくわからなかったりとかするんですよね。
どうするのかなー、と思って調べたら、一発でした。
[GNOME Shellのパネルに日付を表示する方法](https://linuxfan.info/gnome-shell-clock-show-date)

```
$ gsettings set org.gnome.desktop.interface clock-show-date true
$ gsettings set org.gnome.desktop.interface clock-show-seconds true
```

tabによる補完で調べると、他には`clock-show-weekday`があり、
それは既に`true`になってるんでしょうね。
