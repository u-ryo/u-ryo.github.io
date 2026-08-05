---
layout: post
title: "push notification on Angular5"
date: "2018-03-19 18:22"
author: 'u-ryo'
categories: [angular, push notification]
comments: true
published: true
---
もう先月ですが、Web Push Notificationを勉強して自分で試すことで、
理解を深めました。[作ってもみました](https://github.com/u-ryo/my_web_push)。

やりたいアプリは[Jhipster](https://www.jhipster.tech)で
[Angular](https://angular.io)使ってるんですが、
ではそういえば、AngularではWeb Pushってどうやるんだろう?!
ということからまた勉強が始まりました。
調べてみると、簡単に使えるようになっているんですね(Angular CLI 1.6以降は)。

* [3分クッキング ServiceWorkerで阿部寛さんを超える](https://qiita.com/studioTeaTwo/items/157e1c380ca63b372dc0) → いやこれ、単に阿部さんのページはcontents量が少ないから、というだけのjokeですよね
* [Angular CLI 1.5によるAngular Service Workerクイックスタート](https://medium.com/lacolaco-blog/angular-cli-1-5%E3%81%AB%E3%82%88%E3%82%8Bangular-service-worker%E3%82%AF%E3%82%A4%E3%83%83%E3%82%AF%E3%82%B9%E3%82%BF%E3%83%BC%E3%83%88-bd90db99fe13) → 「angular.jpで公式ドキュメントの翻訳が完了しているので、こちらをご覧ください」確かに公式見た方がいいですか。
* [(Angular公式) Service Workerを始める](https://angular.jp/guide/service-worker-getting-started)
* [AngularアプリをPWAにする方法](https://qiita.com/studioTeaTwo/items/648731b61962b7687f5a)
* [Creating PWA with Angular 5. Part 1: Getting started with framework, creating an application, hosting it on github-pages.](https://medium.com/@nsmirnova/creating-pwa-with-angular-5-e36ea2378b5d), [Creating PWA with Angular 5. Part 2: Progressifying the application](https://medium.com/@nsmirnova/creating-pwa-with-angular-5-part-2-progressifying-the-application-449e3a706129) → [PWCat](https://github.com/NastyaSmirnova/PWCat/)を[写経して比較検討しました](https://github.com/u-ryo/PWCat/)
* [Angular Service Worker - Step-By-Step Guide for turning your Application into a PWA](https://blog.angular-university.io/angular-service-worker/)

多くの導きのおかげで、angular5にてどのようにservice workerを導入するか、
はわかってきました。

要するに、アプリをgenerateする時に、Angular CLI(1.6+、現時点では1.7.3)で、

```
ng new application-name --service-worker
```

とするか、既にgenerateしてしまったアプリに対しては、
[(Angular公式) Service Workerを始める](https://angular.jp/guide/service-worker-getting-started)にあるように、

1. `yarn add @angular/service-worker`
1. `ng set apps.0.serviceWorker=true` または `.angular-cli.json`を編集して`apps`の下レベルに`"serviceWorker": true`を挿入
1. `src/app/app.module.ts`にService Workerを`import`して登録(但しその際、base hrefを`/`の他にしたいなら`ServiceWorkerModule.register('./ngsw-worker.js', {enabled: environment.production})`と`./ngsw-worker.js`を相対指定しないとダメ)
1. `src/ngsw-config.json`を作成(雛形をコピペ)
1. `ng build --prod`としてbuild(`ng serve`ではダメ)(base hrefを`/`の他にしたいなら`--base-href /another/directory/`が必要。←最後が`/`でないとダメ)

PWAとしては、あと`manifest.json`が必要?([Progressive Web Apps using the Angular Service Worker](https://docs.google.com/document/d/1F0e0ROaZUnTFftmC0XovpREHWHjcXa4CggiFlmifjhw/edit#), [AngularアプリをPWAにする方法](https://qiita.com/studioTeaTwo/items/648731b61962b7687f5a))

これで、当該projectをweb browserで表示させると、
service workerが読み込まれ、
offline cacheが効くようになる、筈...
なのですが、どう試しても、
offline modeにしてからreloadすると、`504 Gateway Timeout (from ServiceWorker)`になってしまいます。
[PWCat](https://nastyasmirnova.github.io/PWCat/)では上手くcacheされていることから、
多分これは[Angularのbug](https://github.com/angular/angular/issues/21636)と思います。

なのでoffline cacheは今は諦めて、push notificationの方策を探りました。
AngularのService Workerの話は、公式も含めて、
cacheやupdateばかりでpush notificationについては触れられてないんですね。
そういう中、[A new Angular Service Worker — creating automatic progressive web apps. Part 1: theory](https://medium.com/progressive-web-apps/a-new-angular-service-worker-creating-automatic-progressive-web-apps-part-1-theory-37d7d7647cc7),
[A new Angular Service Worker — creating automatic progressive web apps. Part 2: practice](https://medium.com/google-developer-experts/a-new-angular-service-worker-creating-automatic-progressive-web-apps-part-2-practice-3221471269a1)という記事があり、
[PWAtter](https://github.com/webmaxru/pwatter)とその[server](https://github.com/webmaxru/pwa-workshop-api)がありましたので、
非常に参考にさせてもらえました。
というかほぼそのままコピペして使わせてもらってます。
PWAtterはtwitterを拾うので物凄い勢いでpush notificationが来て、
明示的に消さないと消えないので、かなりうざいのですが、
これでAngularで[Push Notificationが実装できるようになりました](https://github.com/u-ryo/angular-swpush/)。
非Angularで作った時は、notificationの形についてはあまり気にしてなくて、
`clickTarget`とか[Notification object spec](https://developer.mozilla.org/en-US/docs/Web/API/Notification/Notification)を無視していましたが、
Angularでは`ngsw-worker.js`に`NOTIFICATION_OPTION_NAMES`があって、
そこで有効なproperty namesが規定されており、
`NOTIFICATION_OPTION_NAMES.filter(name => desc.hasOwnProperty(name))`として
filterしているので、変なproperty書いても効かないんですね。
ちゃんとNotification object specに則った形のJSONをsendするようにすると、
push notificationが表示されるようになりました。

あとは、Notificationをclickしたら消えたり指定のpageに飛ぶように、
と思ったんですが、どうやったらいいのか。
`notification.action`とか色々指定してみたものの、一向に何も起きず。
何でかなーって思ったら、どうやらnot yet implementedなんですね。
現に、`node_modules/@angular/service-worker/ngsw-worker.js`に、
`addEventListener('push', (event)...`
等はあっても、
`addEventListener('notificationclick', (event)...`
は無いんですね。(- -;
workaroundとしては、以前の知見を活かして、
似たようなことを書けばそれだけでpush notificationを実現できることを確認しました。
[push notificationの動作確認済のアプリ](https://github.com/u-ryo/angular-swpush)を作成しました。

既に公式Angularでもissuesに似たような話はありましたので、
comment付けておきました。

* `app.module.ts`中の`/ngsw-worker.js`は相対指定であるべき [ServiceWorker register generated by cli with base-href don't works #8515](https://github.com/angular/angular-cli/issues/8515#issuecomment-374103272)
* `notificationclick` event listenerのworkaroundの紹介 [Provide means to configure notificationclick in service-worker push #20956](https://github.com/angular/angular/issues/20956#issuecomment-374133852), [How to handle Notification click events (cannot find docs) #22311](https://github.com/angular/angular/issues/22311#issuecomment-374143351)

ホントはPull Request作りたかった、です。
上記前者の方は、どこでどうやって書き込んでいるのかわからず、codeに出来ませんでした。
後者は、`angular/packages/service-worker/worker/src/driver.ts`と`angular/packages/service-worker/worker/src/service-worker.d.ts`だというのはわかって、
書いてみたんですが、
いや難しいですね。
まず`event`って`notification`以下のJSONを含んでいるわけですが、
同時に`event.notification.close()`っていうmethodもあるわけで、
こういうのを厳密にobjectとして表現しなきゃいけないようになってるんですね。
凄いなぁ。そういうのが曖昧模糊渾然一体としているのがjavascriptなのに。
また、`clients.openWindow(...)`の`clients`もいきなり出て来るobjectで、
出自がよくわからないのですけど、そういう曖昧さを許さないように出来ています。
他を見てどうやら`this.scope.clients`とすればよさそうなのはわかったんですが、
それでも`./build.sh`をかけると
`driver.ts(246,28): error TS2339: Property 'openWindow' does not exist on type 'Clients'.`と言われて、お手上げでした。
`notification.close()`の方も、
`error TS2339: Property 'notification' does not exist on type 'Object | Client'`と言われて、
どうしていいかわからず。
生成されるべきcodeはわかってるのに、悔しい、です。
折角[Code of Conduct](https://github.com/angular/code-of-conduct/blob/master/CODE_OF_CONDUCT.md)や[CONTRIBUTING.md](https://github.com/angular/angular/blob/master/CONTRIBUTING.md)読んでcoding ruleやcommit message formatを学んだり[CLA(Contributor License Agreement)](http://code.google.com/legal/individual-cla-v1.0.html)登録したり[DEVELOPER.md](https://github.com/angular/angular/blob/master/docs/DEVELOPER.md)読んでbuildやtestの仕方学んだりしたのにー。
でもまぁ確かに、飛び先のURLはどこに書くべきか、とか、
clickしたら飛ばないでただ閉じるように/閉じないようにしたい、とかするには、
`eventListener('notificationclick',(event)=>{...});`を
直接いじらないとならないし、
仕様を含めたもっと別の方策が必要だと思います本当は。

favicon等iconを作るのに、[Real Favicon Generator](https://realfavicongenerator.net/)、いいですね。
