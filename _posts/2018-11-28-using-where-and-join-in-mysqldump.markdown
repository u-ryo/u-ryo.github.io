---
layout: post
title: "Using where and join in mysqldump"
date: "2018-11-28 00:32"
author: 'u-ryo'
categories: [mysql, mysqldump, sql]
comments: true
published: true
---
基本的には[mysqldumpでwhereにjoinを使う](https://qiita.com/asigochan/items/fec45efff78045b33b90)にある通り、

```
mysqldump -uusername -p DatabaseName tableA --single-transaction --were 'id IN (SELECT a.id FROM tableA a JOIN tableB b ON a.b_id=b.id WHERE b.name LIKE "%someone%")'
```

というわけですが、肝は、

* `--where`中で`JOIN`を使うためにsubqueryにする
* `--single-transaction`を付けることで`mysqldump: Couldn't execute 'SELECT... Table 'a' was not locked with LOCK TABLES (1100)`と言われるのを防ぐ
