---
layout: post
title: "Recent package managements"
date: "2016-09-20 14:33"
author: 'u-ryo'
categories: [emacs, java, groovy, node.js, nvm]
comments: true
published: true
---
これも忘れないうちに。

最近のpackage管理は、便利になってるんですけど、用途によって色々と分かれてしまっているので、却って使いづらいというか。`apt-get install`でいいじゃん... ってのは、ダメなんですかねぇ。

* [SDKMAN!](http://sdkman.io/)(GVMの後継。Java系(Groovy,Ant,Kotlin,Maven,Scala,grails,JBoss Forge,...)のinstall、version管理に。installationは`curl -s api.sdkman.io|bash`)
* [n – Interactively Manage Your Node.js Versions](https://github.com/tj/n)(Nodeのinstall、version管理に。`sudo npm install -g n`でinstall、`sudo n latest`で最新のnode.jsが入る)
* [cask](https://github.com/cask/cask)([2015年Emacsパッケージ事情](http://qiita.com/tadsan/items/6c658cc471be61cbc8f6))
* [El-Get](https://github.com/dimitri/el-get)([Caskはもう古い、これからはEl-Get - いまどきのEmacsパッケージ管理](http://tarao.hatenablog.com/entry/20150221/1424518030)