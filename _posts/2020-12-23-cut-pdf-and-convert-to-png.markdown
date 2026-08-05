---
layout: post
title: "cut pdf and convert to png"
date: "2020-12-23 01:07"
author: 'u-ryo'
categories: [imagemagick, pdf, tkpdf]
comments: true
published: true
---
「pdf fileの10ページ目を切り取ってpngにconvertする」やり方。
`pdftk`で`10`ページ目を`cat`して標準出力に渡し、それをpipeで繋げて`pdftoppm`で標準入力から読み込みredirectして`-png`で変換してfileへ。`-png`がないと`.ppm`になるので注意。
```sh
pdftk nenga2021.pdf cat 10 output -|pdftoppm -png - > nenga2021.png
```
