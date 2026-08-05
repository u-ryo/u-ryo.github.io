---
layout: post
title: "ssh access to Koding"
date: "2015-10-28 00:59"
author: 'u-ryo'
categories: [Koding, ssh]
comments: true
published: true
---
Kodingなんですが、sshで繋げてログインすることが出来るんですね。
びっくりポンです。

1. ~/.ssh/authorized_keys に手元のマシンのpublic keyを記述
2. Koding上のマシン名を左側のVM Settingsで確認(毎日変わる模様)
3. 手元からssh(公開鍵認証のみ)

tmuxも使えるので超便利です。
ただ、デフォルトのコントロールキーがC-bになっているので、
最初は戸惑います。
慣れ親しんだC-aに変えるには、
~/.tmux.conf
に

```
set-option -g prefix C-a
```

ちゃんと公式ドキュメントに書いてありました。  
[Using Tmux on Koding](http://learn.koding.com/guides/using-tmux-on-koding/)  
(一旦tmuxを全て終了させないと反映されません)