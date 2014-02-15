---
layout: post
title: "Chef Ohai源码学习"
date: 2014-02-11 08:28
comments: true
categories: chef ohai
---

接触Chef以来一直对Ohai很感兴趣,它收集的系统信息非常全面,很好奇它是怎么收集的,现在终于有时间学习了

### 目标

搞清楚Ohai是如何收集到某一项系统消息的,如IP地址

### 环境搭建

使用`gem install ohai`安装ohai和它的依赖,然后下载ohai的源码

    git clone https://github.com/opscode/ohai.git

可以直接在源码目录通过`./bin/ohai`运行`ohai`,这样可以对源码做一些修改(添加调试代码)然后马上运行查看效果,而不用去找它的源码安装到系统的位置

还可以使用`pry`进行单步调试: `gem install pry`安装`pry`,的要调试的代码前面加上`binding.pry`,在该文件前面加上`require 'pry'`

### 查看源码过程

首先找到入口,当然就是`bin/ohai`文件了,我们需要关系的只有一行

``` ruby bin/ohai
Ohai::Application.new.run
```

我们在看一下`Ohai::Application`的定义,我们感兴趣的是`run`方法的定义

``` ruby lib/ohai/application.rb
def run
  configure_ohai
  configure_logging
  run_application
end
```

这里前两个方法我们目前不关心,看一下`run_application`方法的定义

``` ruby lib/ohai/application.rb
def run_application
  ohai = Ohai::System.new
  ohai.all_plugins(@attributes)

  if @attributes
    @attributes.each do |a|
      puts ohai.attributes_print(a)
    end
  else
    puts ohai.json_pretty_print
  end
end
```

通过加调试代码发现这里的`@attributes`是nil,所以这里的代码可以简化为

``` ruby
ohai = Ohai::System.new
ohai.all_plugins(nil)
puts ohai.json_pretty_print
```

下来去看`Ohai::System`的定义

首先我们看一下`json_pretty_print`的定义

``` ruby lib/ohai/system.rb
def json_pretty_print(item=nil)
  Yajl::Encoder.new(:pretty => true).encode(item || @data)
end
```

它只是把`@data`的数据格式化后输出,所以我们推测实际收集的动作是发生在`all_plugins`方法里

``` ruby lib/ohai/system.rb
def all_plugins(attribute_filter=nil)
  # Reset the system when all_plugins is called since this function
  # can be run multiple times in order to pick up any changes in the
  # config or plugins with Chef.
  reset_system

  load_plugins
  run_plugins(true, attribute_filter)
end
```

`reset_system`方法里只是初始化了一些实例变量,其中包括`@loader`它`@runner`

``` ruby
@loader = Ohai::Loader.new(self)
@runner = Ohai::Runner.new(self, true)
```

这里把`self`传了进去,这里的`self`就就`Ohai::System`的实例,我们看一下loader如何处理

``` ruby lib/ohai/loader.rb
def initialize(controller)
  @controller = controller
  @v6_plugin_classes = []
  @v7_plugin_classes = []
end
```

loader把`Ohai::System`的实例保存到了实例变量`@controller`中

`load_plugins`方法只有一行,调用了`@loader`的`load_all`方法,我们看一下这个方法

``` ruby lib/ohai/loader.rb
def load_all
  plugin_files_by_dir.each do |plugin_file|
    load_plugin_class(plugin_file.path, plugin_file.plugin_root)
  end

  collect_v6_plugins
  collect_v7_plugins
end
```

看一下`plugin_files_by_dir`的定义

``` ruby lib/ohai/loader.rb
def plugin_files_by_dir
  Array(Ohai::Config[:plugin_path]).inject([]) do |plugin_files, plugin_path|
    plugin_files + PluginFile.find_all_in(plugin_path)
  end
end
```

注释中说它搜索所有的plugin路径并返回一个包含`PluginFile`对象的数组

在看一下`PluginFile`对象长什么样子

