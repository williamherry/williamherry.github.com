---
layout: post
title: "Chef基础知识"
date: 2012-07-16 18:57
comments: true
categories: 
---

## 主要内容
本文档主要讲述了一些Chef的基础知识,包括:

- Chef是什么
- Chef能做什么
- Chef怎么做


## Chef是什么
### Chef简介

Chef是，由Ruby开发的服务器的构成管理工具

想像一下我们现在需要搭建一台mysql database slave服务器，安装过程我们手动操作了 没过多久，我们需要第二台，这时候我们会想，如果之后安装第一台的时候把操作过程执行的命令写成脚本 现在安装第二台，运行一下脚本就行了，节约时间而且不容易出错

<!-- more -->

chef就相当于这样的一个脚本管理工具，但功能要强大得多，可定制性强 chef将脚本命令代码化，定制时只需要修改代码，安装的过程就是执行代码的过程

打个比方,Chef就像一个制作玩具的工厂,它可以把一些原材料做成漂亮的玩具, 它有一些模板,你把原材料放进去,选择一个模板(比如怪物史莱克),它就会制造出这个玩具 服务器的配置也是这样,一台还没有配置的服务器,你给它指定一个模板(role或recipe), Chef就会把它配置成你想要的线上服务器

这个只是Chef的一方面,因为可以安装好系统后执行一个脚本也达到同样的目的,Chef还有另一方面 是脚本达不到的,那就是Chef对经过的配置的服务器有远程控制的能力,它可以随时对系统进行进一步的配置或修改 就像前面的玩具工厂可以随时改变它的玩具的颜色,大小(好像现实中不太可能) 你也可以通过手动的方式达到目的, 但是当服务器比较多的时候,可能手动的方法就不是那么欢乐了

Chef提供了更方便统一的方法

### Chef和Puppet比较

在系统的世界中，基本上不会说解决课题的工具只有1个, 各自工具都有自己的长处和短处，所以可能的话要选择最合适的那个

服务器的构成管理工具而言，知名度与Chef平分天下的是叫“Puppet”的工具， 这两个都作为OSS是知名度排名最前的2个。让我们来比较下它们的不同处：

注: 下面的表格来自用Chef来简化服务器管理

```
|------------------+------------------------------------+-----------------------------------------------------------------------|
| 比较             | Puppet                             | Chef                                                                  |
|------------------+------------------------------------+-----------------------------------------------------------------------|
| 历史             | 有一些                             | 还年轻                                                                |
| 用户             | 多,有名的公司也在用                | 还比较少，但我认为有37signals在使用                                   |
| 开发的活跃度     | 中等                               | 活跃（感觉正在旺季）                                                  |
| 文档             | 多                                 | 也足够了                                                              |
| 设定文件         | 用专用的文法书写（外部DSL）        | 用Ruby书写（内部DSL）                                                 |
| 设定的构成       | 有点难懂                           | 相对容易理解,命名等很合适                                             |
| 依存关系的处理   | 运行次序是根据状况由系统端决定的。 | 好像Makefile 基本上是书写顺序。相比Puppet更具脚本风格                 |
| 必要的中间软件   | 没有                               | 服务端需要有CouchDB、RabbitMQ                                         |
| 安装             | 简单，用gem的安装就可以            | 服务端容易死。没有加入包中的话比较困难。客户端的话简单，只要gem就可以 |
| 和其他系统的协作 | 感觉基本上没有                     | 因为使用RESTful的服务API，用JSON可以取值，所以好像能做许多事          |
|------------------+------------------------------------+-----------------------------------------------------------------------|
```

### Chef结构

{% img /images/chef/chef-arch.png %}

这是Chef的结构图,对图做一点解释:

- 有一个中心服务器(运行chef-server)
  - Chef将数据存储在CouchDB数据库里面
  - RabbitMQ和chef-solo等提供搜索的功能
  - Chef还提供了个图形的用户界面(cher-server-webui)
