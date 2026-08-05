---
layout: post
title: "new Map on Typescript"
date: "2018-08-17 14:43"
author: 'u-ryo'
categories: [typescript, map]
comments: true
published: true
---
Angular5で、といいますかTypescriptで、
`Map`を一度にnewしたかったんですが、
どうするのかなーって。

`new Map()`して`.set()`でmethod chainで繋いで行くのもありますが、
一番簡単なのは、
`new Map([['key1','val1'],['key2','val2'],['key3','val3']])`という
ようなnested arrayでしょうか。
cf. [Map & Set](https://codecraft.tv/courses/angular/es6-typescript/mapset/)

ただ、こうすると`map['key1']`や`delete(map['key2'])`は効かず、
`map.get('key1')`や`map.delete('key2')`としないとならない、
というのにハマりました。
