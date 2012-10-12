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

### Cookbook中属性文件的加载顺序

Chef按字母顺序加载cookbook中的属性文件.如果你需要一个属性文件在另一个之前加载(例如,你的Apache的cookbook属性文件的加载需要在Rails的cookbook属性文件加载之后),你可以使用`include_attribute`方法.像这样:

{% codeblock include\_attribute lang:ruby %}
include_attribute "rails"
node.set['apache2']['proxy_to_unicorn'] = node['rails']['use_unicorn']
{% endcodeblock %}

这会在继续处理当前属性之前首先加载`rails/attributes/defaults.rb`

双冒号的写法也适用于`include_attribute`

{% codeblock include\_attribute lang:ruby %}
include_attribute "rails::tunables"
node.set['apache2']['proxy_to_unicorn'] = node['rails']['use_unicorn']
{% endcodeblock %}

这会加载rails cookbook中的`attributes/tunables.rb`文件

### 从Recipes中重新加载Attribute文件

有时候属性依赖于recipes中的动作,所以可能会需要在recipe中重新加载属性.例如,你有一个读防火墙规则的属性,并且有一个安装防火墙软件的recipe,第一次执行这个cookbook防火墙的属性不会被设置.因为`include_attribute`在recipes中不可用,所以你需要手动重新加载`firewall::default`属性

{% codeblock reloading attributes from recipes lang:ruby %}
package 'iptables' do
  notifies :create, 'ruby_block[try_firewall_again]', :immediately
end

ruby_block 'try_firewall_again' do
  block do
    node.load_attribute_by_short_filename('default', 'firewall')
  end
  action :nothing
end
{% endcodeblock %}

### 访问属性的方法

属性访问方法会自动创建并且方法可以和键交换使用.下面的例子和上面的用法等价

{% codeblock cookbooks/apache2/attributes/default.rb lang:ruby %}
default.apache.dir          = "/etc/apache2"
default.apache.listen_ports = [ "80","443" ]
{% endcodeblock %}

使用哪个只是风格问题,你可能会在检索属性的值的时候看到

## Environment中设置属性

*Environment中可以设置default和override属性*

这可以在Environment的Ruby DSL文件里(分别)通过default_attributes和`override_attributes`方法设置,或者在Environment的JSON数据中使用`default_attributes`和`override_attributes`Hashes进行设置.经常会指定一些这个Environment特别有属性.例如在"production"环境和"staging"环境外部负载均衡器的公共DNS可能会不一样

## 在Role中定义属性

*在Role中只可以设置default和override属性,不能设置normal属性.*这可以在Role的Ruby DSL文件里(分别)通过`default_attributes`和`override_attributes`方法设置,或者在Role的JSON数据中使用`default_attributes`和`override_attributes`Hashes进行设置.经常会指定一些各个Role不同的属性.例如一个`php_apache2_server`的role可能会和`mod_perl_apche2_server`使用不同的优化参数

## 在Node中定义属性

最终,Node对象可以直接被修改以设置属性.通常会设置normal优先级的属性.可以通过使用knife工具直接编辑node,或者通过WebUI.或者通过传递JSON数据给Node进行设置

### JSON属性

你可以使用JSON文件指定Node的属性,它们会以normal的优先级添加到Node

```
chef-client --help
[...]
    -j JSON_ATTRIBS                  Load attributes from a JSON file or URL
        --json-attributes
```

例如.设置Apache监听不同的端口

{% codeblock JSON attribute example lang:json %}
{ "apache": { "listen_ports": ["81", "8080"] } }
{% endcodeblock %}

记住.通过JSON文件传递的属性会和Node已经保存的属性合并并且没有办法覆盖,但是如果有冲突,从JSON文件传递的属性会最终被使用

### Automatic属性

这第四种属性不能被修改,因为你做的任何修改都会在下一次运行时被Ohai数据覆盖

## 如果使用属性

- 属性的优势

  - 为应用程序跨平台的抽象例如配置文件的路径
  - 优化参数的默认值例如给处理器多少内存或开启多少个进程
  - 任何其它你想要在Chef运行之间保留(在Node的数据)的信息

### 使用的最佳实践

*属性优先级的通用模式是在cookbooks和roles中使用default属性*

- 如果你要改变一个特定Node的值,使用normal属性
- Overrides属性是指roles可以强制设定一个特定的属性,即使Node已经有相应的属性

肯定有其它的使用方式,但上面的模式是设计的初衷

### 设置同一优先级的属性

*一个通用的使用案例是在cookbook的attribute文件中设置default属性,并且在role中以不同的值设置相同的default属性.*在这种情况下,role中的属性会被[deep merged](http://wiki.opscode.com/display/chef/Deep+Merge)到来自attribute文件中的属性顶端.如果有冲突,在role中设置的属性会被使用.

### 只有在这个属性没有值的时候才设置

在属性文件里,你也可以使用属性优先方法的变种`_unless`方法来做到只有在一个属性没有值时才设置这个属性,这些方法是`default_unless`, `set_unless`和`override_unless`.这些方法有时候非常方便,但要小心使用!

在你使用这些方法的时候,这些加到你的Node中的属性会和你在cookbooks中定义的不同步,当你更新cookbook的时候.这意味着建立一个和已经存在的Node使用相同的recipes和roles会得到不同的配置--会给调试带来麻烦.出于这个原因,你应该尽可能不用`_unless`方法
