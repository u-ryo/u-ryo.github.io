---
layout: post
title: "Path Parameter for DateTime in Spring Boot REST"
date: "2018-08-21 02:25"
author: 'u-ryo'
categories: [java, spring, spring boot, rest, datetime]
comments: true
published: true
---
Spring BootのRESTで、
日付をPath Parameterで受け取りたかったんですね。
[GetMappingでURLパラメータからValueを取得する～LocalDate編～](https://qiita.com/Tsukuru/items/1c872f51d25c6d452fbc)に書いてある通りですけど、
`@DateTimeFormat(pattern="yyyy-MM-dd")`なんて使えるんですね。

```
@GetMapping(value="/element/{since}")
public Element getElement(@DateTimeFormat(pattern="yyyyMMdd") @PathVariable LocalDate since) {...
```
