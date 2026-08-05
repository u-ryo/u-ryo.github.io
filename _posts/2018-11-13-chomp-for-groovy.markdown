---
layout: post
title: "chomp for groovy"
date: "2018-11-13 11:53"
author: 'u-ryo'
categories: [groovy]
comments: true
published: true
---
groovyでcommand executionの結果を拾った(`'command args'.execute().text`)ところ改行がついてたので、除去しようと`.replace('\n', '')`としてたんですが、そっか、`.trim()`でいいんですね。勿論目的にもよりますが。

cf. [Groovyで文字列の先頭・末尾から空白を取り除く](http://orangeclover.hatenablog.com/entry/20110524/1306164109)
