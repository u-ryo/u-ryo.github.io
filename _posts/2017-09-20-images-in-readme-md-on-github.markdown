---
layout: post
title: "Images in README.md on Github"
date: "2017-09-20 00:58"
author: 'u-ryo'
categories: [git, markdown]
comments: true
published: true
---
[Github](https://github.com/)の`README.md`にscreen shotとか画像を貼りたかったんですけど、どうしたものなんでしょう。説明用のimagesをsource codeと一緒に置いておくというのも何か野暮なので。幾つかやり方があるようです。

1. image用の別branchを切る(e.g. [https://github.com/cakecatz/garage](https://github.com/cakecatz/garage))
1. Githubのissueを利用する([GitHub Readme Images Tutorial (screenshots in readmes)](https://www.youtube.com/watch?v=hHbWF1Bvgf4))
1. Githubのwikiを利用する([GitHubに画像ファイルを保存してREADME.mdで表示する方法](https://www.pupha.net/archives/1632/))

issueに画像をDrag and Dropすると、https://user-images.githubusercontent.com/... に自動的にuploadしてURLが得られるなんて知りませんでした。でもこれ、いつまで持っててくれるんでしょう、というのと、消したい時に消せるのかな? というのがあって。やっぱり自分でcontrol持っておきたいでしょう。([GitHubのissueを悪用して画像をホストする](https://qiita.com/kotet/items/a2203a400136ba50b41e))

別branchを切ると、一旦画像以外全部消さないとならなくて、
何となくそれがちょっと嫌だったので、結局wikiを利用しました。
一度何でもいいのでWikiのpageを保存しないとならないみたいです。
一度保存しちゃえば、上記ページに書いてあるように`git clone https://github.com/.../XXX.wiki`で取ってこられて、images作って置けちゃうんですね。へー。
