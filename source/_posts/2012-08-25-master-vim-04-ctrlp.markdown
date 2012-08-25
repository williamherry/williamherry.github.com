---
layout: post
title: "VIM练级日记04-文件搜索CtrlP"
date: 2012-08-25 15:36
comments: true
categories:
---

## CtrlP简介

你可能觉得前面介绍的NERDTree外挂非常好用,但当你过用今天要介绍的这个外挂后,可能NERDTree你就会很少用了

简单的说,CtrlP可以让你以搜索的方式打开文件,你可以输入完整的文件名,或者文件名的一部分,或者路径的一部分,它会帮你自动匹配,如果唯一,就可以回车打开了

看一下效果(可以在图片上右键`view image`看高清大图)

{% img /images/vim/vim-ctrlp.png %}

## CtrlP安装

使用Vundle管理VIM外挂,安装就非常简单了,打开`~/.vimrc`文件加入一行

```
Bundle 'kien/ctrlp.vim'
```

保存退出VIM,在命令行执行:

```
vim +BundleInstall +qa
```

安装已经完成了,可以做一些配置(来自eddie-vim: https://github.cim/kaochenlong/eddie-vim)

{% codeblock vi .vim/plugin/settings/CtrlP.vim %}
noremap <C-W><C-U> :CtrlPMRU<CR>
nnoremap <C-W>u :CtrlPMRU<CR>

let g:ctrlp_custom_ignore = '\.git$\|\.hg$\|\.svn$\|.rvm$'
let g:ctrlp_working_path_mode=0
let g:ctrlp_match_window_bottom=1
let g:ctrlp_max_height=15
let g:ctrlp_match_window_reversed=0
let g:ctrlp_mruf_max=500
let g:ctrlp_follow_symlinks=1
{% endcodeblock %}

## CtrlP使用

使用就非常简单了,打开VIM,按`Ctrl` + `P`, 输入想要打开的文件名,或者文件名的一部分,或者路径,自己试试吧


