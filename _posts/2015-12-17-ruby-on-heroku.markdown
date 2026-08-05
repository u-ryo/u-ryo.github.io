---
layout: post
title: "ruby on heroku"
date: "2015-12-17 17:44"
author: 'u-ryo'
categories: [ruby, heroku, sinatra, ImageMagick]
comments: true
published: true
---
11月から駒沢大学のCTCに派遣されて、
某A空輸の次期予約サイトのMonkey Testをやらされているんですが、
screen shotとかuploadするのに、何故か20kbという制限があります。
そんなの、Gimpとかフリーソフト使えば簡単なんですが、
入れさせてくれないんですよね。
なので、screen shotを撮るにはALT + PrintScreen、
編集はMicro$oft謹製のペイント、
pngでsave出来るようになっただけマシではありますが、
64bit colorでしか保存できないので、
sizeが無駄に大きくなっちゃって、
20kbなんてすぐ超えちゃうんですよね。
ImageMagick使えばそんなの、
`convert infile.png -colors 256 outfile.png`
で済むのに。

仕方無いので、縮小したりしていたんですが、
縮小すると何が書いてあるかわからないし。
そうだ、Web Applicationならinstallしなくてもいい、
と気付いたんですけど、
巷間のserviceに外部秘の画像を上げるのは気が引けたので、
なら自分で作ろうと。

オンプレミスなserverも持ってますが、
イマドキじゃないのと、
よく見たらオンプレミスserverにはImageMagickも入ってないですし。
まぁそれは`apt-get install`一発で行けるからいいんですけど、
shell scriptでconvertすればいいから、
cgiでちゃちゃっと書いちゃおう、と思ったんですが。
POSTされたfile dataをparseするのが面倒かな、
と思ったのと、あと443が開いてないんですね。
なので、herokuに上げよう、と。
そうなるとrubyかな、と思ったので、調べてやってみました。

herokuの公式ruby用sampleを元に作れそう、だったので、
調べて、sinatra使ってるってわかって、
あとImageMagickはherokuに元々入ってて、
rubyはrmagicを使えるらしいということで、やりました。

cf.

1. [Heroku導入メモ](https://gist.github.com/konitter/5370904)
1. [ruby sample](git://github.com/heroku/ruby-sample.git)
2. あと`$ build`?

```
$ git commit -a
$ git push heroku master
```

rmagicを使うには、以下の作業が必要でした。

1. Gemfileに`gem 'rmagick', '~>2.15.4', :require => 'RMagick'`
2. c9上で`sudo apt-get install imagemagick libmagick++-dev`

POSTされたfile dataの取得は、

```
image_filename = "./#{params[:file][:filename]}"
imagedata = params[:file][:tempfile].read
```

image dataの取得は、

```
image = Magick::Image.from_blob(imagedata).first
```

fileとしてのreturnは、

```
content_type = 'image/png'
attachment image_filename
```

image dataのbinary data取り出しは、

```
image.to_blob
```

256色への減色は、

```
mage.format = "PNG8"
```

とするだけで8bit colorで保存されます。

最終的には、

```
require 'sinatra'
require "rmagick"

get '/' do
  erb :index
end

post '/' do
	if params[:file]
	   image_filename = "./#{params[:file][:filename]}"
	   imagedata = params[:file][:tempfile].read
	   image = Magick::Image.from_blob(imagedata).first
	   image.format = "PNG8"
        content_type = 'image/png'
        attachment image_filename
        image.to_blob
	end
end
```

あぁ、erbってのも初めて使いました。
ここへpostするためのplain htmlが書いてあります。
