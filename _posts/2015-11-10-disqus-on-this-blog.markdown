---
layout: post
title: "Disqus on this blog"
date: "2015-11-10 12:44"
author: 'u-ryo'
categories: [blog, Disqus]
comments: true
published: true
---
当初、このblogにはコメント欄がありませんでした。
[Disqus](http://disqus.com)を使える、
ということが他の人のblogを見てわかって、
うちでもSiteConfig.groovyをよく見たら、
`disqus`って項目、
あるんですね。`short_name`を入れるんですが、
Disqusに登録したのはちょっと前の事だったので、
`short_name`って何の事なのか最初よくわかりませんでした。
改めてDisqusに入って、Settings見て、漸く思い出しました。
自分で入れたんですね最初に。

しかし、`short_name`を設定しても、
"We were unable to load Disqus."と言われて出てきませんでした。
DisqusのSettingsでちゃんとURLやtrusted domain指定してるのに...
随分悩みました。
結局、SiteConfig.groovyで、
environments → prod → url を指定してなかったのが、原因でした。
Disqusは、「どのURLからのaccessか」をparameterから見てますから、
ここがSetup → Basic → Site Identity → Website URL と一致してないと
ならないんですね。