---
layout: post
title: "CSV download on Spring Boot REST"
date: "2018-08-24 11:30"
author: 'u-ryo'
categories: [spring boot, csv]
comments: true
published: true
---
Spring BootでCSV downloadを実現するのに、
[Spring MVC で CSV をダウンロードさせる](https://qiita.com/yo1000/items/050c5c47daabf7a10e10)を参考にしました。
要するに、

* `compile "com.fasterxml.jackson.dataformat:jackson-dataformat-csv"` in `build.gradle`
* Bean classの各fieldに`@JsonProperty`
* field出力順を制御したいので`@JsonPropertyOrder({"login", "filename",...})`
* `CsvMapper mapper = new CsvMapper();`して`CsvSchema schema = mapper.schemaFor(SomeBean.class).withHeader();`して`return mapper.writer(schema).writeValueAsString(beans);`
* `compile "com.fasterxml.jackson.datatype:jackson-datatype-jsr310"`を入れて`@JsonFormat(pattern="yyyy/MM/dd HH:mm:ss")`して`mapper.registerModule(new JavaTimeModule());`すると、`ZonedDateTime`を如意に表示できる?[Formatting Java Time with Spring Boot using JSON](http://lewandowski.io/2016/02/formatting-java-time-with-spring-boot-using-json/)
