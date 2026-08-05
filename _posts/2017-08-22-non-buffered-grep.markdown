---
layout: post
title: "non buffered grep"
date: "2017-08-22 11:48"
author: 'u-ryo'
categories: [grep, pipe]
comments: true
published: true
---
grepをpipeで繋いでlogcatをscreeningした時、
なかなかlogが出て来なかったので、
もしやと思って調べたら、bufferingしてるようでした。
それを避けて、出て来たらすぐgrepして出すようにするには、

```
~/Android/Sdk/platform-tools/adb logcat -v time|grep --line-buffered -e send_code1 -e doOnNext -e 'D/After   ([ 1-9][0-9]*): [0-9]'|tee remote_diagnosis.log
```

のように`--line-buffered`が必要でした。