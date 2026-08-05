---
layout: post
title: "Too many open files"
date: "2015-10-30 05:02"
author: 'u-ryo'
categories: [tomcat, linux]
comments: true
published: true
---
tomcat7が止まっていて、catalina.outを見ると、
`Too many open files`と言われていました。

* 現状の確認 `$ ulimit -a`
* プロセス毎なので、`$ cat /proc/プロセス番号/limits` or ``$ cat /proc/`pgrep -f tomcat7`/limits``
* 現状幾つのfilesをopenしているか、は、``$ sudo ls -l /proc/`pgrep -f tomcat7`/fd/|wc -l``
* 増やすには、/etc/init.d/tomcat7 で`ulimit -n 8192`等と追記してrestart
* `$ cat /proc/プロセス番号/limits`で確認