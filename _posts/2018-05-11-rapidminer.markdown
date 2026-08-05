---
layout: post
title: "RapidMiner"
date: "2018-05-11 10:11"
author: 'u-ryo'
categories: [java, ai, machine learning, signate]
comments: true
published: true
---
今、会社の業務の一部(先月から0.75)でAI、機械学習をやっています。
題材として[Signate](http://signate.jp/)の
[【練習問題】銀行の顧客ターゲティング](https://signate.jp/competitions/1)
をやっています。
業務時間中に色々試せるのはありがたいです。
PythonやR、kerasや[Google Colaboratory](https://colab.research.google.com/)や
Azure上のVM with GPUとか貰って試していたのですが、
結局[RapidMiner](http://rapidminer.com/)使ってみました。
[KSKのtutorial](https://www.rapidminer.jp/2016/07/07/【連載】rapidminerで始めるデータ分析～part5：予測モデル/)で勉強して、
色々と回せるようになりました。
確かに簡単ですねこれ。
でも、分かって来るとその限界や欠点も見えてきました。
また、課題に対する知見も溜まってきたので、忘れないうちに。

* 基本的な流れは、`Read CSV(train.csv)`→`Cross Validation`(`Gradient Boosted Trees`→`Apply Model`→`Performance(Binominal Classification)`)と`Read CSV(test.csv)`を合わせて→`Apply Model`→`Select Atributes`→`Write CSV`![RapidMiner flow](https://raw.githubusercontent.com/wiki/u-ryo/u-ryo.github.io/images/rapidminer1.png) ![inside cross validation](https://raw.githubusercontent.com/wiki/u-ryo/u-ryo.github.io/images/rapidminer2.png)
* CSVのreadでは、まず最初の列のidを`id`、最後のyを`label`に指定、その他のattributesについては、yes/no 2値のものを`binominal`、dayを`integer`ではなく`polynominal`にするのがpointか。
* dayとmonthから日付にしては? と思っていたのだが、pdateを見ると同じ月日でも何百日のものもあって複数yearsありそう、且つyearは特定できなさそう(365日毎でもない)
* ならいっそday/monthをfeatureから落とすと精度が落ちる
* せめてmonthだけでも`date`として認識できないかと試したがdate format指定を`mmm`では出来なかった(小文字だから?)
* ウリのAutoModelを試したところ、data file指定、fieldsのcorrelationから落とすattributesを指定、複数種類のmodelを実行、その後modelのparameterをいじれてsimulationが出来、modelに対する理解が進むというもの。AutoModelではGBoostよりDeepLearningが僅差で良かったがAUC 0.89程度
* 自分でcross validationするようにして色々なmodelerを試したところ、Deep Learningも良かったがGradient Boosted TreesがAUC 0.90と最強だった
* なのでGBoostでparameter tuningを。number of treesはdefaultの20からぐっと増やした方が。max depthはちょっとでも大きすぎるとダメ。learning rateもちょっとでも増やすと精度は落ちる
* あんまり細かくtuningしても、結局randomizeの影響が大きい。小数点以下3桁程度のぶれは出てしまう。即ち、全く同じparametersで試しても結果が違う。reproducibleをcheckしても、cross validationの仕方が違うので、全てのrandom性を注意深く固定しないとならないし、randomのseedを探索する事になってしまうのもアホらしい
* 仮にmodelingが出来たとして、これをserviceとして抜き出すのは難しそう。RapidMiner Serverを買えということか。そういうところで彼らは商売をしているのだろう。
* GPUが無くても速い(30秒程度)のは、MultiThreadを使っているからか。`top`を見ていると気持ちいいくらいmulti coresを使っている。流石はJavaである。
* よく言われる`XGBoost`というmodelerは無く、`Gradient Boosted Trees`がそれ相当
* one-hot vector化しようと思って`Nominal to Numerical`を前処理に入れたら精度が落ちた。その出力をよく見たら、これ単に非numericalなattributesを落としてるだけ。何これ。じゃぁっていうんで`Nominal to Binomial`にしてみても全く同じ。えー、RapidMinerではone-hot vector化、簡単に出来ないの!? 他の言語ではチョー簡単なのに。
* RapidMiner、凄く簡単ですけど、これはinput dataがone fileで数万recordsしか無いから、でしょう。これが画像群だったらこうはいかないでしょう。

### 現時点で最良の結果(5/11/2018現在68位)

https://signate.jp/competitions/1/leaderboard#scoreboard

```
number of trees: 205
maximal depth: 5
min rows: 10.0
min split improvement: 0.0
number of bins: 20
learning rate: 0.1
sample rate: 0.8

PerformanceVector:
accuracy: 90.89% +/- 0.42% (mikro: 90.89%)
ConfusionMatrix:
True:	1	0
1:	1638	935
0:	1536	23019
AUC: 0.933 +/- 0.006 (mikro: 0.933) (positive class: 0)
```

これ以上AUCを上げるには、
RapidMinerじゃない何か、RかPythonを使って、
入力dataもone-hot vector化して、とかしないとならないような気がします。
