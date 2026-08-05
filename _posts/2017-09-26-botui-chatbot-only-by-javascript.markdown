---
layout: post
title: "BotUI - ChatBot only by JavaScript"
date: "2017-09-26 00:33"
author: 'u-ryo'
categories: [paiza, javascript, chatbot]
comments: true
published: true
---
[paiza開発日誌](http://paiza.hatenablog.com/entry/2017/09/21/﻿JavaScriptだけで本格的なチャットボットを開発できる)で紹介されていた[BotUI](https://github.com/botui/botui)、なるほど予め型にはまった会話ならこれだけでお手軽にJavaScriptだけで(`.then(function(){...})`で繋ぐだけで)出来ちゃうんですね。注文を取るとか、特定のAPI叩く(Wizardを会話でやる)とか、サポートセンターで特定の電話番号につなぐとか。AIは使ってないので、user側の曖昧な自然言語を受け取って処理する、というものではないですけど、そういうのに繋げればいい? いや、IBMのWatsonとかみると、そういう会話のplatformも含めて提供しているので、そうなるとBotUIの出番は無い筈。
[Git Repositoriesの総数を答えるsample](https://webhacck.github.io/botui-sample/)は、[公式のsamples](https://examples.botui.org)より面白かった(興味深かった)です。
