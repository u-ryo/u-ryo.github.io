---
layout: post
title: "Let's Encrypt"
date: "2016-08-02 15:55"
author: 'u-ryo'
categories: [ssl, certificate, let's encrypt]
comments: true
published: true
---
[Let's Encrypt](https://letsencrypt.org/)、
漸く使えるようになりました。
具体的にどうやって使ったらいいんだろう、
と思ってたんですが、結構簡単に使えますねこれ。
素晴らしい、です。
おかげでbotnetのC&C server等でも広く使われるようになったとかってことですが。

ともあれ、
日本語の要諦サイト[Let's Encrypt 総合ポータル](https://letsencrypt.jp/)もあり、
だいぶわかりやすくなってます。

基本的には、

1. `certbot-auto`を`git`又は`wget https://dl.eff.org/certbot-auto`で取得
1. 認証原理は、
    1. `certbot-auto`がLet's Encrypt側から指定された`http://.../.well-known/XXXXXX`にfileを作る
    1. Let's Encrypt側がそのfileを`GET`、認証
1. なので`standalone`(`certbot-auto`が臨時のhttp serverを建て、Let's Encryptからのrequestを受ける)が基本
1. その間本来のWWW serverを止めねばならないが、止めたくない時は「`--webroot`」で`DocumentRoot`を指定し、`certbot-auto`にfileを作らせる
1. 初回の(というか最後に成功した)command optionsを`/etc/letsencrypt/`以下に保存しておき、次回以降「`renew`」commandでは省略可
1. `renew`を付けて走らせても、証明書期限が30日を切っていないと発動しない(無理やり更新するには`--force-renew` optionが必要)
1. `--post-hook`も指定して、apacheのreloadを自動化(これは`renew`が走らないと発動しない。単純に`renew`の終了status codeだと0になるので「&&」で繋ぐだけでは判別不能だった)
1. 証明書取得がうまく行くと、`/etc/letsencrypt/live/someserver.co.jp/`以下に各種cert filesのsymbolic linkが作成されるので、`/etc/apache2/site-enabled/default-ssl`中で`SSLCertificateFile`等の示す先をここの`cert.pem`等にする(Apache2.4以上やnginxなら証明書＋中間CA証明書に`fullchain.pem`が使える)

こちとら、apache2 serverを止めたくない状況だったので、

```
$ sudo cerbot-auto certonly --webroot -w /var/www/ -m support@company.co.jp -d someserver.co.jp --agree-tos
```

(`-w`: webroot-path, `-d`: domain, `-m`: mail address)

更新は、`/etc/cron.daily/certbot`を作って、

```
#!/bin/sh
cerbot-auto renew -q --post-hook 'service apache2 reload'
```

で良さそうです。`sudo chmod a+x /etc/cron.daily/certbot`を忘れずに。

`certbot-auto`は、実行するといきなり`apt-get update`してからpython2.7とか入れるのと幾つかのpackagesをupdateするので注意です。

また、Let's Encryptにしてから[SSL Server Test](https://www.ssllabs.com/ssltest/)をかけてみると、「Incorrect SNI alerts」というのが出るようになりました。
これは、`/etc/apache2/site-enabled/default-ssl`の`<VirtualHost _default_:443>`に`ServerName XXXXXX`とすることで解決しました。
cf. [SSLテストで”Incorrect SNI alerts”を解決する](https://www.rootlinks.net/2016/02/09/sslテストでincorrect-SNI-alertsを解決する/)

### command log

```
wget https://dl.eff.org/certbot-auto
chmod a+x certbot-auto
sudo chown root:root certbot-auto
sudo mv -i certbot-auto /usr/local/bin/
sudo certbot-auto certonly --webroot -w /var/lib/tomcat7/webapps/ROOT/ -m ls-support@jmtech.co.jp -d cx4.locationsupporter.info --agree-tos
sudo vi /etc/apache2/sites-enabled/default-ssl
================================================
	SSLCertificateFile /etc/letsencrypt/live/cx4.locationsupporter.info/cert.pem
	SSLCertificateKeyFile /etc/letsencrypt/live/cx4.locationsupporter.info/privkey.pem
	SSLCertificateChainFile /etc/letsencrypt/live/cx4.locationsupporter.info/chain.pem
================================================
echo '#!/bin/sh' > certbot
echo "cerbot-auto renew -q --post-hook 'service apache2 reload'" >> certbot
chmod a+x certbot
sudo chown root:root certbot
sudo mv -i certbot /etc/cron.daily/
```