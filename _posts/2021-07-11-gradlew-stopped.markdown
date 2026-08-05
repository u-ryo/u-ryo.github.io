---
layout: post
title: "gradlew stopped"
date: "2021-07-11 01:02"
author: 'u-ryo'
categories: [gradle, grain]
comments: true
published: true
---
実に久し振りにこのblogを書こうとして、

```
./grainw create-post 'some title'
```

としたら、`gradle-1.8.jar`を取りに行って404 not found食らって動きません。
この[grain](https://github.com/sysgears/grain)って古いしもう更新止まってるし、先見の明なかったなぁということなんですが、もうちょっと本気で原因究明です。
https://services.gradle.org/distributions/ はあるのになぜ404? と思ったら、`http://...`で取りに行ってました。gradlewのgradleが古すぎる?
`grainw`のどこで止まってるか、debug print入れて割り出すと、`./gradlew gendeps`でした。これをversion 6.8.1のgradleで手動でやってみる(`gradle gendeps`)と、`build.gradle`の`line: 78`で止まります。なんで?と思って調べると、[「Cannot add task 'wrapper' as a task with that name already exists.」が出た](https://qiita.com/Yu-s/items/13a6a8db8dc191bb3b42)にあった通り、でした。

```
task wrapper(type: Wrapper) {
```

を

```
wrapper {
```

にすれば動きました。良かった。
`gradle wrapper`で`./gradlew`を更新して、`./grainw`も復活しました。
