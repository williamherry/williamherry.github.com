---
layout: post
title: "virt相关命令总结"
date: 2012-09-14 09:35
comments: true
categories:
---

在CentOS下面多部分命令都是有包`libguestfs-tools-c`提供,所以,首先需要安装它


## virt-ls

virt-ls可以列出虚拟机中目录下的文件或目录,用法如下

```
virt-ls [--options] -d domname dir [dir ...]
virt-ls [--options] -a disk.img [-a disk.img ...] dir [dir ...]
```

如

```
virt-ls -d centos2 /etc/
```

可以像使用`ls`一样加一些参数,如`-l`等,具体请看`virt-ls --help`

## virt-what

virt-what可以用来检测当前系统是不是一个虚拟机,如果不是虚拟机,执行virt-what将不会有任何输出,如果是虚拟机,它会打印一系列关于虚拟机的'facts'(如kvm)

virt-what命令由同名包提供,要命令需要先安装(yum -y install virt-what)

## virt-host-validate

这个命令可以用来检测本机是否正确配置以运行虚拟化,如果没有加参数,它会检查它所知道的所有的虚拟化驱动,可选的可以加`qemu`或`lxc`做限制

```
virt-host-validate
```

输出类似这样

```
  QEMU: Checking for hardware virtualization                       : PASS
  QEMU: Checking for device /dev/kvm                               : PASS
  QEMU: Checking for device /dev/vhost-net                         : PASS
  QEMU: Checking for device /dev/net/tun                           : PASS
   LXC: Checking for Linux >= 2.6.26                               : PASS
```

## virt-top

virt-top命令由同名软件包提供,和top命令相似,只是进程换成了虚拟机

```
yum -y install virt-top
virt-top
```

输出

```
virt-top 16:58:01 - x86_64 8/8CPU 2127MHz 7854MB 12.2% 12.0% 12.8% 12.0% 12.1% 12.0% 12.0% 12.0%
4 domains, 3 active, 3 running, 0 sleeping, 0 paused, 1 inactive D:0 O:0 X:0
CPU: 12.7% Mem: 2048 MB (2048 MB by guests)

  ID S RDRQ WRRQ RXBY TXBY %CPU %MEM    TIME   NAME
  40 R    0    0   52    0 12.5  6.0  66:38.82 centos2
  32 R    0    3   22  38K 13.5 16.0  26:28.82 win2003
  40 R    2    0   52    0 12.5  8.0  36:18.82 test
   -                                           (centos3)
```

## virt-cat

virt-cat可以虚拟机中文件的内容,用法如下

```
virt-cat [--options] -d domname file [file ...]
virt-cat [--options] -a disk.img [-a disk.img ...] file [file ...]
```

如

```
virt-cat -d centos2 /etc/passwd
```

domname可以通过`virsh list`得到

也可以对虚拟机的磁盘文件操作

```
virt-cat  -a /opt/images/centos2.img /etc/passwd
```

## virt-edit

这个命令可以修改

```
virt-edit [--options] -d domname file [file ...]
virt-edit [--options] -a disk.img [-a disk.img ...] file [file ...]
```

例如

```
virt-edit -d centos2 /etc/passwd
```

在我的系统中它会用`vim`打开文件,编辑完保存即可修改虚拟机内的文件内容

也可以直接对虚拟机的磁盘文件进行操作

```
virt-edit -a /opt/images/centos2.img /etc/passwd
```

Note: 如果虚拟机正在运行,使用第一种文件修改它的文件会有下面的报错

```
Libguestfs: error: error: domain is a live virtual machine.
Writing to the disks of a running virtual machine ca cause disk corruption.
Either use read-only access, or if the guest is running the guestfsd daemon
specify live access. In most libguestfs tools these options are --ro or
--live respectively. Consult the documentation for further information.
```

但直接对虚拟机磁盘镜像文件操作不会有这个提示,并且可以修改成功,会不会出问题我就不知道了

## virt-copy-out

virt-copy-out这个命令可以把虚拟机里的文件复制出来, 用法如下

```
virt-copy-out -d domname file|dir [file|dir ...] localdir
virt-copy-out -a disk.img file|dir [file|dir ...] localdir
```

例子

```
virt-copy-out -d centos2 /etc/passwd .
```

可以是多个文件或目录

```
mkdir tmp
virt-copy-out -d centos2 /etc /home /root/.bashrc tmp
```

也可以直接对虚拟机磁盘文件操作,只需要将`-d domname`换成`-a path_of_disk_file`

