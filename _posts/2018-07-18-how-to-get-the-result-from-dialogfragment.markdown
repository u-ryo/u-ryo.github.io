---
layout: post
title: "How to get the result from DialogFragment"
date: "2018-07-18 16:30"
author: 'u-ryo'
categories: [android, dialog, dialogfragment]
comments: true
published: true
---
`DialogFragment`をnewしてdialogを表示させ、
そこでのbutton tapによって、元のActivity上で処理をさせたい時、
どうやってfeedbackしたら良いのかなと。
呼び出し元は`Fragment`ではなく`Activity`なので、
[Fragmentで呼び出し元に結果を伝える](https://tech.mokelab.com/android/Fragment/result.html)[Receive result from DialogFragment](https://stackoverflow.com/questions/10905312/receive-result-from-dialogfragment)等にあるように`setTargetFragment()`を使えないんですよね。

結局、`DialogFragment`側に`setCallback(Callback callback)`と
Functional Interfaceとして`Callback`を定義して、
button押したら`callback.call();`とし、
呼び出し元の`Activity`側で`new SomeDialogFragment().setCallback(() -> someMethod())`としてやりたいことを`someMethod()`に込めました。
ちょっと面倒ですけどこのようにcallback駆使するしか無いのかなぁと。
`onActivityResult()`は`getTargetFragment()`が使えないのと
処理を`Activity`側に書きたいというのがあったので。
いや、`onActivityResult()`の中身は`Activity`側ですか。
`setTargetFragment()`の代わりに何か`Activity`のreferenceを
`DialogFragment`側に持たせればよかった?のかな?
いやいや、そもそも`Fragment`から`getActivity()`で取得できる?
からこんなことしなくてよかった?
あれ??
いやーでも`DialogFragment`側は引数の情報を持っておらず、
`Activity`側しか引数持ってないんですよね。
今回のぼくの場合では、
引数を引き回すか、再度SQLで取得するかして`onActivityResult()`でkickするか、
callbackを作るか、ということだったでしょうか。
