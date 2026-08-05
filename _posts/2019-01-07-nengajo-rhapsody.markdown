---
layout: post
title: "Nengajo Rhapsody"
date: "2019-01-07 01:45"
author: 'u-ryo'
categories: [tex, latex, lualatex]
comments: true
published: true
---
いつも年賀状で苦労しています。
毎年微妙に変わりますので。
今年苦労した点をメモしておきます。年末の自分のために。

`LuaLaTeX`になってから、fontの扱いは大分楽になりました。
[LuaLaTeXのフォントの取り扱いについて](https://arxiv.hatenablog.com/entry/2016/11/30/183000)を参考に、
`*.ttf/otf`を`/usr/share/fonts/[true|open]type/xxx/`以下にcopyして、
`.tex`fileに
`\newfontfamily\Text任意名{英文フォントファイル名}`や
`\newjfontfamily\Text任意名{和文フォントファイル名}`と書けば、
`\Text任意名 happy new year!`や`\Text任意名 謹賀新年`というように使えました。

class fileは香田温人さんの`ltjhagaki.cls`を長年使わせて貰っています。
探してみたら今はもう見付からないようですね。

0.81や0.85でLuaLaTeXが少し変わって([LuaTeX や LaTeX や LuaLaTeX が新しくなってアレ(1)](https://zrbabbler.hatenablog.com/entry/20151013/1444700367))、
`\pdfpagewidth`も未定義になっちゃってて、
はがき自体の表示位置もA4のupper centerになっちゃってて。
去年はハガキサイズでPDF出てたのに。
これだと印刷にどうにも苦労したので、
[LuaLaTeX での余白の設定](https://org-technology.com/posts/lualatex-geometry.html#)を参考に
geometryを導入してはがきサイズでpdfを出力するようにしたら
ようやっとうまく行きました。

```
\usepackage[paperwidth=100mm, paperheight=149mm]{geometry}
\geometry{top=0truemm,bottom=0truemm,inner=0truemm,outer=0truemm}
```

これですね。良かったです。

[Easy Drawing Tutorials](http://www.easydrawingtutorials.com/index.php/disney/414-mickey-mouse-body)、[顔の描き方](http://www.easydrawingtutorials.com/index.php/88-disney/81-draw-mickey-mouse)なんてあるんですね凄いなぁ。
