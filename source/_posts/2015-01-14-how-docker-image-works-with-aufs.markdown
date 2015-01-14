---
layout: post
title: "理解Docker镜像的存储原理"
date: 2015-01-14 15:09
comments: true
categories: docker
---

一开始接触Docker就对他镜像的存储原理很感兴趣,有一个问题一直不明白,Commit一个镜像的时候到底把什么存下来了,只知道他是使用分层的AUFS,具体不是特别很清楚,直到看了这篇关于[Linux AuFS的文章](http://www.thegeekstuff.com/2013/05/linux-aufs/),才有一个自己的认识,对不对还不知道

如果你用过Photoshop,那么用类比的方法就很好理解了,PS里作图一般都是一层一层的,上层如果没有内容就会透过去显示下层的内容,如果有旧覆盖下层内容.假设从下到上有ABC三层,A层上有一个红点,B层上靠右的地方有一个绿点(没有和A层上的重叠),如果只有这两层,那我们看到的就是一个红点加几个绿点,现在在上面有加一层C,他上面有一个蓝点和A层上红点的位置重叠,那么他就会覆盖A层的红点,最后我们看到的就是一个蓝点一个绿点

类比到文件也是这样,假设一个目录`/XXX`由自底向上的ABC三层组成,他们每一层都有一个和他们名字对应的文件(A=>A.txt, ..),同时他们都有一个叫`common.txt`文件,里面的内容分别是A,B,C,那么最终的结果是`/XXX`目录共有4个文件,`A.txt`, `B.txt`, `C.txt`和`common.txt`,而`common.txt`里的内容是C(最上层覆盖下层的)

我们可以用一个例子看看:

首先创建三个目录:

```
mkdir -p /tmp/aufs/root
mkdir -p /tmp/aufs/layer1
mkdir -p /tmp/aufs/layer2
```

创建相应的文件

```
echo 'root' > /tmp/aufs/root/root.txt
echo 'layer1' > /tmp/aufs/layer1/layer1.txt
echo 'layer2' > /tmp/aufs/layer2/layer2.txt

echo 'root' > /tmp/aufs/root/common.txt
echo 'layer1' > /tmp/aufs/root/common.txt
echo 'layer2' > /tmp/aufs/root/common.txt
```

然后我们把他们按顺序Mount起来

```
mount -t aufs -o br=/tmp/aufs/layer2:/tmp/aufs/layer1:/tmp/aufs/root none /tmp/aufs/root
```

br后面的参数顺序是越后面越底层

现在`/tmp/aufs/root/`下应该有4个文件,`root.txt`, `layer1.txt`, `layer2.txt`和`common.txt`,并且`common.txt`里面的内容应该是`layer2`.同时由于现在layer2是最上层,那么`/tmp/aufs/root`里面做修改会反映到`/tmp/aufs/layer2`,在`/tmp/aufs/root`里创建文件,`/tmp/aufs/layer2`里也会出现

再回到前面的问题,Commit的时候到底保存的是什么呢,假设我们要基于前面的三层做修改在提交,只要再在上面加一层layer3,然后对`/tmp/aufs/root`做修改

```
mkdir -p /tmp/aufs/layer3
umount /tmp/aufs/root
mount -t aufs -o br=/tmp/aufs/layer3:/tmp/aufs/layer2:/tmp/aufs/layer1:/tmp/aufs/root none /tmp/aufs/root
```

Note: 没有找到可以一层一层加的方法,只能先umount在按顺序mount

然后再`/tmp/aufs/root`下做修改

```
echo 'layer3' > /tmp/aufs/root/layer3.txt
echo 'layer3' > /tmp/aufs/root/common.txt
```

这时候看`/tmp/aufs/layer3`目录,应该看到会有这两个文件,我们就可以把这个目录打个包实现Commit的目的

把这样的每一层保存下来,并想办法记录他们之间的关系(某一层的上层和下层是谁),差不多就可以实现Docker的镜像存储了,不过不一定对,具体怎么做得看了代码才知道

