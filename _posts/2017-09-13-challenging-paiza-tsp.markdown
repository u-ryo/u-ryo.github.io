---
layout: post
title: "Challenging Paiza TSP"
date: "2017-09-13 23:46"
author: 'u-ryo'
categories: [paiza, travelling saleman problem, algorithm]
comments: true
published: true
---
8/13はPaizaの日、ということで、巡回セールスマン問題を出題していました。
[Paizaラーニングでも巡回セールスマン問題のさわり(貪欲法)を取り上げていて](https://paiza.jp/works/algorithm/primer/algorithm3)、
2-OptやSimulatedAnnealing(焼きなまし法)を[学んで](https://github.com/eugenp/tutorials/blob/3abb98e9e8acc7efe3f9f8423fcf0f8934655be7/algorithms/src/main/java/com/baeldung/algorithms/ga/annealing/SimulatedAnnealing.java)、
いざ書いてみたんですが、なかなかうまく行きません。
テストデータは[有名ドコロ](http://comopt.ifi.uni-heidelberg.de/software/TSPLIB95/tsp/)があります。
まずはatt48で2-Optでやってみたものの、
交差線が多く、全然それっぽい経路が得られません。
なんでーーー!?
[Javaによるまんまのコード](http://ist.ksc.kwansei.ac.jp/~tutimura/GraphApplication/)があったので、
読み解いていったんですけど時間かかっちゃって、
結局Paizaの締切9/12に間に合いませんでした...
何たること。

どうも、キモはswapにあるようで、
点を交換すると総延長が短くなる場合、
当然点を交換するんですが、
当該点だけを交換するだけじゃなく、
そこから真ん中へ向かってずっと点を交換していくんですね。
そうかー。

悔しいから、久し振りにPaizaのS問題やってみました。
最も簡単そうな最小辞書順列にしてみました。
けれど、問題の意味を理解するのと、JavaのStreamで書ききったために、
制限の2時間、あっという間に過ぎてしまいました。[0点確定](https://paiza.jp/challenges/share/0njRogbXZnuXG_Dt_noi_u1lW1Bl4R262Je2koUAjNA?source=social)です。トホホ。
しかも、提出したコードは、2つの場合だけ通らずに、80点。
どうしてコケる時があるのか、思い当たるフシもなく、未だにわかっていません。
テストデータ、見せて欲しいです。
