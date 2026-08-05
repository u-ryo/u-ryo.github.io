---
layout: post
title: "Android petite tips"
date: "2018-07-30 18:05"
author: 'u-ryo'
categories: [android, java, textview]
comments: true
published: true
---
お仕事で極悪Androidアプリを改修していて、
今日得た知見をば。

### getTextSize/setTextSize

あるActivityの画面で、
本文とボタンのtext sizeを揃えようとして、
`TextView#getTextSize`してから`setTextSize`したら、
大きくなるんですよね。何でだろう、調べると、
[`TextView#getTextSize`と`setTextSize`のデフォルト単位が違う](http://yamato-iphone.blogspot.com/2012/02/gettextsizesettextsize.html)
のだそう。びっくりです。

```
renewalButton.setTextSize(TypedValue.COMPLEX_UNIT_PX, caution.getTextSize());
```

というように、単位を明示する必要があるそう。


### サイズ自動調整TextView

`TextView`で、指示通りに改行位置を固定しようと思って。
指示中の、禁則処理に失敗しているところも含めて忠実に再現しようと。
そのためにはtext sizeを随意にせねばならず。
[【Android】横幅に合わせてテキストサイズを調整するTextView](https://gist.github.com/STAR-ZERO/2934490)そのままで上手く行きました。
あーでも`onLayout()`の最初の引数`changed`が`true`の時だけ
`resize()`すればよかとです。

今は[Autosizing TextViews](https://developer.android.com/guide/topics/ui/look-and-feel/autosizing-textview)というのがあるそう。
ただ、API 26からなのでまだなかなか使えないでしょうか。


### onLayout後の値の取得

上記のようにtext sizeを変えてから、
その結果のtext sizeに合わせて他のViewのtext sizeを
決定しようとすると、
`onLayout()`が呼ばれ終わってからでないと
目的の値が取得出来ないんですね。
そこで、
[How to know when an activity finishes a layout pass?](https://stackoverflow.com/questions/8418868/how-to-know-when-an-activity-finishes-a-layout-pass)
にあるように、
`myView.getViewTreeObserver().addOnGlobalLayoutListener(() -> {...});`
とすれば良いです。

#### 補足

RobolectricでUnit Test書いてたら、
このclass、testが終わらないんです。
何でかなー、とbreakpointで追ってみると、
延々と`onGlobalLayout()`が呼ばれ続けてるんですね。
えーっと思って。
`RxJavaHooks.setOnIOScheduler(s -> Schedulers.immediate());`
しても、
`ShadowApplication.runBackgroundTasks();`
しても効き目はなく。
[androidでheightやwidthが0になって取得できない時](http://shim0mura.hatenadiary.jp/entry/2016/01/11/013000)
を見ると、用が済んだらすぐremoveするんですね。そっか。
というわけで、Android SDKのversionによって分けて、
`removeOnGlobalLayoutListener(this)`と
`removeGlobalOnLayoutListener(this)`でremoveするように
したんですけど、今度は`this`が効かない。
なるほど、lambdaだと`this`は外側のclass instanceになるんですね。
じゃぁっていうんでlambda自体をListener instanceとして名前付けて、
lambdaの中で`this`じゃなくてその名前で参照しようとしたんですが、
`might not have been initialized`とか言われ、
`null`で初期化すると今度は`effectively final`じゃないと言われて、
うー、とか思って仕方なく諦めて、
初めてlambdaを解いてinner classの記述に戻しました。


### Rx Androidにmaxはない?

探したんですけど見つからなかったので自分で集計しました。

```
float textWidth = rx.Observable
        .from(getText().toString().split("\n"))
        .map(paint::measureText)
        .reduce(Math::max)
        .toBlocking()
        .single();
```

### TextViewで白枠

ある段落を白枠で囲って欲しいと言われました。
調べると、[[android]xmlで枠を指定する](https://qiita.com/Yuki_Yamada/items/15fc68dc88b57734149b)というのがあり、
それ用のdrawable XMLを作ってやって`android:background="@drawable/..."`で、
それを指定すれば、望み通りのものが得られました。
背景色は、これも書いてありますが`#00ffffff`で透明になります。

```
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="rectangle">
    <!-- background -->
    <solid android:color="#00ffffff" />
    <!-- rounded corners -->
    <stroke android:width="1dp"
        android:color="@color/white" />
    <corners android:radius="1dp" />
</shape>
```

### Robolectricで次のActivityへの遷移の確認

`shadowOf(activity).peekNextStartedActivity()`で`Intent`を取得、
`getComponent().getClassName()`が目的のclass nameかをassert。

```
Intent intent = shadowOf(activity).peekNextStartedActivity();
assertThat(Objects.requireNonNull(intent.getComponent()).getClassName(),
        is(MainActivity.class.getName()));
```

### Local PushでNotification

Local PushでNotificationをして欲しい、と言われました。
調べてみると、要するに、
`AlarmManager`に`PendingIntent`をsetして、
それが`set`時の引数のUnix Time(millisec)になると、
これも`set`時引数の`BroadcastReceiver`の子classの
`onReceive(context, intent)`が呼ばれるので、
そこで`NotificationManager.notify()`をする、と。

AndroidのNotificationについては、
sample applicationを作って色々と試してみました。

1. uninstall/端末再起動すれば登録済みのalarmは解除される
1. 多重登録しても`PendingIntent.FLAG_UPDATE_CURRENT`なら最後のNotificationに上書きされる
1. 過去の時日のalarmを登録するとすぐNotifyされてしまう
1. 機種によっては挙動が違う(Huaweiでは、アプリが起動していない時/Sleep時にAlarmを発動させるには「保護されたアプリ」でないとならない、等)
1. 長いtextは全文出ないで端折られる。出したいなら、`.setStyle(new NotificationCompat.BigTextStyle().bigText("..."))`する。但し`.setBigContentTitle(intent.getStringExtra("..."))`も同時に加えるとダメっぽい。
