---
layout: post
title: "Downloading a file on Angular"
date: "2018-08-21 22:04"
author: 'u-ryo'
categories: [angular, java, typescript]
comments: true
published: true
---
JHipsterのAngularでfileをdownloadするUIを作っていました。
AタグlinkからserverのAPI叩いて、
`Content-Disposition: attachment;filename=...`
と返せば済むだろう、
と思ってたんですが、そういえば認証通さねばなりません。
となると一旦browserで全部受けてblobにしてから返さないとならなさそうです。
認証header自体は、JHipsterならHTTPのinterceptorがあって、
フツーに`this.http.get(...)`とかすれば勝手に付けてくれます。
HTTP Headerの付け方は[公式document](https://angular.io/guide/http#adding-headers)にある通りです。
[Angular 2 download .CSV file click event with authentication](https://stackoverflow.com/questions/40966096/angular-2-download-csv-file-click-event-with-authentication?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa)を参考に、
service化しました。
[FileSaver.js](https://github.com/eligrey/FileSaver.js)を使うと楽そうですけど、
そんなに互換性が重視されるわけではないことと大したcode量ではないことから、
自分で書きました。
最後、`window.open(url);`だとwindowが上がってきてしまうので、
[AngularでCSVをAPIからDLするときに色々したお話](http://takasdev.hatenablog.com/entry/2017/06/15/211725)にあるように、
裏で自分で`<a href="...">`作って自分で叩いて自分で消す、
というように書いたら、うまく行きました。
