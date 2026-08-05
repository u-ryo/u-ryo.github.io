---
layout: post
title: "Parameters for FreeRDP"
date: "2015-10-30 05:07"
author: 'u-ryo'
categories: [linux, FreeRDP]
comments: true
published: true
---
某所にFreeRDPのbootable USBを作って納入したのですが、
「壁紙を変えたい」と言われました。
デフォルト「wallpaperはon」と書いてありますけれども、
明示的に「+wallpaper」としないと、効かないようです。

また、「画面が時々ちらつく」と言われました。
どうやら、頻繁にreconnectが生じているための様子。
パケットキャプチャすると、
2秒おきにTCP Keepaliveを出していて、その後、
クライアント側からTCP RST ACKを出しているようです。
なぜ?
ともあれ、「+heartbeat」を指定すると、
頻繁なreconnectはおさまりました。
「-heartbeat」でも効果あったんですが、なぜ?