---
layout: post
title: "Negate Method Reference"
date: "2018-08-17 06:41"
author: 'u-ryo'
categories: [java, method reference, stream]
comments: true
published: true
---
Javaのstreamで、`filter(s -> !s.isEmpty())`を
method referenceに出来ないかなー、と思ったんですが、
`java.util.function.Predicate`を使って、

1. Java11だと`filter(Predicate.not(String::isEmpty))`
1. 現状では`filter(((Predicate<String>) String::isEmpty).negate())`と長くなる
1. 下記のように自分で定義して`filter(not(String::isEmpty))`

```
public static <T> Predicate<T> not(Predicate<T> t) {
    return t.negate();
}
```

cf. [stack overflow: How to negate a method reference predicate](https://stackoverflow.com/questions/21488056/how-to-negate-a-method-reference-predicate)

現状では長くなるので、やめました。