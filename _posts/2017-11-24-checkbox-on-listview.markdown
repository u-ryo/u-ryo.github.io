---
layout: post
title: "CheckBox on ListView"
date: "2017-11-24 18:21"
author: 'u-ryo'
categories: [android, CheckBox, ListView]
comments: true
published: true
---
Androidでのお話です。
`ListView`のそれぞれに`CheckBox`をつけたら、
checkboxはcheck出来るものの、項目選択が出来なくなりました。
どうやら`onItemClick`が呼ばれてない様子。
調べてみると、`CheckBox`がfocusを奪ってしまっているそうでした。
([カスタマイズしたListViewに設定したCheckBoxのon/offを行全体で行う](http://inujirushi123.blog.fc2.com/blog-entry-53.html))

```
android:clickable="false"
android:focusable="false"
```

が必要とのこと。

また、
background処理後、`Adapter`の値を変えただけでは`CheckBox`の見た目に変化はないんですね。
explicitに`setChecked(false)`して回らないとなりません。
その際、`listView.getChildCount()`で取れるcountは、`ListView`の全てではなく、見える範囲のListのobjectなんですね! 確かにscrollすればredrawかかってadapterの値が反映されるからいいんですけど、何かしない限りredrawされないから自分で描画しないとならないんですねー。
