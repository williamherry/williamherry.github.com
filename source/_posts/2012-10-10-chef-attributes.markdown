---
layout: post
title: "Chef基础之Arrtibutes"
date: 2012-10-10 21:42
comments: true
categories:
---

本文档是对官方Wiki[Attributes](http://wiki.opscode.com/display/chef/Attributes)的翻译

## 概述

{% img /images/chef/attributes.jpg %}

属性(Attributes)就是节点(Node)的信息,如IP地址,主机名,加载的内核模块,系统中可用的编程语言的版本以及更多.新的属性可以用多种方式加到节点上.

## 属性类型和优先级

{% img /images/chef/attribute-priority-stack.png %}

有四种类型的属性,按优先级从高到低的顺序排列,它们是:

- automatic
- override
- normal
- default

*用默认属性(default)写你的cookbook, 如果有必要,在role或者node中覆盖默认变量*

Chef没有提供任何方式去修改自动属性(automatic),因为Ohai下次运行的时候回覆盖你做的任何修改.检验[自动属性](http://wiki.opscode.com/display/chef/Automatic+Attributes)看有那些保留的命名

## 属性的持久性

*每一次chef-client运行的开始,默认属性,覆盖属性和自动属性会被完全重置*

然后会利用当前的cookbooks, recipes, roles, environment和Ohai数据重新组建

所以,一量recipe, role或environment从一个node移除并且chef-client在这个node上运行,那么在cookbook的attribute文件,roles, recipes或environments里设置的default和override属性会消失

*Normal属性不会被重置*

而是,第一次chef-client运行,任何以JSON文件形式传递给node的新属性会被合并到node中已经存在的normal属性中(使用[Deep Merge](http://wiki.opscode.com/display/chef/Deep+Merge))

这就意味着,任何在recipe或cookbook中定义的normal属性会仍然存在,即使是cookbook或role已经从node的run list移除后

## 设置属性

属性可以在下面的对象中设置

- cookbooks
- environments (Chef 0.10.0 or above only)
- roles
- nodes

### 优先级

*属性的优先级从低到高排列在下面*

- attributes文件中定义的default属性
- environment中定义的default属性
- role中定义的default属性
- 直接在node的recipe中定义的default属性
- attributes文件中定义的normal或set属性
- 直接在node的recipe中定义的normal或set属性
- attributes文件中定义的override属性
- role中定义的override属性
- environment中定义的override属性
- 直接在node的recipe中定义的override属性
- Ohai产生的automatic属性

在attributes文件里定义的default属性有最低的优先级. Ohai生成的automatic属性有最高的优先级

*用默认属性(default)写你的cookbook, 如果有必要,在role或者node中覆盖默认变量*

[这里](http://wiki.opscode.com/display/chef/Setting+Attributes+%28Examples%29)有例子

## Cookbook中定义属性

Cookbook的属性文件可以在cookbook的attributes子目录中找到.他们在Node对象的上下文中运算并且使用Node的方法设置属性的值

来自Oposcode的Apache cookbook中的例子

{% codeblock cookbooks/apache2/attributes/default.rb lang:ruby %}
default["apache"]["dir"]          = "/etc/apache2"
default["apache"]["listen_ports"] = [ "80","443" ]
{% endcodeblock %}

这里Node对象的使用是隐含的,下面这样写和上面等价

{% codeblock cookbooks/apache2/attributes/default.rb lang:ruby %}
node.default["apache"]["dir"]          = "/etc/apache2"
node.default["apache"]["listen_ports"] = [ "80","443" ]
{% endcodeblock %}

属性也可以在recipe中设置,但这时`node.`就不能省略了

### Cookbook设置属性的方法

使用下面的方法在cookbook的attributes文件或recipe中设置属性.他们对应到和他们名字相同的属性类型(set是normal的一个别名)

- override
- default
- normal(or set)