- 可以有多个Workstation(运行knife工具对Chef进行配置)
  - Workstation上有一个pem文件,knift利用它作为认证来和chef-server通过REST API进行通信
  - Workstation将配置(利用Recipe等描述各Client应该如何配置自己)上传到服务器
  - Workstation和中心服务器可以在同一台机器
- 可以有多个Client(运行chef-server的被配置机器)
  - Client上有一个pem文件,chef-client利用它作为认证来和chef-server通过REST API进行通信
  - 当新加一个Client的时候,需要从中心服务器上拷贝validator.pem到新加的Client
  - 它利用这个pem进行注册得到自己的client.pem进行以后的认证
  - Client连到Chef服务器查看如何配置自己,然后进行自我配置

### Chef的三种管理模式

- Chef-Solo  
由一台普通电脑控制所有的服务器，不需要专设一台chef-server

- Client-Server  
所有的服务器作为chef-client，统一由chef-server进行管理，管理包括安装、配置等工作 chef-server可以自建，但安装的东西较多，由于使用solr作为全文搜索引擎，还需要安装java

- Opscode Platform   
类似于Client-Server，只是Server端不需要自建，而是采用http://www.opscode.com 提供的chef-server服务

## Chef能做什么

Chef能做什么,答案的Anything,这个实际上很好理解,只要你可以对一台服务执行命令 你就可以对这台服务做任何配置(不是有那句话嘛:Where there is a SHELL, there is a way)

这里大家可能对Chef有一些误解,由于Chef使用类似模板的方法对服务进行配置, 大家可能认识它只适合于一些配置比较类似的服务, 这里完全小看Chef了,我从他们的网站上下载过mysql的cookbook,它可以同时支持:

```
debian ubuntu centos suse fedora redhat scientific amazon freebsd windows
```

当你对Chef怎么做有一定的了解后你就不会感到惊讶了

## Chef是怎么做的

如果忽略所有的细节,Chef是这样工作的:

- 在Workstation上定义各个Client应该如何配置自己,然后将这些信息上传到中心服务器
- 每个Client连到中心服务器查看如何配置自己,然后进行自我配置

Chef为了描述Client如何配置自己定义了一些名词,对这些新东西了解有利用理解Chef工作原理

### Resource和Provider

Resource是Chef提供给你的用来描述系统的某一部分希望怎么配置(处于什么状态),请看例子

```
package "vim" do
  action :install
end
```

这就是一条Resource,它想要表达的是希望vim安装(处于安装的状态)

 - 它有一个Resource类型(package)
 - 有一个名字(vim)
 - 可能还会有一些可选的参数(这个例子里没有)
 - 有一个动作(install)(实际上描述一种状态(和Puppet里的ensure类似))

这里package是一个Resource类型,这里列出几个比较常用的Resource:

- Directory
- Execute
- File
- Group
- Package
- Script
- Service
- Template
- User

Provider的概念可能比较抽象,像上面的Resource的例子,我们之所以不关关心vim怎么被安装(apt,yum…),就是因为有Provider 也就是说Provider负责把抽象的Resource对应到实际的命令(如上面的例子可能是:yum -y install vim)

下面我会列一些Resource的例子,如果你对Resource已经有理解了,可以跳到下一节Recipe

#### Resouece示例1

```
service "ntpd" do
  action[:enable,:start]
end
```

这是一条类型为service的Resource,这描述的是:启动ntpd服务并设置成开机启动 注:虽然使用action这个词,但实际上它描述的是一种状态(它不会每次都试图去start)

#### Resource示例2

```
user "random" do
  comment "Random User"
  uid 1000
  gid "users"
  home "/home/random"
  shell "/bin/zsh" 
end
```

上面的Resource的类型为user,名字为random,然后一些参数(uid,gid,home,shell) 它描述的是:在系统上以这些参数创建random这个用户

