---
layout: post
title: "Setting password for unreachable mail user on JHipster"
date: "2018-11-24 07:42"
author: 'u-ryo'
categories: [jhipster]
comments: true
published: true
---
JHipsterでuserを作る時にmail addressは必須で、
基本的にはnew user作るとJHipsterは裏でmailを投げています。
投げて失敗してもnew userは作れるんですが、passwordを設定できません。
どうしたらいいんだろう? と思ってlog見ていたら、
`http(s)://ホスト名/#/reset/finish?key=XXXXXXXXXXXXXXX`にaccessすれば通常の手続きに乗れるんですね。
このkeyはどこで規定されるのかと思ったら、`JHI_USER.RESET_KEY`の値です。
ここだけではダメで、`JHI_USER.RESET_DATE`に`now()`等で日付時刻をsetしないと、そしてその時刻から24時間以内にaccessしないと、ならないです。
あるuserのpassword強制resetにも使える、と思います。

ただこれをやると、`UserService#completePasswordReset`の中の`cacheManager.getCache`でヌルポが出るようでどうしようもなかったです。なので、`UserService.java`中の`cacheManager.getCache(...)`を(当該method以外でも)全て`try...catch`で囲いました。だって無害でしょう?


追記: JHipsterでのlogin dialogには「パスワードを忘れましたか」というlinkがあって、
それを踏むと`http://127.0.0.1:8080/#/reset/request`というpageに遷移し、
mail addressを入れると、上記初期登録時同様にActivateを要求するmailが飛びます。
初めてやってみました。へぇよく出来てますね。
そのmailに書いてあるURLも初期登録時同様`http://127.0.0.1:8080/#/reset/finish?key=XXXXXXXXXX`です。
