---
layout: post
title: "drawing rectangles on an image for groovy"
date: "2018-11-13 11:56"
author: 'u-ryo'
categories: [groovy]
comments: true
published: true
---
今まで、「画像を読み込んで線や図形を描く」というのはGroovyFXでやってたんですが、
JavaFXってheadlessで出来ないんですね!?
びっくりポンです。
`java.awt.headless=true`もそういやAWTなんですね。
うーむ、流石は廃止されるJavaFX、と思ったんですが、
どうせ`im4java`使ってるなら、なんだImageMagickだけで出来るじゃーん、
ということに遅ればせながら気付きました。
ImageMagickってdrawも出来るんですね。

大体、以下の要領です。

1. `import org.im4java.core.*`
1. `op = new IMOperation()`
1. `op.addImage(...)`で画像fileを読み込む
1. `op.fill('rgba(255,100,0,0.5)')`等と塗り潰す色を指定
1. `op.stroke('white')`等と線の色を指定
1. `op.draw('rectangle 0,10,30,30')`で長方形を描画
1. `op.draw('text 0,10 ABCD')`で文字を描画
1. `op.quality(80)`で圧縮率(品質)指定
1. もう一度`op.addImage(...)`で出力画像の名前と形式を指定
1. `new ConvertCmd().run(op)`で実行

[公式page](http://im4java.sourceforge.net/docs/dev-guide.html)にありますけどね。
