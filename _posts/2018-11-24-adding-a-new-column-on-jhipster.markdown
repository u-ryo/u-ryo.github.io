---
layout: post
title: "Adding a new column on JHipster"
date: "2018-11-24 05:16"
author: 'u-ryo'
categories: [jhipster, java, liquibase, database]
comments: true
published: true
---
JHipsterでdatabaseのtableに新しいcolumnを足したいと思いました。
Liquibaseで管理されているので、直接DBを変えちゃうと立ち上がらなくなっちゃいます。
結構面倒臭いんですよねLiquibase。

基本的には[Using JHipster in developmentのDatabase updates](https://www.jhipster.tech/development/#database-updates)に書いてある通り、です。
3通り紹介されていますが、最初に書かれている`entity sub-generator`を使った方が、testやi18n、angular側の面倒も見てくれるので楽です。
但し、generateされるjava/json/ts filesについては、面倒でも一つ一つdiffを見て上書きするか考えた方がいいです。場合によっては上書きしない方がいいcaseもありました(repository class、即ちDBを直接叩くclassとかresource class、即ちRest Controllerとか、i18nのglobal.jsonとか、独自に書いたところがあるものは特に)。

新規にtableごと加える時はこれでいいのかもしれません(以前やってうまく行った薄い記憶がありますがよく覚えてません)。
けれども、既存tableにcolumnを足す、となると、
`年月日_changelog.xml`を作ってくれないので、
いくら`gradle`かけてもDBのtableを変更してくれないんです。
更に、checksumが合わないといって起動しなくなります。
困ります。

なので、2番目の策を合わせて`changelog.xml`を作ってもらいます。
そのためには上記ページに書いてあるように、__一旦buildしてから__、
`gradle liquibaseDiffChangelog -PrunList=diffLog`すると、
class fileのdiffを取って`src/main/resources/config/liquibase/changelog/`以下にchangelogを作ってくれます。
それを`src/main/resources/config/liquibase/master.xml`に手で書き入れてからbuildすると、changelogが働いて`alter table`してくれます。
但しその際、元のtableのxml(`年月日_added_entity_テーブル名.xml`)をもとに戻しておかないと、「checksumが合わない」と言われるでしょう。

...というように、1番目と2番目の合わせ技的なところが最良のように思います。


どうしてもうまく行かない時、最後は以下の手で何とかなります。なりました。

* DBは自分で`alter table`
* checksumは`DATABASECHANGELOG.MD5CHECKSUM`にあるので自分で`update table set ...`
