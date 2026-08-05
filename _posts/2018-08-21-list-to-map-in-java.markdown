---
layout: post
title: "List to Map in Java"
date: "2018-08-21 02:09"
author: 'u-ryo'
categories: [java, stream, list, map]
comments: true
published: true
---
Javaで`List`を`Map`にしたかったんですね。
[【Java入門】List⇔Map変換でJava8のStreamを使う方法](https://www.sejuku.net/blog/15900)を見ました。
ここは、いいこと書いてあるんですけど能書きが長い、Java8以前の話は不要(at least for me)、広告が多い感じがする、のが...

基本は単純に`Collectors.toMap(s->key, s->value)`でいけますと。
但し、duplicated keyがあった場合即Exceptionに。

```
jshell> Stream.of("A quick brown fox jumps over the lazy dog.".split(" ")).collect(Collectors.toMap(s->s.length(),s->s)
   ...> )
|  java.lang.IllegalStateException thrown: Duplicate key 5 (attempted merging values quick and brown)
|        at Collectors.duplicateKeyException (Collectors.java:131)
|        at Collectors.lambda$uniqKeysMapAccumulator$1 (Collectors.java:178)
|        at ReduceOps$3ReducingSink.accept (ReduceOps.java:169)
|        at Spliterators$ArraySpliterator.forEachRemaining (Spliterators.java:948)
|        at AbstractPipeline.copyInto (AbstractPipeline.java:484)
|        at AbstractPipeline.wrapAndCopyInto (AbstractPipeline.java:474)
|        at ReduceOps$ReduceOp.evaluateSequential (ReduceOps.java:913)
|        at AbstractPipeline.evaluate (AbstractPipeline.java:234)
|        at ReferencePipeline.collect (ReferencePipeline.java:511)
|        at (#9:1)
```

なので、`Collectors.toMap(s->key, s->value, (s1,s2)->do)`のようにduplicated keyがあった場合の処理もlambdaとして第三引数に書けます。
後勝ちの例↓

```
jshell> Stream.of("A quick brown fox jumps over the lazy dog.".split(" ")).collect(Collectors.toMap(s->s.length(),s->s,(s1,s2)->s2))
$11 ==> {1=A, 3=the, 4=dog., 5=jumps}
```

SQLっぽく`Collectors.groupingBy(s->key)`を使うと、重複valuesはListに込めてくれるので楽です。

```
jshell> Stream.of("A quick brown fox jumps over the lazy dog.".split(" ")).collect(Collectors.groupingBy(s->s.length()))
$12 ==> {1=[A], 3=[fox, the], 4=[over, lazy, dog.], 5=[quick, brown, jumps]}
```
