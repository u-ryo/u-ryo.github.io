---
layout: post
title: "Comparable of Java"
date: "2018-08-21 01:26"
author: 'u-ryo'
categories: [java, comparable]
comments: true
published: true
---
小数と分数を表すclassなど似たような役割なら、
classを超えて`Comparable`でもいっか、
とか思ってしまって反省しています。

Effective Javaにあるんですね、
[【Effective Java】項目１２：Comparable の実装を検討する](https://www.thekingsmuseum.info/entry/2015/09/04/003117)って。
`compareTo`を定義するなら`equals`と`hashCode`も定義すべきだし、
`a.compareTo(b)==0`と`a.equals(b)==true`が違うなら、
`compareTo`で比較する`TreeSet`と`equals`で比較する`HashSet`で
結果が違っちゃうんですね。
`compareTo`が違うclassの比較を許すなら、
`equals`でも許さねばならず、
流石にそれはおかしいだろう、
ということでしょうか。
実際、

> クラスが適切にパラメータ化されていれば、ClassCastExceptionがスローされます。
> 契約は、クラス間の比較を排除してはいませんが、
> リリース1.6では、Javaプラットフォームライブラリーのどのクラスも、
> クラス間の比較をサポートしません。

[【Effective Java】項目12 Comparableの実装を検討する](http://tatsuyamuku.hatenablog.com/entry/2015/06/14/164803)
