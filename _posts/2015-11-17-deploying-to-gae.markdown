---
layout: post
title: "Deploying to GAE"
date: "2015-11-17 13:48"
author: 'u-ryo'
categories: [GAE]
comments: true
published: true
---
いやぁ、こんなに大変だとは。
web applicationをGAEにdeployするのって。
既に何度も過去にはやっているんですが、
久し振りにやると、なかなか上手く行かないです。

最初、[Cloud9](https://c9.io)でやってみたら、
何度やってもresource(memory?)不足でappengine:updateが途中でkillされます。
仕方無いので、GAE連携がウリの[Codenvy](https://codenvy.com)でやってみたんですが、
original jarにdependsしているんですけど、
それが上手く行かないんです。
[official manual](http://docs.codenvy.com/user/technology-specific-features/#upload-local-libs)
見てrepository追加して、
directoryもgroupId、artifactId、versionと掘って配置して、
buildかけると読んではくれてるんですが、
何故か「jar; error in opening zip file」と言われてこけます。
改めてjar fileをuploadしたんですけど、
何度やってもダメ、同じerrorでした。
何故?

もう仕方無く、localでbuild、updateしようとすると、
今度は「Either the access code is invalid or the OAuth token is revoked.Details: invalid_grant」と言われて。
[heroku](https://heroku.com)はCloud9でサクッとdeploy出来るのに。
やっぱりGAEはもう時代遅れでしょうか。
Java8未対応ですし無料分では。

結局、`~/.appcfg_oauth2_tokens_java`を消したら上手くOAuth再取得してくれました。
やれやれ、です。