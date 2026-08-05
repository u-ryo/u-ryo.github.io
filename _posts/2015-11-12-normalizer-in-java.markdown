---
layout: post
title: "Normalizer in Java"
date: "2015-11-12 01:30"
author: 'u-ryo'
categories: [Unicode, Normalization]
comments: true
published: true
---
同僚に、「JavaのNormalizer.NFCとかって何?」と聞かれたので、
調べてみました。
[ITproの櫻庭さんの解説](http://itpro.nikkeibp.co.jp/article/COLUMN/20071130/288467/)が
わかりやすいかも、と思ったんですが、

> 互換合成 Normalize Function Compativle Composite (NFKC)

とかって、スペルミスもあるし、そもそも[原典](http://www.unicode.org/reports/tr15/)と違うじゃん、
ということに気付いて。

* Normalization Form D (NFD)	Canonical Decomposition
* Normalization Form C (NFC)	Canonical Decomposition,followed by Canonical Composition
* Normalization Form KD (NFKD)	Compatibility Decomposition
* Normalization Form KC (NFKC)	Compatibility Decomposition,followed by Canonical Composition

Canonical Equivalent(正準等価性)が「か+゛」=「が」
Compatibility Equivalent(互換等価性)が「ｶ」=「カ」
なので、

* 「Canonical Decomposition」(正準分解)を「「が」→「か+゛」にすること」
* 「Canonical Composition」(正準合成)を「「か+゛」→「が」にすること」
* 「Compatibility Composition」(互換合成)を「「ｶ」を「カ」とすること」

と読めばいいのでは?

だから例えば、「NFKC」は、
「「ｶ」を「カ」としてから「か+゛」→「が」にすること」

気を付けたいのは、「Compatibility Composition」というのはない! ということですね。
だからよく読むと、各所にある日本語の解説は怪しいかも、です。

[文字コード地獄秘話 第3話：後戻りの効かないUnicode正規化](http://tech.albert2005.co.jp/blog/2014/11/21/mco-normalize/)の解説が良さげです。

```java
groovy:000> import java.text.*
===> java.text.*
groovy:000> str = "神と神㌀㈲¼が"
===> 神と神㌀㈲¼が
groovy:000> println(Normalizer.normalize(str, Normalizer.Form.NFD))
神と神㌀㈲¼か
===> null
groovy:000> println(Normalizer.normalize(str, Normalizer.Form.NFC))
神と神㌀㈲¼が
===> null
groovy:000> println(Normalizer.normalize(str, Normalizer.Form.NFKD))
神と神アハート(有)1⁄4か
===> null
groovy:000> println(Normalizer.normalize(str, Normalizer.Form.NFKC))
神と神アパート(有)1⁄4が
===> null
```