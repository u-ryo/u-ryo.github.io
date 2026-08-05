---
layout: post
title: "mock web server by WireMock on JHipster"
date: "2018-03-26 02:12"
author: 'u-ryo'
categories: [wiremock, junit, mock, web server, spring, spring boot, java, jhipster]
comments: true
published: true
---
外部web siteのresponseのtestに[WireMock](http://wiremock.org/)を使ったのでメモ。

参考: [WireMockを使って通信に関するテストをやってみよう](https://dev.classmethod.jp/etc/wiremock-practice/)

使う時は`build.gradle`で、

```
testCompile 'com.github.tomakehurst:wiremock-standalone:2.15.0'
```

して(`wiremock-standalone`でないと、
jetty系のClassNotFoundExceptionに見舞われた)から、
test classの方で、

```
import com.github.tomakehurst.wiremock.junit.WireMockRule;
import static com.github.tomakehurst.wiremock.client.WireMock.*;
    :
    @Rule
    public WireMockRule wireMockRule = new WireMockRule(8089);
```

とし、各test method内で、

```
        stubFor(post(urlPathEqualTo("/any_path"))
                .withRequestBody(matching(".*arr_stn=%96%BC%8C%C3%89%AE.*"))
                .withRequestBody(matching(".*dep_stn=%93%8C%8B%9E.*"))
                .withRequestBody(matching(".*train=1.*"))
                .willReturn(aResponse()
                            .withStatus(200)
                            .withBodyFile("for_test.html")));
```

とstatic importした
`com.github.tomakehurst.wiremock.client.WireMock.stubFor`を使えば良い。
matching rule詳細は[公式ページ](http://wiremock.org/docs/request-matching/)を。
post body requestに対して`.withQueryParam("...", matching("..."))`は効かなかった。
また、`matching`なので全体にmatchingさせる必要があることに注意。
`find`系はなし。

test対象classでのqueryがここでの指定に合わなければ、404がthrowされる。

上記のように、test用に既に用意してあるhtml fileを`.withBodyFile("...")`で指定でき、
実体は`src/test/resources/__files/`に置く。
`.withBodyFile(...)`は公式Docに無く、JavaDoc APIも無いので、
[Github上のsource code](https://github.com/tomakehurst/wiremock/blob/master/src/main/java/com/github/tomakehurst/wiremock/client/MappingBuilder.java)を眺めて見つけた。

こうしたmappingを`src/test/resources/mappings/`以下に`anyName.json`を置いて、
requestとresponseを指定できるのだが、
それだと「こういうrequest bodyの時はこのようなresponse、
こっちの時はこれこれのresponse」というように、
複数の条件を書くことが出来なかった
(書いてみると、最後のrequest/response条件しか効かず。
ならばと複数fileに分けてみたら、
file名alphabeticalで最後のrequest/responseしか効かず)。
なので、Javaでstub条件書いて各test method内に書くしか無い?


これはSpringの話だが、
testの時だけ`localhost:8089`を見るようにするには、
`〜.config.ApplicationProperties` classにfield定義、setter/getterのaccessorを付け、`src/main/resources/config/application.yml`の`application:`以下に通常時の値を、`src/test/resources/config/application.yml`の`application:`以下にtest時の値(この場合は`http://localhost:8089/any_path`)を記述。
実際に使うには、

```
    @Autowired
    private ApplicationProperties properties;
```

とDIさせて、`properties.get〜()`と普通にgetterでget。

AssertJ、method chainで書けるので、
static importが`assertThat`だけで済んでいいですね。

これはJHipsterの話?(Springかなぁ?)で、
Test Class内で`@Autowired`してるservice classのmockが必要になったら、
特に`build.grade`に記述せずとも`import org.mockito.Mock;`して
mockitoの`@Mock`が使えるんですね。
そう言えば、元々`import org.mockito.MockitoAnnotations;`されてますね。
