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

### simple_form和bootstrap一起使用的问题

simple_form和bootstrap一起使用的时候我遇到的问题是simple_form生成的html没有加进去bootstrap的Class,看不到bootstrap样式加到table的效果,解决办法是在Gemfile里给simple_form指定版本,Rails 4是

```
gem 'simple_form', "~> 3.0.0.rc"
```

Rails 3是

```
gem 'simple_form, "~> 2.1.0"
```

### Rails 4 要使用remote: true的方法执行js需要在action里加上`render layout: false`
