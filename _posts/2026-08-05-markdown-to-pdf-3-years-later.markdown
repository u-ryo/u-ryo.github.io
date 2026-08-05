---
layout: post
title: "Markdown to PDF、3年後"
date: "2026-08-05 13:00"
author: 'u-ryo'
categories: [markdown, pdf, pandoc, typst]
comments: true
published: true
---

前回 (といってももうはるか3年も前ですが)、[markdown to pdf on CLI](/blog/2023/12/11/markdown-to-pdf-on-cli/) という記事で、pandoc/latex の docker image に `tlmgr install collection-langjapanese` で30分ほどかけて日本語PDF環境を作った話を書きました。

実はあの記事を書いてから割とすぐ、[楽にMarkdownをそれっぽいPDFに変換する](https://qiita.com/frozenbonito/items/10a38c5fd4ba97a9bef0) の `frozenbonito/pandoc-eisvogel-ja` を見つけました。pandoc + Eisvogel テンプレート + 日本語フォント焼き込み済みの docker image なので、

```sh
docker run --rm -v $(pwd):/data frozenbonito/pandoc-eisvogel-ja -o doc.pdf doc.md
```

これ一発。表紙も目次も出ます。爾来、今に至るもずっとこちらを愛用しています。以上。

...で終わってしまうのもなんですんで、そういえばきょうび、この界隈はどうなってるんだろう? と思って軽く見渡してみました。

## 2026年の様子

一番の変化は **Typst** の台頭のようです。LaTeX 代替の組版システムで、pandoc が `--pdf-engine=typst` を直接サポートしたので、

```sh
brew install pandoc typst
pandoc doc.md -o doc.pdf --pdf-engine=typst
```

で TeX 環境なしの日本語PDFが出る時代になっていました。tlmgr に22分かけた身からすると隔世の感があります。日本語の組版にこだわるなら CSS 組版の [Vivliostyle](https://vivliostyle.org/ja/) が健在で、技術同人誌方面は大体これみたいですね。

あと身も蓋もない変化として、AI に「これPDFにして」と言うと agent が勝手に pandoc なり headless Chrome なりを使って出してくる、というのがあります。ツール選定という行為自体が消えつつある、のが AI 時代の今、ということでしょうか。

## 3年前に動かなかった md-to-pdf、今なら動くのか

前回の記事で「どうしようもないエラーが出る」と切り捨てた `md-to-pdf`、2026年の今ならどうなのか。試してみました。

```sh
$ npx --yes md-to-pdf sample.md
Error: Could not find Chrome (ver. 151.0.7922.71). This can occur if either
 1. you did not perform an installation before running the script
    (e.g. `npx puppeteer browsers install chrome`) or
 2. your cache path is incorrectly configured
```

一発では動かない。そこは3年前と同じなんですが、理由が進化していて、puppeteer が Chrome を自動ダウンロードしなくなったため、なんですね。エラーメッセージの指示どおりに

```sh
$ npx puppeteer browsers install chrome
$ npx md-to-pdf sample.md
[12:51:55] generating PDF from sample.md [started]
[12:51:59] generating PDF from sample.md [completed]
```

今度は4秒で完了。出てきたPDFは GitHub 風スタイルで、日本語の本文・表・コードブロックとも問題なしでした。「一発では動かない」は3年経っても変わらないですが、エラーメッセージが親切になったので自己解決できる。時代は少しずつ良くなっています。

## 古い Linux でも動くのか

Typst は Rust 製で musl の static build が配られていて、pandoc の公式バイナリも static link なので、原理的には glibc の古い年代物の Linux でも「置くだけ」で動くはずです。docker が使えない・使いたくない環境でこそ試す価値があります。手元の古めの環境で試してみました。

<figure class='code'><div class="highlight"><pre><span></span>mkdir<span class="w"> </span>-p<span class="w"> </span>~/md2pdf/bin<span class="w"> </span><span class="o">&amp;&amp;</span><span class="w"> </span><span class="nb">cd</span><span class="w"> </span>~/md2pdf
curl<span class="w"> </span>-sLO<span class="w"> </span>https://github.com/typst/typst/releases/latest/download/typst-x86_64-unknown-linux-musl.tar.xz
tar<span class="w"> </span>xf<span class="w"> </span>typst-x86_64-unknown-linux-musl.tar.xz<span class="w"> </span><span class="o">&amp;&amp;</span><span class="w"> </span>cp<span class="w"> </span>typst-*/typst<span class="w"> </span>bin/
<span class="c1"># pandoc は GitHub releases の linux-amd64 tarball (static link) を同様に bin/ へ</span>
<span class="nb">export</span><span class="w"> </span><span class="nv">PATH</span><span class="o">=</span>~/md2pdf/bin:<span class="nv">$PATH</span>
<span class="nb">time</span><span class="w"> </span>pandoc<span class="w"> </span>sample.md<span class="w"> </span>-o<span class="w"> </span>out.pdf<span class="w"> </span>--pdf-engine<span class="o">=</span>typst<span class="w"> </span>-V<span class="w"> </span><span class="nv">mainfont</span><span class="o">=</span>IPAMincho
</pre></div></figure>

```
real    0m5.210s
```

glibc 2.23 という年代物の環境で動きました。初回はフォントキャッシュ構築込みで5秒、以後は1秒級。日本語フォントはシステムに入っているものを typst がそのまま拾ってくれるので (`typst fonts` で一覧できます)、うちは IPAMincho を指定しただけ。無いマシンでも otf を1個置いて `TYPST_FONT_PATHS` で指せば済みます。disk 占有は typst + pandoc 合わせて210MB (ほぼ pandoc の static binary です。配布は tar.gz で30MBほどなのに、展開すると5倍に膨らむのは Haskell の static binary らしいところ)。eisvogel-ja の docker image が1.29GBなので、6分の1で済みます。出来上がりの PDF も、表紙情報・見出し・表・コードブロックとも文句なしでした。

## 結論

今日から `pandoc` + `typst` で生きていこうと思います。

凄いですね、ベルリン工科大 Martin Haug さんと Laurenz Mädje さん。「LaTeX のコンパイル遅すぎ・エラー不親切すぎ」に切れて作った、と?! あの Knuth 先生に挑むというその心意気に拍手喝采です。

そういえば、知人が何を使って md to pdf しているのか、気になっています。今度聞いてみます。
