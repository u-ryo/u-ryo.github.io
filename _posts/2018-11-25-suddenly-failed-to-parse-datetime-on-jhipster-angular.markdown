---
layout: post
title: "Suddenly failed to parse DateTime on JHipster Angular"
date: "2018-11-25 01:11"
author: 'u-ryo'
categories: [angular, jhipster, jackson]
comments: true
published: true
---
ある時、JHipsterのAngularのUser Managementを久し振りに見てみると、user名など表示されるのが物凄く遅いことがありました。ポツ、また数秒してポツ、という具合に一つ一つの行が表示されていきます。あれぇ? こんなもんだったかなぁ? あまり気にしなかったのですが、流石に新規Userを作成しようとして失敗する段になって、これは何とかしなければと思い始めました。browserでF12を押してみると、どうやらDate Pipeでのparseに失敗している様子。どうして? 前は出来てたのに。serverから来ているJSONをよく見ると、DBからのDateTime部分のObjectが`epochSecond`とかnano何とかになっていました。検索すると、
[Efficient way to have Jackson serialize Java 8 Instant as epoch milliseconds?](https://stackoverflow.com/questions/37999762/efficient-way-to-have-jackson-serialize-java-8-instant-as-epoch-milliseconds/38004044)を見付けて、中身読んでないですが「jackson-datatype-jsr310」を見掛けてあぁー!っと。

jhipsterで新規application作ってその`build.gradle`見てみると、確かにありました`compile 'com.fasterxml.jackson.datatype:jackson-datatype-jsr310'`。
これを、なくてもいいじゃん、と大分前に取っちゃってたんですね。確かにWebアプリ本体には影響なかったので気付きませんでした。あーあ。こういう影響が出てくるですか。なるほど。
