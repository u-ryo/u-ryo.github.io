---
layout: post
title: "私の AI の使い方変遷"
date: "2026-09-24 15:18"
author: 'u-ryo'
categories: [ai, operation]
comments: true
published: true
---

今や皆さん、当然 AI 使っていますよね。私もそうです。振り返ってみますと、以前から使っていました。

<!-- more -->

何年か前は、Visual Studio Code で、TAB key を押すと source code が補完される、それで喜んでました。それが AI coding の嚆矢だったと思います。

その後、LLM が出てからは、GitHub Copilot を使っていました。Copilot Chat が main で、PR Review とかをさせてもそれはまだなんかいまいちかなぁと。Copilot といっても backend の model は、最初は OpenAI の Codex でしたけれども、後に Claude も選べるようになりました。

当初はそれで、十分でした。Copilot Chat で。Chat window で聞いて、手で直して、と。それが段々と、直接 Copilot 自身に直してもらうようになったりするようになった程度、でした。

ですが、おしおさんの記事 ([Claude Codeで「AI部下10人」を作ったら、勝手にバグ直して「違反は切腹」ルールを追加してきて、オレは適当にしゃべるだけになった](https://zenn.dev/shio_shoppaize/articles/5fee11d03a11a1)) を読んで、自分もやってみたいなぁ、と憧れるようになりました。当時は会社から Copilot のみの提供でしたので、Copilot で Claude Code command を擬して [multi-agent-shogun](https://github.com/yohey-w/multi-agent-shogun/tree/main) を使う方策を調べたりとかしていたのですが、2026年3月末くらいに Claude Code を会社で使えるようにしてもらえました。それで早速試したところ、「ヤバイこれ」3日でそういう結論に至りました。戦国口調も楽しいんですけれども、task を分割して並列で回していくさまがやはりなんとも。正確に言いますと、最初は task を分割してもらえてなかったんですね。どうなるのが正解なのかがわかっていなかったので、shogun とだけ会話してことを済ませてしまっていました。どうやったらおしおさんが言ってるように、みんなでわちゃわちゃと会話しながら task が進んでいくんだろうか? 裏でやっていて見えないだけなのかな? と思っていました。それでも「ヤバイ」と思った、というのは、それは `multi-agent-shogun` というより Claude Code 自体の力量に対する評価ですね。巷間で言われているような、AI agent の力をまざまざと見せつけられた感じがしました。それまでは Copilot で相談して、で特に困ってなかった、というのは井蛙であった、ということです。以後、3ヶ月ほど、`multi-agent-shogun` を楽しく使っていました。ですが、段々と不便を感じてきました。私の仕事の性質によるものなのですけれども、それぞれの task は、あまり分割には適していませんでした。その頃の task で多かったのは、source code を読んで文書にまとめる、というものでした。そういうのは、task 分割はあまり出来なかったのです。ので、足軽を2体に減らしたり、と `multi-agent-shogun` を自分で customize していました。あと、これもわたし側の制約でしたけれど、Claude Code だけなのですね、会社で公式に使える AI Agent が。`multi-agent-shogun` はその名の通り、他の AI にも対応しており、汎用性のため、やり取りが markdown file を介していたので、指示や結果の伝達に結構時間がかかっていました。最初は agent 間のやり取り含め、楽しく見ていたんですけれども、段々その cost を厭うようになってきました。中には、やはり Claude Code しか使わない人が subagent を足軽にするよう独自改良した人もいたようですけれど、そうすると本家の update に追従できなくなるので私は採りませんでした (足軽数を減らすくらいなら parameter 変更程度ですので)。ですが、6月くらいから、shogun 自身に subagent と家老-足軽 ("家中" と呼んでいました) のうち、task に適切な方を自分で選ばせて投げさせていると、殆ど subagent を使い、家中は常に idle 状態になってしまったのですね。ですので、`multi-agent-shogun` を退役させ、Claude Desktop (Claude for Mac) の Code タブで Claude Code を使うようになりました。本家の Claude Desktop では、1つの task の内部を並列化するのではなく、複数の task を同時に走らせられるので、そちらの方が私の使い方に合っていました。

なお、`multi-agent-shogun` の作者のおしおさんご自身も、その後 10 体の家臣団を解散して一体だけを残す [kagemusha](https://github.com/yohey-w/kagemusha) に移られています。[解散の記事](https://zenn.dev/shio_shoppaize/articles/kagemusha-shogun-disband)で挙げられている理由は私のものとは別ですが、行き着いた先は似ていました。

その後も、何かいいものないかな、とアンテナを張り巡らし、引っかかったのが、[MulmoTerminal](https://github.com/receptron/mulmoterminal) ([Claude Code と Codex を並列で動かす — MulmoTerminal 日本語ガイド](https://zenn.dev/singularity/articles/mulmoterminal-guide-ja)) です。これは確か当時の Claude Desktop と違い、`multi-agent-shogun` みたいに複数 terminal を同時に表示させられるものでした。結局その機能は使わず、単一 terminal を最大化してそれを切り替えて使う、という Claude Desktop 的な使い方にすぐなってしまったものの、使ってみて効いたのが、callback です。同時並列に実行させていた task が終わると、当該 window が inactive でも、ポン、と音がして icon が変わってその終了または要対処であることを知らせてくれるんですね。何度かこれに呼ばれるうちに、腹落ちしました。作者の Isamu さんが言っていた「多重並走」は、これのことだったのか、と。これからは、task をバンバン投げといて、向こう (AI 側) から言われたら、人間さまが対応する、という、向きが逆になった気がしました。IoC (Inversion of Control) みたいな。おしおさんとかも MulmoTerminal とかでも、スマホに通知を飛ばしてそれを人間が受けて action する、という話はあって、知ってはいたのですが、やはり自分でそういうようなことをしてみないと、実感としてわからなかったのです。それで1ヶ月程 MulmoTerminal を愛用していたのですけれども、ふと気付くと、本家 Claude Desktop でも、同じことができるんですね今。なので、また Claude Desktop に戻っています。ただ、これも現在進行系であり、他に良いものはないか睥睨しつつ、1年後にはどうなっているか、全然わからないと思っています。

(本文は私が書き、AI は校正と事実確認をしました)

