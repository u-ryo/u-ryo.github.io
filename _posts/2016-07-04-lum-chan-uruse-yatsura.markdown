---
layout: post
title: "Lum-chan(Urusei Yatsura)"
date: "2016-07-04 01:39"
author: 'u-ryo'
categories: [anime, mp4, urusei-yatsura]
comments: true
published: true
---
古巣のJIPDECに派遣されて、
桂史郎くんから「またラム食べに行きましょう」
って言われて、
「ラムちゃん、行くんですね。
大丈夫だっちゃ、とか言うべきでしょうか」
って返したら、高嶋さんしか反応してくれなくて。
でも、それで調べてたら、
うる星やつら、
[全部見られる](http://freeanimedougadesu.blog70.fc2.com/blog-entry-2809.html)んですね。
ルパン三世もそうでしたけど。
何か、今頃になって見ちゃいました。
リアルタイムだと、中学から高校の頃。
その頃って確かにこういうの、見てなかったです全然。
改めて見てみましたが、子供向けのドタバタですねー。
確かにこれならいつまでも続けられそうですね。
でも、なんて言うか、何か見ちゃうですね。
どうせ今見るなら、昔見といた方が良かったのかな。
これ、ラブコメでもあるから、今のぼくにはちょっとキュンときちゃいますね。

というわけで、
command line 1行で見られるようになったので、
忘れないうちにメモ。

### yourupload
youruploadのURL:
`http://yourupload.com/watch/3OHj93`
の最後の5文字を使って、

```
export R=3OHj93;mpv --referrer=http://www.yourupload.com/jwplayer/jwplayer.flash.swf `wget -q -O - http://www.yourupload.com/embed/$R|grep og:video|sed 's/.*content="\(.*\)\/video.mp4.*/\1\/video.mp4/g'`
```

### mp4
mp4のURL:
`http://mp4upload.com/embed-bbzw7vbui2r4-650x370.html`
を利用します。

```
export U=http://mp4upload.com/embed-bbzw7vbui2r4-650x370.html;mpv --referrer=http://www.mp4upload.com/player/J6/jwplayer.flash.swf `wget -q -O - $U|grep '"file": "http:'|awk -F\" '{print $4}'`
```

どちらも、何故か一回で行かない時があります。
二度三度、retryすると、cacheにたまってうまく行くようになりました。
何ででしょう?