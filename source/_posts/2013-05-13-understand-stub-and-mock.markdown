---
layout: post
title: "理解测试中的stub和mock"
date: 2013-05-13 16:46
comments: true
categories: 
---

刚开始学习`rspec`的时候,`stub`和`mock`理解起来有点困难,听了`Terry`和`aNdReW`的讲解后,对它们的理解深入多了,在此表示感谢

先从`stub`说起,什么是`stub`呢,`CodeSchool`给出这样的定义:

> Stub:  
> For replacing a method with code that returns a specified result

简单说就是你可以用`stub`去**伪造(fade)**一个方法,**阻断**对原来方法的调用,例如下面来自`CodeSchool`的例子

我们有一个叫`zombie`的`model`

``` ruby app/models/zombie.rb
class Zombie < ActiveRecord::Base
  has_one :weapon

  def decapitate
    weapon.slice(self, :head)
    self.status = "dead again"
  end
end
```

我们要测试`decapitate`方法,它里面调用了`weapon`的`slice`方法,下面是测试代码:

``` ruby /spec/models/zombie_spec.rb
describe Zombie do
  let(:zombie) { Zombie.create }

  context "#decapitate" do
    it "sets status to dead again" do
      zombie.weapon.stub(:slice)
      zombie.decapitate
      zombie.status.should == "dead again"
    end
  end
end
```

上面代码的第6行就是用`stub`伪造了`weapon`的`slice`方法,阻断了对原来方法的调用

你可能会问为什么我们要这样做,这是因为我们在做单元测试,`weapon`的`slice`可能会非常复杂,里面又调用了其它的方法等等,这是集成测试应该做的工作.事实上这里我们是在测试`decapitate`方法会把`zombie.status`设置成`"dead again"`

接下来我们来说`mock`, `CodeSchool`上给的定义是这样的:

> Mock:  
> A stub with an expectations that the method gets called.

简单来说`mock`就是`stub + expectation`, 说它是`stub`是因为它也可以像`stub`一样伪造方法,阻断对原来方法的调用, `expectation`是说它不仅伪造了这个方法,它还期望你(必须)调用这个方法,如果没有被调用到,这个`test`就`fail`了,看下面的例子

``` ruby /spec/models/zombie_spec.rb
describe Zombie do
  let(:zombie) { Zombie.create }

  context "#decapitate" do
    it "calls weapon.slice" do
      zombie.weapon.should_receive(:slice)
      zombie.decapitate
    end
  end
end
```

这里的第6行伪造了`weapon`的`slice`方法,并期望这个方法在这个测试中被调用.

你可能会想为什么要这样写,这是因为我们仅仅是要测试`decapitate`这个方法确实调用了`weapon.slice`, 可以把`decapitate`想成下面的黑盒,我们蹲在图中的A点,等着看它会不会去调用`weapon.slice`

{% img /images/test/test_outgoing.jpg %}

这个图是`Sandi Metz`在`RailsConf 2013`上的演讲`The Magic Tricks of Testing`

[视频](http://www.justin.tv/confreaks/c/2247122)

[Slides](https://speakerdeck.com/skmetz/magic-tricks-of-testing-railsconf)

这里注意一下顺序,一般的测试先是执行一个动作,然后再去判断状态或其它东西,像前面`stub`的例子,先调用decapitate方法,再去判断`status`的变化,就好像我踢你一脚,看你会不会喊疼,而这里是先有期望再有动作,这就好比老板对你说这个下周前完成,不然就滚蛋一样


