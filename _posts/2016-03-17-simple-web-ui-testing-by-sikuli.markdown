---
layout: post
title: "Simple Web UI Testing by Sikuli"
date: "2016-03-17 07:35"
author: 'u-ryo'
categories: [sikuli, test]
comments: true
published: true
---
[Sikuli](http://www.sikuli.org/)を使うと、
簡単にWeb UIをtest出来ます。
あ、Web UIに限りませんか。Desktop上のアプリ全て、というべきでしょう。
何と言っても、「このボタンをclick」とかっていう指定が、
当該ボタンの画像で示せる、
その画像を取るのもbrowser上で範囲指定すればいいだけ、
というのがなかなかshockingでした。

Sikuli自体は[Java API](https://code.google.com/archive/p/sikuli-api/)があるので、OS free、script化も可能っぽいです。
[SikuliX](http://www.sikulix.com/)でGUIでお手軽テスト自動化、のみならず。
ただ、「ここを選んでclickして」とかっていう手順はすんなり書けるんですが、
Captchaを乗り越えるために裏でlogic組みたかったんですけど、
それがどうもSikuliXではうまく出来ない感じがしたので、諦めました。
あぁ、上記のSikuli-apiを使ってJava Test classとして書けば良かったんですね。
出来ないことはないですか。ぼくが見落としただけです。
「ここ」って示す画像を沢山用意しないとならなさそう、です。

じぇじぇ、GebとSikuli、[一緒に使えそう](https://fbflex.wordpress.com/2012/10/27/geb-and-sikuli/)ですね。
もうGebで書いちゃいましたよトホホ。