``` ruby lib/ohai/loader.rb
class PluginFile < Struct.new(:path, :plugin_root)

  def self.find_all_in(plugin_dir)
    Dir[File.join(plugin_dir, "**", "*.rb")].map do |file|
      new(file, plugin_dir)
    end
  end
end
```

可以看到它只是简单的从`Struct`继承而来,这里所起的作用就是定义了两个访问器`path`和`plugin_root`,打印出来类似这样

```
#<struct Ohai::Loader::PluginFile
 path="/Users/william/Codes/ohai/lib/ohai/plugins/aix/cpu.rb",
 plugin_root="/Users/william/Codes/ohai/lib/ohai/plugins">
```

再回到`load_all`方法

``` ruby lib/ohai/loader.rb
def load_all
  plugin_files_by_dir.each do |plugin_file|
    load_plugin_class(plugin_file.path, plugin_file.plugin_root)
  end

  collect_v6_plugins
  collect_v7_plugins
end
```

对每一个`PluginFile`对象调用了`load_plugin_class`方法

``` ruby lib/ohai/loader.rb
def load_plugin_class(plugin_path, plugin_dir_path=nil)
  # Read the contents of the plugin to understand if it's a V6 or V7 plugin.
  contents = ""
  begin
    contents << IO.read(plugin_path)
  rescue IOError, Errno::ENOENT
    Ohai::Log.warn("Unable to open or read plugin at #{plugin_path}")
    return nil
  end

  # We assume that a plugin is a V7 plugin if it contains Ohai.plugin in its contents.
  if contents.include?("Ohai.plugin")
    load_v7_plugin_class(contents, plugin_path)
  else
    Ohai::Log.warn("[DEPRECATION] Plugin at #{plugin_path} is a version 6 plugin. \
Version 6 plugins will not be supported in future releases of Ohai. \
Please upgrade your plugin to version 7 plugin syntax. \
For more information visit here: docs.opscode.com/ohai_custom.html")

    load_v6_plugin_class(contents, plugin_path, plugin_dir_path)
  end
end
```

他把文件里的内容读了进来,并根据有没有包含`Ohai.plugin`来有选择了调用`load_v7_plugin_class`或`load_v6_plugin_class`,我大概看来一下基本上全都包含`Ohai.plugin`,所以我们从v7追

``` ruby lib/ohai/loader.rb
def load_v7_plugin_class(contents, plugin_path)
  plugin_class = eval(contents, TOPLEVEL_BINDING, plugin_path)
  unless plugin_class.kind_of?(Class) and plugin_class < Ohai::DSL::Plugin
    raise Ohai::Exceptions::IllegalPluginDefinition, "Plugin file cannot contain any statements after the plugin definition"
  end
  plugin_class.sources << plugin_path
  @v7_plugin_classes << plugin_class unless @v7_plugin_classes.include?(plugin_class)
  plugin_class
rescue SystemExit, Interrupt
  raise
rescue Ohai::Exceptions::InvalidPluginName => e
  Ohai::Log.warn("Plugin Name Error: <#{plugin_path}>: #{e.message}")
rescue Ohai::Exceptions::IllegalPluginDefinition => e
  Ohai::Log.warn("Plugin Definition Error: <#{plugin_path}>: #{e.message}")
rescue NoMethodError => e
  Ohai::Log.warn("Plugin Method Error: <#{plugin_path}>: unsupported operation \'#{e.name}\'")
rescue SyntaxError => e
  # split on occurrences of
  #    <env>: syntax error,
  #    <env>:##: syntax error,
  # to remove from error message
  parts = e.message.split(/<.*>[:[0-9]+]*: syntax error, /)
  parts.each do |part|
    next if part.length == 0
    Ohai::Log.warn("Plugin Syntax Error: <#{plugin_path}>: #{part}")
  end
rescue Exception, Errno::ENOENT => e
  Ohai::Log.warn("Plugin Error: <#{plugin_path}>: #{e.message}")
  Ohai::Log.debug("Plugin Error: <#{plugin_path}>: #{e.inspect}, #{e.backtrace.join('\n')}")
end
```

