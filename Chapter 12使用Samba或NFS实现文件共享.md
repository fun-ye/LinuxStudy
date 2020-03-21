# Chapter 12使用Samba或NFS实现文件共享

[TOC]

## 12.1 SAMBA文件共享服务

SMB（Server Messages Block，服务器消息块）协议，Samba服务程序现在已经成为在Linux系统与Windows系统之间共享文件的最佳选择。

```
# yum install samba
# cat /etc/samba/smb.conf
```

表12-1                   Samba服务程序中的参数以及作用

| [global]   |                                                              | #全局参数。                                         |
| ---------- | ------------------------------------------------------------ | --------------------------------------------------- |
|            | workgroup = MYGROUP                                          | #工作组名称                                         |
|            | server string = Samba Server Version %v                      | #服务器介绍信息，参数%v为显示SMB版本号              |
|            | log file = /var/log/samba/log.%m                             | #定义日志文件的存放位置与名称，参数%m为来访的主机名 |
|            | max log size = 50                                            | #定义日志文件的最大容量为50KB                       |
|            | security = user                                              | #安全验证的方式，总共有4种                          |
|            | #share：来访主机无需验证口令；比较方便，但安全性很差         |                                                     |
|            | #user：需验证来访主机提供的口令后才可以访问；提升了安全性    |                                                     |
|            | #server：使用独立的远程主机验证来访主机提供的口令（集中管理账户） |                                                     |
|            | #domain：使用域控制器进行身份验证                            |                                                     |
|            | passdb backend = tdbsam                                      | #定义用户后台的类型，共有3种                        |
|            | #smbpasswd：使用smbpasswd命令为系统用户设置Samba服务程序的密码 |                                                     |
|            | #tdbsam：创建数据库文件并使用pdbedit命令建立Samba服务程序的用户 |                                                     |
|            | #ldapsam：基于LDAP服务进行账户验证                           |                                                     |
|            | load printers = yes                                          | #设置在Samba服务启动时是否共享打印机设备            |
|            | cups options = raw                                           | #打印机的选项                                       |
| [homes]    |                                                              | #共享参数                                           |
|            | comment = Home Directories                                   | #描述信息                                           |
|            | browseable = no                                              | #指定共享信息是否在“网上邻居”中可见                 |
|            | writable = yes                                               | #定义是否可以执行写入操作，与“read only”相反        |
| [printers] |                                                              | #打印机共享参数                                     |
|            | comment = All Printers                                       |                                                     |
|            | path = /var/spool/samba                                      | #共享文件的实际路径(重要)。                         |
|            | browseable = no                                              |                                                     |
|            | guest ok = no                                                | #是否所有人可见，等同于"public"参数。               |
|            | writable = no                                                |                                                     |
|            | printable = yes                                              |                                                     |

