---
layout: post
title: "glassfish small tips"
date: "2017-08-22 17:11"
author: 'u-ryo'
categories: [glassfish]
comments: true
published: true
---
ちょっとハマったところを。

- `WEB-INF/glassfish-web.xml`の`context-root`の最後が`/`だと、
ログイン後、トップページに(`http://...//`という形で)飛ばされる
- Glassfish管理GUIにおいて、`glassfish-web.xml`の`context-root`と違うcontext名でdeployした場合、
CUIでrestart(=disable/enable)すると、
`glassfish-web.xml`の`context-root`のcontext名になってしまい、
当該web applicationにaccess出来ないように見えてしまう