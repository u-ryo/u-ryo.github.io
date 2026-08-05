---
layout: post
title: "site search in this blog"
date: "2016-10-30 17:57"
author: 'u-ryo'
categories: [grain, search]
comments: true
published: true
---
この自分のblog、検索したいことがよくあるんですが、
右上のsearch box、単なるgoogle searchなので、
何でblog内検索がないのかなー、
って思ってたんです。
いい加減不便なので、
調べてみました。
すると、theme/includes/navigation.html にて、

```
<input type="hidden" name="q" value="site:${site.url}" />
```

と書いてあるので、
site限定searchをやろうとしていた意図は窺えました。
これがうまく行っていないので、
調べて下のようにしたところ、
google検索結果に「site:...」がついてくれるようになりました。

```
<input type="hidden" name="sitesearch" value="${site.url}" />
```

しかし、結果にRecent Postでの見出しも含まれてしまうなど、
かなり広めに出て来ちゃうようなので、
Lunr.js Pluginでも入れた方がいいのかなー。
grainにoctopress用pluginが入るのか?!
というのはありますが。