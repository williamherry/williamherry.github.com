---
layout: post
title: "Ruby元编程读书笔记"
date: 2015-07-17 14:18
comments: true
categories: "notes"
---

> 由于C语言绝大多数编译时的信息在运行时都丢失了，所以C语言不支持元编程或內省

实例方法：在类中定义，在实例中使用

Ruby的class关键词更像一个作用域操作符而不是类型声明语句

一个对象仅仅包含它的实例变量以及一个对自身类的引用

方法存放在类中而不是实例中

类自身也是对象，作为对象它的类是Class

超类是指类之间的继承关系

任何以大写字母开头的引用（包括类名和模块名），都是常量

方法查找：如果不引入eigenclass, 方法查找被称为“向右一步，在向上”， 也就是说，先向右一步来到接收者所在的类中查找，然后沿着祖先链向上直到找到给定的方法。如果有eigenclass，那么最先会在接收者的eigenclass中查找

默认类是从Object类继承而来，即

    class MyClass
    end

和

    class MyClass < Object
    end

是一样一样的

类和模块的定义中（并且在任何方法之外），self的角色由这个类或模块担任

私有方法服从一个简单的规则：不能明确指定一个接收者调用一个私有方法

    class A
      include B
    end

include会把被引入模块B插入到引入模块A祖先链中该模块A的上方

    A.ancestors # => [A, B, Object, Kernel, BasicObject]

没有明确指定接收者的调用都会作用于self

当调用一个方法时，实际上是给一个对象发送了一条消息

符号和字符串没有关系，并且它们属于完全不同的类

符号是不可变的

一些操作（比如比较操作）针对符号运行的更快

动态调用方法叫动态派发

动态定义方法叫动态方法

method_missing叫幽灵方法

什么事拟态方法？

什么事类宏？

只有在调用一个方法时才可以定义一个块，yield回调这个块，Kernel#block_given?()方法判断是否包含块

什么事闭包？

当定义一个块时，它会获取当时环境中的绑定，并且把它传给一个方法时，它会带着这些绑定一起进入该方法

块会覆盖具有相同名字的局部变量

类和模块定义中的代码会被立即执行，相反，方法定义中的代码只有在方法被调用时被执行

&操作符的真正含义：这是一个Proc对象，我想把它当成一个块再使用

可调用对象可以有一下方式：

  - 块（虽然它们不是真正的对象，但是它们是可调用的）：在定义它们的作用域中执行
  - proc：Proc类的对象，跟块一样，它们在定义自身的作用域中执行
  - lambda：也是Proc类的对象，但是它跟普通的proc有细微的区别。它跟块和proc一样都是闭包，因此可以在定义自身的作用域中执行
  - 方法：绑定于对象，在所绑定对象的作用域中执行

可调用对象的区别：

  - 在方法和lambda中，return语句从可调用对象中返回
  - 在块和proc中，return语句从定义可调用对象的原始上下文中返回
  - 对传入参数的处理：方法最严格，lambda同样严格（它与方法相比，在某些极端情况下略为宽松），而proc和块则要宽松一些

类方法就是类的单件方法，它存在于该类的eigenclass中

eigenclass的超类就是超类的eigenclass（有什么用?）

`class << obj` 进入对象的eigenclass

当类包含模块时，它获得的是该模块的实例方法－－而不是类方法。类方法存在于模块的eigenclass中

如果在对象的eigenclass中包含模块，就可以把该模块的实例方法导入到该对象的eigenclass中，从而成为该对象自己的方法

Object#extend只是在接收者eigenclass中包含模块的快捷方式

关于别名：

可以把方法看成有名字（name）和方法体（body）两部分组成，名字指向方法体

    def m1
      # body
    end

经过

    alias m2 m1

后，m2也指向类m1的方法内容b1
这时候在重定义m1

    def m1
      # body 2
    end

m1就会指向一个新的方法体b2，原来的方法体只能通过m2来引用了

注意alias是一个关键字，后面两个参数中间没有逗号

环绕别名：先定义别名，再重定义方法，在其中使用别名

    alias new_name old_name

class是保留词，除了使用klass替代外还可以用clazz

类扩展混入：

    module MyMixin
      def self.included(base)
        base.extend(ClassMethods)
      end

      module ClassMethods
        def x
          "x()"
        end
      end
    end
