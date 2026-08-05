---
layout: post
title: "Content-Type for returning attachment file"
date: "2016-05-13 6:46"
author: 'u-ryo'
categories: [cgi, download]
comments: true
published: true
---
ファイルをダウンロードさせるには、

```
print "Content-Type: application/octet-stream\nContent-Disposition: attachment; filename=$line_number$day.oud\n\n";
```