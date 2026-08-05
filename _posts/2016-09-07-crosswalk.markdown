---
layout: post
title: "Crosswalk"
date: "2016-09-07 22:15"
author: 'u-ryo'
categories: [ionic, crosswalk, android, hybrid, WebView]
comments: true
published: true
---
Crosswalk という、WebViewを独自に持つことでAndroid間の動作性を統一するlibraryを使ってみました。最近のAPI levelに対応するだけなら、敢えてこんなの使わなくてもいいのかも。

わけあって、Crosswalk sample applicationの一つをcompileしてみました。

how to installは、[official page](https://crosswalk-project.org/documentation/android/system_setup.html)のまま。

`ANDROID_HOME`は、Sdk directoryを指す模様。

それから、`android` commandを裏で動かすので、そこへのPATHが必要でした。

```
$ ANDROID_HOME=/home/u-ryo/Android/Sdk PATH=$PATH:/home/u-ryo/Android/SDK/tools crosswalk-app check android
$ ANDROID_HOME=/home/u-ryo/Android/Sdk PATH=$PATH:/home/u-ryo/Android/SDK/tools crosswalk-pkg -p android client
```

`android` commandはGUIを要求するので、Cloud9のconsoleでは動かせませんでした。

[Android Video Calling with CrossWalk and PeerJS](https://www.sitepoint.com/android-video-calling-with-crosswalk-and-peerjs/)
見てやってみたんですが、ionic2からは`ionic browser list`というのがなくなって、`ionic plugin add cordova-plugin-crosswalk-webview`になったことと、その後`ionic platform add android`すると、

```
Installing "cordova-plugin-crosswalk-webview" for android


        After much discussion and analysis of the market, we have decided to discontinue support for Android 4.0 (ICS) in Crosswalk starting with version 20.

        So the minSdkVersion of Cordova project is configured to 16 by default.
```

と言われてしまいました...

動かしたかったtargetは、Android 4.0.4なので、Crosswalkダメですね。
実際、「CPU Mismatch」と言われて、x86のものでもarmのでも動きませんでした。
あうー。