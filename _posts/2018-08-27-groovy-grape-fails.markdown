---
layout: post
title: "Groovy Grape fails"
date: "2018-08-27 13:53"
author: 'u-ryo'
categories: [groovy, grape]
comments: true
published: true
---
時々、「全く同じgroovy scriptなのにあるマシンでだけ
起動に失敗する」ことがあります。
error messageはcaseによりますが、例えば直近では以下のようなものでした。

```
$ groovy webpush.groovy
org.codehaus.groovy.control.MultipleCompilationErrorsException: startup failed:
General error during conversion: Error grabbing Grapes -- [download failed: com.google.guava#guava;19.0!guava.jar(bundle)]

java.lang.RuntimeException: Error grabbing Grapes -- [download failed: com.google.guava#guava;19.0!guava.jar(bundle)]
        at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
        at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62)
        at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
        at java.lang.reflect.Constructor.newInstance(Constructor.java:423)
        at org.codehaus.groovy.reflection.CachedConstructor.invoke(CachedConstructor.java:83)
        at org.codehaus.groovy.reflection.CachedConstructor.doConstructorInvoke(CachedConstructor.java:77)
        at org.codehaus.groovy.runtime.callsite.ConstructorSite$ConstructorSiteNoUnwrap.callConstructor(ConstructorSite.java:84)
        at org.codehaus.groovy.runtime.callsite.CallSiteArray.defaultCallConstructor(CallSiteArray.java:59)
        at org.codehaus.groovy.runtime.callsite.AbstractCallSite.callConstructor(AbstractCallSite.java:238)
        at org.codehaus.groovy.runtime.callsite.AbstractCallSite.callConstructor(AbstractCallSite.java:250)
        at groovy.grape.GrapeIvy.getDependencies(GrapeIvy.groovy:464)
   :
   :
```

要するに「guava-19.0.jarのdownloadに失敗した」と。
`~/.groovy/grapes/com.google.guava/guava/`を見ると、
確かに他のlibrariesのように`jars` directoryがありませんし、
当然jar fileもありません。
じゃぁっていうんで、自分で`guava-19.0.jar`を落としてきて、
`jars/`作ってその下に置いても、同じ結果です。
Groovyが悪いのかJavaが悪いのかGrapeが悪いのか設定が悪いのか
versionが悪いのか、何なのか全然分かりません。
普通に動くマシンでのJava/Groovyとversionを合わせてもダメ、
Grapeする各libraryのversionを上げてもダメ、
Guava 19.0を明示的に`@Grab`に入れてもダメ、
[Groovy - Grab - download failed](https://stackoverflow.com/questions/16871792/groovy-grab-download-failed)を見て`~/.groovy`以下や`~/.groovy/grapes`を削除してもダメ、
と途方に暮れました。
いつもは面倒なので諦めて、
その動くマシンでやっていつのですけれども、
今回は粘って原因究明してみました。

結局、以下のどちらかの方法で立ち上がるようになりました。

1. `~/.groovy/grapes/com.google.guava/guava/jars`ではなく`~/.groovy/grapes/com.google.guava/guava/bundles`にして、その下に落としてきた`guava-19.0.jar`を置く
1. 他の多くのlibrariesと同様に`~/.groovy/grapes/com.google.guava/guava/jars`に落としてきた`guava-19.0.jar`を置き、`~/.groovy/grapes/com.google.guava/guava/ivy-19.0.xml`の`<publications>`下の`<artifact name="guava" type="bundle" ext="jar" conf="master"/>`の`type` attributeを`bundle`から`jar`にする

上手く行っていたマシンでは、
`~/.groovy/grapes/com.google.guava/guava/bundles`が出来ており、
その下に`guava-19.0.jar`もフツーにありました。
上手く行かないマシンではなぜ`bundles`が出来ないのか、は未だ不明ですが、
取り敢えず前に進める方法がわかったので記しています。


あぁ、あと、このようにGrapeに失敗すると、
`~/.groovy/grapes/`以下に`resolved-caller-all-caller-working73.properties`と`resolved-caller-all-caller-working73.xml`といったfilesが澱のように溜まっていくので、都度削除した方がいいです。
