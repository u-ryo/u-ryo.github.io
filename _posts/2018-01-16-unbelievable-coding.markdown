---
layout: post
title: "Unbelievable Coding"
date: "2018-01-16 10:03"
author: 'u-ryo'
categories: [java, android]
comments: true
published: true
---
Androidのお仕事で、あるclassのcodeを読んでいて。

* onTouchListenerの上にonClickListenerを上書き  
あるbuttonを`setOnTouchListener(this);`してて。
buttonを`onTouchListener`っていうのもなんですが、
`onTouch(...)`で更に`setOnClickListener(...);`してるという...
* timer止めずに新しいinstanceを上書き  
`CountDownTimer`をinstance fieldとして持ってて、
途中でnewしてるんですが、それが複数箇所あるんですよね...
直前にcancel処理とか特に無いし。大丈夫なのかこれ。
* カタカナや"$","/"にtoLowerCase()/toUpperCase()してindexOf()  
`"半角カタカナ".toLowerCase()`してから`indexOf(...)>-1`して、
`contains(...)`と同じことしてました。`indexOf`はまだしも、
`toLowerCase`したからって
全角カタカナが半角カタカナになるわけじゃないのに。
え、まさか、とか思ってしまった自分が情けないです。
同様に、記号に対しても`"$".toLowerCase()`とか謎すぎます。
* 他の(inflateもincludeもしてない)View上のR.idをfindViewById()  
当然`null`です`findViewById()`しても。実質無害なcodeではありますが。
どうやら他から何も考えずにコピペしたから、らしいです。
* listの2度回し  
なるべく一度で済むように書きますよねぇ、フツーは。
ちょっと違う処理をするから、なのか、
同じlistを直後に2度回して、って。
まー他でも同じtableのDB accessを3回してたりしますからねーこのcode。
* 1830秒?  
随分謎なMagic Numberです。
* loop回すのに中で値を上書き(結局見てるのは最後の値だけ)  
`for(i in list){v = i}`みたいな。`v=list[lastIndex]`でいいじゃん。
そういうことされると意図が読めないんですよね。困ります。
* loopの空回し  
waitしたいみたいなんですが、
`while(true){if(!flag)break;}`ってこれじゃぁCPU無駄遣いでしょ。
改善したっていって`do{i=0;}while(!flag);`って、あのねー...
* `synchronized wait()`で同期
他Activity(dialog)に遷移させ、
その同期に`synchronized(this){wait();}`って使ってます。
そういうthread jugglingはやめて欲しい、です。
こういうのってホントはRXですよね。
* method/field名が大文字で始まっててclass名と区別がつかない、
なんていうのは可愛い方で、もう気にもならなくなってますそういえば。
methodも長いし条件分岐も複雑で、
state patternとかなんて知らないんだろうなぁと。
* というか全てがfat ActionでFragmentもなければApplicationもないという
(基本的には。後から「訳も分からずダーッとコピペした部分」にはありますが)。

...というように。
こういうcodeと共に仕事するのは、嫌で嫌で仕方ありません。
