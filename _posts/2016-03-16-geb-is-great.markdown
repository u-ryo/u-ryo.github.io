---
layout: post
title: "geb is great"
date: "2016-03-16 02:13"
author: 'u-ryo'
categories: [geb, groovy, test, selenium]
comments: true
published: true
---
[Geb](http://www.gebish.org/)([jeb])が凄いです。
要はSeleniumのwrapperなんですが、
Page Modelとか駆使すると、
GUI testもspockのSpec本体には具体的なtag名出さずにlogicalに書けますね。
frameなpageでは、

```
<frameset>
  <frame id="head">...</frame>
  <frame id="main">...</frame>
</frameset>
```

とあったら、Page Objectに

```
static content = {
  header(page: HeaderPage) { $('#head') }
  main(page: MainPage) { $('#main') }
}
```

と書いて、

```
withFrame(main) {
  ...
}
```

と使えます。

しかし、
そもそもイマドキframeなんか使う方がおかしいんですけどね。
html5ではaccessibilityの観点から廃止されてるのに。

最近よくある、mouse overしてmenuをdrop downさせるものについては、

```
interact {
  moveToElement(...)
  click(...)
}
```

`<select>`やradio buttonsといったform関係要素については、
以下のようにmoduleを使って簡単に操れます。

```
import geb.module.RadioButtons
import geb.module.Select
import geb.module.Textarea
$('select#...').module(Select).selected = 'some value'
$('select#...').module(Select).selectedText == 'label text'
$('select#...').module(RadioButtons) = 'some value'
$('select#...').module(RadioButtons).checked == 'some value'
$('select#...').module(Textarea).text = 'A sentence with\na new line.'
```

`withFrame`でのassertionは、末尾に`|| true`が必要でした。

```
withFrame(main) {
  leaveRequest.approver.selectedText == '西　部長'
  leaveRequest.approvalDate.text == new Date().format('yyyy/MM/dd')
} || true
```

frame内の変化があるので、Page Module内要素は`required: false`つけまくりでした。

```
static content = {
  requestType(wait: true, required: false) { $('input#rdo12').module(RadioButtons) }
  requester(required: false) { $('span#select2-chosen-1') }
}
```

あと適宜`waitFor {...}`が必要ですね。

browser alertの処理は厄介でした。
`driver.switchTo().alert().accept()`かと思ったら、`withAlert(wait: true){...}`でいいんですか。
普通の(javascriptによる)popup dialogに対しても。
「browser alertが出たら」ってどう書くんでしょう。

```
withAlert(wait: true) {
  ...
} == 'modal dialog message'
```

まぁ、[geb manual](http://www.gebish.org/manual/current/)を読めってことですね。

これはgroovyのことですが、日付のformatが簡単です。

```
leaveRequest.approvalDate.text == new Date().format('yyyy/MM/dd')
```

http accessはHttpURLClientを用い、またcookieの取得も簡単です。

```
@Grab('org.codehaus.groovy.modules.http-builder:http-builder')
import groovyx.net.http.HttpURLClient
def jsessionId = driver.manage().getCookieNamed("JSESSIONID").value
def http = new HttpURLClient(url: 'https://my-ocr.herokuapp.com/')
tryLogin(userId, password, http.request(path: jsessionId).data)
```