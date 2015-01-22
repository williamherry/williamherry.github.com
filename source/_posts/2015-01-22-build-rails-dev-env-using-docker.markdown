---
layout: post
title: "使用Docker搭建一个Rails开发环境"
date: 2015-01-22 14:56
comments: true
categories: docker rails devops
---

如果使用的fig的话那就太简单了,照着fig官方文档分分钟就可以搞定,但是它的方法存在一个问题,他是直接用的ruby,postgres,redis的image,整个加起来就上G了,国内的这破网速,虽然只需下载一次就好了,并且国内也有镜像加速的服务如daocloud,但实际使用发现任然无法接受.所以我的方法是使用一个基本的ubuntu镜像,其他镜像在此基础上构建而成

首先是多官方的ubuntu镜像做稍微修改,这个修改就是把源换成网易的,然后制作成一个镜像,其他的镜像都一次为基础

想看一下相关的文件:

```
# tree .
├── Dockerfile
├── docker
│   ├── gemrc
│   ├── postgres
│   │   ├── Dockerfile
│   │   ├── pg_hba.conf
│   │   ├── postgresql.conf
│   │   └── run
│   └── redis
│       ├── Dockerfile
│       └── redis.conf
└── fig.yml
```

我们使用fig来编排,所以fig.yml就是主要的入口了,内容如下:

```
db:
  build: ./docker/postgres
redis:
  build: ./docker/redis
web:
  build: .
  command: bundle exec rails s -b 0.0.0.0 -p 3000
  volumes:
    - .:/myapp
  ports:
    - "3000:3000"
  links:
    - db
    - redis
```

这里有三个部分,前面两个都是知道从什么地方找Dockerfile来build镜像(fig官方的例子这里是用的image: postgres)

第三个就是实际跑rails的image了,它指定了一个image启动后运行的命令(其他两个镜像也有启动后要执行的命令,是在Dockerfile里指定),然后有一个volumes把代码目录镜像到images是,这样修改了代码不用做操作就可以看到效果

ports指令把image里的端口映射到宿主机上,宿主机并不一定就是我们的开发机,像Mac使用boot2docker安装的docker,那么开发机就是运行docker的虚拟机了,可以用boot2docker ip得到ip地址,等之后配置完成了就可以用这个地址加3000端口查看网页了

最后的links会在web的image执行的时候(就是container)在其的/etc/hosts里加一条到其他container地址的对应,要知道每个container运行IP都是不一样的,这样就可以在web去连db和redis了,相应的database.yml里把host写成db就可以了

我们一个一个来看他们是怎么build的

```
# cat docker/postgres/Dockerfile
FROM williamherry/ubuntu
MAINTAINER William Herry <WilliamHerryChina@Gmail.com>
ENV DOCKERFILE_VERSION 0.0.1.1

RUN locale-gen en_US.UTF-8
RUN update-locale LANG=en_US.UTF-8

RUN apt-get -qqy update && DEBIAN_FRONTEND=noninteractive apt-get install -qqy postgresql-9.3 postgresql-contrib-9.3 postgresql-9.3-postgis-2.1 libpq-dev sudo

# /etc/ssl/private can't be accessed from within container for some reason
# (@andrewgodwin says it's something AUFS related)
RUN mkdir /etc/ssl/private-copy; mv /etc/ssl/private/* /etc/ssl/private-copy/; rm -r /etc/ssl/private; mv /etc/ssl/private-copy /etc/ssl/private; chmod -R 0700 /etc/ssl/private; chown -R postgres /etc/ssl/private

ADD postgresql.conf /etc/postgresql/9.3/main/postgresql.conf
ADD pg_hba.conf /etc/postgresql/9.3/main/pg_hba.conf
RUN chown postgres:postgres /etc/postgresql/9.3/main/*.conf

ADD main /var/lib/postgresql/9.3/main
RUN chown -R postgres:postgres /var/lib/postgresql/9.3/main

ADD run /usr/local/bin/run
RUN chmod +x /usr/local/bin/run

EXPOSE 5432
CMD ["/usr/local/bin/run"]
```

这个是直接抄的https://github.com/orchardup/docker-postgresql, 刚发现他github项目主页就是这个已经是Deprecated,请使用官方镜像

简单说就是安装postgresql,把配置文件改改,让访问权限宽松一点,在启动起来

