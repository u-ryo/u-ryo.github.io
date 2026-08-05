---
layout: post
title: "Knapsack Problem"
date: "2017-10-20 23:48"
author: 'u-ryo'
categories: [knapsack problem]
comments: true
published: true
---
ここ最近、Knapsack Problemが流行っている感じがしています。

[応用情報技術者過去問題 平成29年春期 午後問3](http://www.ap-siken.com/kakomon/29_haru/pm03.html)では基本「全探索」、改善提案が「枝刈り」でしたが、それでは全然不十分です。この問題は要するにKnapsack Problemのちょっとした変形で、code量もさしたることなく書けますし、折角FEではなくAPなのだし、日本の若者のためにも、Knapsackで書くよう誘導すべきだったのでは、と「禿げしく」思ったものでした。

すると、今度は[応用情報技術者過去問題 平成29年秋期 午後問3](http://www.ap-siken.com/kakomon/29_aki/pm03.html)でKnapsack Problemを正面から出してきましたね。0-1ではなく普通の。ぼくは前回のrevengeではないかと思ってみています。
問題自体は、あまりにも普通のKnapsack Problemなので、ツッコミようがなくてつまんなかったデス。

この他にアジアでも。そっちは0-1でした。

### References

* Wikipedia[En](https://en.wikipedia.org/wiki/Knapsack_problem), [Ja](https://ja.wikipedia.org/wiki/%E3%83%8A%E3%83%83%E3%83%97%E3%82%B5%E3%83%83%E3%82%AF%E5%95%8F%E9%A1%8C)
* [Aizu Online Judge](http://judge.u-aizu.ac.jp/onlinejudge/commentary.jsp?id=DPL_1_B)
* [「最強最速アルゴリズマー養成講座」最新記事一覧](http://www.itmedia.co.jp/keywords/algorithmer.html), [病みつきになる「動的計画法」、その深淵に迫る](http://www.itmedia.co.jp/enterprise/articles/1005/15/news002.html), [アルゴリズマーの登竜門、「動的計画法・メモ化再帰」はこんなに簡単だった](http://www.itmedia.co.jp/enterprise/articles/1003/06/news002.html)
* [片鱗懐古のブログ 01ナップサック問題を動的計画法で解く場合の考え方](http://pieceofnostalgy.blogspot.jp/2013/12/01.html)
* [【Java】ナップサック問題(knapsack)[動的計画法]  ](http://fantom1x.blog130.fc2.com/blog-entry-174.html)

ちなみに日本語では「ナップザック」、登山用語はドイツから入ってきたから、です。
