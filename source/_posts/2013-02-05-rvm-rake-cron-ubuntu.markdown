---
layout: post
title: "最简单的在crontab中执行rake命令的方法"
date: 2013-02-05 11:26
comments: true
categories: 
---

stackoverflow上很多都说要添加变量什么的,这是我发现的最简单而且能工作的方法,原文在[这里](http://www.wyliethomas.com/blog/2011/08/24/rvm-rake-and-cron-on-ubuntu/)

方法很简单,在crontab中可以定义环境变量

```
crontab -e
```

```
SHELL = /home/william/.rvm/bin/rvm-shell

* 4 * * * /bin/bash -l -c 'cd /path/to/app && RAILS_ENV=production rake mytask –silent'
```
