---
layout: post
title: "Trigger as 3 times service down in a row on Zabbix"
date: "2016-08-03 17:33"
author: 'u-ryo'
categories: [zabbix]
comments: true
published: true
---
Zabbixで「3回連続してservice downを観測したらalert、
且つ1回service upを観測したら復帰」とかって、
実は結構面倒なんですね。

cf.[トリガーの条件式における記述方法について](http://www.zabbix.jp/node/2573)

```
({TRIGGER.VALUE}=0&{host名:http.max(#3)}=0)|({TRIGGER.VALUE}=1&{host名:http.last(0)}=0)
```

→何か騙された感じです。色々試すと、単に`max(#3)`だけで良さげですよ。

```
{host名:http.max(#3)}=0
or
{host名:http.max(#3)}=0|{host名:https.max(#3)}=0
```

↑これだけで、「3回連続してservice downを観測したらalert、
且つ1回service upを観測したら復帰」を実現出来ました。