```
ADD main /var/lib/postgresql/9.3/main
RUN chown -R postgres:postgres /var/lib/postgresql/9.3/main
```

上面这两行是我添加的,实现把一些初始数据导入,把postgresql数据目录的文件整个放在`docker/postgres/main`下面build镜像,就可以一启动就有seed数据了

下面看看redis的Dockerfile

```
# cat docker/redis/Dockerfile
FROM williamherry/ubuntu
MAINTAINER William Herry <WilliamHerryChina@Gmail.com>
ENV DOCKERFILE_VERSION 0.0.1

RUN apt-get update -qqy && apt-get install -qqy redis-server

ADD redis.conf /etc/redis/redis.conf
CMD ["redis-server", "/etc/redis/redis.conf"]

EXPOSE 6379
```

这个就更简单了,安装,加个配置文件,启动

配置文件主要修改了两个地方:

```
daemonize no
bind 0.0.0.0
```

似乎docker在后台运行是不行的

最后我们看看web的Dockerfile

```
# cat Dockerfile
FROM williamherry/ubuntu
MAINTAINER William Herry <WilliamHerryChina@Gmail.com>
ENV DOCKERFILE_VERSION 0.0.1

ENV RUBY_MAJOR 2.1
ENV RUBY_VERSION 2.1.5
ENV RUBY_SRC_DIR /usr/src/ruby

# Essentials
RUN apt-get update -qqy \
  && apt-get install -qqy \
    autoconf \
    build-essential \
    curl \
    git \
    imagemagick \
    libbz2-dev \
    libcurl4-openssl-dev \
    libevent-dev \
    libffi-dev \
    libglib2.0-dev \
    libjpeg-dev \
    libmagickcore-dev \
    libmagickwand-dev \
    libmysqlclient-dev \
    libncurses-dev \
    libpq-dev \
    libreadline-dev \
    libsqlite3-dev \
    libssl-dev \
    libxml2-dev \
    libxslt-dev \
    libyaml-dev \
    procps \
    zlib1g-dev \
  && rm -rf /var/lib/apt/lists/*

RUN apt-get update -qqy \
  && apt-get install -qqy locales \
  && rm -rf /var/lib/apt/lists/* \
  && localedef -i en_US -c -f UTF-8 -A /usr/share/locale/locale.alias en_US.UTF-8
ENV LANG en_US.utf8

# Ruby specifics
RUN apt-get update -qqy \
  && apt-get install -qqy \
    bison \
    ruby \
  && rm -rf /var/lib/apt/lists/* \
  && mkdir -p $RUBY_SRC_DIR \
  && curl -s -SL "http://ruby.taobao.org/mirrors/ruby/2.1/ruby-$RUBY_VERSION.tar.bz2" \
    | tar -xjC $RUBY_SRC_DIR --strip-components=1

WORKDIR $RUBY_SRC_DIR

RUN autoconf \
  && ./configure \
    --disable-install-doc \
  && make -j"$(nproc)" \
  && apt-get purge -qqy --auto-remove bison ruby \
  && make install \
  && rm -r /usr/src/ruby

RUN mkdir /myapp
WORKDIR /myapp
ADD . /myapp
ADD config/database.yml.fig /neo_lion/config/database.yml
ADD config/secrets.yml.fig /neo_lion/config/secrets.yml
ADD docker/gemrc /root/.gemrc
RUN gem install bundler
RUN bundle install
```

前面那部分是网上找的安装ruby的配制方法,你可能不能理解为什么要自己安装ruby,你可以直接用官方ruby镜像,只要把FROM后面改成ruby就可以了,就是比较大而已,我发现下载安装好ruby的镜像还没从ubuntu镜像安装来的快

设置好之后就可以用下面的命令启动开发环境了(如果有数据放docker/postgres/main下,fig up就够了)

```
fig up -d db
fig run web rake db:setup
fig up
```

从mkdir开始才是正在起作用的,太简单就不用说了,这里的.gemrc里知道了taobao的源,这样安装bundler就会快一点

最后说说踩过的一个坑,你可能注意到了每一个Dockerfile前面都有`ENV DOCKERFILE_VERSION 0.0.1`这一行,刚开始是`ENV VERSION 0.0.1`,这样会有一个问题,就是在执行migration的时候干脆不跑了,花了我一天的时间debug,最后发现他会把这个变量传进去,`rake db:migrate`就相当于`rake db:migrate VERSION=0.0.1`


