---
layout: post
title: "Spock and JMockit"
date: "2017-12-19 23:27"
author: 'u-ryo'
categories: [spock, jmockit, junit, junit5]
comments: true
published: true
---
バイトのtestで、Spockでさらっとtest書いて、
いざ全pattern cross check! と思ったら、
無限loopになるpatternが多くて。
`@Timeout`導入でサクッと、と思ったら、
loop回るのが早くて凄いmemoryと時間を食って、
`@Timeout`では抑えきれません。
うーむ、それじゃぁGroovyのmetaClassでmethodの振る舞いを変更だ!
と思ったら、どうやっても振る舞いが変わりません。
悩んだ末、GroovyのmetaClassでの再定義は、
外側のJava classには及ばない(Groovyの世界の中だけ)、と結論。
仕方無いじゃぁJMockitだ!
って思ったんですが、今度はSpockとJMockitは相容れないようで、

```
Caused by: java.lang.UnsupportedOperationException: Attempted to redefine class loaded from custom class loader
```

なので`JAVA_OPTS=-javaagent:.../jmockit-1.37.jar`をつけて実行すると今度は、

```
mockit.internal.ClassFile$NotFoundException: Unable to find class file for Test2$1
```

spockは諦めました。

ならばせめてJUnit5だ! と思ったんですが、`build.gradle`で、

```
junitAnt 'org.apache.ant:ant-junit4:1.10.1'
```

と指定しているので、敢え無く撃沈。
調べるとどうも、JUnit5とAntを組み合わせる動きはないんですよね。
[それでも Ant を使いたい人のための JUnit 5](https://devlog.arksystems.co.jp/2017/12/12/4436/)というのもありましたけど、これは、

```
<java fork="true" classname="org.junit.platform.console.ConsoleLauncher">
```

で無理矢理実行してるだけですから、ちょっと違います。
こちとら、`build.gradle`から`ant.junitAnt(...)`で呼んでいるので、
`ConsoleLauncher`だと困るんです(→どうやって書いたらいいかよくわからない)。

結局、JUnit4 + JMockit で書きました。
`Timeout`や`ExpectedException`は`@Rule`にしてちょこっと今っぽくしましたけど、
そのくらいが関の山でした。

`ant.junitAnt(...)`で`fork:true`にして、また更に
`junit4.jar`より`jmockit.jar`を先に書いておかないと
`IllegalStateException`が出る([gradle + jmockitでjava.lang.IllegalStateExceptionって出たので対応](http://cadeveloper.hatenablog.com/))、
というのでそう書いたし`gradle --stacktrace`してload順確認したんですけど、
それでもException出たので、

```
jvmarg(line:"-javaagent:${configurations.jmockit.asPath.split(':')[0]}")
```

を入れて漸く動きました。

けど、折角JMockit入れたのに、
やっぱり別の時には`Timeout`も必要で。
JUnit4だと`@Rule`つけて`Timeout.millis(200)`でいいんですね今。
そしたら、JMockitで振る舞い変えなくても、
GroovyでなくJavaなら`Timeout`だけで済むじゃーないですかー!
うぁーん。

あとはParameterized Testにしました。
[JUnitParams](https://github.com/Pragmatists/JUnitParams)が簡便そうです。
何がいいって、[JUnit4標準の方法](https://github.com/junit-team/junit4/wiki/parameterized-tests)だとinstance fieldがparameterになるので、
全methodについて回っちゃいませんか? というのが不安で。
JUnitParamsなら、明示的に単一methodに対してparameter指定できるので安心です。

...ダメです。JUnitParams使うと、Timeout効かないことがわかりました。
悲しい。
