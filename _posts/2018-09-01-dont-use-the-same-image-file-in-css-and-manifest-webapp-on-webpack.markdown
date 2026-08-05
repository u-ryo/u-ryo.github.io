---
layout: post
title: "Dont use the same image file in css and manifest.webapp on webpack"
date: "2018-09-01 20:26"
author: 'u-ryo'
categories: [webpack, angular, jhipster, pwa]
comments: true
published: true
---
notificationに使っていたlogicを変えたいと思って、
アプリとしてinstallしていたPWAを一旦uninstallしてから
browserでそのpageを読んでも、「ホーム画面に追加」が現れなかったんですね。
何でんだろー、一旦uninstallしちゃうとダメなのかな?
いやそんなことないだろうな、とは思いつつも
明確に「uninstallしても大丈夫」だと書いてあるところもなく、
心配だったんです。
「ホーム画面に追加」が出ないのは何が悪いのか、わからないんですよね。

「ホーム画面に追加」が出る条件、というのがあって、
[公式ページ(ウェブアプリのインストール バナー)](https://developers.google.com/web/fundamentals/app-install-banners/?hl=ja)によると、

1. `short_name`(ホーム画面で使用)
1. `name`(バナーで使用)
1. 192x192のpngアイコン(アイコンの宣言にはMIMEタイプ`image/png`の指定が必要)
1. 読み込み先の`start_url`

というんですが、
[「ホーム画面に追加」からはじめる『PWA(Service Worker)』](https://qiita.com/narikei/items/4240f03542f29e313989)には192のpngの話はなく、
また[【3ステップではじめる】PWAによる「ホーム画面に追加」バナーの実装](https://amymd.hatenablog.com/entry/2017/10/12/001612)では`fetch` eventの定義が必要だとあります。
こちとら、一度は出来たわけだし、実際には
[PWAで「ホーム画面に追加」が表示されない時に確認する事](https://www.bunkei-programmer.net/entry/2018/04/09/125015)にある「manifest.json内のアイコン画像が404である」といったところが一番多いんじゃないでしょうか。
この人もそうだったといいますし、ぼくもそうでした。

ただ、その原因は困ったものでした。
webpackでhash化された名前の付いたpng fileが壊れていたからです。
とても悩みました。調べてもそんな事例、出てきません。
色々試行錯誤した挙句わかったのは、
`css`内と`manifest.webapp`(`manifest.json`)内で
指定しているpngが同じだと、
webpackがpng fileをhash名化する時にバッティングするようで、
生成後のpng fileはviewerで見えないし、
`file`コマンドで見ると「data」とか出るし。
`css`でのpng fileを`../../../../../../webpack/logo-jhipster.png`
といったようにして`manifest.webpack`と別物にすると、うまく行きました。
`./src/...`以下でなくてもいいんですね!