上面的例子并没有action,这是因为'user'这个Resource的默认动作为create,可以省略

#### Resource示例3

```
file "/tmp/something" do
  owner "root"   
  group "root"   
  mode "0755"   
  action :create
  content "just test" 
end
```

这个Resource的类型为file, 名字为'/tmp/something',动作为'create',还有一些参数 它描述的是:

  - 在被管理服务器上创建文件'/tmp/something',
  - 这个文件的拥有者和拥有组都是root,
  - 这个文件的模式为0755,
  - 这个文件内容为'just test'

#### Resource示例4

```
template "/tmp/config.conf" do
  source "config.conf.erb" 
  variables(
    :config_var => node[:configs][:config_var]
  )
end
```

这是一个类型的template的Resource,它把服务器上的config.conf.erb文件传到客户机上,重命名为config.conf并做变量替换(template后面会详细的讲一下)

### Recipe

简单的说把多个Resource写到一起就是Recipe,客户端会把Recipe里面的Resouce按照顺序(重要)一条一条的应用到自身

看一个配置ntp服务的Recipe

{% codeblock vim /var/chef/cookbooks/ntp/recipes/default.rb lang:rb %}
package "ntp" do
  action [:install]
end

template "/etc/ntp.conf" do
  source "ntp.conf.erb"   variables( :ntp_server => "time.nist.gov" )
end

service "ntpd" do
  action[:enable,:start]
end
{% endcodeblock %}

将把这个Recipe加到客户的run list(后面会说到)时,它会按顺序(重要)应用这些Resouce到自身:

- 安装ntp
- 从服务器上复制配置文件
- 启动并设置开机启动ntp服务

这里顺序很重要,如果整个反过来,可能就不能正确配置ntp服务了

Recipe可以包含其它的Recipe,如这个Recipe

{% codeblock vim /var/chef/cookbooks/mysql/recipes/default.rb lang:rb %}

include_recipe "ntp::default" package "mysql-server" do
  action :install
end

service "mysql-server" do
  action :start
end
{% endcodeblock %}

当这条Recipe被加到客户服务器的run list里时,它会首先把ntp::default这条Recipe展开成一条一条的Resource应用到自身,然后再应用后面的Resource

总结一下Recipe:

- 是Resource的组合
- 按顺序应用
- 可以包含其它的Recipe

### Node和Role

把它们写在一起是因为他们都有一个run_list,而且他们的run_list都可以包含Recipe和Role

```
knife node show client1.chefdemo.com

# here is output
Node Name:   client1.chefdemo.com
Environment: _default
FQDN:        client1.chefdemo.com
IP:          192.168.122.10
Run List:    recipe[ntp::default], recipe[mysql::default]
Roles:       
Recipes:     ntp::default, mysql::default
Platform:    centos 6.2
```

```
knife role show mysql

# here is output
chef_type:            role
default_attributes:   
description:          
env_run_lists:        
json_class:           Chef::Role
name:                 mysql
override_attributes:  
run_list:            
    recipe[ntp::default]
    recipe[mysql::default]
```

Role可以用来描述一台服务器希望被配置成什么样子(配置成web服务器,mysql服务器,甚至是一个论坛)

它有一个run_list, 里面包含了要把一台服务器配置成这个样子所需要的Recipe和Role(Role可以包含Role)

Node很好理解,每一个被Chef管理的服务器(运行chef-client)就是一个Node

当一个Node上的chef-client启动的时候,它会去chef-server查到自己的run_list里包含那些Role和Recipe,然后把它们按顺序依次展开并应用到自身

Role和Node的关系是:可以把Role应用到Node,实际上这个过程只是简单的把这个Role的run_list里包含的东西(Role和Recipe)加到这个Node的run_list里(一个Node可以包含任意多个Role或Recipe)

这里举一个例子帮助理解,有两个Recipe: ntp::default和mysql::default

