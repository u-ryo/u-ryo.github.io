---
layout: post
title: "lualatex error"
date: "2019-01-23 21:59"
author: 'u-ryo'
categories: [tex, latex, lualatex, ubuntu]
comments: true
published: true
---
Ubuntu 18.04になってからか、これまで難なく通っていたLuaLaTeXが
通らなくなっていました。

```
(/usr/share/texlive/texmf-dist/tex/luatex/luatexja/patches/lltjp-stfloats.sty)
ABD: EverySelectfont initializing macros
! Undefined control sequence.
<argument> ...x \ifdim \paperheight >0pt\relax \pdfpagewidth
                                                  =\paperwidth \pdfpageheigh...

l.5 \begin{document}
?
```

version upしたからかな、
そのうち通るようになるかな、
それまでは16.04捨てられないな、
とか思ってほっぽっておいてたんですが、
一向に良くならないので、ちょっと調べてみました。
LuaLaTeX追ってる人には何でもない情報でしょうが、
色々変わってたんですね。

結論から示しますと、
`graphicx`のdriverを`pdftex`から`luatex`にする
(→defaultの自動判定に任せる)ようにすれば良かったです。
具体的には、

```
\usepackage[pdftex]{graphicx}
   ↓
\usepackage{graphicx}
```

だけでした(ref.[TeX Live 2016 の新しい LuaTeX あれこれ](http://acetaminophen.hatenablog.com/entry/2016/04/23/141922))。
primitive名が色々変わって、
LuaTeXのprimitiveに対応した`luatex.def`が出来て、
graphicx driverの`luatex`への自動判定(`pdftex`ではなく)が
導入されていました。
