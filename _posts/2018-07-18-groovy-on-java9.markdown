---
layout: post
title: "groovy on Java9+"
date: "2018-07-18 07:43"
author: 'u-ryo'
categories: [java, groovy, java9, modules]
comments: true
published: true
---
いつも忘れちゃうので。
Java9以上のGroovyでmoduleが足りない、
具体的には、

```
Caught: java.lang.NoClassDefFoundError: Unable to load class groovy.xml.jaxb.JaxbGroovyMethods due to missing dependency javax/xml/bind/JAXBContext
java.lang.NoClassDefFoundError: Unable to load class groovy.xml.jaxb.JaxbGroovyMethods due to missing dependency javax/xml/bind/JAXBContext           
```

と言われる時。
groovyではどうやって`--add-modules`すればいいのかなって。

```
$ JAVA_OPTS='--add-modules=ALL-SYSTEM' groovy ...
```