---
layout: post
title: "rename files"
date: "2016-01-05 23:58"
author: 'u-ryo'
categories: [rename, mv, sed, find]
comments: true
published: true
---
### ファイル名一括置換
「rename」を使えばよいとのことだったので、

```
$ rename 'IMG_' 'img_0' ~/photo/20151227/*.JPG
```

しかし、

```
Bareword "IMG_" not allowed while "strict subs" in use at (eval 1) line 1.

```

と言われて動かなかったので、

```
find /home/u-ryo/photo/20151227/ -maxdepth 1 -name "IMG_*.JPG"|sed 'p;s|IMG_\([0-9]*\).JPG|img_0\1.jpg|g'|xargs -n2 mv -i
```

とすれば動いた。