---
layout: post
title: "unit test on jhipster"
date: "2018-03-23 22:30"
author: 'u-ryo'
categories: [jhipster, junit, gradle, spring, spring boot, spring security]
comments: true
published: true
---
JHipsterでのunit testでいくつかハマったのでメモ。

* unit test実行は`gradle test`
* 単一test classを指定するには`gradle test --tests FQDN.class.name`
* test class内の`log.debug(...)`を出力するには、`build.gradle`に`test.testLogging.showStandardStreams = true`を記述し、また`src/test/resources/application.yml`に以下を記述(`src/main/resources/application-dev.yml`には記述あり)
```
logging:
    level:
        ROOT: DEBUG
        bz.mydns.walt.canmatch: DEBUG
        io.github.jhipster: DEBUG
```

* test classにおいてRESTで認証したuserを表したい時には`@WithMockUser`(`import org.springframework.security.test.context.support.WithMockUser;`)
* testされる側のREST methodでは、`java.security.Principal`を勝手に引数に加えることでlogin userを取得出来、それをtestする場合にはtest classの`MockMvc`で`restUserInfoMockMvc.perform(post("/api/user-infos")...)`などとする時に`org.springframework.security.test.web.servlet.request.SecurityMockMvcRequestPostProcessors.user`を使って`.user("foo")`とすると、認証された"foo"というuserでのaccessをmock出来るが、`org.springframework.security.core.context.SecurityContextHolder.getContext().getAuthentication()`というstatic methodで`java.security.Principal`が取れ、それに`.getName()`で名前を取得可能。その方がspringっぽいのかな。test class側では、認証accessが必要なmethodに上記の`@WithMockUser`をつければ良い。`@WithMockUser(username="foo", password="pw", roles={"USER"})`などとすれば色々指定可能だが基本的には不要

### 参照
* [WithMockUser](https://docs.spring.io/spring-security/site/docs/4.2.5.BUILD-SNAPSHOT/apidocs/org/springframework/security/test/context/support/WithMockUser.html)
* [Spring Security 使い方メモ　テスト](https://qiita.com/opengl-8080/items/eaa8f4eb9286a3df7986)(2017-08-02)
* [11. Testing Method Security](https://docs.spring.io/spring-security/site/docs/current/reference/html/test-method.html)(Spring公式doc)
* [Spring Securityを使っているWebアプリのUnitテスト](https://ishiis.net/2016/11/12/spring-boot-security-unit-test/)(2016/11/12)