```
package "ntp" do
  action [:install]
end # 后面把这一条Resource简称为: 安装ntp的Resource

service "ntpd" do
  action[:enable,:start]
end # 后面把这一条Resource简称为: 启动ntp的Resource 
```

```
package "mysql-server" do
  action :install
end # 后面把这一条Resource简称为: 安装mysql-server的Resource 

service "mysql-server" do
  action :start
end # 后面把这一条Resource简称为: 启动mysql-server的Resource 
```

我们创建一个名叫ntp_and_mysql的Role并把这两个Recipe加到里面,相应的命令为

```
EDITOR=vim knife role create ntp_and_mysql
```

这条命令会用vim打开一个文件让你编辑这个role,修改成这样然后保存退出

```
{
  "override_attributes": {
  },
  "chef_type": "role",
  "env_run_lists": {
  },
  "json_class": "Chef::Role",
  "name": "ntp_and_mysql",
  "run_list": [
    "recipe[ntp::default]",
    "recipe[mysql::default]"   
  ],
  "default_attributes": {
  },
  "description": "" 
}
```

然后把这个Role应用到一个Node上(实际上就是把这个Role的runlist里的Recipe加到Node的runlist里)

```
knife node run list add client1.chefdemo.com 'role[ntp_and_mysql]' 
```

最后client1.chefdemo.com这个Node会把它展开为4条Resource(按顺序)

安装ntp的Resource  
启动ntp的Resource  
安装mysql-server的Resource  
启动mysql-server的Resource  

再由Provider将其转为对应的命令,最后这个Node所要做的就是:

安装ntp   
启动ntp   
安装mysql-server   
启动mysql-server  

### Cookbook

Cookbook实际上就是Recipe等一些东西的打包,像前面的ntp::default,ntp就是一个Cookbook

Cookbook的目录结构类似这样

```
tree /var/chef/cookbooks/ntp/

/var/chef/cookbooks/ntp/
├── attributes
├── definitions
├── files
│   └── default
├── libraries
├── metadata.rb
├── providers
├── README.md
├── recipes
│   ├── default.rb
│   └── ntp.rb
├── resources
└── templates
    └── default
        └── ntp.conf.erb

10 directories, 5 files
```

更多Cookbook的内容请查看官方Wiki

### Data Bag

由于创建用户的那个Recipe就用到了Data Bag, 所以这里简单说一下

Data Bag提供了定义全局信息的方法,直接看例子

首先我们创建一个Data Bag

```
knife data bag create admin
```

这条命令在chef-server上创建一个Data Bag, 可以在里面存储信息

```
mkdir -p /var/chef/data_bags/admin
vim /var/chef/data-bags/admin/william.json

{
    "id": "william",
    "shell": "/bin/bash",
    "comment": "william",
    "action": "create",
}
```

然后上传到服务端

```
cd /var/chef
knife data bag from file admin william.json
```

现在就可以在Recipe里访问这些信息,可以有两个方法:data_bag和data_bag_item

- data_bag

```
data_bag("admin")# => ["william"] 
```

'# =>' 这里指返回的意思,也就是说如果这样写:

```
username=data_bag("admin")
```

那么username就等["william"]

- data_bag_item

```
data_bag_item('admins', 'charlie')# => {"id"=>"william", "shell"=>"/bin/bash", "comment"=>"william", "action"=>"create"} 
```

### 总结

现在你已经知道这些新的概念了,我们再来看下Chef如何工作

- 在Workstation上定义各个Client应该如何配置自己,然后将这些信息上传到中心服务器
  - 可以分为两个方面
    - Chef利用Recipe和Role定义出来一些模板
      - 如一个名为mysql的Role可能描述怎么配置才成成为一个mysql服务器
      - 利用它的runlist里包含的Role和Recipe实现这种描述
    - Chef再指定各个Client应用那些模板
      - 如给Client1指定mysql的Role
      - 实际上只是将mysql的run_list里的东西加到Client1的run_list里
