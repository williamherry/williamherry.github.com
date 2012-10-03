---
layout: post
title: "安装Git服务器Gitolite"
date: 2012-10-03 18:38
comments: true
categories:
---

Gitolite的代码做了更改,网上好多安装教程都不再适用,所以在这里简单记录一下我的安装方法

首先安装必要的软件

    yum -y install git

添加用户

    useradd --system --shell /bin/bash --create-home git

设置密码

    passwd git

拷贝公钥

Gitolite管理的方式是给你一个特殊的仓库,修改,提交,推送到服务器就可以了,这个仓库只可以管理员访问,现在把管理员的公钥复制到服务器上(可以和Git服务器在同一台服务器上也可以在不同的服务器上)

    # 如果没有key先生成
    ssh-keygen
    ssh-copy-id git@git-server

安装Gitolite

现在在Git服务器上切换到git用户进行安装

    # change to git user
    su - git

    # gitolite will install here
    mkdir bin

    # authorized_keys is admin's pubkey
    mv ~/.ssh/authorized_keys ~/git.pub

    # get the source code
    git clone git://github.com/sitaramc/gitolite.git

    # install
    ~/gitolite/install -to ~/bin

    # setup
    ~/bin/gitolite setup -pk ~/git.pub

安装已经完成了,可以在以管理用户进行测试(前面运行`ssh-copy-id`的服务器)

    ssh git@chef-server

应该看到类似这样的输出:

    hello git, this is git@desktop running gitolite3 v3.04-20-g6328ec2 on git 1.7.9.5

      R W   gitolite-admin
    Connection to localhost closed.closed

可以把管理仓库克隆下来管理Git服务器

    git clone git:git-server:gitolite-admin.git

要添加用户,只要把用户的公钥复制到这个仓库里的`keydir`下并以`username.pub`命名,然后提交并推送到服务器
