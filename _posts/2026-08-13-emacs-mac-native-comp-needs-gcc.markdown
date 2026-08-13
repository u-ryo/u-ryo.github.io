---
layout: post
title: "emacs-mac の native compilation には libgccjit だけでなく gcc が要る"
date: "2026-08-13 14:06"
author: 'u-ryo'
categories: [emacs, macos, homebrew]
comments: true
published: true
---

emacs-mac@29 (29.4-mac-10.1、Apple Silicon) で native compilation が全滅した話です。パッケージを読み込むたびに *Warnings* にこれが積み上がる:

```
libgccjit.so: error invoking gcc driver
Internal native compiler error: failed to compile
```

libgccjit はちゃんと brew で入っている。なのに全部の .el がコンパイルに失敗する。同じ症状の方は、結論だけ知りたければ最後のコマンド1行だけどうぞ。

<!-- more -->

## 本当のエラーを見る

*Warnings* バッファのメッセージは要約されすぎていて役に立ちません。verbose な本当のエラーは、batch モードで1ファイルだけコンパイルすると見えます:

```sh
emacs --batch --eval '(native-compile "foo.el" "foo.eln")'
```

これで正体が出ました:

```
ld: library 'emutls_w' not found
```

## 原因: libemutls_w.a は gcc formula 側にいる

`libemutls_w.a` を探すと、`libgccjit` formula (16.1.0) には同梱されておらず、`gcc` formula の中にいます:

```
/opt/homebrew/Cellar/gcc/16.1.0/lib/gcc/current/gcc/aarch64-apple-darwin25/16/libemutls_w.a
```

つまり libgccjit がリンク段階で gcc 付属のライブラリを必要とするのに、macOS の Homebrew では gcc が libgccjit の依存として宣言されていない。libgccjit だけ入れた環境では、コンパイルの最終段でリンカが死ぬわけです。

## 対処: brew install gcc、以上

```sh
brew install gcc
```

これだけで全部直りました。init.el の変更 (`native-comp-driver-options` や `LIBRARY_PATH` をいじる系の対処がネットには出てきますが) は不要でした。

注意点をひとつ: `gcc` と `libgccjit` は**ペアで必要**です。「gcc なんて使ってないから」と brew cleanup で消すと再発します。

エラーメッセージから gcc に辿り着くまでが長かったので、誰かの検索に引っかかることを祈って置いておきます。

(下書きはうちのAIが書き、文責は私にあります)
