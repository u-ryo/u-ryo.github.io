---
layout: post
title: "Logging on Angular"
date: "2018-09-02 16:48"
author: 'u-ryo'
categories: [angular, logging]
comments: true
published: true
---
Angularでloggingをしようと思いました。
[slf4j(Simple Logging Facade 4 Java)](https://www.slf4j.org/)みたいなのないかなー、って思って[ng2-logger](https://www.npmjs.com/package/ng2-logger)使いました。
まぁ良かったんですけど、
logのlevel指定が、
指定level以下、とかではなく、
それぞれ指定しなくてはならないというのがちょっと不便です。
何でそんな設計にしたのかなー。
例えば、`Log.create('books', Level.WARN)`とすれば`Level.ERROR`も
出てくるというわけではなく、明示的にそれぞれ
`Log.create('books', Level.ERROR, Level.WARN)`と指定しなくてはならない、
というわけです。
`Log.setProductionMode()`で全てのlogは出て来なくなります。
これは、階層関係なくどこかに1箇所書いてあれば効くようです。
使う時はフツーに`log.d('object',obj)`と`i`、`w`ですが、
errorだけは`log.er('object',obj)`なので注意です。
あと出てくるlogの見た目はcolorfulできれいです。
