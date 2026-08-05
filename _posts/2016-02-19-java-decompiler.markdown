---
layout: post
title: "Java Decompiler"
date: "2016-02-19 08:53"
author: 'u-ryo'
categories: [Java, Decompiler]
comments: true
published: true
---
Javaのdecompilerといえばjadが昔から有名ですが、jadってとうに開発止まってるんですね。
知りませんでした。1.5くらいから解析不能というのでは、
もうちょっと使えませんか。
今は[JD-GUI](http://jd.benow.ca/)が簡便そうですが、
これとてJava8のlambda expressionや、そもそもLocal Method Classes(method内inner class?)に非対応とな。
Java8の昨今ではう～ん、です。
と思ったら、[CFR](http://www.benf.org/other/cfr/)というのが使えるそうです。
実際試して、使えました。
[online decompiler](http://www.javadecompilers.com/)もあるので、色々気軽に試せます。
良かった、今でもjava decompileは気軽に出来るわけですね。