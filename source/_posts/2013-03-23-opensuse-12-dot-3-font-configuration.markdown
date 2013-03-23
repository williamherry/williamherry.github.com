---
layout: post
title: "openSUSE 12.3 字体美化及无线网问题"
date: 2013-03-23 02:36
comments: true
categories: 
---

今天终于把openSUSE12.3安装上了,记得上次安装openSUSE应该是好几年前吧,那时候刚开始玩Linux.记得当时选发行版的时候很纠结,老想用openSUSE但当时它还是有很多问题,尤其严重的是显示上有点问题,图形界面和字符界面不能同时和显示器边对齐,以当时的水平解决不了这个问题,就放弃而一直用的Ubuntu

今天安装上了最新的12.3,比以前进步太多了,fcitx输入法安装就能用(以前很麻烦的,尤其是环境是英文的),各种驱动自动帮你安好,进系统就可以开启3D,KDE4.10已经非常稳定了,不像以前动不动就Crash. 总的来说我非常满意,以后就用她了

不过openSUSE 12.3的默认字体还是非常难看的,需要做一些配置才可以让人看着舒服

其实根本的问题并不是没有好看的字体,而是字体的渲染不给力,使字体看起来很虚,锯齿很厉害,我从网上找的这个办法可以开启抗锯齿,分享给大家

原文地址在[这里](http://tuxperience.blogspot.com/2012/12/fix-ugly-fonts-in-kubuntu-or-kde-make.html)

```
Fix: Ugly Fonts in Kubuntu or KDE | Make your fonts crystal clear

By default, KDE Desktop and Kubuntu have a very ugly font behavior compared to GNOME or Ubuntu. But no worries. Fix is easy!

How to fix it?

1- Go to "System Settings > Application Appearance > Fonts"
2- Enable anti-ailising.
3- Set Use sub-pixel rendering to RGB and Slight.
4- Go Advanced tab of Desktop Effects and set scale method to Crisp.

To improve your fonts more, change your fonts to DejaVu and for a better console set console fonts to DejaVu Sans Mono 9.5.

Now you have very cyrstal looking fonts! Have fun!
```

我刚开始照他的做也看不到效果的,花了好长时间,发现一个很有意思的事情,就是把字体放大到16以后和缩小到8以后字体的渲染开始变得非常漂亮,后来才发现在第三步设置anti-ailising的时候点击后面的configure在弹出的窗口中有`Exclude range`正好是8和16,我是勾选了的,原来我把8到16之间的字体除过不渲染,去掉之后漂亮的字体终于出来了.现在你可以随便选择你喜欢的字体,都会有很好的效果

PS: 这些文字就是在新的openSUSE 12.3下面写的,看来以后要和Ubuntu说再见了

UPDATE: 无线网络的问题

Release Note中有说安装完无线网没有启动,需要手机重启系统,但是我发现我还遇到一个问题,有时候开机无线网络是没有连上了,点开直接是空的,我在[这里](http://forums.opensuse.org/english/get-technical-help-here/network-internet/470567-wireless-problems-opensuse-12-1-a.html)找到了解决办法,简单说就是默认他用的`dhcpd`好像不太稳定,改成`dhclient`就好了

在`/etc/sysconfig/network/dhcp`中修改下面两行

```
DHCLIENT_BIN="dhclient"
DHCLIENT_DEBUG="yes"
```

加第二行是因为有一个[bug](https://bugzilla.novell.com/show_bug.cgi?id=732910) (不过好像已经修复了)
