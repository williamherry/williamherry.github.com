---
layout: post
title: "Weechat-命令行下的IRC客户端工具"
date: 2012-08-29 20:15
comments: true
categories:
---

以前一直用XChat做为IRC的客户端工具,最近无意中发现了这个命令行下面的工具,一般这样的博客都会说XChat怎么怎么的不好,所以开始用这个新东西,但是,我还真说不出XChat那里不好了,都是非常优秀的软件,如果你喜欢尝试新东西,或者钟情于命令行,可以试试这个东西

首先看一下效果图(右键图形选择`view image`可以看到高清大图哦):

{% img /images/weechat/weechat.png %}

在Ubuntu下安装非常简单

```
sudo apt-get install weechat
```

安装完之后运行`weechat-curses`就可以打开软件了

在软件下方的输入框里输入`/connect freenode`就可连接到freenode的服务器,输入`/join #ruby`就可以加入到`#ruby`群组里了

你可能会想,能不能打开软件到自动连到freenode上,自动加入一些群组呢,这当然可以了,包括你的名字都可以配置后保存起来:

自动连接到freenode

```
/set irc.server.freenode.autoconnect on
```

设置nicks,username,realname

```
/set irc.server.freenode.nicks "nickname"
/set irc.server.freenode.username "username"
/set irc.server.freenode.realname "realname"
```

自动认证nickname

```
/set irc.server.freenode.command "/msg nickserv identify xxxxxx"
```

自动加入群组

```
/set irc.server.freenode.autojoin "#channel1,#channel2"
```

知道这些估计就差不多了,还有就是可以同时加入到多个Channel里,用`Ctrl + P`和`Ctrl + N`来切换,聊天的内容不能通过滚动鼠标滚轮来查看(可以了,看更新),需要按`PageUp`和`PageDown`

更多的高级的使用可以看它的[手册页](http://www.weechat.org/doc/)

## 更新

### 使用weechat会有连不上freenode的问题

解决办法是端口改成8001

```
/set irc.server.freenode.addresses "chat.freenode.net/8001"
```

### 加入鼠标然支持

启动鼠标支持

``` ruby
/moune enable
```

启动weechat的时候就支持鼠标

``` ruby
/set weechat.look.mouse on
```

### 屏蔽无关信息
经常会有一些`aaa join the channel, bbb leave this channel`等的信息,屏蔽的方法是

``` ruby
/filter add irc_smart *,!*weechat* irc_smart_filter *
```
