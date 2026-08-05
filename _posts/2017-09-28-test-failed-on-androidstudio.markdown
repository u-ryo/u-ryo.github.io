---
layout: post
title: "Test Failed on AndroidStudio"
date: "2017-09-28 14:12"
author: 'u-ryo'
categories: [android studio, android]
comments: true
published: true
---
Android Studio(2.3.1)で久し振りにtestを動かしてみると、

```
Caused by: java.lang.ClassNotFoundException: android.view.View$OnClickListener
```

と言われて動かなくなりました。
instrumentation testではなくフツーのtestですjunit4とmockitoの。
[java.lang.NoClassDefFoundError:android and junit test](https://stackoverflow.com/questions/14213219/java-lang-noclassdeffounderrorandroid-and-junit-test)を見付けて、えーとか思いつつもやってみたら、確かに直りました。

```
$ rm -rf .gradle
```

### 追記
projectを`clean`した後、いくらbuildしても「`databinding` classが見つからない」と言われて困った時にも効きました。
