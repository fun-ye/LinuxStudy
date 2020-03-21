# Chapter 10 使用Apache服务部署静态网站

[TOC]

## 10.1 网站服务程序

目前能够提供Web网络服务的程序有IIS、Nginx和Apache等。

**IIS**（Internet Information Services，互联网信息服务）是Windows系统中默认的Web服务程序，这是一款图形化的网站管理工具，不仅可以提供Web网站服务，还可以提供FTP、NMTP、SMTP等服务。

**Nginx**程序作为一款轻量级的网站服务软件，因其稳定性和丰富的功能而快速占领服务器市场，但Nginx最被认可的还当是系统资源消耗低且并发能力强

**Apache**程序是目前拥有很高市场占有率的Web服务程序之一，其跨平台和安全性广泛被认可且拥有快速、可靠、简单的API扩展。Apache服务程序可以运行在Linux、UNIX系统甚至是Windows系统中，支持基于IP、域名及端口号的虚拟主机功能，支持多种认证方式，集成有代理服务器模块、安全Socket层（SSL），能够实时监视服务状态与定制日志消息，并有着各类丰富的模块支持。

```
# yum install httpd
# systemctl start httpd
# systemctl enable httpd
```

## 10.2 配置服务文件参数

表10-1                        Linux系统中的配置文件

| 服务目录     | /etc/httpd                 |
| ------------ | -------------------------- |
| 主配置文件   | /etc/httpd/conf/httpd.conf |
| 网站数据目录 | /var/www/html              |
| 访问日志     | /var/log/httpd/access_log  |
| 错误日志     | /var/log/httpd/error_log   |

<img src="C:\Users\方也\AppData\Roaming\Typora\typora-user-images\image-20200314121628390.png" alt="image-20200314121628390" style="zoom:67%;" />

表10-2             配置httpd服务程序时最常用的参数以及用途描述

| ServerRoot     | 服务目录                  |
| -------------- | ------------------------- |
| ServerAdmin    | 管理员邮箱                |
| User           | 运行服务的用户            |
| Group          | 运行服务的用户组          |
| ServerName     | 网站服务器的域名          |
| DocumentRoot   | 网站数据目录              |
| Listen         | 监听的IP地址与端口号      |
| DirectoryIndex | 默认的索引页页面          |
| ErrorLog       | 错误日志文件              |
| CustomLog      | 访问日志文件              |
| Timeout        | 网页超时时间，默认为300秒 |

DocumentRoot参数用于定义网站数据的保存路径，其参数的默认值是把网站数据存放到/var/www/html目录中；而当前网站普遍的首页面名称是index.html，因此可以向/var/www/html目录中写入一个文件，替换掉httpd服务程序的默认首页面，该操作会立即生效。

```
# echo "Welcome To LinuxProbe.Com" > /var/www/html/index.html
把保存网站数据的目录修改为/home/wwwroot目录
# mkdir /home/wwwroot
# echo "The New Web Directory" > /home/wwwroot/index.html
# vim /etc/httpd/conf/httpd.conf 
119 DocumentRoot "/home/wwwroot"
124 <Directory "/home/wwwroot">
# systemctl restart httpd
```

## 10.3 SELinux安全子系统

SELinux（Security-Enhanced Linux）是美国国家安全局在Linux开源社区的帮助下开发的一个强制访问控制（MAC，Mandatory Access Control）的安全子系统

SELinux安全子系统就是为了杜绝此类情况而设计的，它能够从多方面监控违法行为：对服务程序的功能进行限制（SELinux域限制可以确保服务程序做不了出格的事情）；对文件资源的访问限制（SELinux安全上下文确保文件资源只能被其所属的服务程序进行访问)

SELinux服务有三种配置模式

> enforcing：强制启用安全策略模式，将拦截服务的不合法请求。
>
> permissive：遇到服务越权访问时，只发出警告而不强制拦截。
>
> disabled：对于越权的行为不警告也不拦截。

```CQL
# vim /etc/selinux/config
# getenforce
# setenforce 0
# ls -Zd /var/www/html
drwxr-xr-x. root root system_u:object_r:httpd_sys_content_t:s0 /var/www/html
# ls -Zd /home/wwwroot
drwxrwxrwx. root root unconfined_u:object_r:home_root_t:s0 /home/wwwroot
```

用户段system_u代表系统进程的身份，角色段object_r代表文件目录的角色，类型段httpd_sys_content_t代表网站服务的系统文件。

