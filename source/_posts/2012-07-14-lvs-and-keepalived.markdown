---
layout: post
title: "LVS和KEEPALIVED实现高可用负载均衡"
date: 2012-07-14 21:16
comments: true
categories:
---

## 背景

利用LVS和Keepalived实现高可用负载均衡已经不是什么新鲜玩意了,网上的资料的也非常多,但大多都是转载的NetSeek大侠的博文,我学习的过程中也遇到一些困难,所以就有了写一篇自己的文档的想法.本文档没有太多自己的东西,只是将网上的资料做一些整理的工作

本文档主要包括以下内容

- RSYNC和SERSYNC实现后端服务器内容同步
- LVS实现负载均衡
- KEEPALIVED实现高可用

<!-- more -->

基本的拓扑图(来自互联网)

{% img /images/lvs/lvs-and-keepalived.png %}

上面是两台负载均衡调度器(Load Balancer)利用KEEPALIVED实现高可用,下面是4台服务器(Real Server),本文档使用自底向上的方法,先介绍Real Server内容同步,然后是LVS调度(负载均衡),最后加上KEEPALIVED实现高可用

## RSYNC和SERSYNC实现后端服务器内容同步

可以利用RSYNC和SERSYNC实现后端服务器内容同步,为了能说的更清楚,分开介绍它们

### RSYNC

