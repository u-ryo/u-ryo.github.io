---
layout: post
title: "set my postfix able to send over OP25B(Outbound Port 25 Blocking) using Mailgun"
date: "2015-11-15 05:00"
author: 'u-ryo'
categories: [mail, postfix, mailgun, smtp]
comments: true
published: true
---
OP25Bが始まってから、ってもう大分前のことですが、
自前でmailを送るのは諦めて、
GMailを経由して送ってました。
でも、GMailを経由すると、
Fromが強制的に自分のaccountになっちゃうんですよね。
だから、例えば自前serverで定義したMLも、
上手く機能しなくなっちゃいました。
もうそういうのは自前でやる時代じゃないのかな、
と諦観してたんですけど、
[Mailgun](http://mailgun.com)を使えばspfとかcleanにして外に出してくれるんですね。
今頃になって知ったので、試してみました。

### cf.
* [Mailgunを使ってOB25P対策(postfix)](http://mjhd.hatenablog.com/entry/defeat-ob25p-with-mailgun)
* [Postfix SMTP relay setup for Sendgrid/Mailgun](https://community.rackspace.com/products/f/28/t/63)
* [How to start receiving inbound email](https://documentation.mailgun.com/quickstart-receiving.html#how-to-start-receiving-inbound-email)

### 自分でちょっとハマったところ
0. postfixのmain.cfで、`relayhost = [smtp.mailgun.net]:587`
と[]で囲う必要があった事(でも囲わなくても行けるみたいですよ
`/etc/postfix/password`の方でも囲わなければ)
0. MyDNSにおいて、spfを指定する場合には、Hostに`_spf`と入れなければならないこと
(MyDNSのlocal ruleでした)
0. DKIM(domainkey)の指定がなかなか反映されなかったようであること
(その他の項目は全て反映されてたし、
`$ host -t txt ...`で引くと見えてたので、大丈夫だろうと思ってたんですが、
そうでもなかったようです。
小一時間待ったら、認識されました。
ここが通らないとVerifiedにならず、使えないので、途方に暮れかけました)
0. 受信の設定はroutesなるものを設定しないとならないこと
(MXを直接自前serverに向ければいいんですが、
mailgunで受信するにはどうしたらいいんだろう?
と思っていると、そういうこと[=routesを設定する必要がある]だったんですね)
自分は、GMailにforwardするようにしました。
そうすればGMail見てるだけで済みますので。

久し振りに自前serverのmail logを見たら、
中国からのbrute force attackがずっと続いていたことがわかりました。
嫌なので、mail serverは落として、
GMailへのforwardingにすることにしました。