---
layout: post
title: "定制Linux发行版(CentOS-6.2)"
date: 2012-09-09 01:18
comments: true
categories:
---

主要内容:

- 基本的定制过程
- 改变ISO中包含的软件包
- 集成Kickstart
- 更换背景图片

## 基本的定制过程

ISO的定制简单来说就是将ISO挂载,修改里面的内容然后打包回去,这一节我们不修改ISO里的内容,只熟悉一个这个过程

- 安装需要要的软件包

```
yum -y installl genisoimage createrepo
```

- 挂载ISO

```
mount -o loop /opt/CentOS-6.2-x86_64-bin-DVD1.iso /media/
```

- 将所有人文件复制到一个目录

```
cp -r /media mycentos
```

- 做修改

这里我们不做修改,只熟悉这个过程

- 重新生成软件包列表

如果添加或删除了ISO里包含的软件包,需要运行下面命令重新生成列表(注意下面命令最后的`.`)

```
cd mycentos
createrepo -v -g repodata/3a27232698a261aa4022fd270797a3006aa8b8a346cbd6a31fae1466c724d098-c6-x86_64-comps.xml .
```

上面的很长的文件名可以从`repodata/repomd.xml`文件里找到,在这个文件中寻找`type="group"`

```
...
 <data type="group">
    <location xml:base="media://1323560292.885204#1" href="repodata/3a27232698a261aa4022fd270797a3006aa8b8a346cbd6a31fae1466c724d098-c6-x86_64-comps.xml"/>
    <checksum type="sha256">3a27232698a261aa4022fd270797a3006aa8b8a346cbd6a31fae1466c724d098</checksum>
    <timestamp>1324003565</timestamp>
 </data>
...
```

- 重新打包回去

运行下面命令重新生成修改过的ISO保存到`/tmp`下(注意最后面的`.`)

```
cd mycentos
mkisofs -joliet-long -T -o /tmp/mycentos.iso -b isolinux/isolinux.bin -c isolinux/boot.cat -no-emul-boot -boot-load-size 4 -boot-info-table -R -m TRANS.TBL .
```

## 改变ISO中包含的软件包

如果觉得CentOS的ISO太大,可以对里面的软件包做一些删减,只包含最小化安装需要的包可能是一个好选择,在一个最小化安装的系统中`/root/install.log`里包含了最小化安装所需要的包,所以这一节很简单,在Packages目录下只留下需要的包,重新生成软件列表,重新打包

- 只复制需要要软件包

```
cd mycentos
rm -rf Packages/*
for i in `cat /root/install.log | grep Installing | sed 's/Installing //'`; do cp -v /media/Packages/$i* Packages/; done
```

- 重新生成软件包列表


```
cd mycentos
createrepo -v -g repodata/3a27232698a261aa4022fd270797a3006aa8b8a346cbd6a31fae1466c724d098-c6-x86_64-comps.xml .
```

- 重新打包回去

```
cd mycentos
mkisofs -joliet-long -T -o /tmp/mycentos.iso -b isolinux/isolinux.bin -c isolinux/boot.cat -no-emul-boot -boot-load-size 4 -boot-info-table -R -m TRANS.TBL .
```

如果要需要一些自己制作的软件包,需要把他们依赖的包也复制到Packages目录里

## 集成Kickstart

集成Kickstart比较简单,将做好的Kickstart文件复制到ISO里(可以是isolinux下),然后修改`isolinux/isolinux.cfg`文件

```
label autoinstall
  menu label ^Auto install
  menu default
  kernel vmlinuz
  append initrd=initrd.img ks=cdrom:/isolinux/ks.cfg
```

重新打包即可

## 更换背景图片


