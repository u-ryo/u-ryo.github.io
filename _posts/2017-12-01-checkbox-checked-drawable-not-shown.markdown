---
layout: post
title: "CheckBox.checked drawable not shown"
date: "2017-12-01 16:33"
author: 'u-ryo'
categories: [android, checkbox, context]
comments: true
published: true
---
`ListView`で、各行にcheckboxを表示させるような話があって。
暗い背景なので、defaultのdesignだと見にくいんですね。
なのでcustomの白っぽいのに差し替えようとしたんですが、
なかなかうまく行かなかったのです。
基本的には、`res/drawable/`に、

```
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:state_enabled="false" android:drawable="@drawable/ic_check_disabled"/>
    <item android:state_checked="true" android:state_pressed="false" android:drawable="@drawable/ic_check_on"/>
    <item android:state_checked="false" android:state_pressed="false" android:drawable="@drawable/ic_check_off"/>
    <item android:state_checked="true" android:state_pressed="true" android:drawable="@drawable/ic_check_on_pressed"/>
    <item android:state_checked="false" android:state_pressed="true" android:drawable="@drawable/ic_check_off_pressed" />
</selector>
```

と書いて(状態判定は上の行から順番になされる模様)、layoutで、

```
<CheckBox
  android:id="@+id/historySelected"
  style="@style/historyListCheckBox"
  android:button="@drawable/selector_checkbox"
/>
```

というように参照すればいいんです。
が、それだとcheckしても絵が変わらなかったんですね。
別途`OnClickListener`に、

```
checkbox.setButtonDrawable(checkbox.isChecked() ? R.drawable.ic_check_on : R.drawable.ic_check_off);
```

が必要でした、というのはまだわかるんですが、
これを書いても`ic_check_on`の絵にならなかったんですね(`ic_check_off`の絵のまま)。
なんでだろ～、1日程悩みました。

結局、
stackoverflowの[Can't create custom arrayadapter with appcompat elements inside of it](https://stackoverflow.com/questions/34508164/cant-create-custom-arrayadapter-with-appcompat-elements-inside-of-it)に書いてあったんですけど、
`ListView`のAdapterを作る時の`Context`が、
`getApplicationContext()`で得られたものであったこと、
が敗因でした。`getApplication()`でもダメでした。
`this`でないと、`ic_check_on`がdrawされませんでした。

```
adapter = new SimpleAdapter(this, someList, R.layout.some_listview, new String[]{...}, new int[]{R.id.someId,...});
```

`this`で引き回すと、使ってるfieldとか色々引きずるから
なるべく`getApplicationContext()`にしましょうね、
というのを聞いたことがあるのですが、
なるほどと思ってそうすると、
結構色んな箇所で出るべきものが出なくなるんですよね。
気を付けないとなりません。
