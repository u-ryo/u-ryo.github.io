---
layout: post
title: "LuaLaTeX"
date: "2015-11-12 12:02"
author: 'u-ryo'
categories: [LuaLaTeX, LaTeX, png]
comments: true
published: true
---
表題の通り、LaTeXをよく使っています。
pLaTeXが長かったので、
LuaLaTeXについてのメモ書きです。
例によって、ちょっと調べればいくらでも出てますけど。

1. preambleは`\documentclass{ltjsarticle}`
2. usepackageは`luatexja` (`\usepackage{luatexja}`)
3. `\usepackage[dvipdfmx]{graphics}`
4. pngを貼りこむ時には、普通に`\includegraphics{XXX.png}`で良いが、
```
$ extractbb XXX.png
```
1. `zw`は`\zw`に

reportもMarkdownで書いてみようかな...