**semanage**命令

semanage命令用于管理SELinux的策略，格式为“semanage [选项] [文件]”

> -l参数用于查询；
>
> -a参数用于添加；
>
> -m参数用于修改；
>
> -d参数用于删除。

向新的网站数据目录中新添加一条SELinux安全上下文，让这个目录以及里面的所有文件能够被httpd服务程序所访问到：

```
# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot
# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/*
# restorecon -Rv /home/wwwroot/  //使用restorecon命令将设置好的SELinux安全上下文立即生效,-Rv参数对指定的目录进行递归操作，以及显示SELinux安全上下文的修改过程
```

## 10.4 个人用户主页功能

**第1步**：在httpd服务程序中，默认没有开启个人用户主页功能。

```
# vim /etc/httpd/conf.d/userdir.conf 
 17 # UserDir disabled
 24   UserDir public_html
```

**第2步**：在用户家目录中建立用于保存网站数据的目录及首页面文件。另外，还需要把家目录的权限修改为755，保证其他人也有权限读取里面的内容

```
[linuxprobe@linuxprobe ~]$ mkdir public_html
[linuxprobe@linuxprobe ~]$ echo "This is linuxprobe's website" > public_html/index.html
[linuxprobe@linuxprobe ~]$ chmod -Rf 755 /home/linuxprobe
```

**第3步**：重新启动httpd服务程序，在浏览器的地址栏中输入网址，其格式为“网址/~用户名”

getsebool命令查询并过滤出所有与HTTP协议相关的安全策略。其中，off为禁止状态，on为允许状态。

```
[root@linuxprobe ~]# getsebool -a | grep http
```

```
[root@linuxprobe ~]# setsebool -P httpd_enable_homedirs=on
```

网站的拥有者并不希望直接将网页内容显示出来，只想让通过身份验证的用户访客看到里面的内容，这时就可以在网站中添加口令功能了

```
# htpasswd -c /etc/httpd/passwd linux
# vim /etc/httpd/conf.d/userdir.conf //编辑个人用户主页功能的配置文件。
```

## 10.5 虚拟网站主机功能

利用虚拟主机功能，可以把一台处于运行状态的物理服务器分割成多个“虚拟的服务器”。

Apache的虚拟主机功能是服务器基于用户请求的不同IP地址、主机域名或端口号，实现提供多个网站同时为外部提供访问服务的技术，如图10-12所示，用户请求的资源不同，最终获取到的网页内容也各不相同

<img src="C:\Users\方也\AppData\Roaming\Typora\typora-user-images\image-20200314225115910.png" alt="image-20200314225115910" style="zoom:67%;" />

### 10.5.1 基于IP地址

**第1步**：分别在/home/wwwroot中创建用于保存不同网站数据的3个目录，并向其中分别写入网站的首页文件

```
[root@linuxprobe ~]# mkdir -p /home/wwwroot/10
[root@linuxprobe ~]# mkdir -p /home/wwwroot/20
[root@linuxprobe ~]# mkdir -p /home/wwwroot/30
[root@linuxprobe ~]# echo "IP:192.168.10.10" > /home/wwwroot/10/index.html
[root@linuxprobe ~]# echo "IP:192.168.10.20" > /home/wwwroot/20/index.html
[root@linuxprobe ~]# echo "IP:192.168.10.30" > /home/wwwroot/30/index.html
```

**第2步**：在httpd服务的配置文件中大约113行处开始，分别追加写入三个基于IP地址的虚拟主机网站参数，然后保存并退出

```
[root@linuxprobe ~]# vim /etc/httpd/conf/httpd.conf
113 <VirtualHost 192.168.10.10>
120 </VirtualHost>
121 <VirtualHost 192.168.10.20>
128 </VirtualHost>
129 <VirtualHost 192.168.10.30>
136 </VirtualHost>
```

第三步：手动把新的网站数据目录的SELinux安全上下文设置正确

```
[root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot
[root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/10
[root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/10/*
[root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/20
[root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/20/*
[root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/30
[root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/30/*
[root@linuxprobe ~]# restorecon -Rv /home/wwwroot
```

### 10.5.2 基于主机域名

当服务器无法为每个网站都分配一个独立IP地址的时候，可以尝试让Apache自动识别用户请求的域名，从而根据不同的域名请求来传输不同的内容。/etc/hosts是Linux系统中用于强制把某个主机域名解析到指定IP地址的配置文件

