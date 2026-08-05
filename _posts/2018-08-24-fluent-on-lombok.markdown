---
layout: post
title: "fluent on Lombok"
date: "2018-08-24 11:20"
author: 'u-ryo'
categories: [java, lombok]
comments: true
published: true
---
[Lombok](https://projectlombok.org)で`@Accessors(fluent=true)`にしたら、
setterのみならず[getterもfluentになる](https://projectlombok.org/features/experimental/Accessors)んですね。
`getXxx()`がないと言われて焦りました。
