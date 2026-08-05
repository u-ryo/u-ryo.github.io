---
layout: post
title: "Arukas Cloud"
date: "2018-08-22 00:31"
author: 'u-ryo'
categories: [docker, sakura, arukas, alpine, nginx]
comments: true
published: true
---
[Sakura](https://www.sakura.ad.jp/)がやっているからそのreverse spellingだという[Arukas](https://arukas.io/)、
docker hostingをやっていて、
credit card登録は強制されますが1 containerなら無料というので、
専用SSH machine作って[JHipster](https://jhipster.tech/)の開発/buildに使おうと企みました。

* [arukas cloudとDockerでお手軽に開発環境をゲットする #Arukas](https://blog.stormcat.io/post/entry/arukas-development/)
* [Arukas.ioを使ってWEBページを公開する](https://qiita.com/ProjectEuropa/items/3909bd51454fcf4ef16d)

無料枠では[Docker Hub](https://hub.docker.com/)にあるものしか使えないとはいえ、
SSH serverの入ったdocker imageなんて色々あります。
でも使ってみると、基本的にはrootでlogin、
当然root loginをpermitしていて、
securityを維持するにはrootのpasswordを変える、
userを作って`.ssh/authorized_keys`を作る、
とかなわけですが、けどroot loginのpermissionを切って
sshdをrestartすると、docker終わっちゃうんですよね当たり前ですが。
何かもっとこう、最初からUser作ってrootは禁止して、
とかっていうの無いかなー、
と物色していると、どれだったか忘れましたがgithub.comから
自分が登録したkeyを`ADD`してるものがあって、
あーなるほどー、と思って。
`ADD https://github.com/$GITHUB_USER.keys /home/$GITHUB_USER/.ssh/authorized_keys`ということですね。

で、結局どうしても自分の欲しいものがなかったので、
`Dockerfile`自作することにしました。
元になるimageはDocker Hub上になければいけないので、
[Docker Hubを使ってGitHubにあるDockerfileからimageを自動生成する](https://simple-it-life.com/2016/03/27/dockerhub/)を参考に、
Github → Docker Hub → Arukasという流れになるように、
[自分のGithub repository](https://github.com/u-ryo/docker_alpine_sshd/)
→ [自分のDocker Hub repository](https://hub.docker.com/r/uryooo/docker_alpine_sshd/)
→ [自分のArukas App](https://app.arukas.io/apps/e45aaaa5-df9d-474d-ad42-d3eff1702d76)
と繋げて、
回るようにします。

Arukasでは変数定義が出来るというので`Dockerfile`でGithub user名を`ENV`定義して、
でもそうするとDocker Hubではbuildにコケるので`ENV`には何かdefault値がないとダメで、
でもdefault値を付けちゃうとそのdefault値がArukas Appまで残っちゃうんですよね。
[Docker コンテナの動作に必要な設定を起動時に渡す](https://blog.amedama.jp/entry/2018/01/30/230221)などを見て、
え、shell script内じゃないとダメ?
じゃぁっていうんで`entrypoint.sh`作って試してみたんですけど、
Arukasで起動に失敗しました。Docker Hubではbuild通ったのに。
Arukasでは起動時のmessageとかは出ないので、何が悪いのか分かりません。
困り果ててArukasのchatで相談投げると、翌営業日にはすぐreplyをくれて、
動作確認のためということでpull requestまで作ってくれました。
まずは早速そのまま取り込んで起動してみたんですが、
やはり起動に失敗。
けれども、別のaccountの方で`ENV`を編集することで再起動させてみたところ
うまく行ったので、何だったんでしょう?
結局今はどちらでも上手く立ち上がるようになりました。
ありがとうございました。

他、Arukasでハマったのは、開放するPort、
後で色々service立ち上げようと思って3000番とか8080番とか色々書いておいたら、
Endpointが有効にならないんですね。
これもArukasのchatで聞いて教えてもらったんですが、
確かによく読むと、「Docker image立ち上がった時点で
書いたPortが全て開いてないとEndpointが有効にならない」
と書いてありますね。
「一番上に書いたPortだけEndpointに繋がる」というのは
「アプリ編集」画面のEndpointのinfoに書いてあるからわかったんですが。

baseとなるdistributionは、
軽いという[Alpine Linux](https://www.alpinelinux.org)に。
アプリの管理も`apk add ...`コマンドで出来ますし、
packageも`screen`やら`openjdk8`やら`gradle`やら
結構色々あってびっくりです。
何かubuntuじゃなくてalpineで十分な気がしてきました。

目的としていたjhipster applicationのbuildは失敗しました。
specがしょぼい(128MB memory)のか通信量に制限があるのか、
`jhipster`コマンドの途中で止まり、
applicationのscaffoldingが出来ません。
まー、無料枠なのでmachine specについては文句言えませんから、
しょーがないのかなーと。

ただ、Endpointが`https`で手に入るので、
[Nginxによるリバースプロキシの設定方法](https://qiita.com/schwarz471/items/9b44adfbec006eab60b0)を参考に、
nginxによるreverse proxyとして使おうかなー、
と思っています。
custom domainも使えるとのことですが、
CDNと同じくDNSを自分でcontrol出来ねばならないので、
dynamic DNSでは使えず、custom domainは諦めました。

* [docker hubの成果物](https://hub.docker.com/r/uryooo/docker_alpine_sshd/builds/)(といってもpull requestくれた山田さんのおかげモノですが。山田さんありがとうございます!)

Githubにkeyを登録してあれば、Arukas AppでImageにこれ↑を指定し、
`ENV`で`GITHUB_USER`をGithubのuser名、
`PROXY_PASS`をreverse proxyしたいURL、
にして立ち上げると、
当該hostにssh出来、
またEndpointで指定したURLでreverse proxy出来ます。

```
ln -sf /dev/stdout /var/log/nginx/access.log
ln -sf /dev/stderr /var/log/nginx/error.log
```

こんな技があるんですねーなるほどー。