异常处理我们不关心,直接删掉来看

``` ruby lib/ohai/loader.rb
def load_v7_plugin_class(contents, plugin_path)
  plugin_class = eval(contents, TOPLEVEL_BINDING, plugin_path)
  unless plugin_class.kind_of?(Class) and plugin_class < Ohai::DSL::Plugin
    raise Ohai::Exceptions::IllegalPluginDefinition, "Plugin file cannot contain any statements after the plugin definition"
  end
  plugin_class.sources << plugin_path
  @v7_plugin_classes << plugin_class unless @v7_plugin_classes.include?(plugin_class)
  plugin_class
end
```

这里第一行把插件文件里的代码执行,返回的结果赋给`plugin_class`,把`plugin_path`保存到其中,并把它收集到实例变量`@v7_plugin_classes`中

我们找最简单了plugin看里面是什么

``` ruby lib/ohai/plugins/command.rb
Ohai.plugin(:Command) do
  provides "command"

  collect_data do
    command Mash.new
  end
end
```

它是调用`Ohai`的`plugin`方法,还传给它一个block,看看这个方法的定义

``` ruby lib/ohai/dsl/plugin.rb
def self.plugin(name, &block)
  raise Ohai::Exceptions::InvalidPluginName, "#{name} is not a valid plugin name. A valid plugin name is a symbol which begins with a capital letter and contains no underscores" unless NamedPlugin.valid_name?(name)

  plugin = nil

  if NamedPlugin.strict_const_defined?(name)
    plugin = NamedPlugin.const_get(name)
    plugin.class_eval(&block)
  else
    klass = Class.new(DSL::Plugin::VersionVII, &block)
    plugin = NamedPlugin.const_set(name, klass)
  end

  plugin
end
```

这个方法重点是这两行

```
klass = Class.new(DSL::Plugin::VersionVII, &block)
plugin = NamedPlugin.const_set(name, klass)
```

第一行用传进来了block创建了一个继承自`DSL::Plugin::VersionVII`的类,然后一传进来的`name`为常量名保存到模块`NamedPlugin`中

我们在回到前面的plugin文件

``` ruby lib/ohai/plugins/command.rb
Ohai.plugin(:Command) do
  provides "command"

  collect_data do
    command Mash.new
  end
end
```

这里还有两个方法`provides`和`collect_data`,你没有猜错,它们就是定义在继承来的`DSL::Plugin::VersionVII`中

``` ruby lib/ohai/dsl/plugin/versionvii.rb
def self.provides(*attrs)
  attrs.each do |attr|
    provides_attrs << attr unless provides_attrs.include?(attr)
  end
end
```

它只是它传进来的参数收集到`provides_atts`,而`provides_atts`是前面生成的类的实例变量

``` ruby lib/ohai/dsl/plugin/versionvii.rb
def self.provides_attrs
  @provides_attrs ||= []
end
```

在看`collect_data`

``` ruby lib/ohai/dsl/plugin/versionvii.rb
def self.collect_data(platform = :default, *other_platforms, &block)
  [platform, other_platforms].flatten.each do |plat|
    if data_collector.has_key?(plat)
      raise Ohai::Exceptions::IllegalPluginDefinition, "collect_data already defined on platform #{plat}"
    else
      data_collector[plat] = block
    end
  end
end
```

由于前面的plugin中调用这个方法的时候只传了一个快进来,所从这个方法只是把传进来了块赋值给了以`:default`为key的`Mash`对象`data_collector`中

``` ruby lib/ohai/dsl/plugin/versionvii.rb
def self.data_collector
  @data_collector ||= Mash.new
end
```

我们再一次回到`load_all`方法中

``` ruby lib/ohai/loader.rb
def load_all
  plugin_files_by_dir.each do |plugin_file|
    load_plugin_class(plugin_file.path, plugin_file.plugin_root)
  end

  collect_v6_plugins
  collect_v7_plugins
end
```

