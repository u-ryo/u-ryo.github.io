---
layout: post
title: "Flatten lists in a list in Java"
date: "2018-08-21 01:45"
author: 'u-ryo'
categories: [java, stream]
comments: true
published: true
---
Javaでlistの中にlistが複数あるものを、
一つのflatなlistにしたい時、どうするのかなー、と。
[Java 8 で ruby の flatten](https://qiita.com/macoshita/items/4d4aaf5cea9848ff9dbf)に書いてありました。

```
jshell> List.of(List.of("a","b","c"),List.of("d","e"),List.of("f","g")).stream().flatMap(Collection::stream).collect(Collectors.toList())
$7 ==> [a, b, c, d, e, f, g]
```

`flatMap(Collection::stream)`がキモですね。
こんなの思い付かないですよ。
