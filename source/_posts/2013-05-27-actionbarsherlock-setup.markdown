---
layout: post
title: "ActionBarSherlock使用"
date: 2013-05-27 21:50
comments: true
categories: 
---

从Android 3.0(API Level 11)开始,Android加入了一个新的API, ActionBar.但目前市面上还有好多的Android 2.3(API Level 8),并不支持ActionBar,还好有人开发了ActionBarSherlock可以让2.x的系统也能使用ActionBar,而且它有一个特点,如果你在3.0以上的机子上使用,那么它会调用系统原生的ActionBar.这里简单记录一下设置过程.

首先从[官方网站](http://actionbarsherlock.com/)上下载压缩包

解开里面会有一个actionbarsherlock的目录,在eclipse中导入这个项目(eclipse中file -> new -> other -> android类下的android project from existing code,选择解压开的actionbarsherlock目录)

导入之后会多了一个名为actionbarsherlock的项目

然后在你要使用ActionBarSherlock的项目名字上右键选择properties,弹出的窗口中点击左边的Android标签,然后点击add选择刚才导入的项目

完成后就可以在项目中使用ActionBarSherlock了
