---
layout: post
title: "how to build my web application product"
date: "2016-02-19 09:04"
author: 'u-ryo'
categories: [bower, npm, node, gradle]
comments: true
published: true
---
## Private memo

To build hondaPortal,
`cd` to hondaPortalView,
`bower install` and `npm install` first to set up javascript libraries.
And return to the parent directory and try `gradlew`.
If the build process encounters an error on imagemin in hondaPortalView,
I found a workaround at [Fatal error: Cannot read property 'contents' of undefined #330](https://github.com/gruntjs/grunt-contrib-imagemin/issues/330).

```
cd node_modules/grunt-contrib-imagemin
npm install imagemin@4.0.0
```