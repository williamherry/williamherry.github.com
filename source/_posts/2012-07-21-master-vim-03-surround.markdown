---
layout: post
title: "VIM练级日记03-配对外挂Surround"
date: 2012-07-21 22:46
comments: true
categories:
---

写程序时,尤其在C/C++程序中,有大量的配对的符号(surrounding): `(` `)` `[` `]` `{` `}` `<` `>` `'` `"`，另外标记语言(如XML)更是由无数的配对标记组成.如果能够快速地处理这些surroundings，就能大大提升编程的效率.

今天给大家介绍的这个外挂就是用来简化这些事情的,首先我们从安装开始说起

## 安装Surround

如果你已经知道怎么用Vundle管理VIM外挂,那么安装这个外挂非常简单(不知道Vundle? 请看[这里](http://williamherry.com/blog/2012/07/16/master-vim-01/))

在[Github](https://github.ocm)上搜索打到surround这个外挂(我找到的是[这个](https://github.com/tpope/vim-surround))

在`~/.vimrc`里添加一行

```
Bundle 'tpope/vim-surround'
```

打开VIM运行`:BundleInstall`,安装完成下面会有提示

重启VIM,现在你就可以使用了

## 使用Surround

可以先看一下Surround自带的帮助,在VIM中输入`:help surround`,我这里也简单的总结一下

### 添加surrounding

#### 给一个单词添加

使用命令`ysiw` + `surrounding`,例如下面的例子,想要给`William`这个单词两边添加一个单引号,只需要将水标称到这个单词中,然后输入`ysiw'`

```
Old text           Command         New text
William Herry      ysiw'           'William' Herry
```

再多几个例子

```
Old text           Command         New text
William Herry      ysiw"           "William" Herry
William Herry      ysiw*           *William* Herry
William Herry      ysiw)           (William) Herry
William Herry      ysiw(           ( William ) Herry
William Herry      ysiw<html>      <html>William</html> Herry
```

你可能已经注意到了使用`(`和`)`的不同,`{}`和`[]`也是一样的,使用左边的会多出一空格

还有就是它也支持HTML的Tag(如`<html></html>`, `<body></body>`这些

#### 给一整行添加

使用`yss` + `surrounging`,请看例子

```
Old text           Command         New text
William Herry      yss"            "William Herry"
```

其它的类似,请自己做实验验证

#### 给选中的区域添加

使用Visual模式选中后按`S` + `surrounding`,例如光标在`a`上,我按`Ctrl` + `v`再按4下`l`选中`am He`后按`S%`,就会得到下面的结果

```
Old text           Command         New text
William Herry      vllllS%         Willi%am He%rry"
```

### 修改surrounding

修改的时候是把surrounding分成两类的,一类是像`'`,`"`...这样的,修改时使用命令`cs` + `现在的surrounding` + `要修改成的surrounding`,一类是HTML的Tag,如`<div></div>`这样的,修改时使用`cst` + `要修改成的Tag`.修改时水标需要移动到surrounding内

```
Old text              Command         New text
"William" Herry       cs"'            'William' Herry
<p>William</p> Herry  cst<div>        <div>William</div> Herry
```

### 删除surrounding

删除的时候也是分成两类的,使用`ds` + `surrounging`删除一般的surrounding,`dst`删除HTML的Tag

```
Old text              Command         New text
"william" herry       ds"'            william Herry
<p>william</p> herry  dst             William Herry
```

基本的使用就这些,一些高级的我自己还没搞清楚,比如加surrounding的时候自动缩进等等

这里还有一个Youtube上的视频

{% youtube 5HF4jSyPpvs %}

### 更新

选中,更改,删除和复制引号(`'` `"` `{`....)内的内容是VIM原生支持的,不需要什么外挂,记住下面的就够了

```
ci' ci" ci( ci[ ci{ ci< - 更改这些配对标点符号中的内容
di' di" di( di[ di{ di< - 删除这些配对标点符号中的内容
yi' yi" yi( yi[ yi{ yi< - 复制这些配对标点符号中的内容
vi' vi" vi( vi[ vi{ vi< - 选中这些配对标点符号中的内容
```

如果把上面的`i`改在`a`可以连配对标点一起操作
