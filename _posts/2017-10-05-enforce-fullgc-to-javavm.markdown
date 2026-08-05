---
layout: post
title: "Enforce FullGC to JavaVM"
date: "2017-10-05 20:51"
author: 'u-ryo'
categories: [java, gc]
comments: true
published: true
---
あ、FullGCって強制的にかけられるんですね。

[JavaでFull GCを実行する方法](http://cco.hatenablog.jp/entry/2013/05/20/223212)

要するに、

```
jmap -histo:live <pid>
or
jcmd <pid> GC.heap_dump <output_filename>
```

「生存中のオブジェクトのみ抽出したメモリーマップを作成するために、直前にFull GCを実行するのでこれを利用する」のだそう。「対象のJavaプロセスを実行しているユーザで実行する必要があるので注意」ご尤も。
