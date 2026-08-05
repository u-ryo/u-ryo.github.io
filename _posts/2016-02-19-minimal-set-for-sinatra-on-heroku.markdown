---
layout: post
title: "minimal set for sinatra on heroku"
date: "2016-02-19 09:18"
author: 'u-ryo'
categories: [ruby, sinatra, heroku, cloud9]
comments: true
published: true
---
heroku久しぶりだったので、やり方忘れちゃいましたよ。

Cloud9上で、

```
$ cat Gemfile
source 'https://rubygems.org'
gem 'sinatra'

$ cat Procfile
web: exec ruby app.rb -p $PORT

$ cat app.rb
require 'sinatra'

get '/' do
  'Hello World!'
end
```

最低この3ファイル、なんですが、
gitにcommitしてherokuにdeployするのは
あと自動生成される `Gemfile.lock` も必要になります。
`Gemfile.lock` は、一旦rubyを動かさないと出来ないのかな?
`ruby app.rb` で試せます。

herokuへは、

```
$ heroku create my-application
$ git init
$ git add Gemfile Gemfile.lock Procfile app.rb
$ git commit -a
$ bundle install
$ git push heroku master
```

で上手く行く筈、なんですが、
git remoteを設定しないとダメかも。
その辺は適宜。