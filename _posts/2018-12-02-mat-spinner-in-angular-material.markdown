---
layout: post
title: "mat-spinner in Angular Material"
date: "2018-12-02 09:39"
author: 'u-ryo'
categories: [angular, material, spinner, loader]
comments: true
published: true
---
Angularで「ロード中」を簡単に実現する方法を探していて、
[Angular Material](https://material.angular.io/components/progress-spinner/overview)にあるんですね。`<mat-spinner></mat-spinner>`だけ。凄く簡単。
でも、実際にやってみても出て来なかったので不思議でした。
いや、[sample](https://stackblitz.com/angular/qyqovrjarbx?file=app%2Fprogress-spinner-overview-example.html)とかいくら見ても、特に何も`import`しなくてもいきなりtag書くだけで使えるよ、っていうんですけど、自分のには出て来ません。なぜ??
[動画](https://www.youtube.com/watch?v=Z6JdFWXh1Nc)見てやっとわかりました。
確かに当該component fileではimportの必要ありませんけど、
`app.module.ts`なりで`MatProgressSpinnerModule`のimport/importsが必要なんですね。
もし入ってなければ`BrowserAnimationsModule`も。
sampleはdefaultでMat系Module全部入りだからよくわかんなかったんですよ。
