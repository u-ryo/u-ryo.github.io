---
layout: post
title: "service worker in private browsing mode"
date: "2018-08-28 10:11"
author: 'u-ryo'
categories: [service worker, browser]
comments: true
published: true
---
Service WorkerでWeb Pushを色々試していて、
違うaccountでloginしてみようとprivate browsing mode用いると、
何か出来ないんですねこれ。
`https`か`localhost`じゃないとダメ、というのは知ってましたが。
気を付けましょう。
以上。

> Firefox ではプライベートブラウジングモードでサービスワーカー API を利用することはできません。
> [サービスワーカーの概念と使い方](https://developer.mozilla.org/ja/docs/Web/API/ServiceWorker_API#Service_worker_concepts_and_usage)

chromeはsecret mode(private window)でのService Workerも大丈夫、のようですか。
