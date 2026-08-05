---
layout: post
title: "illegal forward reference"
date: "2018-08-30 10:02"
author: 'u-ryo'
categories: [java, lambda]
comments: true
published: true
---
Javaの珍しいcompile error messageに、
`illegal forward reference`というのがあります。
これを題材に、後輩氏に課題を出しました。

最も単純な形にして挙例すると、

```
import java.util.function.Function;
class Test {
    Function f = new Function() {
            @Override
            public Object apply(Object t) {
                return o;
            }
        };
    Object o;
}
```

↑これをlambda式に直しなさい、といったようなものです。

実はこれ、単純にlambda式にすると、compile errorになります。

```
class Test {
    java.util.function.Function f = t -> o;
    Object o;
}
$ javac /tmp/Test.java
/tmp/Test.java:2: error: illegal forward reference
    java.util.function.Function f = t -> o;
                                         ^
1 error
```

error messageからしてどう直せばいいかは自明かと思ったんですが、
英語が不自由な後輩氏の1日弱かけた答えはこうでした。

```
class Test {
    java.util.function.Function f = t -> o;
    static Object o;
}
```

流石に「何でもかんでも`static`にすればいいってもんじゃない」
ってことはわかってて、「これがFA」とまでは言ってませんでしたけど、
[ここ](https://stackoverflow.com/questions/1746758/illegal-forward-reference-in-java)をみて、
「`static`にすればいいんじゃね、って書いてある」と思ったんだそう。
そうは書いてないんだけどなー。

でも、こういう回答は想像してなくて、新鮮でした。

そもそもdeclarationが後ろに書いてあったこと自体がおかしい、とは思いますけどね。
そういう「色々おかしい」ソフトを直しています。