还剩下两行代码,我们只看一下`collect_v7_plugins`

``` ruby lib/ohai/loader.rb
def collect_v7_plugins
  @v7_plugin_classes.each do |plugin_class|
    load_v7_plugin(plugin_class)
  end
end
```

对收集的`plugin_class`调用`load_v7_plugin`方法

``` ruby lib/ohai/loader.rb
def load_v7_plugin(plugin_class)
  plugin = plugin_class.new(@controller.data)
  collect_provides(plugin)
  plugin
end
```

这个方法把传进来的类实例化,并把`Ohai::System`实例的data传了进去,然后调用了`collect_provides`

``` ruby lib/ohai/loader.rb
def collect_provides(plugin)
  plugin_provides = plugin.class.provides_attrs
  @controller.provides_map.set_providers_for(plugin, plugin_provides)
end
```

还记得前面最简单的plugin的代码吗,这里的`plugin.class.provides_attrs`就是`provides`后面的参数("command")

这里的provides_map是在`Ohai::System`的`reset_system`赋值的,是`ProvidesMap`的实例

```
@provides_map = ProvidesMap.new
```

然后它又调用了`set_providers_for`方法

```
def set_providers_for(plugin, provided_attributes)
  unless plugin.kind_of?(Ohai::DSL::Plugin)
    raise ArgumentError, "set_providers_for only accepts Ohai Plugin classes (got: #{plugin})"
  end

  provided_attributes.each do |attribute|
    attrs = @map
    parts = normalize_and_validate(attribute)
    parts.each do |part|
      attrs[part] ||= Mash.new
      attrs = attrs[part]
    end
    attrs[:_plugins] ||= []
    attrs[:_plugins] << plugin
  end
end
```

这里把传来的keys它plugin收集到了providesmap的实例变量`@map`中,类似这样

```
[2] pry(#<Ohai::ProvidesMap>)> @map
=> {"cpu"=>
     {"_plugins"=>
       [#<Ohai::NamedPlugin::CPU:0x00000101a6a4d0
         @data={},
         @has_run=false,
         @source=
           ["/Users/william/Codes/ohai/lib/ohai/plugins/aix/cpu.rb",
             "/Users/william/Codes/ohai/lib/ohai/plugins/darwin/cpu.rb",
             "/Users/william/Codes/ohai/lib/ohai/plugins/freebsd/cpu.rb",
             "/Users/william/Codes/ohai/lib/ohai/plugins/linux/cpu.rb",
             "/Users/william/Codes/ohai/lib/ohai/plugins/netbsd/cpu.rb",
             "/Users/william/Codes/ohai/lib/ohai/plugins/openbsd/cpu.rb",
             "/Users/william/Codes/ohai/lib/ohai/plugins/sigar/cpu.rb",
             "/Users/william/Codes/ohai/lib/ohai/plugins/solaris2/cpu.rb",
             "/Users/william/Codes/ohai/lib/ohai/plugins/windows/cpu.rb"
           ],
         @version=:version7>
       ]
     }
   }
```

我们再回到`all_plugins`方法,前面是`load_plugins`方法的深入执行过程,现在来看`run_plugins`方法

``` ruby lib/ohai/system.rb
def run_plugins(safe = false, attribute_filter = nil)
  # First run all the version 6 plugins
  @v6_dependency_solver.values.each do |v6plugin|
    @runner.run_plugin(v6plugin)
  end

  # Then run all the version 7 plugins
  begin
    @provides_map.all_plugins(attribute_filter).each { |plugin|
      @runner.run_plugin(plugin)
    }
  rescue Ohai::Exceptions::AttributeNotFound, Ohai::Exceptions::DependencyCycle => e
    Ohai::Log.error("Encountered error while running plugins: #{e.inspect}")
    raise
  end
end
```

我们只看v7的,这里比较简单,对每一个plugin调用`@runner`的`run_plugin`方法(`@runner`)是在`reset_system`中定义的

