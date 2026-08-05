---
layout: post
title: "Push Notification from command line by ntfy"
date: "2018-08-24 14:14"
author: 'u-ryo'
categories: [python, ntfy, push, notification]
comments: true
published: true
---
コマンドラインからPush通知が出来るというのでやってみました。
([Linuxのコマンドラインやスクリプトからスマホにプッシュ通知する。（ntfyというツールの紹介）](https://qiita.com/kjtanaka/items/8b0c90d28909e50e4a6d))
[PushBullet](https://www.pushbullet.com/)を使ってみました。
ChromeやFirefoxだとextension、
スマホだとアプリを入れます。

```
sudo apt install python-pip
pip install ntfy
```

で`ntfy`を入れ、`~/.ntfy.yml`にPushBulletのaccess_tokenを入れ、
`~/.local/bin/ntfy -t 'Title' send 'notification contents'`
とすると、送れました。
スマホ側の音とか振動は、アプリでの設定にて調整可能です。

最初よくわかってなかったのですが、
PushBulletのアプリを入れると、
電話帳やSMS(Short Message Service)もぶっこ抜いてくるんですね。
それと知らずにSMS送ってしまい、相手に不審がられてしまいました。
