---
layout: post
title: "VIM练级日记02-目录外挂NERDTree"
date: 2012-07-19 19:16
comments: true
categories:
---

## NERDTree简介

上一篇日记介绍了Vundle这个外挂管家,并简单演示了如何使用它来管理NERDTree这个外挂,今天主要介绍一个NERDTree这个外挂

NERDTree是一个用于浏览文件系统的树形资源管理外挂,它可以让你像使用Windows档案总管一样在VIM中浏览文件系统并且打开文件或目录.

看一下它的效果图(可以在图片上右键选择`view image`看高清大图)

{% img /images/nerdtree/nerdtree.png %}

这里还有一个Youtube上的视频演示

{% youtube CPu9mDpSYj0 %}

## 安装使用

如果你已经配置好了Vundle这个外挂(可以参考[这里](http://williamherry.com/blog/2012/07/16/master-vim-01/),那么安装NERDTree真的是太简单了,只要在你的`～/.vimrc`文件里添加一行:

```
Bundle 'scrooloose/nerdtree'
```

重启VIM并执行:BundleInstall

使用也很简单,只要在Normal模式下执行`:NERDTree` (可以自动补全),就可以看到一个树状的目录在左边了,光标移动(当然可以用JK键)到某个文件(或目录)上回车就可以打开了.

可以给它绑定一个按键,这样就不用每次输入`:NERDTree`了,例如要绑定到`F2`上,只要在`～/.vimrc文件里加入下面一行:

```
nmap <F2> :NERDTreeToggle <CR>
```

这样在`Normal`模式下按一下`F2`就出现NERDTree了,再按一下就消失了(也可以按`q`让它消失)

这里是NERDTree中的一些按键

`o` 打开关闭文件或者目录
`t` 在标签页中打开
`T` 在后台标签页中打开
`!` 执行此文件
`p` 到上层目录
`P` 到根目录
`K` 到第一个节点
`J` 到最后一个节点
`u` 打开上层目录
`m` 显示文件系统菜单（添加、删除、移动操作）
`?` 帮助
`q` 关闭

## 问题

我遇到的问题是,由于我的家目录下文件和目录比较多,当在我的家目录下打开VIM并按`F2`调出NERDTree的时候它会出现下面的提示,按一下回车才能调出NERDTree

```
NERDTree: Please wait, caching a large dir ... DONE (124 node
Press ENTER or type command to continue
```

不知道怎么解决,虽然只在打开VIM第一次会出现这个情况,还是感觉有点不爽 :(


