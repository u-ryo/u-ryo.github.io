---
layout: post
title: "encryption by GnuPG"
date: "2015-11-04 02:38"
author: 'u-ryo'
categories: [GnuPG, encryption]
comments: true
published: true
---
GnuPGによる暗号化は、調べりゃいくらでも載ってますので、都度調べた方がいい感じです。

1. `gpg --gen-key`でキーを作成(これが結構面倒・時間かかる)
2. `gpg --export mail@address -o outputfile` で公開鍵を出力して他人に渡す
3. `gpg --import outputfile` で他人の公開鍵を入力
2. `gpg --list-public-key`で受領者IDを確認
3. `gpg -e -a -r ID... plain_file` で暗号化(-e:encrypt, -a:ascii, -r:receiver)
3. `gpg --passphrase ... --batch encrypted_file` で復号