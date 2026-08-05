---
layout: post
title: "Security Check"
date: "2016-02-20 20:12"
author: 'u-ryo'
categories: [security, blacklist, botnet, spam]
comments: true
published: true
---
NHK Specialの[CYBER SHOCK 狙われる日本の機密情報](http://www.nhk.or.jp/docudocu/program/46/2586764/)見てたら、
標的型攻撃による情報流出が怖くなりました。
[JIPDEC](http://www.jipdec.or.jp)とか、表面化してないだけで絶対やられてるんじゃないのかな、
とか思うんですが、それをみるために、
gatewayで宛先IPとportをcheckしたらいいんじゃないのかな、
と思いました。
まず自分のマシンでやってみようと思いました。
隗より始めよ、ですね。
iptablesでOUTPUTのACKなtcpのdportとIPをLOGすれば、
と思ったんですけど、
それだったらsnortの方がいいですか。
でもsnortでoutboundな通信のlogってどうやるんでしょう。
また、それでIPあげつらったとしても、
それをどう検証するのか。
blacklistでもないかなぁ、と思ったら、
カペルスキーとかセキュリティ専門会社は独自blacklist持ってるんですね。

...と、この辺まで書いてたら、
Kodingに「週間使用量をexceedしている」とかいわれて止められちゃいました。
書こうとしてた時に勢いを止められるのはつらいです。
書いた文章もURLも消えちゃいましたし、何書こうとしてたか忘れちゃいました。

blacklist、IP addressで検索していたら、
whatismyipaddressの[Blacklist Check](http://whatismyipaddress.com/blacklist-check)、
でもこれは、接続元IPがspammerのblacklistに載っているか、
一括して調べるもの。どこかに載ってたら、
過去にspamを送っていたことがある、
何らかのmalwareに感染してた可能性がある、
ということです。

[check your very own IP for any botnet infections](https://www.reddit.com/r/Malware/comments/3u8719/check_your_very_own_ip_for_any_botnet_infections/)
というのがあって、これも同じようなserviceなんでしょうか。

C&C serverとのcommunicationは、
DNSのTXT recordでやるそうですね。
そんなのってdetect出来るんでしょうかね。出来そうですね。
何かtoolないのかな。

[脆弱性診断研究会]("https://www.facebook.com/sec.testing.study.session)には、
よさ気な情報が載ってます。
↑に載ってましたが、
[VAddy](http://vaddy.net/ja/)というJenkinsに組み込んでCIで脆弱性診断をしてくれる
serviceがあるそうです。free planだとSQL injection、XSSしか見てくれないそうですが、
freeで何度も使えるならいいですね。

## 脆弱性検査tools

- OWASP ZAP
- Vega(Java, included in Kali Linux)

Bug hunterとして生計を立てている[Kinugawa masatoさんのページ](http://masatokinugawa.l0.cm/)が
とても勉強になります。[徳丸先生の日記](http://blog.tokumaru.org/)は言わずもがな。
[ヒロミチュ先生のところ](https://takagi-hiromitsu.jp/diary/)は、
最近技術的な話題少ないんですかね。