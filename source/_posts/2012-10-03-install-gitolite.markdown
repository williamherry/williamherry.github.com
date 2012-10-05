---
layout: post
title: "安装配置Git服务器Gitolite"
date: 2012-10-03 18:38
comments: true
categories:
---

Gitolite的代码做了更改,网上好多安装教程都不再适用,所以在这里简单记录一下我的安装方法

### 安装

#### 首先安装必要的软件

    yum -y install git

#### 添加用户

    useradd --system --shell /bin/bash --create-home git

#### 设置密码

    passwd git

#### 拷贝公钥

Gitolite管理的方式是给你一个特殊的仓库,修改,提交,推送到服务器就可以了,这个仓库只可以管理员访问,现在把管理员的公钥复制到服务器上(可以和Git服务器在同一台服务器上也可以在不同的服务器上)

    # 如果没有key先生成
    ssh-keygen
    ssh-copy-id git@git-server

#### 安装Gitolite

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

### 添加用户示例(jenny)

    cd gitolite-admin
    cp path/of/jenny's/pubkey keydir/jenny.pub
    git add keydir
    git commit -m "add new user jenny"
    git push

### 创建仓库示例

假如我们要创建一个名为`notes`的仓库,让刚创建的`jenny`有读写权限

    cd gitolite-admin
    vi conf/gitolite.conf

添加类似下面这内容进去

    repo notes
        RW    =   jenny

保存,提交并推送到远和服务

    git add -u
    git commit -m 'add new repo notes and assign RW to jenny'
    git push

推送的时候应该看到类似这样的信息

    Counting objects: 7, done.
    Delta compression using up to 4 threads.
    Compressing objects: 100% (3/3), done.
    Writing objects: 100% (4/4), 395 bytes, done.
    Total 4 (delta 1), reused 0 (delta 0)
    remote: Initialized empty Git repository in /home/git/repositories/notes.git/
    To git@desktop:gitolite-admin
       6de90b8..52737aa  master -> master

注意`remote`开头的一行,它已经帮你创建了这个仓库

### 通配符仓库的创建示例

通配符仓库事先不能确定名字,所以不会帮你创建,在你clone的时候才会创建

编辑`conf/gitolite.conf`文件在里面加入类似下面的内容

    repo sandbox/.+$
      C      =   jenny
      RW+C   =   jenny

注意`C  =  username`的一行必不可少,这里的`C`是指创建仓库的意思,下一行的`RW+C`中的`C`是指创建引用(branch,tag)的意思

保存后提交并推送到服务上去

    git add -u
    git commit -m 'add wildcard repo'
    git push

注意看`push`时输出的信息,应该没有创建仓库的信息

这时`jenny`克隆仓库的时候会自动创建

    # as jenny user
    git clone git@desktop:sandbox/test.git

输出应该类似这样

    Cloning into 'test'...
    Initialized empty Git repository in /home/git/repositories/sandbox/test.git/
    warning: You appear to have cloned an empty repository.

如果你的输出报这样的错

    FATAL: R any sandbox/test jenny DENIED by fallthru
    (or you mis-spelled the reponame)
    fatal: The remote end hung up unexpectedly

一般是没有`C  =  username`这一行,注意是只有`C`的一行
