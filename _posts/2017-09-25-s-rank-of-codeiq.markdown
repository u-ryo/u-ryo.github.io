---
layout: post
title: "S Rank of CodeIQ"
date: "2017-09-25 05:01"
author: 'u-ryo'
categories: [codeiq, java]
comments: true
published: true
---
もう昨日のことですが、[Paiza](https://paiza.jp/challenges/share/Ku6dnsSr8gw5zINY0ItXIQ1UXOOdmh6Xl_0akPz7zwo)に続いてようやっと[CodeIQ](https://codeiq.jp)でも[Sランク取りました](https://codeiq.jp/badge/3116)。

まぁ、u-ryoさんなら当然だよね、とか言われておしまいでしょうけど。

[Paizaにあった似た問題](https://paiza.jp/challenges/share/UwlvLJ8wbTeDuWz-kV7g2vpFehW0vZE-wI-qg56WM98)(←これで見ると失敗もありますが、これは最初に提出したcodeについてで、後でちゃんと全部通るようにしてます)からcode引っ張ってきて、でも勿論そのままでは使えなくて。土曜未明の一晩で終えるつもりが、間にバイトやmachine troubleを挟んで日曜の午後までかかっちゃいました。ふと探してみると、[まんまのcode](http://ideone.com/pfoy7z)があるのにはびっくり。流石にそのままっていうのは癪なので、「直前の方向を使う」というideaと検算にだけありがたく使わせてもらいました。
探索順の違いによるエラーをなかなか潰せなくて。そっか全探索なのね、そのためには... BFSではなくDFSか、と辿って、あーそっか、その違いはstackかqueueかだけか、に最後の最後気付いてgoalでした。何か色々忘れてて、こういうのやり続けてないと錆び付いてたなーというのを思い知らされました。この手のalgorithm系は集中してやらないと!

コメント書きすぎでしょうか。でもまんまの置いてあるよりは遥かにマシかと。
