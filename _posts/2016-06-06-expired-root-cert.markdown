---
layout: post
title: "Expired Root Cert"
date: "2016-06-06 13:23"
author: 'u-ryo'
categories: [fortigate, network, security, certificate]
comments: true
published: true
---
Fortigateの認証proxy(且つSSLも検閲できるよう間に独自Certをかます)を有効にしたら、
突然見られなくなったサイトが。
[DUNS Number検索](https://duns-number-jp.dnb.com/search/jpn/login.asp)っていうところなんですけど、
なんでだろー、って調べてったら、とんでもないことが判明。

```
$ openssl s_client -connect duns-number-jp.dnb.com:443 -showcerts|awk -v b=0 '{if($2~/CERTIFICATE/){b++};if(b==5){print}}END{print "-----END CERTIFICATE-----"}'|openssl x509 -enddate -noout
depth=2 C = US, O = DigiCert Inc, OU = www.digicert.com, CN = DigiCert High Assurance EV Root CA
verify error:num=20:unable to get local issuer certificate
verify return:0
DONE
notAfter=Sep 30 18:19:47 2015 GMT
```

<%= img src: '/images/duns-number-jp-1.png', alt: 'SSL Report by SSL Labs' %>

<%= img src: '/images/duns-number-jp-2.png', alt: 'SSL Report by SSL Labs' %>

[SSL Report by SSL Labs](https://www.ssllabs.com/ssltest/analyze.html?d=duns-number-jp.dnb.com)

そんな、わざわざ期限切れのRoot CAなんて配んなくっていいのに...

ただ、そういえば、自分もSSL Cert更新時、
中に含めていた中間証明書をそのまま使い回してverifierにかけたら、
中間証明書が古いって言われて慌てて差し替えたことあります。
自分で作ったcertは期限気にしますが、
間に含めたcertsまではあんまり気にしないんですよね確かに。

気持ちは、わかります。