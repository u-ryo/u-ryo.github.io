---
layout: post
title: "Grain on CodeAnywhere"
date: "2017-09-06 22:02"
author: 'u-ryo'
categories: [grain, codeanywhere]
comments: true
published: true
---
ちょっと、やっぱりダメですね。

```
java.lang.RuntimeException: While executing class com.sysgears.grain.registry.Registry.compile
        at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
        :
        :
Caused by: java.net.SocketTimeoutException: Accept timed out
        at java.net.PlainSocketImpl.socketAccept(Native Method)
```

といわれて、`./grainw generate`も`./grainw gendeploy`も出来ません。
あーあ、なぁんだ。
