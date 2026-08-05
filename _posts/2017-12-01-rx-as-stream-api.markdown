---
layout: post
title: "Rx as Stream API"
date: "2017-12-01 17:35"
author: 'u-ryo'
categories: [android, rx, rxandroid, rxjava, stream, java]
comments: true
published: true
---
周知のように、Androidではlambdaは書けるようになりましたが
Stream APIのようにCollectionsを扱えません。
折角Java8で覚えたのに。
ですが、RxJavaを使うとほぼStream APIのように書けるんですねーへーーー。
[非同期や並列処理にも役立つRxJavaの使い方](https://qiita.com/disc99/items/1b2e44a1105008ec3ac9)
おかげでloopを回さず一文になったので、
ifの条件節に直接書けるようになりました。
`Optional`も出来るんですね。
書いてありますが、キモは`toBlocking().single()`でしょうか。
