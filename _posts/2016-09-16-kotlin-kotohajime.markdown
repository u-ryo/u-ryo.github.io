---
layout: post
title: "Kotlin KOTOHAJIME"
date: "2016-09-16 11:47"
author: 'u-ryo'
categories: [kotlin, android, android studio, java]
comments: true
published: true
---
## Kotlin事始め
最初、title間違えました。
タイプミスです。
Ergodox EZ(キー配列をカスタマイズ出来るキーボード)が必要かもしれません...

Android開発を、というかIoTに乗ってSmart Glass(Vuzix M100)の
アプリ開発を任命付けれられています。
それはそれでありがたいのですが、
今更ながらAndroid開発を学んでいます。
素のAndroid Javaを使う、というのはあまりにもイケてなさそうなので、
いちいち`on何々`なんてやってられませんよ、ということで、
Kotlinを使ってイマドキのReactive Programmingを、
と思っています。
Kotlinやってると、Groovyでもいいんじゃないの?
と思ってしまうのですが、
殊Androidについては、同梱libが大きくなってしまうなど、
やはり後発の強みもあってKotlinのようです。
ただ、Android開発者がJava loveだそうで、
今後のJava8の取り込みようによっては、
Javaだけでも行けるのでは? という気も少しします。
しかしそれは普及率を考えるとまだまだ遠いので、
今はやはりKotlinかと。

しっかし、Android開発って結構めんどいんですね。
IDEがないと開発できないって、
古いtypeの人間としてはそこから苦痛なんですが...

最初は、[森洋之,基本からしっかり身につくAndroidアプリ開発入門 Android Studio 2.x対応 プロが本気で教えるアプリ作りの基本「技」 (ヤフー黒帯シリーズ),SBクリエイティブ,2016.7](http://www.sbcr.jp/products/4797384505.html) を間違い直しながら写経することで
Android開発を「Hello World」から卒業してから、
[長澤太郎,Kotlinスタートブック,リックテレコム,2016.7](http://taro.hatenablog.jp/entry/2016/06/16/203136) を写経しています。
そこでハマったことを。

1. 実行しても、エラーメッセージを吐かずにemulatorが一瞬立ち上がった後終了 → Run対象を`MainActivityTest`にしていたから
1. Dagger2でinjectionするようにしたらprofileImageUrlを取れなくなった → `.dagger.ClientModule#provideRetrofit`で`.addConverterFactory(GsonConverterFactory.create(gson))`の`gson`を抜かしてしまっていた
1. Dagger2でinjectionしたtestでCastがこけてうまく行かない → `build.gradle`で`testInstrumentationRunner 'sample.qiitaclient.MockTestRunner'`に変更しないとならないのと、`Run` → `Edit Configurations` → `Android Tests` → `MainActivityTest` で`Specific instrumentation runner (optional):` を `〜.qiitaclient.MockTestRunner` に変更しないとならない

IDEに慣れてないので、fileばっかり見比べてました。downloadしたお手本sampleではDagger2 Testがうまく行くのに、何で自分のはうまく行かないんだろう、ってずっと悩んでました。IDEでのproject settingに問題があったとは。
[stackoverflowのTest running failed: Unable to find instrumentation info for: ComponentInfo{} — error trying to test in IntelliJ with Gradle](http://stackoverflow.com/questions/24002212/test-running-failed-unable-to-find-instrumentation-info-for-componentinfo)で見付けました。
こんなところに設定があるなんて、知りませんよー。