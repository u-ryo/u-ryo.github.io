---
layout: post
title: "Schedulers.newThread() or Schedulers.io()"
date: "2018-10-10 09:57"
author: 'u-ryo'
categories: [android, java]
comments: true
published: true
---
Robolectricでtestを書いていて、
どうも上手く行かないところがありました。
「上手く行かない」というのは「Test class内で(RxJava1系なので)`RxJavaHooks.setOnIOScheduler(s -> Schedulers.immediate());`しているにも関わらず、
test対象classで`subscriber`が動いてしまう」というものです。

どうしてかなぁ、`.subscribe(...)`の中で`.subscribe(...)`してるからかなぁ、
[RxJava のテスト(1): TestSubscriber, test(), TestScheduler](https://hydrakecat.hatenablog.jp/entry/2016/12/12/RxJava_のテスト(1):_TestSubscriber,_test(),_TestScheduler)を見て、
じゃぁっていうんで`TestScheduler scheduler = Schedulers.test();`して何度か`scheduler.triggerActions();`しても現象変わらずでした。

悩んだ末、わかったのは、
test対象classでは`.subscribeOn(Schedulers.newThread())`していた、
ということでした。
なぁんだ。

そういえばちょっと前に書いたところだったので、
まだ`Schedulers.newThread()`にしちゃってたんですね。
今回`Schedulers.io()`に変えました。
Androidでも効くんでしょうか。
