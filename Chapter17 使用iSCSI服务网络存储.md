# Chapter17 使用iSCSI服务网络存储

[TOC]

## 17.1 iSCSI技术介绍

当前的硬盘接口类型主要有IDE、SCSI和SATA这3种。

> IDE是一种成熟稳定、价格便宜的并行传输接口。
>
> SATA是一种传输速度更快、数据校验更完整的串行传输接口。
>
> SCSI是一种用于计算机和硬盘、光驱等设备之间系统级接口的通用标准，具有系统资源占用率低、转速高、传输速度快等优点

互联网小型计算机系统接口（iSCSI，Internet Small Computer System Interface）。这是一种将SCSI接口与以太网技术相结合的新型存储技术，可以用来在网络中传输SCSI接口的[命令](https://www.linuxcool.com/)和数据。这样，不仅克服了传统SCSI接口设备的物理局限性，实现了跨区域的存储资源共享，还可以在不停机的状态下扩展存储容量。

## 17.2 创建RAID磁盘列阵

使用mdadm命令创建RAID磁盘阵列。其中，-Cv参数为创建阵列并显示过程，/dev/md0为生成的阵列组名称，-n 3参数为创建RAID 5磁盘阵列所需的硬盘个数，-l 5参数为RAID磁盘阵列的级别，-x 1参数为磁盘阵列的备份盘个数。在命令后面要逐一写上使用的硬盘名称。

```
# mdadm -Cv /dev/md0 -n 3 -l 5 -x 1 /dev/sdb /dev/sdc /dev/sdd /dev/sde
# mdadm -D /dev/md0	//查看设备的详细信息
```

## 17.3 配置iSCSI服务端

iSCSI技术在工作形式上分为服务端（target）与客户端（initiator）。iSCSI服务端即用于存放硬盘存储资源的服务器，它作为前面创建的RAID磁盘阵列的存储端，能够为用户提供可用的存储资源。iSCSI客户端则是用户使用的软件，用于访问远程服务端的存储资源

```
# yum -y install targetd targetcli //-y参数，在安装过程中就不需要再进行手动确认了
# systemctl start targetd
# systemctl enable targetd
```

配置iSCSI服务端共享资源。targetcli是用于管理iSCSI服务端存储资源的专用配置命令，它能够提供类似于fdisk命令的交互式配置功能，将iSCSI共享资源的配置内容抽象成“目录”的形式

在执行targetcli命令后就能看到交互式的配置界面了。在该界面中可以使用很多Linux命令，比如利用ls查看目录参数的结构，使用cd切换到不同的目录中。/backstores/block是iSCSI服务端配置共享设备的位置

```
# targetcli
```

创建iSCSI target名称及配置共享资源。iSCSI target名称是由系统自动生成的，这是一串用于描述共享资源的唯一字符串.会在/iscsi参数目录中创建一个与其字符串同名的新“目录”用来存放共享资源。

设置访问控制列表（ACL）。

设置iSCSI服务端的监听IP地址和端口号。

配置妥当后检查配置信息，重启iSCSI服务端程序并配置防火墙策略。

## 17.4 配置Linux客户端

```
[root@linuxprobe ~]# yum install iscsi-initiator-utils 
# vim /etc/iscsi/initiatorname.iscsi
# systemctl restart iscsid
# systemctl enable iscsid
```

iscsiadm是用于管理、查询、插入、更新或删除iSCSI数据库配置文件的命令行工具，用户需要先使用这个工具扫描发现远程iSCSI服务端，然后查看找到的服务端上有哪些可用的共享存储资源。其中，-m discovery参数的目的是扫描并发现可用的存储资源，-t st参数为执行扫描操作的类型，-p 192.168.10.10参数为iSCSI服务端的IP地址：

```
# iscsiadm -m discovery -t st -p 192.168.10.10
```

接下来准备登录iSCSI服务端。其中，-m node参数为将客户端所在主机作为一台节点服务器，-T iqn.2003-01. org.linux-iscsi.linuxprobe.x8664:sn.d497c356ad80参数为要使用的存储资源（大家可以直接复制前面命令中扫描发现的结果，以免录入错误），-p 192.168.10.10参数依然为对方iSCSI服务端的IP地址。最后使用--login或-l参数进行登录验证。

```
# iscsiadm -m node -T iqn.2003-01.org.linux-iscsi.linuxprobe.x8664:sn.d497c356ad80 -p 192.168.10.10 --login
```

```
# file /dev/sdb 
# mkfs.xfs /dev/sdb
# blkid | grep /dev/sdb
# vim /etc/fstab
# iscsiadm -m node -T iqn.2003-01.org.linux-iscsi.linuxprobe.x8664:sn.d497c356ad80 -u
```

## 17.5 配置Windows客户端

