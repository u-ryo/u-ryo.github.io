---
layout: post
title: "Digest Authentication and File Realm on Glassfish"
date: "2017-10-04 19:27"
author: 'u-ryo'
categories: [glassfish, java, realm, digest, authentication]
comments: true
published: true
---
客先から「192.168.1.134にdiget認証のrelmsを作成してください」と
相変わらずわけわかんない指令を受けたので、
「何かに使う『digest』認証を任意の『Realm』で設定」
と解釈して取り組みました。
一番簡単な`FileRealm`でいいっしょ、
これは4848のGUIからRealms辿って`Manage Users`ボタンからuser足せばいいし、
Digest Authは`<auth-method>DIGEST</auth-method>`で瞬殺、
とか思ってたら、ハマりました。
`index.html`と`web.xml`のみの最小構成でproof application作ったのですが、
どうしても認証できないのです。
`server.log`見ると、

```
SEC1105: A PasswordCredential was required but not provided.
```

と言われています。えー!? 何で?
試しにBASIC認証にしてみると、(紆余曲折ありましたが要するに)通ります。
だから、`keyfile`の書き方が悪いわけでは無いんですね。
勿論色々ググりました。が、なかなか出て来ません類似例。
辛うじて同じような症状があっても、Answerがついてません。
半日悩みました。
GlassfishのRealmって、
JAAS Contextを指定する必要があって、これは`.../domains/domain/config/login.conf`で規定されているんですね。
これを見ると、
fileについては`fileRealm`は一つなのに、
jdbcについては`jdbcRealm`と`jdbcDigestRealm`ってあるぢゃないですか。
をぉ、と思って、source treeを探してみると、
`com.sun.enterprise.security.auth.login.DigestLoginModule`
というのもあるんですね。
これどうやって使うんでしょうね。
使い方とかも例も、
JavaDocに書いてないしググっても出て来ませんでした。
試しに`digestRealm`なんて`login.conf`に他のを真似して、
上の`DigestLoginModule`を指定して作って使ってみたのですが、
fileから読み込むようにはなっていないようで、

```
WEB9102: Web Login Failed: com.sun.enterprise.security.auth.login.common.LoginException: Login failed: unable to instantiate LoginModule: null
```

と言われて失敗してました。
...って、これ`abstract class`だから当たり前じゃないですか恥ずかしい。
`DigestLoginModule`をextendsしてるのは`JDBCDigestLoginModule`だけだから、
file realmで使うなら`JDBCDigestLoginModule`みたいなのを自前で作らないとならないんじゃないでしょうかね。
でもそうならどっかにそう書いといてよねー。

結局、大人しく`jdbcDigestRealm`を使うように変更することで面倒を回避しました。
まぁ、それだけ世にDigest Authenticationのdemandがない、
という証左なのかなと思いました。

### minimum web.xml

```
<?xml version="1.0" encoding="UTF-8" ?>
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
                             http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
         version="3.0">
  <security-constraint>
    <web-resource-collection>
      <web-resource-name>Digest Authentication Test</web-resource-name>
      <url-pattern>/*</url-pattern>
    </web-resource-collection>
    <auth-constraint>
      <role-name>someRole</role-name>
    </auth-constraint>
  </security-constraint>
  <login-config>
    <auth-method>DIGEST</auth-method>
    <realm-name>someCreatedRealmUsingjdbcDigestRealmOnJAASContext</realm-name>
  </login-config>
  <security-role>
    <role-name>someRole</role-name>
  </security-role>
</web-app>
```

### WEB-INF/glassfish-web.xml
glassfishの場合は、以下のように`role-name`と`group-name`を結びつける`glassfish-web.xml`もないとダメでした。

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE glassfish-web-app PUBLIC "-//GlassFish.org//DTD GlassFish Application Server 3.1 Servlet 3.0//EN" 
"http://glassfish.org/dtds/glassfish-web-app_3_0-1.dtd">
<glassfish-web-app error-url="">
  <security-role-mapping>
    <role-name>user</role-name>
    <group-name>user</group-name>
  </security-role-mapping>
  <class-loader delegate="true"/>
  <jsp-config>
    <property name="keepgenerated" value="true">
      <description>.</description>
    </property>
  </jsp-config>
</glassfish-web-app>
```
