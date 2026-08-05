---
layout: post
title: "number to string on Angular/TypeScript"
date: "2018-11-13 12:42"
author: 'u-ryo'
categories: [angular, typescript]
comments: true
published: true
---
Angularを書いていて、
`Type 'string' is not assignable to type 'number'.`と言われて驚いたので調べました。
えー、そのくらいよしなに変換してくれるんじゃないのー?!
結構型に厳しいんですね。
[Typescript, toFixed. Type 'string' is not assignable to type 'number'](https://stackoverflow.com/questions/39956988/typescript-tofixed-type-string-is-not-assignable-to-type-number)に`(12.32).toFixed(2)`とあるので試すとその通りでした。
この2ってなんだろう? と思ったら、「小数点以下第2位までの表示(第3位で四捨五入、無ければ0埋め)」ってことなんですね。
