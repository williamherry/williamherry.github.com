---
layout: post
title: "Ruby/Rails小技巧收集"
date: 2013-07-29 11:43
comments: true
categories: 
---

收集一些当时做的时候花了好多时间去Google,但本身又很短的小问题的答案,大部分来算StackOverflow

### jQuery on的用法

on的基本用法为:

```
$(selector1).on 'event', 'selector2', ->
  ...
```

我遇到的问题是对通过ajax加载的部分on不起作用,解决办法是确保selector1不是通过ajax加载的
