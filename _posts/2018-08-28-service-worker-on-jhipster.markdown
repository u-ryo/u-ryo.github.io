---
layout: post
title: "service worker on JHipster"
date: "2018-08-28 10:24"
author: 'u-ryo'
categories: [jhipster, java, spring, angular, workbox, service worker, web push, pwa]
comments: true
published: true
---
[JHipster](https://jhipster.tech)でService WorkerでWeb Pushを、
と思っていて、[Angular](https://angular.io/)だから、
[前やった](https://u-ryo.github.io/blog/2018/03/19/push-notification-on-angular5/)ように
`app.modules.ts`で`ServiceWorkerModule.register('./service-worker.js',...);`してから`@angular/service-worker`で`SwPush`をinjectして、
って思ってたんですが、違うんですね。
JHipsterでは[workbox](https://developers.google.com/web/tools/workbox/)でService Worker使うようになってるんですね予め。
確かに`index.html`で`service-worker.js`を有効にして挙動を見てると、
cachingは綺麗に行っている様子です。
えーでも調べてみると、workboxってWeb Pushはないじゃーないですかー。

> 特に私の知っている範囲ではworkboxはプッシュ通知のロジックを作ってくれないので、そこは自分で書いてやる必要があります。
> [ServiceWorkerを簡単に書けるworkbox-swの使い方](https://qiita.com/nazonohito51/items/32b61cabdac8b24769bd)

しかも、自分のcodeとmergeする、
即ち`workboxPlugin(...)`に`swSrc:`を追加すると、
`generateSW`が使えず自分で書かないといけないの!? ですか?
いやー、jhipsterで生成される`service-worker.js`とか見ると、
色んなfilesにhash値?が付与されているから、
これを自分で作るというのはあり得ないでしょー。
どーしたらいーのー?!

と、途方に暮れました。

workboxを捨ててAngularのSwPushにする?
いやー、でもjhipsterでのbuildでは`ng`実行されないので、
いくら`.angular-cli.json`に`"serviceWorker": true`と書いても効かないんですよー。
Angularのservice workerは生成されないわけですね。

jhipsterで生成された`build/www/service-worker.js`を見ていると、
上部のコメントに、

```
 * The rest of the code is auto-generated. Please don't update this file
 * directly; instead, make changes to your Workbox build configuration
 * and re-run your build process.
 * See https://goo.gl/2aRDsh
```

とあります。言われるままに[そのURL](https://goo.gl/2aRDsh)を見てみると、
`importScripts`というoptionがあることがわかりました。
これを指定するとどうなるのかなー、試してみます。
指定する場所は、webpackでworkboxのservice worker生成をやっているので、
`webpack/webpack.prod.js`になります。
それの最下段、`new WorkboxPlugin.GenerateSW({...})`の中に
`importScripts: ['push-notifications.js']`と書いてみると、
自動生成された`build/www/service-worker.js`中に、
`importScripts("push-notifications.js",...)`と出ます。
なるほど。
ではこの`push-notifications.js`というのを作ってwww root folderに置き、
そこにpush notificationのlogicを書いておけばいいんじゃーん。
でも、基本的には全てのfile名はhash化?されてしまいます。
どうやって名前をhash化されないようにするの?!
`favicon.ico`等の例から手探りで探し当てました。
`webpack/webpack.common.js`の`new CopyWebpackPlugin([..])`に
書いておけばいーんですね。なるほど。

結局要するにworkboxの`importScripts`を使うということで、具体的には、

1. `src/main/webapp/push-notification.js`にWeb Push通知時の処理`addEventListener('push', function(event) {...});`と通知をclickした時の処理`addEventListener('notificationclick', function(event) {...});`を書いておく
1. `webpack/webpack.prod.js`の`new WorkboxPlugin.GenerateSW({...})`に`importScripts: ['push-notifications.js']`を書く
1. `webpack/webpack.common.js`の`new CopyWebpackPlugin([..])`に`{ from: './src/main/webapp/push-notifications.js', to: 'push-notifications.js' },`を書く

で目出度くJHipster application上でのWeb Push通知が出来ました。

尚、通知の許可を求める処理やendpoint、auth、publicKeyを求める処理では、
Angularの`SwPush`を使うことが出来ます。
JavaScriptでゴリゴリ書かなくても`this.swPush.requestSubscription()`だけで済むのでラクです。
