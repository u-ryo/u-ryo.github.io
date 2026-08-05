---
layout: post
title: "Created my blog by Grain"
date: "2015-10-26 17:10"
author: 'u-ryo'
categories: [grain, gradle]
comments: true
published: true
---
## Blog、はじめました

非常に遅まきながら、blogを作ってみました。

きしださんの[2014-02-25(火) コミュニティに入るか入らないかでエンジニアとしての幸福度がかわる](http://d.hatena.ne.jp/nowokay/20140225)を見まして、
なるほどそうだなぁと思い、
そのためには発信していかなきゃ、
ということで、
始めました。

blog環境は世に数多あれど、
editorで書きたい、
dataを手元に置いておきたい、
ということから、
Qiitaやはてぶ等ではなく、
Github Pagesを用いました。
Github Pagesというのがあってblogに使える、
というのも今回初めて知りました。
この場合の世の趨勢は「Github Pages + Jekyll」、
ただ数が多くなると重くなるということで、
最近は「Github Pages + Hugo」に乗り換える人が多いようですが、
Java屋さんとしては、
Gradleで行けるGrainというのがあるというのを知り、
それを使ってみました。
わかってしまえば、結構簡単に入ります。

1. まず、https://github.com で自分のGithub Pages repository(必ず「ユーザーID.github.io」という名前)を作成します。
中身は空で構いません。

2. それから、GrainのOctopress用テーマを持ってきます。
これがGrainを含んでおり、また現状唯一のblog用のテーマです。
[https://github.com/sysgears/grain-theme-octopress/releases](https://github.com/sysgears/grain-theme-octopress/releases)
で適宜最新版を確認してみて下さい。
```
$ wget -O - https://github.com/sysgears/grain-theme-octopress/archive/v0.4.5.tar.gz|tar xvfz -
```

3. この状態で、プレビュー出来ます。
```
$ cd grain-theme-octopress-0.4.5
$ ./gradlew
```
とすると、
[http://localhost:4000/](http://localhost:4000/)
でプレビューできます。
(本当は「./grainw 」でプレビューなんですがうまく行きませんでした)

4. 新しい記事を作成するには、以下のようにします。
```
$ ./grainw create-post 'Any blog post title'
```
すると、
「./content/blog/yyyy-mm-dd-上で指定したタイトル.markdown」

というfileが出来ます。それを編集します。

5. Github Pagesにデプロイするには、
SiteConfig.groovy という設定ファイルの中に、
github のURLを書きます。
私の場合は、↓こうでした。
```groovy
gh_pages_url = 'git@github.com:u-ryo/u-ryo.github.io.git'
```
sshでgithubにアップロード出来るように、
[https://github.com/settings/ssh](https://github.com/settings/ssh)
で手元のマシンのssh public keyを登録しておきます。

6. そうしておいてから、
```groovy
$ ./grainw gendeploy
```
とすると、私(ユーザーID:u-ryo)の場合は、
[http://u-ryo.github.io/](http://u-ryo.github.io/)
でblogが見られる、という塩梅です。

7. タイトル等このblogの設定は、
SiteConfig.groovy の中身を適宜変更すれば良いです。

...というようなことを、これから少しずつ発信していきたいと思います。

### はまった点

* grain-theme-octopress をgit cloneするとうまく行かず(./grainw generateで止まりました)、tar.gzファイルを落として展開するとうまく行った(./grainw generateが通った)
* 記事ヘッダの「author:」で、文字列はシングルクォートないしダブルクオーテーションで囲む必要がある(groovyで処理するので、「文字列」にしておかないと後でエラーになります)
* emacsのバックアップファイル(拡張子の最後に「~」が付くもの)もblogファイルと見做されてしまうため、消す必要がある(また、書いている途中だと「.#〜」というロックファイルが出来るが、これがあるとプレビューが立ち上がらない)
