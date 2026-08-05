---
layout: post
title: "Let's Encrypt without certbot-auto"
date: "2016-09-13 11:23"
author: 'u-ryo'
categories: [ssl, let's encrypt, bash]
comments: true
published: true
---
Let's Encrypt、素晴らしいんだけど、
とあるserverで、使っているのがUbuntu 11.04(Natty)、
package systemが何か壊れてて、
依存関係がおかしくて解消できなくて、
というcaseがありました。
基本的にどこを調べても、
Let's Encryptを使うには既存のpackaging systemをbaseにしたものばかりで、
Stackoverflowなんかでも「backportsを使って既存のpackaging依存関係を
修築してから」なんていう話しかなくて。
なんかこう、`certobot-auto`を使わないで、
っていう話って意外となくて途方に暮れていると、
こんなのがありました。

[An ACME Shell script: acme.sh](https://github.com/Neilpang/acme.sh)

要は、bashだけでLet's Encryptの証明書を使える、
というもので、
なーんだ最初から全部これでいーじゃん、
という感じです。

`wget -O - https://get.acme.sh | sh`でも行けるというのですが、
何かserver certが違うらしく、`--no-check-certificate`を付けてもうまく行かず。
幸いgitはあってくれたので、cloneして`cd acme.sh` `./acme.sh --install`すると、
home直下に.acme.shというdirectoryを掘り、
その下にCSRやLet's Encryptからの証明書を自動で作ってくれる、
更に実行userでcron登録してくれる、
というものなので、実行だけならroot権限不要とはいえ、
証明書filesが一般user home directory以下というのもなんなので、
rootでやった方が良さげです。
というわけで、

```
# git clone https://github.com/Neilpang/acme.sh
# cd acme.sh
# ./acme.sh --install
# ./acme.sh --issue -d dev.locationsupporter.info -w /var/lib/tomcat7/webapps/ROOT/
# vi /etc/apache2/sites-enabled/default-ssl
=============================================
SSLCertificateFile /root/.acme.sh/dev.locationsupporter.info.cer
SSLCertificateKeyFile /root/.acme.sh/dev.locationsupporter.info.key
SSLCertificateChainFile /root/.acme.sh/fullchain.cer
=============================================
# crontab -l
```

これで、pytho2.7もpackage updateも不要だし、
更にお手軽にLet's Encryptを使えるというものでしょう。