```
def run_plugin(plugin)
  unless plugin.kind_of?(Ohai::DSL::Plugin)
    raise Ohai::Exceptions::InvalidPlugin, "Invalid plugin #{plugin} (must be an Ohai::DSL::Plugin or subclass)"
  end

  if Ohai::Config[:disabled_plugins].include?(plugin.name)
    Ohai::Log.debug("Skipping disabled plugin #{plugin.name}")
    return false
  end

  begin
    case plugin.version
    when :version7
      run_v7_plugin(plugin)
    when :version6
      run_v6_plugin(plugin)
    else
      raise Ohai::Exceptions::InvalidPlugin, "Invalid plugin version #{plugin.version} for plugin #{plugin}"
    end
  rescue Ohai::Exceptions::Error
    raise
  rescue Exception,Errno::ENOENT => e
    Ohai::Log.debug("Plugin #{plugin.name} threw exception #{e.inspect} #{e.backtrace.join("\n")}")
  end
end
```

简单说它就是转而去调用`run_v7_plugin`

```
def run_v7_plugin(plugin)
  visited = [ plugin ]
  while !visited.empty?
    next_plugin = visited.pop

    next if next_plugin.has_run?

    if visited.include?(next_plugin)
      raise Ohai::Exceptions::DependencyCycle, "Dependency cycle detected. Please refer to the following plugins: #{get_cycle(visited, plugin).join(", ") }"
    end

    dependency_providers = fetch_plugins(next_plugin.dependencies)

    # Remove the already ran plugins from dependencies if force is not set
    # Also remove the plugin that we are about to run from dependencies as well.
    dependency_providers.delete_if { |dep_plugin|
      dep_plugin.has_run? || dep_plugin.eql?(next_plugin)
    }

    if dependency_providers.empty?
      @safe_run ? next_plugin.safe_run : next_plugin.run
    else
      visited << next_plugin << dependency_providers.first
    end
  end
end
```

简单说就是去调用plugin自己的`safe_run`方法(因为在定义@runner的时候有传第二个参数true)

```
def safe_run
  begin
    self.run
  rescue Ohai::Exceptions::Error => e
    raise e
  rescue => e
    Ohai::Log.debug("Plugin #{self.name} threw #{e.inspect}")
    e.backtrace.each { |line| Ohai::Log.debug( line )}
  end
end
```

它又调用了`run`方法

```
def run
  @has_run = true
  run_plugin
end
```

它又调用了`run_plugin`方法,这个方法是在`lib/ohai/dsl/plugin/versionvii.rb`中定义的

```
def run_plugin
  collector = self.class.data_collector
  platform = collect_os

  if collector.has_key?(platform)
    self.instance_eval(&collector[platform])
  elsif collector.has_key?(:default)
    self.instance_eval(&collector[:default])
  else
    Ohai::Log.debug("No data to collect for plugin #{self.name}. Continuing...")
  end
end
```

它用`instance_eval`执行了前面动态构造Plugin类的时候保存下来的块(collect_data后面跟的块)

整个过程过了一遍,但有一点还没明白,最开始的地方我们发现显示出来的信息都收集在`Ohai::System`的实例`@ohai`的实例变量`@data`里,执行插件里的代码怎么会修改到它呢?

这是因为在实例化plugin的时候它把实例变量`@data`传了进去

``` ruby lib/ohai/loader.rb
def load_v7_plugin(plugin_class)
  plugin = plugin_class.new(@controller.data)
  collect_provides(plugin)
  plugin
end
```

在看一下plugin的部分代码

```
Ohai.plugin(:Command) do
  provides "command"

  collect_data do
    command Mash.new
  end
end
```

传给`collect_data`的块中的command是不是很奇怪,它是个什么东西?

答案的下面的代码中

``` ruby lib/ohai/dsl/plugin.rb
def method_missing(name, *args)
  return get_attribute(name) if args.length == 0

  set_attribute(name, *args)
end

def get_attribute(name)
  @data[name]
end
```

到此所有迷雾都解开了, Yeah!!!
