---
layout: post
title: "Stubbing in Spock"
date: "2018-08-21 23:49"
author: 'u-ryo'
categories: [spock, groovy, java, stub]
comments: true
published: true
---
[Spock](http://spockframework.org/)でstubbingして
methodのcall回数をassertする必要がありまして。
`someClass = Spy(SomeClass)`でspyにすると、
`3 * someClass.targetMethod(_, _)`といったようにassert出来るのですが、
そのtargetMethodを呼ぶ大元のmethodのcallは、
`then:`ではなく`when:`になければならなかった、という話です。

即ち、

```
when:
  ...
then:
  someClass.method() == 'answer'
  3 * someClass.targetMethod(_, _)
```

ではダメで、

```
when:
  ans = someClass.method()
then:
  ans == 'answer'
  3 * someClass.targetMethod(_, _)
```

でないとなりませんでした、と。

あと、
`3 * someClass.targetMethod(_, _)`の部分には、
変数とか入れられません。即ち、
`(3 + n) * someClass.targetMethod(_, _)`とかはダメでした。
