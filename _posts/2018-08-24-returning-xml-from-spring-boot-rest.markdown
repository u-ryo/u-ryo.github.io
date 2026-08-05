---
layout: post
title: "Returning XML from Spring Boot REST"
date: "2018-08-24 10:38"
author: 'u-ryo'
categories: [java, spring boot, rest, xml]
comments: true
published: true
---
Spring BootでRESTやってて、XMLでdataを返したいと。
[Java: Spring Boot で REST なアプリを作ってみる](http://blog.rakugakibox.net/entry/2014/11/23/java_spring_boot_rest)を見てみて、
`@XmlRootElement`を付ければいい?
ついでにtag名がclass名と違うので、
`@XmlRootElement(name="annotation")`としました。

あと、[Lombok](https://projectlombok.org)を使ったせいなのか、
各fieldに`@XmlElement`を付けないと、出て来ませんでした。
そういうもの?!
