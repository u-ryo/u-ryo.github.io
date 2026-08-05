---
layout: post
title: "Robolectric + PowerMock"
date: "2017-10-17 18:26"
author: 'u-ryo'
categories: [Robolectric, PowerMock, android, test, RoboSpock, ElectricSpock, spock]
comments: true
published: true
---
[Robolectric](https://robolectric.org)のquick startは、[本家](http://robolectric.org/writing-a-test/)が詳しい。

* `build.gradle`に、以下が必要(Android Studio 2系の場合)。
```
testCompile 'org.robolectric:robolectric:3.4.2'
testCompile 'org.robolectric:shadows-multidex:3.3.2'
testCompile 'org.powermock:powermock-module-junit4:1.7.3'
testCompile 'org.powermock:powermock-module-junit4-rule:1.7.3'
testCompile 'org.powermock:powermock-api-mockito2:1.7.3'
testCompile 'org.powermock:powermock-reflect:1.7.3'
testCompile 'org.powermock:powermock-classloading-xstream:1.7.3'
```
* Test Classは、Android Studioで開いた実class java fileの`public class CLASS名`のところで黄色いヒントをclickして`Create Test class`を選択、`JUnit4`で作成
* 既存test classの`Whitebox`は`org.powermock.reflect.Whitebox`で置き換え
* 既存test classの`@RunWith`の`MockitoJUnitRunner`は`org.mockito.junit.MockitoJUnitRunner`で置き換え
* `org.mockito.exceptions.misusing.UnnecessaryStubbingException:`というwarningが出るようになったので`@RunWith(MockitoJUnitRunner.Silent.class)`にすると解消  
cf. [How to resolve Unneccessary Stubbing exception](https://stackoverflow.com/questions/42947613/how-to-resolve-unneccessary-stubbing-exception)
* `Robolectric`なTestは、1.`@RunWith(RobolectricTestRunner.class)` 2.`activity = Robolectric.setupActivity(SomeActivity.class);`で`Activity`を起動
* `RuntimeException: Multi dex installation failed`と言われるので`shadows-multidex`が必要  
cf. [Robolectric と Multidex でテストが落ちる問題の対応](https://qiita.com/kuwapp/items/942f0e44adbd45adff10)
* static methodのmockは[PowerMock](https://github.com/powermock/powermock)と。`@RunWith`がかぶっちゃうよ、どうしよう! → [本家に解説](https://github.com/robolectric/robolectric/wiki/Using-PowerMock)あり。要は、`@PowerMockIgnore`でmockito、robolectric、android標準classesを除外、`@PrepareForTest`でstatic methodを持つclassを指定し、`@Rule`を入れ(使わないのによくわからないが必要)、`PowerMockito.mockStatic(...)`で当該classを指定
* `NoClassDefFoundError: org/powermock/classloading/ClassloaderExecutor`と言われるので、`powermock-classloading-xstream`が必要  
cf. [version 1.5.5 java.lang.ClassNotFoundException: org.powermock.classloading.DeepCloner #597](https://github.com/powermock/powermock/issues/597)
* `NoClassDefFoundError: org/mockito/cglib/proxy/MethodInterceptor`と言われるので、`powermock-api-mockito2`と`2`でないとならない  
cf. [Problem with org.mockito.plugins.MockMaker and loading MethodInterceptor #819](https://github.com/powermock/powermock/issues/819)
* `javax.xml.parsers.FactoryConfigurationError: Provider ...DocumentBuilderFactoryImpl cannot be cast to javax.xml.parsers.DocumentBuilderFactory`と言われるので`@PowerMockIgnore`に`"javax.xml.*", "org.xml.sax.*", "org.w3c.dom.*", "org.apache.log4j.*"`が必要  
cf. [Powermock + Mockito + Spring = DocumentBuilderFactoryImpl](https://groups.google.com/forum/#!topic/powermock/YJYPgBLpkqk)
* `org/powermock/default.properties is found in 2 places`と言われてerrorにはならないけどwarningが出るので、`@PowerMockIgnore`に`"org.powermock.*"`も入れておく(試行錯誤の末なので参照なし)
* `AsyncTask`があっても、特段その終了を待たずにtestが終了してしまう。`Robolectric.getBackgroundThreadScheduler().pause();`で`AsyncTask#doInBackground()`を止める必要がある(`AsyncTask#onPreExecute()`は実行される)。
* [`ShadowAsyncTaskTest.java`](https://github.com/robolectric/robolectric/blob/master/robolectric/src/test/java/org/robolectric/shadows/ShadowAsyncTaskTest.java)を見ると、`setUp()`で`Robolectric.getBackgroundThreadScheduler().pause();`(と`Robolectric.getForegroundThreadScheduler().pause();`?)でthread止めて、`asyncTask.execute()`すると`onPreExecute()`が動き、次に`ShadowApplication.runBackgroundTasks();`すると`doInBackground()`、`ShadowLooper.runUiThreadTasks();`すると`onPostExecute()`が動く(ようだが、試してみると`ShadowApplication.runBackgroundTasks()`で返ってこなくなった。何故?!←これは単に`AsyncTask`中でdialog出して止まっていたため)
* target class内でnewしているもののmockは、
`PowerMockito.whennew(XXX.class).thenReturn(mock);`
だと、
```
org.mockito.exceptions.base.MockitoException: 
ClassCastException occurred while creating the mockito mock :
...
You might experience classloading issues, please ask the mockito mailing-list.
```
と言われて失敗する。
* shadow classでもstatic methodのmockが出来る。PowerMock使わずとも良い様子。いちいちShadow class作って各method毎に`@Implements`書くのは面倒ではあるが、PowerMockを`@Rule`して並存させると上述のようにclass loaderがどうのと言われて失敗したので、Robolectric一本で頑張った方がよさ気。PowerMock使わないなら`testCompile`も`robolectric`と`shadows-multidex`の2つで済むし、PowerMock導入に伴って変更したMockito部分も変更不要になる。
* Custom Shadow classesの追加でcustom TestRunnerは作成不要、単に`@Config`に`shadows={ShadowXXX.class}`と追記していけば良い。
* Shadowについて。Android APIのclassesについては、全て`ShadowXXX`というclassが揃っている(e.g. `ShadowActivity`)。まるっとmockしたものを返したい場合には、custom shadow methodで`return Shadow.newInstanceOf(ShadowBluetoothDevice.class);`で良い。
* [constructorもshadow出来る](http://robolectric.org/extending/#shadowing-constructors)。constructorの場合には単に`public void __constructor__(...){...}`でよく、`@Implementation` annotationは不要(あっても害はない)。
* `extends`してるclassのconstructorの場合には、super classのconstructorのshadowingも必要。さもなくばsuper classの当該constructorが実行されてしまう。また、super classのconstructorもshadowingする場合、当該Shadow classの方も`extends`しないと`ClassCastException`に見舞われる。`A extends B`で`A`のconstructorをshadowingしたら`B`のconstructorもshadowingし、`ShadowA extends ShadowB`にする必要がある。
* `Shadows.shadowOf(myDialog).hasBeenDissmissed()`といったようにUIの状態を取得できる。
* `context.getPackageManager().getLaunchIntentForPackage("package name")`がRobolectricsでやると`null`を返しやがるのでヌルポで失敗しくさる。多くの人が困っている模様。cf. [PackageManager#getLaunchIntentForPackage() returns null #747](https://github.com/robolectric/robolectric/issues/747) ←これによると2.2の頃から。3.4から`PackageManager`周りは`RobolectricPackageManager`がdeprecatedになって他と同じように`ShadowPackageManager`を使えと[Migrating from 3.3 to 3.4](http://robolectric.org/migrating/#migrating-from-33-to-34)にはあるが、`shadowOf(RuntimeEnvironment.application.getPackageManager());`としても、versionを3.3に落として`RuntimeEnvironment.setRobolectricPackageManager(packageManager);`としても、testにおける`ApplicationPackageManager#getLaunchIntentForPackage`は`null`を返す。仕方なく、`ShadowApplicationPackageManager`をextendsしてcustom PackageManagerを作ってみても、何を`@implements`したらいいのか。`PackageManager.class`では効かないし(抽象クラス?なのでそれは仕方ないのだろう)、`android.app.ApplicationPackageManager.class`では何故か名前解決に失敗してcompile出来ない。[RobolectricPackageManagerTest.java](https://github.com/robolectric/robolectric/commit/5e082743821857f057ab45945e838d5ef6b69e37)を見ると、`notNullValue()`でassert出来そうなのだが、うまく行かなかった(Step Overしてやってみても、そもそも`ShadowApplicationPackageManager`ではなく`android.app.ApplicationPackageManager#getLaunchIntentForPackage`が何かにmethod callを取られて空で返している感じ)。色々探して結局諦めた。ホントは、↓というようにやりたかったのだが。
```
dialog.getButton(DialogInterface.BUTTON_POSITIVE).performClick();
assertThat(shadowOf(activity).getNextStartedActivity().getAction(),
               is("jp.ideacross.allcardia.main.SplashActivity"));
```


## Robospock -> ElectricSpock or Spock for Android

せめてresourceの場所なりと。

* [RoboSpock](http://robospock.github.io/RoboSpock/)ですがちょっと更新が鈍いということで[ElectricSpock](https://github.com/hkhc/electricspock)。但し新しい分情報少なし
* [Spock for Android](https://github.com/AndrewReitz/android-spock)もあり
* どちらも、directory structureがstandardでないとならない様子(要するに`app/src/main/java/...`にsourceがあり`app/src/test/groovy/...`にSpock Testcodeがある)。`build.gradle`での`android.sourceSets.test.setRoot(...)`は効かないようだった
* 要は、`buildscript.dependencies`で`classpath 'org.codehaus.groovy:groovy-android-gradle-plugin:1.2.0'`を指定、`apply plugin: 'com.android.application'`と`apply plugin: 'groovyx.android'`を指定、`dependencies`に`testCompile 'org.spockframework:spock-core:1.0-groovy-2.4'`を指定すれば素のSpock、`testCompile 'com.github.hkhc:electricspock:0.6'`ならElectricSpock、`androidTestCompile 'com.andrewreitz:spock-android:2.0'`ならSpock for Android(←これだけ`androidTestCompile`なのに注意)

という感じでしょうか。

## Robolectric3 + RxJava(RxAndroid)1 + Retrofit2
RxJava + Retrofitなんて鉄板だからRobolectricによるtestなんてすぐ見つかると思ってたんですが、意外に手こずりました。要は、

* Retrofit2に対しては[MockWebServer](https://github.com/square/okhttp/tree/master/mockwebserver)([OkHttp3 の MockWebServer を使う](https://qiita.com/toastkidjp/items/4986caee5d776a4c9e6c))
* RxJavaに対しては`RxJavaHooks`([RxJava のテスト(2): RxJavaHooks, RxAndroidPlugins](http://hydrakecat.hatenablog.jp/entry/2016/12/14/RxJava_のテスト(2)%3A_RxJavaHooks%2C_RxAndroidPlugins))
* `MockWebServer`は、例にあるように基本`new`して`MockResponse`を`enqueue`して`url(...)`すればstartしてreturn valueにURL(`http://localhost:XXXXX/`←random port number)が入っているのでそれをRetrofitに食わせればいいのだけれども、URLをsetする部分はShadowの中なので、test classから直接食わせられず。なので固定port番号を使いたく、その場合`server.url("/...");`は不要で、
```server.start(portNumber);```
でおk
* ↑`http`になると`isCleartextTrafficPermitted()`まわりで失敗するようになった。これは、[`isCleartextTrafficPermitted()` fails on OpenJDK 8 + Robolectric #2533](https://github.com/square/okhttp/issues/2533#issuecomment-223093100)にあるように、`NetworkSecurityPolicy`をShadowしてやればよい。
* RxJavaの`onNext`や`onCompleted`が実行されない問題は、`Robolectric.flushBackgroundThreadScheduler();`ではなく、
```RxJavaHooks.setOnNewThreadScheduler(s -> Schedulers.immediate());```
によって別threadじゃなくmain threadで実行するようにすればおk
* 上記の話は、`Retrofit2`のService interfaceで`Observable<...>`を返す場合のもの。`Call<...>`を返す形にして`enqueue()`して`Callback<...>`で`onResponse()`、`onFailure()`でhandleする場合には、こうは行かなかった(`onResponse()`も`onFailure()`も実行されない)。`ShadowLooper.runUiThreadTasks()`でうまく行くようなことを書いてある情報([Testing retrofit 2 with robolectric, callbacks not being called](https://stackoverflow.com/questions/37909276/testing-retrofit-2-with-robolectric-callbacks-not-being-called))もあったが、症状変わらず。[OkHttpのMockWebServerとRobolectricでFragmentの動作をテストする](https://qiita.com/noboru_i/items/5eeb8b8d5684622aee95)にRetrofit2内で使っている`OkHttpClient.Builder#newBuilder`をshadowしてうまく行く話があったので、試すと確かに`onResponse()`が呼ばれた! ただ、今回ぼくは実classの方で`new Retrofit().newBuilder().client(new OkHttpClient().newBuilder().build())`とかって`client`methodを使っておらずdefaultで裏でimplicitlyに生成される`OkHttpClient`そのまま使っており、それだと`newBuilder()`呼ばれないので、色々辿ってった挙句、`okhttp3.Dispatcher#executorService`をshadowして、前述のpageにあったようにすぐ`command.run()`する`execute`methodを持つ`AbstractExecutorService`classを返してやると、うまく行った。`Dispatcher#executorService`って`java.util.concurrent.ThreadPoolExecutor`をdefaultでは使っており、Androidのthreadとは違うから、uncontrollableだったんですね。考えてみるに、RxAndroidと違いRetrofitはAndroid専用ではないので、`java.util.concurrent`の`Executor`使ってるのも当然ですか。

## AccountManager with Robolectric(というかMockito)
* 基本的には、`AccountManager.get(Context)`はJUnit Test内でもtarget class内でも同じobjectを返すので、そのままassertion可能
* ただ、例えば`manager.blockingGetAuthToken(...)`でExceptionを起こさせたい時は、`AccountManager manager = spy(AccountManager.get(application));`した`manager`を`getSystemService(Context.ACCOUNT_SERVICE)`で`doReturn`するようにした`Application`を`spy`して、その`application`を`RuntimeEnvironment.application`の代わりにねじ込む必要がある([Using mockito to mock AccountManager](https://stackoverflow.com/questions/26937001/using-mockito-to-mock-accountmanager))。具体的には、
```
@Rule
public ExpectedException thrown = ExpectedException.none();
  :
Account account = new Account("any name", CarCloudAuthUtil.ACCOUNT_TYPE);
Application application = spy(RuntimeEnvironment.application);
util = new CarCloudAuthUtil(application);
AccountManager manager = spy(AccountManager.get(application));
doReturn(manager)
        .when(application)
        .getSystemService(Context.ACCOUNT_SERVICE);
manager.addAccountExplicitly(account, "any key", new Bundle());
manager.setAuthToken(account, CarCloudAuthUtil.AUTH_TOKEN_TYPE, "any string");
doThrow(AuthenticationException.class)
        .when(manager)
        .blockingGetAuthToken(eq(account), eq(CarCloudAuthUtil.AUTH_TOKEN_TYPE), eq(true));
thrown.expect(AuthenticationException.class);
thrown.expectMessage(new IsNull());
```

* `Exception`のassertionは、`@Test(expected=...)`でも良いが、`@Rule`でも書ける([JUnitでの例外テストの書き方](https://qiita.com/su-kun1899/items/5c9f0294a7de1986e542#ruleを使った書き方))。その場合、`Exception#message`が`null`の場合のassertionは`org.hamcrest.core.IsNull`を用いて`thrown.expectMessage(new IsNull());`とする([ExpectedException.expectMessage((String) null) is not working](https://stackoverflow.com/questions/35199026/expectedexception-expectmessagestring-null-is-not-working))。
* mocking method実行時に他のことをしたい時には、`when(mock.methodCall()).thenAnswer(m -> {...});`とlambdaで書ける([mockitoとJMockitについてのメモ](https://qiita.com/kazurof/items/1171c7e038050453c6c9#mockitoでのサンプル))。
