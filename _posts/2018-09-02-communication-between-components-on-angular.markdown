---
layout: post
title: "Communication between components on Angular"
date: "2018-09-02 18:49"
author: 'u-ryo'
categories: [angular]
comments: true
published: true
---
[Angular](https://angular.io/)でcomponent間通信しようと思いました。
これまで一つにしていたcontrol formとそれをはんえいするimage viewを
別componentにして、その間で情報伝達したり、
あるtimingで他方のfunctionを叩いたり、
とかしたくなるわけですね。
[コンポーネント間のデータ授受メモ](https://qiita.com/gambare/items/b75f9c9dc997ae45c092)とか[Angular2でComponentをまたがったデータのやり取り](https://christina04.hatenablog.com/entry/2017/01/19/041235)とか[コンポーネント間でデータを共有する](https://st40.xyz/one-run/article/200/)とかちょっと調べればもう色々書いてあって、
親子だとどうとか兄弟間だとこうとかあるわけです。
今回は兄弟間だったので間を取り持つService作って兄から書いて弟で読むとかその逆とかやってたんですが、
[ngx-store](https://www.npmjs.com/package/ngx-store)の`@SharedStorage`使えば一発ですよそんなの。
最初から知ってればもっともっと簡単に済んだのに。
[JHipster](https://www.jhipster.tech/)では
[ngx-webstorage](https://www.npmjs.com/package/ngx-webstorage)が
予め入っているのでこれを使えればいいんでしょうけど、
`this.$sessionStorage.store('key', 'value')`して
`this.$sessionStorage.retrieve('key')`取り出すというように、
`ngx-store`と比べてちょっと面倒です。
っていうか、JHipsterでも`ngx-store`使ってくれればよかったのに。
Local Storageにも保存できますし、ってそれはどちらも同じですが、
`ngx-store`の方は特にstore/retrieveとかしなくても、
意識しなくても変数に`@SharedStorage`とか`@LocalStorage`とすれば
他のcomponentでも変更された値がそのまま参照できるというスグレモノです。
全部これ使えばいーんですよ。