## virt-copy-in

virt-copy-in是将文件复制到虚拟机里面,用法和virt-copy-out基本相同,这里只举一个例子

```
virt-copy-in -d centos2 test.txt /opt/
```

不出你的所料,如果虚拟机正在运行,上面的命令也会报错

## virt-df

这个命令是比较简单了,就是将在虚拟机中执行`df`命令的结果输出

```
virt-df -d centos2
virt-df -a /opt/images/centos2.img
```

可以加`-h`参数以`human-readable`显示

## virt-alignment-scan

旧的操作系统安装时会使用不对齐的分区,这会引起一些不必要的I/O,这个命令的作用是检查是否正在不对齐的问题,如果存在,只是警告(Warns)你,当前这个工具不会帮你解决这个问题

```
virt-alignment-scan -d centos2
```

输出类似这样

```
/dev/sda1    1048576     1024K   ok
```

## virt-inspector2

这个命令可以显示虚拟机的操作系统版本和其它一些信息,包含的信息非常多,用法非常简单

```
virt-inspector2 -d centos2
```

输出类似这样

```
<?xml version="1.0"?>
<operatingsystems>
  <operatingssystem>
    <root>/dev/sda1</root>
    <name>linux</name>
    <arch>x86_64</arch>
    ...
    there are too many
    ...
```

## virt-resize

- virt-resize可以调整虚拟机磁盘的大小,调整或删除任何分区
- virt-resize不可以就地调整磁盘,不应该对正在运行的虚拟机进行磁盘调整,为了确保一致性,调整先需要关闭虚拟机
- virt-resize调整的过程非常慢,从35G的磁盘进行扩展需要差不多10分钟
- virt-resize调整所花的时候只和开始磁盘的大小有关,从35G扩展到40G和扩展到135G所花的时间差不多
- 如果你使用qcow2磁盘格式,个人建议先转成raw,调整完后再转回去,因为直接对qcow2做调整,比较35G的qcow2磁盘镜像文件可能只有1G大小(ls查看),通过virt-resize调整后就会变成35G大小了(ls查看)(也可能是我的方法不对),先转成raw调整完大小后再转回去可以避免这个问题

概要

```
virt-resize [--resize /dev/sdaN=[+/-]<size>[%]]
  [--expand /dev/sdaN] [--shrink /dev/sdaN]
  [--ignore /dev/sdaN] [--delete /dev/sdaN] [...] indisk outdisk
```

- 示例1.给一个分区增加5G

首先关闭虚拟机

```
virsh destroy centos2
```

查看分区情况

```
virt-filesystems --long -h --all -a /opt/images/centos2.img
```

```
Name      Type        VFS       Label     MBR     Size    Parent
/dev/sda1 filesystems ext4      -         -       35G     -
/dev/sda1 partition   -         -         83      35G     /dev/sda
/dev/sda  device      -         -         -       35G     -
```

把qcow2格式的磁盘镜像转成raw

```
cd /opt/images
cp centos2.img centos2.img.orig
qemu-img convert -f qcow2 -O raw centos2.img centos2.raw
```

利用truncate创建一个新的文件,大小比centos2.raw大5G

```
truncate -r centos2.raw newdisk
truncate -s +5G newdisk
```

开始调整

```
virt-resize --expand /dev/sda1 centos2.raw newdisk
```

你应该看到类似这样的信息

```
Examining centos2.img.raw ...

Summary of changes:

/dev/sda1: This partition will be resized from 35.0G to 40.0G. The
    filesystem ext4 on /dev/sda1 will be expanded using the 'resize2fs'
    method.

**********
Setting up initial partition table on newdisk ...
Copying /dev/sda1 ...
```

然后是持续好久的刷屏信息,好在有倒计时

最后应该看到类似下面的信息

```
Expanding /dev/sda1 using the 'resize2fs' method ...

Resize operation completed with no errors. Before deleting the old
disk, carefully check that the resized disk boots and works correctly.
```

调整完后转回qcow2格式

```
qemu-img convert -f raw -O qcow2 centos2.raw centos2.qcow2
```

虚拟机里面不用再做操作,现在使用新的磁盘镜像文件启动虚拟机应该可以看到/dev/sda1已经从35G变为40G了

