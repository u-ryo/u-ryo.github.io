---
layout: post
title: "AuthType None"
date: "2015-10-27 15:11"
author: 'u-ryo'
categories: [web, apache, authentication]
comments: true
published: true
---
さっき同僚に聞かれて調べました。
apacheにおけるBasic認証/Form認証等を
子ディレクトリにおいては無効にしたい場合、
.htaccessに、

```apache
Satisfy Any
```

と書けば良い、という説が諸所にあったのですが、
ちゃんとキャンセルするには、

```apache
AuthType None
Require all granted
```

とすべき、みたいです。
「AuthType None」だけだと、
「No authentication done but request not allowed
without authentication for /~u-ryo/test/test/.
Authentication not configured?」と言われて500
Internal Server Errorになってしまいます。

親ディレクトリではIP制限とBasic認証を同時にかけて、
子ディレクトリではIP制限だけ、
且つBasic認証のポップアップは出ないように
(IP制限で弾かれれば即403)、
という要件だったので、

### 親
```apache
AuthType Basic
AuthName test
AuthUserFile password.txt
require valid-user
Allow from 127.0.0.1
Satisfy Any
```

### 子
```apache
AuthType None
Require all granted
Order deny,allow
Allow from 127.0.0.1
Deny from all
Satisfy All
```

といった感じで解決しました。