- 每个Client连到中心服务器查看如何配置自己,然后进行自我配置
  - Client连到中心服务器查看自己的run_list里都有什么东西(Role和Recipe)
  - 把需要的Cookbook传一份过来
    - 把run_list里的Role展开成Recipe,就得到一个Recipe的列表了
    - 这些Recipe都属于那些Cookbook,这些Cookbook可能就是被传的对象了
  - Client把run_list里的东西按顺序(重要)展开成Resource(得到一个Resource的列表)
  - Client按顺序(重要)应用这个Resource列表来进行自我配置
    - Provider负责把这个抽象的Resource对应到具体的系统命令

##  Chef扩展

### template

这里简单的说一个template这个Resource,因为可以正好从侧面消除大家对Chef的误解 它非常适合用来管理配置文件,先看一个例子

```
template "/etc/ntp.conf" do
  source "ntp.conf.erb"   variables(
    :ntp_server => node[:ntp_server]
  )
end
```

这个Resource用来从服务端复制配置文件到客户端,复制的过程中还进行了变量替换(ntpserver),也就是不同的服务器可以有各自的ntpserver

这个模板文件(ntp.conf.erb)内容为:

```
# generated by Chef. restrict default kod nomodify notrap nopeer noquery
restrict -6 default kod nomodify notrap nopeer noquery
restrict 127.0.0.1
restrict -6 ::1
server <%= @ntp_server %>
server  127.127.1.0     # local clock driftfile /var/lib/ntp/drift
keys /etc/ntp/keys
```

这个文件保存在什么地方呢,如果对前面的Cookbook的目录结构有印象,应该还记得有template这个目录 这个目录下有default这个目录,一般就保存在这里

假设有一个Node(主机名为client1.chefdemo.com, 安装了centos6.2的系统)的runlist里有这一条Resource, 那么它从服务端传这个文件的时候,它会按这个顺序在服务端查找

```
.../template/host-client1.chefdemo.com/ntp.conf.erb
.../template/centos-6.2/ntp.conf.erb
.../template/centos/ntp.conf.erb
.../template/default/ntp.conf.erb
```

如果客户机是Ubuntu12.04,它会按这样的顺序查找

```
.../template/host-client1.chefdemo.com/ntp.conf.erb
.../template/ubuntu-12.04/ntp.conf.erb
.../template/ubuntu/ntp.conf.erb
.../template/default/ntp.conf.erb
```

也就是说不同的客户端(Platform和版本)可以有各自的配置文件

总结一下:

- 不同的客户端可以有各自的一些变量
- 不同的客户端可以有各自的配置文件

从这两点我们可以看到Chef不只适用于配置比较相似的情况,同样可以胜任个性化的配置

### 初始化的Recipe(init)

这只是一个非常简单的Recipe,对ssh做一些配置,安装一些软件,加上超级管理员的key(重点)

```
### ssh ###

if platform?("redhat", "centos", "fedora")
    ssh_server = "sshd"
else
    ssh_server = "ssh"
end

service ssh_server do
    action :nothing
end

template "/etc/ssh/sshd_config" do
    source "sshd_config.erb"
    notifies :restart, resources(:service => ssh_server), :immediately
end

template "/etc/ssh/ssh_config" do
    source "ssh_config.erb"
    notifies :restart, resources(:service => ssh_server), :immediately
end

# generate ssh key, create /root/.ssh
execute "ssh-keygen" do
    command "ssh-keygen -t dsa -f /root/.ssh/id_rsa -N \"\""
    if File.exists?("/root/.ssh/id_rsa")
        action :nothing
    end
end

# add admins' keys
template "/root/.ssh/.keys" do
    source "keys.erb"
    owner "root"
    group "root"
    mode "0600"
end

# add group cyops and add root to it
group "cyops" do
    system true
    members "root"
end

# packages 
package "vim" do
    action :install
end
```

