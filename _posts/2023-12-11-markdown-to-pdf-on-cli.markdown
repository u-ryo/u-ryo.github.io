---
layout: post
title: "markdown to pdf on CLI"
date: "2023-12-11 22:11"
author: 'u-ryo'
categories: [markdown, pdf, docker]
comments: true
published: true
---
markdownで快調に書いたものを、「印刷して提出せよ」ということになっていたので、「へ?」とか思って。今時ねぇ、印刷なんてねぇ、何の意味があるのか。一応evidenceってことなんだろうけど。

調べると、意外とないんですよね日本語markdown to pdf。popularなのはMS Visual Studio Codeの拡張機能Markdown PDF使うものですけど、たったそれだけのためにVisual Studio Code入れたかぁないですし。
nodeの`markdown-pdf`や`md-to-pdf`とかサクッと出来そうだったんですけど、古いのか、どうしようもないエラーが出ます。

```sh
docker run --rm -it -v $PWD:/work -w /work -e https_proxy=http://192.168.120.1:3128 node:alpine sh
# npm i -g markdown-pdf
# markdown-pdf some.md
node:events:497
      throw er; // Unhandled 'error' event
      ^

Error: spawn /usr/local/lib/node_modules/markdown-pdf/node_modules/phantomjs-prebuilt/lib/phantom/bin/phantomjs ENOENT
  :
# npm i -g md-to-pdf
# md-to-pdf some.md
(node:29) [DEP0040] DeprecationWarning: The `punycode` module is deprecated. Please use a userland alternative instead.
(Use `node --trace-deprecation ...` to show where the warning was created)

  Puppeteer old Headless deprecation warning:
    :

  ✖ generating PDF from some.md
    → TROUBLESHOOTING: https://pptr.dev/troubleshooting
ListrError: Something went wrong
    at /usr/local/lib/node_modules/md-to-pdf/node_modules/listr/index.js:102:18
     :
```

pandocもいいんですが、texの環境色々入れるのがかったるいなー。
というわけで、docker baseで。
探すと、参考文献がありました。proxyも使わないとならないので。

- [楽にDockerで日本語Pandocする](https://qiita.com/kojix2/items/1d2db46858ce202628d2)
- [[proxy対応]brewとtlmgrによるTeX環境構築 (ヒラギノ, times埋め込み設定)](https://qiita.com/yyamnk/items/2da2791bcee82643984f)

```sh
$ PROXY=http://192.168.120.1:3128
$ docker run --rm -it -e https_proxy=$PROXY -e http_proxy=$PROXY -e ftp_proxy=$PROXY -e use_proxy=yes -v $PWD:/data --entrypoint sh pandoc/latex
/data # tlmgr install collection-langjapanese
   :
tlmgr: package repository ftp://tug.org/historic/systems/texlive/2022/tlnet-final (not verified: valid signature with expired key)
   :
[108/108, 22:16/22:16] install: collection-langjapanese [1k]
running mktexlsr ...
   :
/data # pandoc some.md -o some.pdf -C --pdf-engine=lualatex -V linkcolor=blue -V documentclass=ltjsarticle -V luatexjapresetoptions=fonts-noto-cjk
```

- 何か今どきftpで取りに行くので、`ftp_proxy`が必要
- `use_proxy=yes`もないとproxy経由と認識してくれなかった
- working directoryはdefaultで`/data`なのでそこにmountする
- `entrypoint`が入っているので、optionで上書きしないと`tlmgr`を動かせない
- `pandoc/core`ではなく`pandoc/latex`であることに留意
    - 500MB超とsizeも結構ある
- `tlmgr install collection-langjapanese`は22分かかった
    - 総計30分弱はかかる
- 基本的には`pandoc md_file -o pdf_file`だが、`--pdf-engine`に`lualatex`を指定したり`documentclass`に`ltjsarticle`を指定したり、日本語特有の指定が必要
- `###`が本文と同じ大きさの文字になるので、思っているよりlevelを1段階上げとかないと、思ったような出力にならない
