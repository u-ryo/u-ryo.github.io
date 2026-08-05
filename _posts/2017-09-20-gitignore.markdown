---
layout: post
title: "gitignore"
date: "2017-09-20 00:54"
author: 'u-ryo'
categories: [git, gitignore]
comments: true
published: true
---
前も書いたような気がしますけど、`.gitignore`は[gitignore.io](https://gitignore.io/)で。ここでgenerateしたものに置き換えてみたんですが、`git status`はあんまり変わりませんでした。記述量は確かに増えていたので、良くはなっているんでしょう...

AndroidStudio用に引っ張ってきたんですけど、そのままだと`.idea/kotlinc.xml`が入ってきてしまうようです。`kotlinc.xml`はversion管理不要ですよね?
逆に、`.idea/gradle.xml`はversion管理下に、という話が[Android Studioのバージョン管理対象ファイル](http://www.torutk.com/projects/swe/wiki/Android_Studioのバージョン管理対象ファイル)、[Android Studioでバージョン管理下に置かないもの](http://qiita.com/komax/items/d1aaecaec0a22cb5bc4e)、[第35回　バージョン管理 ─プロジェクト管理ファイルについて［後編］](http://gihyo.jp/dev/serial/01/android_studio/0035)と出て来るんですが、`gitignore.io`のfileではexplicitlyにignore対象なんですよね...

試してみると、特に`.idea/`が無くてもAndroid Projectとして開けますね。じゃ全部要らないのかな。