重点看一下template "root.ssh/.keys"这个Resource 如果所有的服务器都有这一条Recipe,那么我要知道所有服务上都有那些人的key, 我只要查看workstation上的这个模板文件就可以了,因为所有其它服务器上的这个文件都是从这里复制过去的, 要添加删除某个用户的key,也只要修改这个文件然后upload

### 账号管理的Recipe(user)

这个也很简单,用户数据保存在Data Bag,这个Recipe从Data Bag解析得到用户的数据,然后利用user这个Resource进行添加删除的动作

还记得前面Data Bag的例子吗,如果admin这个Data Bag里包含这么几个文件:

    charlie.json herry.json william.json

那么users就等于["charlie", "herry", "william"]

```
users = data_bag('admin')

users.each do |login|
    # 每次处理一个用户,利用下面的方法得到一个用户的数据
    username = data_bag_item('admin', login)
    _action = username['action']

    # 利用user这个Resource添加删除用户
    user(login) do
        shell     username['shell']
        comment   username['comment']

        home      "/home/#{login}"
        supports  :manage_home => true
        if _action == "delete"
            action :remove
        end
    end

    # 创建.ssh目录,将key放里面
    if _action != "delete"
        directory "/home/#{login}/.ssh" do
            owner "#{login}"
        end

        file "/home/#{login}/.ssh/.keys" do
            content username['keys']
            owner "#{login}"
        end
    end
end
```

写了个脚本简化添加删除用户的动作,由于前面将数据和代码分开,只要更新Data Bag的用户数据就可以了

将下面的代码放到$PATH路径里(如/usr/local/bin/)就可以用它来添加删除用户了

```
mv user.sh /usr/local/bin/user
chmod +x /usr/local/bin/user
user add admin william /home/william/.ssh/id_rsa.pub
```

```
#!/bin/bash

chef_dir='/var/chef'
echo $#

function usage()
{
    echo "two few args"
    echo -e "Usage: \nuser add user_type user_name path_of_users_key"
    echo "user del user_type user_name"
    echo "user list user_type"
    echo "user list"
    exit
}

action=$1

if [ "$action" == "add" ]; then
    usertype=$2
    username=$3
    keypath=$4

    ls ${chef_dir}/data_bags/${usertype} &> /dev/null
    if [ "$?" != "0" ]; then
        echo "${usertype} doesn't exist"
        exit
    fi
    cat > ${chef_dir}/data_bags/${usertype}/${username}.json << EOF
{
    "id": "${username}",
    "shell": "/bin/bash",
    "comment": "${username}",
    "action": "create",
    "keys": "`cat ${keypath}`"
}
EOF

    cd $chef_dir
    knife data bag from file ${usertype} ${username}.json

    cd -
    cp ${keypath} ${chef_dir}/cookbooks/user/templates/default/${usertype}/${username}.key
    knife cookbook upload user
elif [ "$action" == "del" ]; then
    usertype=$2
    username=$3
    keypath=$4
    sed -i -e 's/create/delete/' ${chef_dir}/data_bags/${usertype}/${username}.json
    cd $chef_dir
    knife data bag from file ${usertype} ${username}.json
elif [ "$action" == "list" ]; then
    if [ "$2" != "" ]; then
        user_type=$2
        echo "Users in ${user_type}":
        echo '---------------------------'
        for i in `grep -R 'create' /var/chef/data_bags/${user_type}/ | awk '{print $1}' | sed -e "s/.*${user_type}\///" -e 's/\.json://'`
        do
            echo $i
        done
    else
        for i in `cd /var/chef/data_bags/; ls`
        do
            user_type=$i
            echo "Users in ${user_type}":
            echo '---------------------------'
            for i in `grep -R 'create' /var/chef/data_bags/${user_type}/ | awk '{print $1}' | sed -e "s/.*${user_type}\///" -e 's/\.json://'`
            do
                echo $i
            done
            echo
        done
    fi

else
    usage
fi
```

