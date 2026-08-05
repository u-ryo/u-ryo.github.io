---
layout: post
title: "MySQL on JHipster"
date: "2018-09-02 21:02"
author: 'u-ryo'
categories: [jhipster, mysql]
comments: true
published: true
---
これはもう[書いてあります](https://www.jhipster.tech/development/#using-mysql-mariadb-or-postgresql-in-development)けど、
実際、これまでdevだったのをprod環境にして、
Timastampを含むTableのあるMySQLをbackendに据えてみると、
`-Pdev`環境ではうまく立ち上がっていたのに、
`-Pprod`だと`ERROR 1067 (42000): Invalid default value for 'create_date'`
とかって言われて立ち上がりません。
エラーに直面すると、前書いてあったことなんて忘れてて、
改めて探し回っちゃいました。
結局、ですね、ubuntuの場合は、
`/etc/mysql/mysql.conf.d/mysqld.cnf`の`[mysqld]`のところに
以下を加えてrestartすれば済みます。特に2行目ですね。

```
character-set-server=utf8
explicit_defaults_for_timestamp=on
```
