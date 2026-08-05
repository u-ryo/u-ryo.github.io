---
layout: post
title: "ssh with empty password"
date: "2018-10-10 10:22"
author: 'u-ryo'
categories: [linux, ssh, pam]
comments: true
published: true
---
某ubuntu起動diskをmountして`/etc/passwd`のpasswordを潰して、
console login出来るようにはなったんですけれども、
sshでは入れません。
あーそっか、と思って、いい機会だからと後輩氏に課題として与えてみました。

新規user作って乗り越えてくれまして、それはそれで良い解だと思いますが、
`/etc/ssh/sshd_config`の`PermitEmptyPasswords yes`でも行けるよー
(このhostを繋ぐのは内部networkだし)、
と垂れて、試したところ、ものの見事に失敗。

あれー、なんでー?!

更に調べてみると、もう一つ施策が必要でした。

[公開鍵なし/パスワードなしでSSHログインする設定](https://qiita.com/FGtatsuro/items/4893dfb138f70d972904)にあるように`UsePAM no`にするか、
PAMを使っているということなのでならばPAMでempty passwordを許容するように
`/etc/pam.d/common-auth`の
`auth	[success=1 default=ignore]	pam_unix.so nullok_secure`の
`nullok_secure`を消す([Re: nullok_secure option](https://www.redhat.com/archives/pam-list/2005-September/msg00001.html))か、
ですがPAMいじるのは怖いですよねぇ。
