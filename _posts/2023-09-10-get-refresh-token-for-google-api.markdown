---
layout: post
title: "Get Refresh Token for Google API"
date: "2023-09-10 15:03"
author: 'u-ryo'
categories: [google, python]
comments: true
published: true
---
何かいつの間にかGoogle APIの認証方法が変わってて、security上の理由からというので仕方ないんでしょうけど。

`AUTHORIZATION_CODE`を得ようと、以前のように↓のURLにaccessすると、
https://accounts.google.com/o/oauth2/v2/auth?response_type=code&client_id=...apps.googleusercontent.com&redirect_uri=urn:ietf:wg:oauth:2.0:oob&scope=https://www.googleapis.com/auth/photoslibrary&access_type=offline
`アクセスをブロック: ... のリクエストは無効です`

と出て、あれ?!と。

> エラー 400: invalid_request
> The out-of-band (OOB) flow has been blocked in order to keep users secure. Follow the Out-of-Band (OOB) flow migration guide linked in the developer docs below to migrate your app to an alternative method.

で、[帯域外（OOB）フロー移行ガイド](https://developers.google.com/identity/protocols/oauth2/resources/oob-migration?hl=ja)へ誘導され。

このoobフローはなくなったと!? んじゃどうせぃっていうんじゃねん。

「デスクトップクライアント」に相当するから、[ループバック IP アドレス（localhost または 127.0.0.1）フロー](https://developers.google.com/identity/protocols/oauth2/native-app?hl=ja#redirect-uri_loopback)に飛ばされたものの、こっちもなくなってて、[ループバック IP アドレスフローの移行ガイド](https://developers.google.com/identity/protocols/oauth2/resources/loopback-migration?hl=ja)に飛ばされて、結局library使う方法しか書いてなくて。こちとら、shell scriptで使いたいんですけど。`refresh_token`欲しいだけなのに、なんでこんなに苦労せなかんの??🤔

...って嘆いても仕方ないので、library使って`refresh_token`取得だけします。

ちょっとめんどくさいんですけど、local環境を汚染しない形で。

参考:

- [PythonからGooglePhotoに画像や動画をアップロード](https://qiita.com/sey323/items/4c7140efaed750e690eb)
- [PythonからGoogleDriveにファイルをアップロード](https://qiita.com/sey323/items/875c0ab1585044772ab2#%E8%AA%8D%E8%A8%BC%E6%83%85%E5%A0%B1%E3%81%AE%E4%BD%9C%E6%88%90)

Google APIの[認証情報のpage](https://console.cloud.google.com/apis/credentials)から「OAuthクライアントをダウンロード」して`client_secret.json`として保存。

```sh
$ docker run --rm -it -v $PWD/client_secret_....json:/root/client_secret.json python:alpine sh
# apk add w3m screen
# pip install google-api-python-client google-auth-oauthlib
# screen
# python
>>> from google_auth_oauthlib.flow import InstalledAppFlow
>>> credential=InstalledAppFlow.from_client_secrets_file('client_secret.json',["https://www.googleapis.com/auth/photoslibrary.appendonly"]).run_local_server()
```

ここで`w3m`が開くので、`Q`を押して閉じるとURLが表示され、そこをChromeなりで開いて、突き進んで承認します。

そうすると、http://localhost:8080/?state=.... へ回されて止まるので、`C-a`で別screenを開いて、そこで`w3m 'http://localhost:8080/?state=...'`とすると、`The authentication flow has completed. You may close this window.`と言われます。徐に元のscreenに戻ると`credential` objectができているので、`credential.refresh_token`とすると、漸く`REFRESH_TOKEN`を得られます。1時間有効な`ACCESS_TOKEN`は`credentail.token`で得られます。

これらを使うと、やっと以前のようにshell scriptで回せるようになります。ふぅ。