使用cat[命令](https://www.linuxcool.com/)读入主配置文件，再在grep[命令](https://www.linuxcool.com/)后面添加-v参数（反向选择），分别去掉所有以井号（#）和分号（;）开头的注释信息行，对于剩余的空白行可以使用^$参数来表示并进行反选过滤，最后把过滤后的可用参数信息通过重定向符覆盖写入到原始文件名称中。

```
# mv /etc/samba/smb.conf /etc/samba/smb.conf.bak
# cat /etc/samba/smb.conf.bak | grep - v "#" | grep -v ";" | grep -v "^$"
```

### 12.1.1 配置共享资源

全局配置参数用于设置整体的资源共享环境，对里面的每一个独立的共享资源都有效。区域配置参数则用于设置单独的共享资源，且仅对该资源有效。

表12-2  用于设置Samba服务程序的参数以及作用

| 参数                                                  | 作用                       |
| ----------------------------------------------------- | -------------------------- |
| [database]                                            | 共享名称为database         |
| comment = Do not arbitrarily modify the database file | 警告用户不要随意修改数据库 |
| path = /home/database                                 | 共享目录为/home/database   |
| public = no                                           | 关闭“所有人可见”           |
| writable = yes                                        | 允许写入操作               |

**第1步**：创建用于访问共享资源的账户信息。在RHEL 7系统中，Samba服务程序默认使用的是用户口令认证模式（user）。这种认证模式可以确保仅让有密码且受信任的用户访问共享资源，只有建立账户信息数据库之后，才能使用用户口令认证模式。

pdbedit命令用于管理SMB服务程序的账户信息数据库，格式为“pdbedit [选项] 账户”。

表12-3                    用于pdbedit命令的参数以及作用

| 参数      | 作用                   |
| --------- | ---------------------- |
| -a 用户名 | 建立Samba用户          |
| -x 用户名 | 删除Samba用户          |
| -L        | 列出用户列表           |
| -Lv       | 列出用户详细信息的列表 |

```
# id linuxpro
# pdbedit -a -u linuxpro
```

**第2步**：创建用于共享资源的文件目录。在创建时，不仅要考虑到文件读写权限的问题，而且由于/home目录是系统中普通用户的家目录，因此还需要考虑应用于该目录的SELinux`安全上下文`所带来的限制。

```
# mkdir /home/database
# chown -Rf linux:linux /home/database
# semanage fcontext -a -t samda_share_t /home/database //设置安全上下文
# restorecon -Rv /home/database		//立即生效
```

**第3步**：设置SELinux服务与策略，使其允许通过Samba服务程序访问普通用户家目录。执行getsebool命令，筛选出所有与Samba服务程序相关的SELinux域策略，根据策略的名称（和经验）选择出正确的策略条目进行开启即可：

```
# getsebool -a | grep samba
```

**第4步**：在Samba服务程序的主配置文件中，根据表12-2所提到的格式写入共享信息

```
# vim /etc/samba/smb.conf
```

**第5步**：Samba服务程序的配置工作基本完毕。接下来重启smb服务并清空iptables防火墙，

```
[root@linuxprobe ~]# systemctl restart smb
[root@linuxprobe ~]# systemctl enable smb
[root@linuxprobe ~]# iptables -F
[root@linuxprobe ~]# service iptables save
```

### 12.1.2 Windows挂载共享

无论Samba共享服务是部署Windows系统上还是部署在Linux系统上，通过Windows系统进行访问时，其步骤和方法都是一样的。

表12-4        Samba服务器和Windows客户端使用的操作系统以及IP地址



| 主机名称        | 操作系统  | IP地址        |
| --------------- | --------- | ------------- |
| Samba共享服务器 | RHEL 7    | 192.168.10.10 |
| Linux客户端     | RHEL 7    | 192.168.10.20 |
| Windows客户端   | Windows 7 | 192.168.10.30 |

### 12.1.3 Linux挂载共享

```
# yum install cifs-utils
Samba服务的用户名、密码、共享域的顺序将相关信息写入到一个认证文件中
[root@linuxprobe ~]# vim auth.smb
username=linuxprobe
password=redhat
domain=MYGROUP
[root@linuxprobe ~]# chmod -Rf 600 auth.smb
在Linux客户端上创建一个用于挂载Samba服务共享资源的目录，并把挂载信息写入到/etc/fstab文件中，以确保共享挂载信息在服务器重启后依然生效：
[root@linuxprobe ~]# mkdir /database
[root@linuxprobe ~]# vim /etc/fstab
进入到挂载目录/database后就可以看到Windows系统访问Samba服务程序时留下来的文件了（即文件Memo.txt）
[root@linuxprobe ~]# cat /database/Memo.txt
```

## 12.2 NFS网络文件系统

NFS（网络文件系统）服务可以将远程Linux系统上的文件共享资源挂载到本地主机的目录上，从而使得本地主机（Linux客户端）基于TCP/IP协议，像使用本地主机上的资源那样读写远程Linux系统上的共享文件。

```
# yum install nfs-utils
```

**第1步**：为了检验NFS服务配置的效果，我们需要使用两台Linux主机

**第2步**：在NFS服务器上建立用于NFS文件共享的目录，并设置足够的权限确保其他人也有写入权限。

```
[root@linuxprobe ~]# mkdir /nfsfile
[root@linuxprobe ~]# chmod -Rf 777 /nfsfile
```

**第3步**：NFS服务程序的配置文件为/etc/exports，默认情况下里面没有任何内容。我们可以按照“共享目录的路径 允许访问的NFS客户端（共享权限参数）”的格式，定义要共享的目录与相应的权限。

表12-7                 用于配置NFS服务程序配置文件的参数

| 参数           | 作用                                                         |
| -------------- | ------------------------------------------------------------ |
| ro             | 只读                                                         |
| rw             | 读写                                                         |
| root_squash    | 当NFS客户端以root管理员访问时，映射为NFS服务器的匿名用户     |
| no_root_squash | 当NFS客户端以root管理员访问时，映射为NFS服务器的root管理员   |
| all_squash     | 无论NFS客户端使用什么账户访问，均映射为NFS服务器的匿名用户   |
| sync           | 同时将数据写入到内存与硬盘中，保证不丢失数据                 |
| async          | 优先将数据保存到内存，然后再写入硬盘；这样效率更高，但可能会丢失数据 |

```
[root@linuxprobe ~]# vim /etc/exports
/nfsfile 192.168.10.*(rw,sync,root_squash)
```

**第4步**：启动和启用NFS服务程序。由于在使用NFS服务进行文件共享之前，需要使用RPC（Remote Procedure Call，远程过程调用）服务将NFS服务器的IP地址和端口号等信息发送给客户端。因此，在启动NFS服务之前，还需要顺带重启并启用rpcbind服务程序，并将这两个服务一并加入开机启动项中。

```
[root@linuxprobe ~]# systemctl restart rpcbind
[root@linuxprobe ~]# systemctl enable rpcbind
[root@linuxprobe ~]# systemctl start nfs-server
[root@linuxprobe ~]# systemctl enable nfs-server
```

使用showmount命令（以及必要的参数，见表12-8）查询NFS服务器的远程共享信息，其输出格式为“共享的目录名称 允许使用客户端地址”。

表12-8                 showmount命令中可用的参数以及作用

| 参数 | 作用                                      |
| ---- | ----------------------------------------- |
| -e   | 显示NFS服务器的共享列表                   |
| -a   | 显示本机挂载的文件资源的情况NFS资源的情况 |
| -v   | 显示版本号                                |

```
# showmount -e 192.168.10.10
在NFS客户端创建一个挂载目录
[root@linuxprobe ~]# mkdir /nfsfile
[root@linuxprobe ~]# mount -t nfs 192.168.10.10:/nfsfile /nfsfile
[root@linuxprobe ~]# vim /etc/fstab 	//永久有限
192.168.10.10:/nfsfile /nfsfile nfs defaults 0 0
```

## 12.3 AutoFs自动挂载服务

mount命令不同，autofs服务程序是一种Linux系统守护进程，当检测到用户试图访问一个尚未挂载的文件系统时，将自动挂载该文件系统。

```
[root@linuxprobe ~]# yum install autofs
```

autofs服务程序的主配置文件中需要按照“挂载目录 子配置文件”的格式进行填写。挂载目录是设备挂载位置的上一级目录。

```
[root@linuxprobe ~]# vim /etc/auto.master
/media /etc/iso.misc  //挂载子目录
```

在子配置文件中，应按照“挂载目录 挂载文件类型及权限 :设备名称”的格式进行填写

```
[root@linuxprobe ~]# vim /etc/iso.misc
iso   -fstype=iso9660,ro,nosuid,nodev :/dev/cdrom
[root@linuxprobe ~]# systemctl start autofs 
[root@linuxprobe ~]# systemctl enable autofs 
```