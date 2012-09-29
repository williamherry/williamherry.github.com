---
layout: post
title: "用rvm安装ruby-1.9.3"
date: 2012-08-29 19:47
comments: true
categories:
---

Octopress已经可以使用最新的ruby-1.9.3了,每次安装都会遇到一些问题,自己又记不住,每次都要Google,所以在这里简单的记录一下

首先安装curl

```
sudo apt-get -y install curl
```

安装rvm

```
curl -L https://get.rvm.io | bash -s stable
```

这时你还不能执行rvm这个命令,需要重新打开一个终端或者`source ~/.bashrc`

安装依赖包

```
rvm requirements
```

输出中会看到类似这样的一行

```
Additional Dependencies:
# For Ruby / Ruby HEAD (MRI, Rubinius, & REE), install the following:
  ruby: /usr/bin/apt-get install build-essential openssl libreadline6 libreadline6-dev curl git-core zlib1g zlib1g-dev libssl-dev libyaml-dev libsqlite3-dev sqlite3 libxml2-dev libxslt-dev autoconf libc6-dev ncurses-dev automake libtool bison subversion pkg-config
```

按提示安装这些依赖包

```
sudo apt-get install build-essential openssl libreadline6 libreadline6-dev curl git-core zlib1g zlib1g-dev libssl-dev libyaml-dev libsqlite3-dev sqlite3 libxml2-dev libxslt-dev autoconf libc6-dev ncurses-dev automake libtool bison subversion pkg-config
```

安装`python-dev`,不然`rake generate`的时候会报`Could not open library 'lib.so'`的错

```
sudo apt-get install python-dev
```

如果你要使用`tk`,安装ruby前安装`tk-dev`可以省去好多麻烦

```
sudo apt-get install tk-dev
```

如果要使用rails,安装这些可以省去好多麻烦

```
sudo apt-get install build-essential git-core curl libmysqlclient-dev nodejs
```

最后安装ruby

```
rvm install 1.9.3
```

指定默认使用1.9.3

```
source ~/.rvm/scripts/rvm
rvm use 1.9.3 --default
```

打完收工
