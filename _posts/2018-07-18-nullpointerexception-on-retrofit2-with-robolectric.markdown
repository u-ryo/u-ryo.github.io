---
layout: post
title: "NullPointerException on Retrofit2 with Robolectric"
date: "2018-07-18 14:19"
author: 'u-ryo'
categories: [java, android, robolectric, retrofit]
comments: true
published: true
---
Android ApplicationをRobolectricでtestしていて、
どうにも困ったのでメモです。

状況は、Android Applicationで、
Robolectricを使っていて、
Retrofit2で`POST`しにいく部分(受け手はMockWebServer)のunit testで、
突然`NullPointerException`になって`subscribe`の`error`に入ってしまう、というもの。
breakpointで追っていっても`call()`で突如NPEに入ってしまって、
具体的にどこでNPEに陥っているのかよく分かりませんでした。
Googleで探してみると、
[Stack Overflow](https://stackoverflow.com/questions/39827081/unit-testing-android-application-with-retrofit-and-rxjava)にそれらしき投稿があり、
`.observeOn(AndroidSchedulers.mainThread())`は
`LooperScheduler`なのでここでNPEになる、
だから`RxAndroidPlugins`の`registerSchedulersHook()`で
`Schedulers.immediate()`してやると良い、
と書いてあって、やったー!
と思ったものの、効果なく。

結局そうではなくて、
MockWebServer使っているから`http://localhost:NNNN/...`に
requestを改装しているせいなんですけど、
SecurityPolicy絡みのExceptionが裏で出ているようで、
以下のようなShadowを用意して`@Config({shadows=...})`に
書いてやればそのままですんなり行きました。

```
import android.security.NetworkSecurityPolicy;

import org.robolectric.annotation.Implementation;
import org.robolectric.annotation.Implements;

@Implements(NetworkSecurityPolicy.class)
public class ShadowNetworkSecurityPolicy {
    @Implementation
    public static NetworkSecurityPolicy getInstance() {
        try {
            Class<?> shadow = ShadowNetworkSecurityPolicy.class.forName("android.security.NetworkSecurityPolicy");
            return (NetworkSecurityPolicy) shadow.newInstance();
        } catch (Exception e) {
            throw new AssertionError();
        }
    }

    @Implementation
    public boolean isCleartextTrafficPermitted(String hostname) {
        return true;
    }
}
```

勿論、`.subscribeOn(Schedulers.io())`に対しては
`RxJavaHooks.setOnIOScheduler(s -> Schedulers.immediate());`
した上で、です。
