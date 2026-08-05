---
layout: post
title: "Developing and Building Cordova App"
date: "2015-11-10 07:36"
author: 'u-ryo'
categories: [Cordova, Cloud9, PhoneGap, GitHub, Android, iOS]
comments: true
published: true
---
all cloud環境で、hybrid applicationsを開発する方法がわかりました。

1. [Cloud9](http://c9.io)で作る
2. [GitHub](https://github.com)に置く
3. [PhoneGap Builder](https://build.phonegap.com)でbuild

### Cloud9
templateは何を選んでもnode.jsが付いてくるみたいです。
[Developing Ionic (Cordova) apps in the cloud with Cloud9](http://daniel.favand.net/2014/11/21/developing-ionic-cordova-apps-in-the-cloud-with-cloud9/)
に従って、以下project nameを「firstproject」とします。

```
$ npm -g install cordova ionic
$ ionic start firstproject
$ cd firstproject
```

これで準備が出来ます。
[official document](https://docs.c9.io/docs/ionic)にもありますが、
`$ ionic start firstproject sidemenu`とすれば、
tabではなくsidemenu型のprototypeが用意されるようです。

### GitHub

buildには、上述のように、PhoneGap Builderを使います。
そのためには、一度GitHubにpushせねばなりません。

まずGitHubに行って、repositoryを作ります。
その上で、

```
$ git init
$ git add .
$ git commit -m "Initial commit"
$ git remote add origin https://github.com/u-ryo/firstproject.git
$ git push -u origin master
```

### PhoneGap Builder

そうしておいてから、
[PhoneGap Builder](https://build.phonegap.com)(passwordは英小文字大文字8文字以上)に行って、
buildして、端末にはQR codeで読み込ませれば、
install、実行が出来ます。

はぁ〜、イマドキのアプリって、こういう風に作れちゃうんですねー。