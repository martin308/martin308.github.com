---
layout: post
title: "File encoding of your package.json file"
date: 2013-09-22 11:33
comments: true
tags: [coding, javascript, grunt]
---

If you are using [NPM](https://npmjs.org/) or [Grunt](http://gruntjs.com) on Windows and are having issues running your tasks it would pay to check the file encoding on your package.json file. I was receiving the following error while trying to run Grunt:

```
"Warning: C:\dev\project\package.json : Unexpected token ? Use --force to continue."
```

The issue happened to be that VisualStudio had sneakily saved the file as UTF8-BOM, changing the encoding to UTF8 without the BOM solved this. I found the NPM [bug](https://github.com/isaacs/npm/issues/3358) which has been closed without a fix. Hopefully this will allow you to solve this quicker than I did!
