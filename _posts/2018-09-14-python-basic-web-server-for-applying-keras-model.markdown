---
layout: post
title: "Python Basic Web Server for Applying keras Model"
date: "2018-09-14 16:56"
author: 'u-ryo'
categories: [python, keras, Machine Learning]
comments: true
published: true
---
VGG16をfine tuningしたmodelを簡単にapply(predict)するsystemが
NotePC程度で動くことを証明したく、
PythonのBaseHTTPServerで1日でちゃちゃっと作りました。

といっても「PythonのBaseHTTPServer」なんて触るの初めてです。
[bradmontgomery/dummy-web-server.py](https://gist.github.com/bradmontgomery/2219997)を参考にしました。
なるほど、classを定義して、`BaseHTTPRequestHandler`を引数にして、
`do_GET`や`do_POST`を書くわけですね。
routingは自分でやれと。面倒なのでmethodで分けました。
このsystem自体をone fileで済ませたかったので、
`GET`で`index.html`を返したいと思い、
そのためにはhere documentを書きたいな、
と思ったので、
[Pythonのヒアドキュメント](https://qiita.com/ykhirao/items/c7cba73a3a563be5eac6)を参考に、
あぁ、普通に`'''`(single quote 3つ)でいいのね、とわかりました。
single quoteでも中で変数展開されるんですね。へー。
[PythonでJSONを受けて処理をする](http://d.hatena.ne.jp/matasaburou/20151003/1443882557)を参考に、
`POST`時のparameter(=request body)の受け取り方を。
HTTP Request Headerの`Content-Length`を`int`として記憶し、
`self.rfile.read()`でそのbyte分読んで、
`decode()`でstringにすると。
parameterizedされるわけではなく、bodyがまるっと読まれるので、
parameterで分けるのは自分でやれということですね。

あと、base64として来た文字列をdecodeして`Image`化するには、
先頭の`image/jpeg;base64,`を除去し(`replace()`)て純粋にBase64文字列だけにし、
`base64.b64decode(content)`したものを`io.BytesIO()`し、
それを`Image.open()`すると。

`predict()`したものは2次元配列になっているので、
0番目の`argmax()`したものが答えの添字、なのですね。

ともあれ、このpython predict server script 1つと
keras model fileの2 filesだけでサクッと動くものを、
[github](https://github.com/u-ryo/python_prediction_server)に置きました。
