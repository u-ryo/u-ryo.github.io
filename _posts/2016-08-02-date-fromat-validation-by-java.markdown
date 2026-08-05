---
layout: post
title: "Date fromat validation by Java"
date: "2016-08-02 17:26"
author: 'u-ryo'
categories: [java, date, format, validation]
comments: true
published: true
---
Javaで日付フォーマットの検証をしたいと言われました。
ぱっと思いつくのは、`SimpleDateFormat`で`setLenient(false)`にしてparseする、
ですが、これだとException投げるんですよね。
Exception catchをlogicに使うのは良くない、
ということで、他を探してみると意外となかなかないんですねこれが。
commons-langの`DateUtils.parseDateStrictly`もException返しますし。

色々探してみると、Apacheの[commons-validator](https://commons.apache.org/proper/commons-validator/)というのがありました。

```
@Grab('commons-validator:commons-validator')
import org.apache.commons.validator.routines.CalendarValidator

pattern = "yyyy/MM/dd"
validator = CalendarValidator.getInstance()

assert validator.isValid("2016/02/29", pattern)
assert !validator.isValid("2016/02/30", pattern)

assert validator.isValid("2016/08/31", pattern)
assert !validator.isValid("2016/08/32", pattern)
```