参考资料: [RSYNC架设手册](http://wenku.baidu.com/view/55325cfafab069dc50220159.html)

rsync是一个快速增量文件传输工具,它可以用于在同一主机备份内部的备分,我们还可以把它作为不同主机网络备份工具之用.相对tar和wget来说,rsync也有其自身的优点,比如速度快,安全,高效.

rsync可以提供C/S的模式,rsync服务器可以监视一个或多个目录,并打开873等待客户端来连接,客户端连到服务器既可以将服务器上的某个目录同步到本地,也可以将本地某个目录同步到服务器

#### RSYNC的安装

现在Linux各大发行版都提供这个软件包,安装十分简单,如Redhat系统发行版可使用下面命令进行安装

```
yum -y install rsync
```

#### RSYNC服务器器配置文件

我们可以根据自己的喜好创建RSYNC的配置文件(可以在启动的时候指定配置文件的位置),例如:

```
mkdir /etc/rsync
touch /etc/rsync/rsync.conf
```

rsync.conf是rsync服务器主要配置文件,我们来个简单的示例,比如我们想要备份/data这个目录

{% codeblock vim /etc/rsync/rsync.conf lang:bash %}
port = 873
uid = root
gid = root
use chroot = yes
read only = no

max connections = 5
timeout = 300

log file = /var/log/rsync.log
pid file = /var/run/rsync.pid

[data]
path = /data
list=yes
ignore errors = yes

hosts allow = 192.168.1.0/255.255.255.0 10.0.1.0/255.255.255.0
hosts deny = *
{% endcodeblock %}

#### 启动rsync服务器

启动rsync服务器非常简单,–daemon是让rsync以服务器模式运行

```
rsync --daemon --config=/etc/rsync/rsync.conf
```

可以利用netstat查看873端口确定rsync有没启动

#### 通过rsync客户端来同步数据

首先,我们可以看看rsync服务器上提供了哪些可用的数据(在另外的机器上),这里rsync服务器的IP为192.168.1.3

```
rsync --list-only 192.168.1.3::
```

可以列出data下的文件

```
rsync --list-only 192.168.1.3::data
```

如果要将服务器上的/data目录的文件同步到本地(保存到data_local目录下),可以使用以下命令

```
rsync -avzP 192.168.1.3::data datata_local

# here is output
receiving incremental file list
./
1.txt
           0 100%    0.00kB/s    0:00:00 (xfer#1, to-check=0/2)

sent 48 bytes  received 106 bytes  308.00 bytes/sec
total size is 0  speedup is 0.00
```

如果要将本地的一个目录同步到rsync服务器去,可以使用以下命令

```
# 从本地同步到服å¡器需要服务器配置文件里read only = no
rsync -avzP data_local 192.168.1.3::data

# here is output
data/
data/1.txt
           0 100%    0.00kB/s    0:00:00 (xfer#1, to-check=1/3)
data_local/2.txt
           0 100%    0.00kB/s    0:00:00 (xfer#2, to-check=0/3)

sent 139 bytes  received 50 bytes  378.00 bytes/sec
total size is 0  speedup is 0.00
```

RSYNC已经差不多了,下面我们看看SERSYNC

### SERSYNC

SERSYNC可以用来监控一个目录,当目录里有文件变化时调用RSYNC将变化同步到设置好的RSYNC服务器上去

#### SERSYNC原理

使用Linux2.6内核的inotify监控Linux文件系统事件，被监听目录下如果有文件发生修改,sersync将通过内核自动捕获到事件,并将该文件利用rsync同步到多台远程服务器.sersync 仅仅同步发生增、删、改事件的单个文件或目录,不像rsync镜像同步那样需要比对双方服务器整个目录下数千万的文件,并且支持多线程同步,因此效率非常高.
安装RESYNC

到这里可以下载最新的二进制安装包,现在最新的版本为2.5

```
cd /user/local/src
wget http://sersync.googlecode.com/files/sersync2.5.4_64bit_binary_stable_final.tar.gz
tar xf sersync2.5*
mv GNU-Linux-x86 /usr/local/sersync
```

这样SERSYNC就安装完成了,下面介绍配置和使用

#### 配置SERSYNC

按照前面的安装路径,SERSYNC的配置文件会是/usr/local/sersync/confxml.xml,我们只需要做简单的修改

{% codeblock vim /usr/local/sersync/confxml.xml %}
...
        <localpath watch="/var/www/html">
            <remote ip="192.168.1.3" name="data"/>
        </localpath>
...
{% endcodeblock %}

这样设置,当启动sersync服务器的时候,它就会监视/var/www/html这个目录,当里面的变化时它会利用rsync去连接192.168.1.3上的rsync服务器并将变化同步到data定义的目录

#### 启动SERSYNC服务

在开启实时监控之前可以对监控目录与远程目标机目录进行一次整体同步

```
/usr/local/sersync/sersync2 -r --config=/usr/local/sersync/confxmlonfxml.xml

# here is output
set the system param
execute：echo 50000000 > /proc/sys/fs/inotify/max_user_watches
execute：echo 327679 > /proc/sys/fs/inotify/max_queued_events
parse the command param
option: -r    rsync all the local files to the remote servers before the sersync work
daemon thread num: 10
parse xml config file
host ip : localhost   host port: 8008
config xml parse success
please set /etc/rsyncd.conf max connections=0 Manually
sersync working thread 12  = 1(primary thread) + 1(fail retry thread) + 10(daemon sub threads)
Max threads numbers is: 22 = 12(Thread pool nums) + 10(Sub threads)
please according your cpu ，use -n param to adjust the cpu rate
------------------------------------------
rsync the directory recursivly to the remote servers once
working please wait...
execute command: cd /var/www/html && rsync -artuz -R --delete ./ 192.168.1.3::data >/dev/null 2>&1
run the sersync:
watch path is: /var/www/html
```

现在可以在/var/www/html下创建文件或目录,看会不会同步到远程rsync服务器

可以加-d参数让sersync在后台运行

```
/usr/local/sersyncersync/sersync2 -d --config=/usr/local/sersync/confxml.xml
```

现在你已经知道RSYNC和SERSYNC如何工作了,怎么用在自己的环境中就看你怎么方便了,这里是我想到一个应用:假设我有三台Real Server rl1(192.168.1.11), rl2(192.168.1.12), rl3(192.168.1.13),我希望只修改rl1,rl2和rl3会自动同步,并且在一台服务器backup(192.168.1.14)上做备份,rl2和rl3的配置可能有点像这样

{% codeblock vim /etc/rsync/rsync.conf %}
...
[html]
path = /var/www/html
list = yes
ignore errors = yes

hosts allow = 192.168.1.11
hosts deny = *
...
{% endcodeblock %}

backup服务器的配置可能类似这样(/backup目录需要手动创建)

{% codeblock vim /etc/rsync/rsync.conf lang:bash %}
...
[backup]
path = /backup
list = yes
ignore  errors = yes

hosts allow = 192.168.1.11
hosts deny = *
...
{% endcodeblock %}

rl1的配置可能类似这样(rl1上安装sersync,有变化时推送到rl2,rl3和backup)

{% codeblock vim /usr/local/sersync/confxml.xml %}
...
          <localpath watch="/var/www/html">
            <remote ip="192.168.1.12" name="html"/>
            <remote ip="192.168.1.13" name="html"/>
            <remote ip="192.168.1.14" name="backup"/>
          </localpath>
...
{% endcodeblock %}

注: 如果不能正常工作,请查看SELINUX,IPTABLES,和配置文件里hosts allow的配置

## LVS实现负载均衡

参考资料1: [James的LVS手记](http://ishare.iask.sina.com.cn/f/25384267.html)
参考资料2: [lvs_tutorial](http://ishare.iask.sina.com.cn/f/14622189.html)

LVS(Linux Virtual Server: Linux虚拟服务器)是一个项目,框架,概念,IPVS(IP Virtual Server)是LVS的一种实现,ipvsadm是IPVS管理器,它是用户空间工具,我们利用它来管理IPVS,就像利用iptables管理防火墙一样

LVS提供了三种调度模式和10种调度算法,这都不是本文档的重点,这方面内容请自己找资料,这里只介绍DR模式

首先看一下DR模式的拓扑图

{% img /images/lvs/lvs-dr-tutorial.png %}

从这个图我们可以看到进来的数据是通过调度器的(Linux Director),返回的时候是从Real Server直接传给客户端的

这是James的实验图

{% img /images/lvs/lvs-dr-james.png %}

上图中为了清晰使用了两个物理网卡和两个交换机,一个也可以工作

我们将IPVS称为调度器,两个web服务器称为Real Server

DR的工作原理如下:

- 客户端发出请求数据包,通过网关到达 IPVS 主机
- Ipvs根据调度算法,从私有网络(亮黄色链接的网络)中询问被选中的,真实服务器的mac地址.客户端请求数据包被网关转发过来后,改变该组数据帧中的mac目的地址为被选中服务器的mac.再经私网(eth1)接口广播出去
- 真实服务器收到数据帧后,发现包内的目标地址在本地接口(local interface)上.根据系统中的路由规则,把所要返回给客户端的数据包从外部(eth0)接口递给网关

LVS DR模式的要点可以总结为:

调度器和Real Server都设置了VIP,但是只有调度器接收到VIP的数据包,Real Server不回应到VIP的arp请求,它只用VIP发送数据包给客户端
Real Server的设置

### 设置WEB服务器

```
yum -y install httpd
echo 'realserver1' > /var/www/html/index.html
/etc/init.d/httpd start
```

接下来我们要让Real Server拥有VIP但只用它来发送数据,可以通过以下方法做到

```
#   抑制arp
echo "1" > /proc/sys/net/ipv4/conf/lo/arp_ignore
echo "2" > /proc/sys/net/ipv4/conf/lo/arp_announce
echo "1" > /proc/sys/net/ipv4/conf/all/arp_ignore
echo "2" > /proc/sys/net/ipv4/conf/all/arp_announce
# 添加VIP
ip addr add $VIP dev lo
```

Real Server的设置就完成了,我们先不设置内容的同步以便于观察LVS调度的工作情况

多个Real Server可以使用类似的配置

### LVS调度的配置

使用DR模式的LVS所要做的工作是:当它收到访问VIP的数据包后,会从Real Server里找一台把数据包转发给它,不改变IP数据包的源地址和目标地址,通过MAC地址将数据包转给特定的Real Server

#### 添加VIP

```
ip addr add $VIP dev eth0
```

调度需要用VIP接收数据包,所以不要做隐藏的工作

#### 安装ipvsadm

对LVS的管理都是通过ipvsadm这个用户空间工具来完成了,所以我们需要安装它

```
yum -y install ipvsadm
```

#### 添加ipvs规则

假设我有三台VIP为10.0.0.10,三台Real Server的IP为192.168.1.11-13,可以这样添加规则

```
ipvsadm -A -t 10.0.0.10:80
ipvsadm -a -t 10.0.0.10:80 -r 192.168.1.11 -g
ipvsadm -a -t 10.0.0.10:80 -r 192.168.1.12 -g
ipvsadm -a -t 10.0.0.10:80 -r 192.168.1.13 -g
```

#### 开启内核转发
{% codeblock vim /etc/sysctl.conf lang:bash %}
...
net.ipvsadmpv4.ip_forward = 1
...
{% endcodeblock %}

```
sysctl -p
```

现在你差不多可以测试了,打开浏览器访问10.0.0.10并不断刷新

## KEEPALIVED实现高可用


