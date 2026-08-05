---
layout: post
title: "LocalDate to ZoneDateTime"
date: "2018-08-17 06:31"
author: 'u-ryo'
categories: [java, date time]
comments: true
published: true
---
調べれば出てきますが。

`LocalDate`で指定された日付だけの情報を、
`ZonedDateTime`と日付時刻情報にconvertするには。

[stack overflow: Java 8 - Convert LocalDate to ZonedDateTime](https://stackoverflow.com/questions/32001716/java-8-convert-localdate-to-zoneddatetime#)

```
ZonedDateTime zdt = localDate.atStartOfDay(ZoneId.of("Asia/Tokyo"));
```

method名通り、当該日の0:00になります。