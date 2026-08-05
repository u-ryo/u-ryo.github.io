---
layout: post
title: "Getting the first in Spring DATA JPA"
date: "2018-08-23 17:59"
author: 'u-ryo'
categories: [spring, java, jpa]
comments: true
published: true
---
Spring DATA JPAで、「最新のもの一つ」を取得したかったんです。
Spring DATA JPAは、`findFirstBy...`とかってmethodに命名すれば
自動的にSQL作ってくれるらしいんですが([【Spring Data JPA】自動実装されるメソッドの命名ルール](https://qiita.com/sndr/items/af7d12be264c2cc4b252))、
目的のものではlogin userを自動的にparameterizeしたかったので、
それが出来ませんでした。←`User` objectは別途取得しておいて、
それをparameterに入れれば良かったかも、ですけど。
ともあれ、`findFirstBy...`で出来ないなら、
`Pageable`を付けるしかなさそうだ、ということで、
SQL文にはMySQLでいうところの`limit=1`などはつけずに引数の最後に`Pageable`を添え、
`new PageRequest(0, 1, DESC, "to")`として範囲を指定しました(`org.springframework.data.domain.Sort.Direction.DESC`)。

### 参考URLs
1. [yamkazu/springdata-jpa-example](https://github.com/yamkazu/springdata-jpa-example/blob/pageable/src/test/java/org/yamkazu/springdata/EmpRepositoryTest.java)
1. [Spring Data JPA Tutorial: Pagination](https://www.petrikainulainen.net/programming/spring-framework/spring-data-jpa-tutorial-part-seven-pagination/)
1. [setMaxResults for Spring-Data-JPA annotation?](https://stackoverflow.com/questions/9314078/setmaxresults-for-spring-data-jpa-annotation?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa)
