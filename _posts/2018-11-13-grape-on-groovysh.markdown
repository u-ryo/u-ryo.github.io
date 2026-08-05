---
layout: post
title: "grape on groovysh"
date: "2018-11-13 12:28"
author: 'u-ryo'
categories: [groovy]
comments: true
published: true
---
groovyshでgrapeを使ってlibraryを使いたい時がありました。

```
groovy:000> :i groovy.grape.*
===> groovy.grape.*
groovy:000> Grape.grab(group:'mysql',module:'mysql-connector-java',version:'8.0.11')
===> null
groovy:000> Grape.grab('mysql:mysql-connector-java:8.0.11')
ERROR java.lang.RuntimeException:
Error grabbing Grapes -- [unresolved dependency: groovy.endorsed#mysql:mysql-connector-java:8.0.11;2.5.0: not found]
```

versionも文字列でないとならないのに気を付けます。
いつもの`'mysql:mysql-connector-java:8.0.11'`の形式はダメでした。

`@GrabConfig(systemClassLoader=true)`はどうやってもダメっぽいので、
結局groovyshからのMySQL accessは諦めましたが。

参考: [groovysh で Maven リポジトリにあるライブラリを使う](https://qiita.com/yukung/items/6e1f62e7c2d0aae95bee)
