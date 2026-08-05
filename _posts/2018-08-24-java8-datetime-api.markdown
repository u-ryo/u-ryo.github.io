---
layout: post
title: "Java8 DateTime API"
date: "2018-08-24 10:17"
author: 'u-ryo'
categories: [java, datetime]
comments: true
published: true
---
Java8から入ったDateTime API(`java.time` package)では、
`a.plusMinutes(3).isBefore(now)`
なんていう感じで日付時刻計算や比較が出来ます、
という話です。現在時刻は`ZonedDateTime.now(ZoneId.of("Asia/Tokyo"))`。
あ、`ZonedDateTime.now()`でもいいんじゃーんdefaultのtimezoneでいぃなら。

* [Javaで日時を扱う（Java8）](https://qiita.com/kurukurupapa@github/items/f55395758eba03d749c9)
* [Java 8日時APIの主なメソッドとフォーマット用のパターン文字の使い方 (2/6)](http://www.atmarkit.co.jp/ait/articles/1501/29/news016_2.html)
