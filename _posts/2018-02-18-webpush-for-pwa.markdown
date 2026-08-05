---
layout: post
title: "Web Push Notification for PWA"
date: "2018-02-18 22:32"
author: 'u-ryo'
categories: [pwa, javascript, spring, java, webpush, notification]
comments: true
published: true
---
PWAの売りとしてWeb Push Notificationが出来るのはつとに聞いていましたが、
具体的にどうやるんだろう? と思ってました。
仕組みから考えて、WebSocketが必須なのかな?
とかって勝手に思ってたんですが、違うんですね!
「やってみた」系の記事見て、Firebaseを使うものばっかりだったので、
えーっ! Firebase hosting必須なの?!
と思いきや、
VAPID(Voluntary Application Server Identification)も
あるということで一安心。
あーでも、別にFCM(Firebase Cloud Messaging)をhostingじゃなく使うことも出来るので、
その方が楽なのかな。
browser毎じゃなくてuser毎に送れるっていうし。
でもvendor lock onも嫌なので、
VAPIDからやってみました。

Web Pushの仕組みは、[PWAのプッシュ通知の仕組み](https://ajike.github.io/how-pwa-push-works/)がよくまとまってました。
PWAとWeb Push Notificationは別なんですね。

browser上からは、[Push Demo](https://gauntface.github.io/simple-push-demo/)などですぐ試せます。

自分の手に馴染ませないとよくわからないので、[作ってみました](https://github.com/u-ryo/my_web_push)。

1. push server側で、VAPIDのkey(public, private)を決めておく。key generationは、[web-push](https://github.com/web-push-libs/web-push/)の`client.js`をcommand lineから使って`$ node_modules/web-push/src/cli.js generate-vapid-keys`とせよ、とか[書いてあるもの](https://dzone.com/articles/web-push-notifications-1)も他に多くありましたが、Googleの[Adding Push Notifications to a Web App](https://developers.google.com/web/fundamentals/codelabs/push-notifications/)にあるように[Push Companion](https://web-push-codelab.glitch.me/)で簡単に作れますね。試しに作ってみたものではscriptで毎回自動生成してclient(=web browser, html and javascript側)に読ませてみるようにしてみましたが、実運用上はベタに固定値書いておいた方が楽だしわかりやすいし再起動の度に値変わらないからその方がいいのかな。
1. あとserver側は各clientのnotificationEndPoint、publicKey、authを知ってないといけない。これらはclientが`serviceWorkerRegistration.pushManager.subscribe(subscribeParams)`した時に得られる。`subscribeParameter`とは、server側のpublicKeyを`urlB64ToUint8Array()`したものと`userVisibleOnly: true`は固定値。そうして得られた3つの値をserver側に引き渡さねばならないのだが、そのまま渡すのではなく、`btoa(String.fromCharCode.apply(null, new Uint8Array(key)))`などとしてString化する必要がある。
1. 上記でもらう`notificationEndPoint`って、mozillaなら`https://updates.push.services.mozilla.com/wpush/v2/...`、chrome(chromium)なら`https://fcm.googleapis.com/fcm/...`というように、browser毎にmaker別のpush server持ってるんですね。ってことは、browserは裏ではこうしたsiteと常にconnection張ってるということ?
1. もっと深い原理(具体的にどういうheader/encryptionでpush serviceに送っているか)は[Web Pushの実装まとめ（Chrome/Firefox/Android共通）](http://adiary.blog.abk.nu/0391)参照。
1. client側では、`main.js`で、まず`Notification.requestPermission()`で許可を得る。許可を求めるtimingは、page load直後かと思いきや、最初にいきなり許可求めると拒否られることが多いということで、button式に。button押したら許可求める、と。
1. 許可が得られたら`navigator.serviceWorker.register('/sw.js')`でService Worker scriptを登録。`navigator.serviceWorker.ready`で利用可能になるのを待つ。これのreturn value(の型)は`ServiceWorkerRegistration`であり`navigator.serviceWorker`ではないので注意。これから`pushManager`を得ることになる。
1. その後、push serverからserverのpublicKeyをGET。
1. `subscribeParams`を作って`serviceWorkerRegistration.pushManager.getSubscription(subscribeParams)`してみる。既に登録があったらunsubscribeしないとならない。上述のように、parameterではserver側のpublicKeyを`urlB64ToUint8Array()`したものと`userVisibleOnly: true`は固定値。
1. それから`serviceWorkerRegistration.pushManager.subscribe(subscribeParams)`でsubscribeする。
1. そうすると、endpoint、key、authが得られるので、それらをpush server側に通知。
1. `sw.js`では、push通知が喜た場合`push` eventがfireされるので、`event.waitUntil(registration.showNotification(...)`する。この`registration`がどこから来るのかよくわからなかったが、ともあれ明示的に`waitUntil`して`showNotification`しないと表示されない(pushが来て勝手に表示されるわけではない)。
1. 表示されたNotificationをclickすると`notificationClick` eventがfireされるので、clickしたらどこかへ遷移したい場合にはこのevent listenerを`sw.js`に書いておく必要がある。
1. PWA用に、`sw.js`の`install` eventと`fetch` event listenerでfile chacheの作成とその利用をcodeする(`install`でcacheに加え、`fetch`ではcacheにあったらそれを、なければfetchするようにする)。
1. notificationをpushするには、push server側でまず`new PushService(publicKey, privateKey, "http://localhost")`してから、clientから貰った情報で`push.send(new Notification(...))`する。中では色々と暗号化しているが、[nl.martijndwarsのwebpush-java](https://github.com/web-push-libs/webpush-java)を使えばVAPIDのkey生成やnotificationのsendもone methodでよろしくやってくれる。他の言語も[web-push-libs](https://github.com/web-push-libs)に各種あり。

実際に自分で真似して書いてみて、
値やcodeを色々変えて試してみることで、
大分わかってきました。
