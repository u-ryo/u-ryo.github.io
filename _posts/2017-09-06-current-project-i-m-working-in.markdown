---
layout: post
title: "Current Project I'm working in"
date: "2017-09-06 22:02"
author: 'u-ryo'
categories: [android]
comments: true
published: true
---
今、グループ内会社のAndroidアプリ開発に売られてるんですけど、そこのソフトの作りがひどくてひどくて泣けてきます。

1. 「結果が表示されなくなった」というので見てみたら、`toString()`が変わっていたのが原因。より根本的な原因は、`toString()`というdebug用途のmethodをoverrideしてmainのlogicに使っていること。まぁ、Activity跨ぐstructured dataを`Parcelable`にする時間が無かった、という事情は分かるんですけど、`StringBuilder#toString`ですら使わず`new String(StringBuilder)`とするくらいなのでぼくは。
1. 「途中で落ちる」というので見てみると、ヌルポが出てました。どうしてかなー、と見ていくと、途中でnullを代入しているmethodが呼ばれています。どうしてこれを呼ぶよう変えたのか聞いてみると、終了処理をちゃんとするようしてる時に、comment outしてあったこのfinishっぽいmethodをcomment inしたんだそう。それがどういう効果を持つのかわからぬまま、そうしたんだって。えーーーっ!?
1. 極めつけは、今日わかったんですが、`HashMap`を`List`にしてその0番目を使ってるんですね。えーーーっ! どうしてAndroid 4.4.2ではうまく動かないの? というのを探っていったら、そこに行き着きました。逆に、これまでよく動いていましたねぇ。素晴らしい!! 先月までいた派遣のフリー技術者が書いたcodeの一部でしたけど、わざと書いたならいざ知らず、もし意識せず書いたのなら、恐ろしいです。
