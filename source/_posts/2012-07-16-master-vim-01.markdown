---
layout: post
title: "VIM练级日记01-外挂总管Vundle"
date: 2012-07-16 22:02
comments: true
categories: 
---

## 背景

已经使用VIM好多年了,但从来都不去配置,最多只是在写Python的时候加个ts=4,前几天不小心看到台湾[高見龍](http://blog.eddie.com.tw)老师的VIM视频,马上被VIM加外挂(插件)后强大的功能震惊了,所以决定花一些时候把VIM了解的更深一点,并通过BLOG记录下来,也就有了VIM练级日记系列博文,主要是介绍一些好用的VIM外挂(插件)的使用,这是第一篇,介绍一个Vundle这个外挂(台湾朋友喜欢用这个词)

## 外挂总管Vundle

高老师的视频中使用的是pathogen这个外挂来管理其它VIM外挂,但我通过Google发现有人说Vundle比pathogen要好,就试了下,果然很不错的说

Vundle利用git,插件的git repo以及vim-scripts维护的GitHub repo, 自动安装, 更新和卸载插件. 它把这些繁杂的工作变得简单, 甚至, 成为一种享受.

## 安装

Vundle的github地址在[这里](https://github.com/gmarik/vundle),上面有安装和使用方法,我在这里也简单的写一下

```
git clone https://github.com/gmarik/vundle.git ~/.vim/bundle/vundle
```

## 配置

{% codeblock vim ~/.vimrc lang:vim %}
set nocompatible               " be iMproved
filetype off                   " required!

set rtp+=~/.vim/bundle/vundle/
call vundle#rc()

" let Vundle manage Vundle
" required! 
Bundle 'gmarik/vundle'

" My Bundles here:

filetype plugin indent on     " required!
"
" Brief help
" :BundleList          - list configured bundles
" :BundleInstall(!)    - install(update) bundles
" :BundleSearch(!) foo - search(or refresh cache first) for foo
" :BundleClean(!)      - confirm(or auto-approve) removal of unused bundles
"
" see :h vundle for more details or wiki for FAQ
" NOTE: comments after Bundle command are not allowed..
{% endcodeblock %}

## 使用

前面非常简单的步骤已经安装好了,那要怎么使用呢,这里以一个例子做说明,假设我们想安装NERDTree(下篇介绍)这个外挂,我的方法是:

- 在github上搜索nerdtree找到它,我找到的是: https://github.com/scrooloose/nerdtree

- 在～/.vimrc文件里添加一行:

```
Bundle 'scrooloose/nerdtree'
```

- 退出并重新打开VIM,执行:BundleInstall

新的外挂NERDTree已经安装好了,可以在~/.vim/bundle/目录下看到它,下一篇我会简单介绍它的使用

用Vundle管理外挂就是这么简单

