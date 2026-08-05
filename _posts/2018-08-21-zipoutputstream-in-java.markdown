---
layout: post
title: "ZipOutputStream in Java"
date: "2018-08-21 23:20"
author: 'u-ryo'
categories: [java, zip]
comments: true
published: true
---
Javaで複数filesをまとめてzipにして出力する(=downloadさせる)必要があり、
何かlibraryないかなーと探したところ、あまりなくて。
ということは、標準的な手法でもさほど手間はかからないということですね。
確かに、`new ZipOutputStream(response.getOutputStream())`してから
`new ZipEntry("file/name")`して`zos.putNextEntry(entry)`して
`zos.write(...)`していけばいいだけですもんね。
[Zipファイル](http://www.ne.jp/asahi/hishidama/home/tech/java/zip.html)によりますと、
`new ZipOutputStream(new BufferedOutputStream(...))`というように
`new BufferedOutputStream()`でくくった方が3.6倍速い(110ms→30ms)とのこと。
ホント?!
