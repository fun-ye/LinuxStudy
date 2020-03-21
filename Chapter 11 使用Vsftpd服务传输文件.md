# Chapter 11 使用Vsftpd服务传输文件

[TOC]

## 11.1 文件传输协议

FTP是一种在互联网中进行文件传输的协议，基于客户端/服务器模式，默认使用20、21号端口，其中端口20（数据端口）用于进行数据传输，端口21（[命令](https://www.linuxcool.com/)端口）用于接受客户端发出的相关FTP[命令](https://www.linuxcool.com/)与参数。FTP服务器普遍部署于内网中，具有容易搭建、方便管理的特点。

<img src="C:\Users\方也\AppData\Roaming\Typora\typora-user-images\image-20200317225555661.png" alt="image-20200317225555661" style="zoom:67%;" />

FTP服务器是按照FTP协议在互联网上提供文件存储和访问服务的主机，FTP客户端则是向服务器发送连接请求，以建立数据传输链路的主机。FTP协议有下面两种工作模式。

> **主动模式**：FTP服务器主动向客户端发起连接请求。
>
> **被动模式**：FTP服务器等待客户端发起连接请求（FTP的默认工作模式）。

vsftpd（very secure ftp daemon，非常安全的FTP守护进程）是一款运行在Linux操作系统上的FTP服务程序

表11-1                   vsftpd服务程序常用的参数以及作用

| 参数                                              | 作用                                                         |
| ------------------------------------------------- | ------------------------------------------------------------ |
| listen=[YES\|NO]                                  | 是否以独立运行的方式监听服务                                 |
| listen_address=IP地址                             | 设置要监听的IP地址                                           |
| listen_port=21                                    | 设置FTP服务的监听端口                                        |
| download_enable＝[YES\|NO]                        | 是否允许下载文件                                             |
| userlist_enable=[YES\|NO] userlist_deny=[YES\|NO] | 设置用户列表为“允许”还是“禁止”操作                           |
| max_clients=0                                     | 最大客户端连接数，0为不限制                                  |
| max_per_ip=0                                      | 同一IP地址的最大连接数，0为不限制                            |
| anonymous_enable=[YES\|NO]                        | 是否允许匿名用户访问                                         |
| anon_upload_enable=[YES\|NO]                      | 是否允许匿名用户上传文件                                     |
| anon_umask=022                                    | 匿名用户上传文件的umask值                                    |
| anon_root=/var/ftp                                | 匿名用户的FTP根目录                                          |
| anon_mkdir_write_enable=[YES\|NO]                 | 是否允许匿名用户创建目录                                     |
| anon_other_write_enable=[YES\|NO]                 | 是否开放匿名用户的其他写入权限（包括重命名、删除等操作权限） |
| anon_max_rate=0                                   | 匿名用户的最大传输速率（字节/秒），0为不限制                 |
| local_enable=[YES\|NO]                            | 是否允许本地用户登录FTP                                      |
| local_umask=022                                   | 本地用户上传文件的umask值                                    |
| local_root=/var/ftp                               | 本地用户的FTP根目录                                          |
| chroot_local_user=[YES\|NO]                       | 是否将用户权限禁锢在FTP目录，以确保安全                      |
| local_max_rate=0                                  | 本地用户最大传输速率（字节/秒），0为不限制                   |

## 11.2 Vsftpd服务程序

**匿名开放模式**：是一种最不安全的认证模式，任何人都可以无需密码验证而直接登录到FTP服务器。

**本地用户模式**：是通过[Linux系统](https://www.linuxprobe.com/)本地的账户密码信息进行认证的模式，相较于匿名开放模式更安全，而且配置起来也很简单。但是如果被黑客破解了账户的信息，就可以畅通无阻地登录FTP服务器，从而完全控制整台服务器。

**虚拟用户模式**：是这三种模式中最安全的一种认证模式，它需要为FTP服务单独建立用户数据库文件，虚拟出用来进行口令验证的账户信息，而这些账户信息在服务器系统中实际上是不存在的，仅供FTP服务程序进行认证使用。这样，即使黑客破解了账户信息也无法登录服务器，从而有效降低了破坏范围和影响。

### 11.2.1 匿名访问模式

这种模式一般用来访问不重要的公开文件（在生产环境中尽量不要存放重要文件）

表11-2                 可以向匿名用户开放的权限参数以及作用

| 参数                        | 作用                               |
| --------------------------- | ---------------------------------- |
| anonymous_enable=YES        | 允许匿名访问模式                   |
| anon_umask=022              | 匿名用户上传文件的umask值          |
| anon_upload_enable=YES      | 允许匿名用户上传文件               |
| anon_mkdir_write_enable=YES | 允许匿名用户创建目录               |
| anon_other_write_enable=YES | 允许匿名用户修改目录名称或删除目录 |

开启匿名模式：

```
[root@linuxprobe ~]# vim /etc/vsftpd/vsftpd.conf
1 anonymous_enable=YES
2 anon_umask=022
3 anon_upload_enable=YES
4 anon_mkdir_write_enable=YES
5 anon_other_write_enable=YES
# ftp 192.168.10.10
```

### 11.2.2 本地用户模式

表11-3                  本地用户模式使用的权限参数以及作用

| 参数                | 作用                                              |
| ------------------- | ------------------------------------------------- |
| anonymous_enable=NO | 禁止匿名访问模式                                  |
| local_enable=YES    | 允许本地用户模式                                  |
| write_enable=YES    | 设置可写权限                                      |
| local_umask=022     | 本地用户模式创建文件的umask值                     |
| userlist_deny=YES   | 启用“禁止用户名单”，名单文件为ftpusers和user_list |
| userlist_enable=YES | 开启用户作用名单文件功能                          |

vsftpd服务程序所在的目录中默认存放着两个名为“用户名单”的文件（ftpusers和user_list）

```
# vim /etc/vsftpd/user_list    //禁止访问名单
# cat /etc/vsftpd/ftpusers
# getsebool -a | grep ftp	//启SELinux域中对FTP服务的允许策略
ftpd_full_access --> off
# setsebool -P ftpd_full_acess=on
```

### 11.2.3 虚拟用户模式

```
第1步：创建用于进行FTP认证的用户数据库文件，其中奇数行为账户名，偶数行为密码。
# cd /etc/vsftpd/
# vim vuser.list
zhangsan
redhat
用db_load命令用哈希（hash）算法将原始的明文信息文件转换成数据库文件，并且降低数据库文件的权限（避免其他人看到数据库文件的内容），然后再把原始的明文信息文件删除。
# db_load -T -t hash -f vuser.list vuser.db 
# file vuser.db
# chmod 600 vuser.db
# rm -f vuser.list
第2步：创建vsftpd服务程序用于存储文件的根目录以及虚拟用户映射的系统本地用户。
# useradd -d /var/ftproot -s /sbin/nologin virtual
# ls -d /var/ftproot/
# chmod -Rf 755 /var/ftproot
第3步：建立用于支持虚拟用户的PAM文件。
PAM（可插拔认证模块）是一种认证机制，通过一些动态链接库和统一的API把系统提供的服务与认证方式分开，使得系统管理员可以根据需求灵活调整服务程序的不同认证方式
```

通俗来讲，PAM是一组安全机制的模块，系统管理员可以用来轻易地调整服务程序的认证方式，而不必对应用程序进行任何修改。PAM采取了分层设计（应用程序层、应用接口层、鉴别模块层）的思想，其结构如图11-2所示。

<img src="C:\Users\方也\AppData\Roaming\Typora\typora-user-images\image-20200319103323190.png" alt="image-20200319103323190" style="zoom: 67%;" />

新建一个用于虚拟用户认证的PAM文件vsftpd.vu，其中PAM文件内的“db=”参数为使用db_load命令生成的账户密码数据库文件的路径，但不用写数据库文件的后缀：

```
# vim /etc/pam.d/vsftpd.vu
auth       required     pam_userdb.so db=/etc/vsftpd/vuser
account    required     pam_userdb.so db=/etc/vsftpd/vuser
```

**第4步**：在vsftpd服务程序的主配置文件中通过pam_service_name参数将PAM认证文件的名称修改为vsftpd.vu，PAM作为应用程序层与鉴别模块层的连接纽带，可以让应用程序根据需求灵活地在自身插入所需的鉴别功能模块。当应用程序需要PAM认证时，则需要在应用程序中定义负责认证的PAM配置文件，实现所需的认证功能。

表11-4              利用PAM文件进行认证时使用的参数以及作用

| 参数                       | 作用                                                        |
| -------------------------- | ----------------------------------------------------------- |
| anonymous_enable=NO        | 禁止匿名开放模式                                            |
| local_enable=YES           | 允许本地用户模式                                            |
| guest_enable=YES           | 开启虚拟用户模式                                            |
| guest_username=virtual     | 指定虚拟用户账户                                            |
| pam_service_name=vsftpd.vu | 指定PAM文件                                                 |
| allow_writeable_chroot=YES | 允许对禁锢的FTP根目录执行写入操作，而且不拒绝用户的登录请求 |

```
[root@linuxprobe ~]# vim /etc/vsftpd/vsftpd.conf
1 anonymous_enable=NO
2 local_enable=YES
3 guest_enable=YES
4 guest_username=virtual
5 allow_writeable_chroot=YES
14 pam_service_name=vsftpd.vu
```

**第5步**：为虚拟用户设置不同的权限。

```
[root@linuxprobe ~]# mkdir /etc/vsftpd/vusers_dir/
[root@linuxprobe ~]# cd /etc/vsftpd/vusers_dir/
[root@linuxprobe vusers_dir]# touch lisi
[root@linuxprobe vusers_dir]# vim zhangsan
anon_upload_enable=YES
anon_mkdir_write_enable=YES
anon_other_write_enable=YES
[root@linuxprobe ~]# vim /etc/vsftpd/vsftpd.conf
user_config_dir=/etc/vsftpd/vusers_dir
[root@linuxprobe ~]# systemctl restart vsftpd
[root@linuxprobe ~]# systemctl enable vsftpd
```

**第6步**：设置SELinux域允许策略，然后使用虚拟用户模式登录FTP服务器

```
[root@linuxprobe ~]# getsebool -a | grep ftp
[root@linuxprobe ~]# setsebool -P ftpd_full_access
```

## 11.3 TFTP简单文件传输协议

简单文件传输协议（Trivial File Transfer Protocol，TFTP）是一种基于UDP协议在客户端和服务器之间进行简单文件传输的协议。

TFTP在传输文件时采用的是`UDP`协议，占用的端口号为`69`，因此文件的传输过程也不像FTP协议那样可靠。但是，因为TFTP不需要客户端的权限认证，也就减少了无谓的系统和网络带宽消耗，因此在传输琐碎（trivial）不大的文件时，效率更高。

```
[root@linuxprobe ~]# yum install tftp-server tftp
[root@linuxprobe ~]# vim /etc/xinetd.d/tftp
  disable                 = no
  [root@linuxprobe ~]# systemctl restart xinetd
[root@linuxprobe ~]# systemctl enable xinetd
[root@linuxprobe ~]# firewall-cmd --permanent --add-port=69/udp
success
[root@linuxprobe ~]# firewall-cmd --reload 
success
```

TFTP的根目录为/var/lib/tftpboot

表11-5                     tftp命令中可用的参数以及作用

| 命令    | 作用                |
| ------- | ------------------- |
| ?       | 帮助信息            |
| put     | 上传文件            |
| get     | 下载文件            |
| verbose | 显示详细的处理信息  |
| status  | 显示当前的状态信息  |
| binary  | 使用二进制进行传输  |
| ascii   | 使用ASCII码进行传输 |
| timeout | 设置重传的超时时间  |
| quit    | 退出                |

```
[root@linuxprobe ~]# echo "i love linux" > /var/lib/tftpboot/readme.txt
[root@linuxprobe ~]# tftp 192.168.10.10
```