**第1步**：手工定义IP地址与域名之间对应关系的配置文件，保存并退出后会立即生效

```
# vim /etc/hosts
192.168.10.10 www.linuxprobe.com bbs.linuxprobe.com tech.linuxprobe.com
```

**第2步**：分别在/home/wwwroot中创建用于保存不同网站数据的三个目录，并向其中分别写入网站的首页文件。

```
[root@linuxprobe ~]# mkdir -p /home/wwwroot/www
[root@linuxprobe ~]# mkdir -p /home/wwwroot/bbs
[root@linuxprobe ~]# mkdir -p /home/wwwroot/tech
[root@linuxprobe ~]# echo "WWW.linuxprobe.com" > /home/wwwroot/www/index.html
[root@linuxprobe ~]# echo "BBS.linuxprobe.com" > /home/wwwroot/bbs/index.html
[root@linuxprobe ~]# echo "TECH.linuxprobe.com" > /home/wwwroot/tech/index.html
```

**第3步**：在httpd服务的配置文件中大约113行处开始，分别追加写入三个基于主机名的虚拟主机网站参数

```
[root@linuxprobe ~]# vim /etc/httpd/conf/httpd.conf
113 <VirtualHost 192.168.10.10>
120 </VirtualHost>
121 <VirtualHost 192.168.10.10>
127 </Directory>
128 </VirtualHost>
129 <VirtualHost 192.168.10.10>
```

**第4步**：置网站数据目录文件的SELinux安全上下文，使其与网站服务功能相吻合

```
[root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot
[root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/www
[root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/www/*
[root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/bbs
[root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/bbs/*
[root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/tech
[root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/tech/*
[root@linuxprobe ~]# restorecon -Rv /home/wwwroot
```

### 10.5.3 基于端口号

基于端口号的虚拟主机功能可以让用户通过指定的端口号来访问服务器上的网站资源。

**第1步**：分别在/home/wwwroot中创建用于保存不同网站数据的两个目录，并向其中分别写入网站的首页文件。

```
[root@linuxprobe ~]# mkdir -p /home/wwwroot/6111
[root@linuxprobe ~]# mkdir -p /home/wwwroot/6222
[root@linuxprobe ~]# echo "port:6111" > /home/wwwroot/6111/index.html
[root@linuxprobe ~]# echo "port:6222" > /home/wwwroot/6222/index.html
```

**第2步**：在httpd服务配置文件的第43行和第44行分别添加用于监听6111和6222端口的参数。

```
# vim /etc/httpd/conf/httpd.conf
 43 Listen 6111
 44 Listen 6222
```

**第3步**：在httpd服务的配置文件中大约113行处开始，分别追加写入两个基于端口号的虚拟主机网站参数，

```
[root@linuxprobe ~]# vim /etc/httpd/conf/httpd.conf
113 <VirtualHost 192.168.10.10:6111>
120 </VirtualHost>
121 <VirtualHost 192.168.10.10:6222>
128 </VirtualHost>
```

**第4步**:须要正确设置网站数据目录文件的SELinux安全上下文，

用semanage命令查询并过滤出所有与HTTP协议相关且SELinux服务允许的端口列表。

```
[root@linuxprobe ~]# semanage port -l | grep http
[root@linuxprobe ~]# semanage port -a -t http_port_t -p tcp 6111
[root@linuxprobe ~]# semanage port -a -t http_port_t -p tcp 6222
[root@linuxprobe ~]# semanage port -l| grep http
```

## 10.6 Apache的访问控制

它通过Allow指令允许某个主机访问服务器上的网站资源，通过Deny指令实现禁止访问。在允许或禁止访问网站资源时，还会用到Order指令，这个指令用来定义Allow或Deny指令起作用的顺序，其匹配原则是按照顺序进行匹配

**第1步**：先在服务器上的网站数据目录中新建一个子目录，并在这个子目录中创建一个包含Successful单词的首页文件。

```
[root@linuxprobe ~]# mkdir /var/www/html/server
[root@linuxprobe ~]# echo "Successful" > /var/www/html/server/index.html
```

**第2步**：打开httpd服务的配置文件，在第129行后面添加下述规则来限制源主机的访问。这段规则的含义是允许使用Firefox浏览器的主机访问服务器上的首页文件，除此之外的所有请求都将被拒绝。

```
[root@linuxprobe ~]# vim /etc/httpd/conf/httpd.conf

```