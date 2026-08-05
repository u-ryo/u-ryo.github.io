---
layout: post
title: "Codenvy for GAE"
date: "2015-11-11 05:09"
author: 'u-ryo'
categories: [Cloud IDE, GAE, Google App Engine]
comments: true
published: true
---
[Codenvy](http://codenvy.com)はGAEに直接deploy出来るからいい、
って読んだんですけど([もうEclipseはいらない? codenvy IDEによるクラウド開発 (1/6)](http://libro.tuyano.com/index3?id=912006))、
じゃぁっていうんで使ってみたら、
うーん、command lineが使えないので、
JARなprojectとかうまく動かせない(buildしても成果物をどうやって取り出すのか、
どうやってoptionつけて実行して試せるのか)感じでした。
あと、月20時間?しかtest起動、buildが出来ないみたいですね。

[Cloud9](http://c9.io)ではGAE連携できないのかな、
と調べてみると、GAEってgitでもdeploy出来るって?!
へーと思ってGAEを見てみると、今は出来ないみたいですね...
Google Cloud Consoleからソースの閲覧っていメニュー、
無くなってます。
まぁ、普通にcommand叩けば良いのかな? という感じです。

[Koding](https://koding.com/)も、
無料だと1プロジェクトしか作れませんが、
どうせGitHubに置いておくので、
とっかえひっかえすればいいのかなぁ。
でも流石に面倒そうですよねぇ。