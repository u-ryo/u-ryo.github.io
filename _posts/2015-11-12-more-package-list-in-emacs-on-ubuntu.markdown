---
layout: post
title: "more package-list in emacs on ubuntu"
date: "2015-11-12 13:11"
author: 'u-ryo'
categories: [emacs, markdown]
comments: true
published: true
---
emacsでmarkdownを書く時に、
markdown-modeが欲しいと思ったので、
`list-package`したら、
なかったんですね選択肢に。
調べると、`.emacs`に、

```
(require 'package)
(add-to-list 'package-archives '("melpa" . "http://melpa.milkbox.net/packages/"))
(add-to-list 'package-archives '("marmalade" . "http://marmalade-repo.org/packages/"))
(package-initialize)
```

が必要でした。

cf.[init-loader.el と package.el を導入して快適 Emacs ライフ](http://qiita.com/catatsuy/items/5f1cd86e2522fd3384a0)


### Preview
あと、emacsで書いてpreviewだけ別にしたいな、
と思ったぼくには、
Chromeの[Markdown Preview Plus](https://chrome.google.com/webstore/detail/markdown-preview-plus/febilkbfcbhebfnokafefeacimjdckgl)
extentionが便利でした。
install後、
Manage extentionsで「Allow access to file URLs」にチェックを入れないとならない、
というのがハマりポイントでした。

### PDFへの変換
[Pandoc で Markdown から PDF へ変換](https://gist.github.com/bouzuya/5989695)にある通り、
apt-getで入るpandocを使えば楽かなぁと。
header.texが必要というのがtrickyですね。
これもLuaLaTeXで変換出来ました。

header.tex

```
\usepackage{luatexja}
\setmainfont{TakaoPMincho}
\setsansfont{TakaoPGothic}
\setmonofont{TakaoGothic}
```

```
$ pandoc --latex-engine=lualatex -o /tmp/test.pdf -H ~/header.tex /tmp/test.md
```

### Table
[Table Generator](http://www.tablesgenerator.com/markdown_tables)というのがあります。
orgtbl-modeは、よくわからないので使ってません。


### online editor
ってうか、既にbrowser上でedit/preview出来るもの、あるじゃないですか。
もう古い記事ですが、
[Webブラウザで使えるMarkdownエディタの比較](http://yoshimov.com/list/markdown-online-editor-list/#wripe)を見ますと、
今はもっとあるんでしょうね...
2015年の記事もありますね。[ブラウザ上で使えるMarkdownエディタ](http://shgam.hatenadiary.jp/entry/2015/01/11/032651)

* [wri.pe](https://wri.pe/)
* [Online Markdown Editor - Dillinger, the Last Markdown Editor ever.](http://dillinger.io/)
* [Online Markdown Editor](http://www.ctrlshift.net/project/markdowneditor/)

wri.pe凄いですね。
GitHub accountで入れますし、
スッと使えるinterfaceを感じます。