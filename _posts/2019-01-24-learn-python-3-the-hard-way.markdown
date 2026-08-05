---
layout: post
title: "Learn Python 3 the Hard Way"
date: "2019-01-24 03:30"
author: 'u-ryo'
categories: [python]
comments: true
published: true
---
知人が[Learn Python 3 the Hard Way](https://www.amazon.co.jp/Learn-Python-Hard-Way-Introduction-ebook/dp/B07378P8W6)を[翻訳しました](https://www.amazon.co.jp/Learn-Python-Hard-Way-%E6%9B%B8%E3%81%84%E3%81%A6%E8%A6%9A%E3%81%88%E3%82%8BPython%E5%85%A5%E9%96%80/dp/4621303287)。
凄いなぁと思うと同時に、
見てみると既に[英語版](https://learnpythonthehardway.org/)で[勉強した人がいる](http://blog.sayajewels.com/posts/janiota-python-practice/)というので、
その人がやったという初心者用練習問題を、
ジャニヲタじゃないので、
というか初心者ではないので「一行で」やってみました。

```
>>> def average_age(members):
...  return sum([x[2] for x in members])/len(members)
...
>>> average_age([('s','s',35),('a','a',34),('n','n',34),('d','o',36),('j','j',34)])
34.6

>>> def oldest_member(members):
...  return max(members, key=(lambda x:x[2]))[0]
...
>>> oldest_member([('s','s',35),('a','a',34),('n','n',34),('d','o',36),('j','j',34)])
'd'

>>> def second_oldest_member(members):
...  return sorted(members, key=(lambda x:x[2]))[1][0]
...
>>> second_oldest_member([('s','s',35),('a','a',34),('n','n',34),('d','o',36),('j','j',34)])
'n'

>>> def next_older_member(members, nickname):
...  return min([a for a in members if a[2] > [b for b in members if b[0]==nickname][0][2]],key=(lambda x:x[2]))[0]
...
>>> next_older_member([('st','ms',19),('m','my',17),('k','nk',23),('f','kf',22),('sh','ss',20)],'f')
'k'

>>> def check_future_age(current_members, future_members):
...  return len({i[2]-j[2] for i,j in zip(current_members,future_members)}) == 1
...
>>> check_future_age([('st','ms',19),('m','my',17),('k','nk',23),('f','kf',22),('sh','ss',20)],[('st','ms',22),('m','my',20),('k','nk',26),('f','kf',25),('sh','ss',23)])
True
>>> check_future_age([('st','ms',19),('m','my',17),('k','nk',23),('f','kf',22),('sh','ss',20)],[('st','ms',22),('m','my',20),('k','nk',26),('f','kf',25),('sh','ss',24)])
False
```

あぁ、なんてアホなことに時間と頭を使ってしまった...

`min`、`max`、`sorted`ってリスト内包表記で出来ないんですか?

印税は初版で20〜30万、あとは増刷でガッポガッポ、だそうです。
1年半かかったって。お疲れさまでした。
ぼくも販促に協力しましょうかね。
そしておこぼれにあずかれれば!