分区的缩减我们一般用不到,没有做测试,lvm的调整可以参考[这里](http://askaralikhan.blogspot.tw/2011/07/expanding-kvm-guest-disk-image-using.html)

## virt-install

virt-install命令由`python-virtinst`提供,需要安装python-virtinst才可以使用virt-install

```
yum -y install python-virtinst
```

安装例子

    virt-install \
        --name kvm-test-centos-6.2-x64 \
        --ram 1024 \
        --vcpus 4 \
        --cdrom /opt/isos/CentOS-6.2-x86_64-bin-DVD1.iso \
        --network bridge:virbr0 \
        --vnc --vncport=5910 --vnclisten=localhost \
        --disk /opt/images/kvm-test-centos-6.2-64bit.img,size=20

如果磁盘文件不存在,会自动创建(需要size参数),默认创建是raw磁盘镜像,可以用qemu-img手动创建磁盘文件

    qemu-img create -f raw kvm-test-centos-6.2-64bit.img 20G

这里磁盘格式常见的有两种,raw和qcow2(还有qed正在开发中,据说性能更好)
raw的读写性能要比qcow2好,但如果你需要快照等高级特性,可以选择qcow2
如果使用qcow2,加上preallocation性能会有所提升

    qemu-img create -f qcow2 -o preallocation=metadata kvm-test-centos-6.2-64bit.img 20G

不同的bus类型,cache类型和aio选择都会有性能有影响,所以你可希望把这些也加进去,格式类似这样

    ...
        --disk path=/opt/images/kvm-test-centos-6.2-64bit.img,size=20,bus=virtio,cache=none,aio=threads,format=qcow2 \
    ...

这里可供选择的参数有

    bus: virtio, ide
    cache: unsafe, none, writeback, writethrough, directsync
    aio: threads, native

    不同的网卡驱动类型会影响到虚拟机的网络性能,可以这样指定

    ...
        --network bridge:virbr0,model=e1000 \
    ...

可供选择的参数有

    # 可以通过qemu-system-x86_64 -net nic,model=?查到支持的参数
    model:  ne2k_pci,i82551,i82557b,i82559er,rtl8139,e1000,pcnet,virtio

virbr0是系统自动创建的桥,可以手动创建桥接设备然后指定虚拟机使用
如果你的虚拟机是windows并且想使用virtio做为硬盘驱动和网卡驱动,你需要下载两个驱动文件
这两个文件可以从这里下载

[磁盘驱动](http://ishare.iask.sina.com.cn/f/24109437.html)

[网卡驱动](http://alt.fedoraproject.org/pub/alt/virtio-win/latest/images/bin/)

    virt-install \
        --name kvm-test-win2003 \
        --ram 1024 \
        --vcpus 1 \
        --cdrom /opt/isos/cn_win_srv_2003_r2_enterprise_x64_with_sp2_vl_cd1_X13-47314.iso \
        --network bridge:virbr0,model=virtio \
        --vnc \
        --disk path=/opt/images/kvm-test-win2003.img,bus=virtio,cache=none,format=qcow2,size=20 \
        --disk path=/opt/drivers/virtio-win-1.1.16.vfd,device=floppy \
        --disk path=/opt/drivers/virtio-win-0.1-12.iso,device=cdrom,perms=ro

如果不出问题,应该可以看到安装窗口了(需要安装virt-viewer)
如果没有安装virt-viewer或者你在没有安装图形的服务器上操作
你仍然可以通过以下方法访问到安装窗口

    vncviewer vnclisten:vncport

前面的–vnclisten就不能写localhost了

第二种方法是:

    virt-viewer --connect qemu+ssh://user@ip_address:port/system name_of_instance

kickstart安装也可以用在virt-install中

    yum -y install httpd
    mkdir /var/www/centos
    mount -o loop /opt/isos/CentOS-6.2-x86_64-bin-DVD1.iso /var/www/centos
    cp /root/ks.cfg /var/www/centos
    /etc/init.d/httpd restart

    virt-install \
        --name kvm-test-centos-6.2-x64 \
        --ram 1024 \
        --vcpus 4 \
        --location http://192.168.122.1/centos/
        --network bridge:virbr0 \
        --vnc --vncport=5910 --vnclisten=localhost \
        --disk /opt/images/kvm-test-centos-6.2-64bit.img,size=20
        --extra-args "ks=http://192.168.122.1/ks.cfg ip=192.168.122.10 ip=192.168.122.10 netmask=255.255.255.0 gateway=192.168.122.1 dns=8.8.8.8"

安装好之后,就可以使用virsh对虚拟做一些操作了
virt-install的更多参数可以参考[这里](http://my.oschina.net/williamherrychina/blog/56463)


