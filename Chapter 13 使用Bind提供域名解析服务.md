# Chapter 13 使用Bind提供域名解析服务

[TOC]

## 13.1 DNS域名解析服务

鉴于互联网中的域名和IP地址对应关系数据库太过庞大，DNS域名解析服务采用了类似目录树的层次结构来记录域名与IP地址之间的对应关系，从而形成了一个分布式的数据库系统

<img src="C:\Users\方也\AppData\Roaming\Typora\typora-user-images\image-20200321195926744.png" alt="image-20200321195926744" style="zoom:67%;" />

DNS三种类型服务器：

> **主服务器**：在特定区域内具有唯一性，负责维护该区域内的域名与IP地址之间的对应关系。
>
> **从服务器**：从主服务器中获得域名与IP地址的对应关系并进行维护，以防主服务器宕机等情况。
>
> **缓存服务器**：通过向其他域名解析服务器查询获得域名与IP地址的对应关系，并将经常查询的域名信息保存到服务器本地，以此来提高重复查询时的效率。

DNS域名解析服务采用分布式的数据结构来存放海量的“区域数据”信息，在执行用户发起的域名查询请求时，具有递归查询和迭代查询两种方式。所谓`递归查询`，是指DNS服务器在收到用户发起的请求时，必须向用户返回一个准确的查询结果。如果DNS服务器本地没有存储与之对应的信息，则该服务器需要询问其他服务器，并将返回的查询结果提交给用户。而`迭代查询`则是指，DNS服务器在收到用户发起的请求时，并不直接回复查询结果，而是告诉另一台DNS服务器的地址，用户再向这台DNS服务器提交请求，这样依次反复，直到返回查询结果。

**DNS服务器之间是迭代查询，用户与DNS服务器之间是递归查询。**

<img src="C:\Users\方也\AppData\Roaming\Typora\typora-user-images\image-20200321200348573.png" alt="image-20200321200348573" style="zoom:50%;" />

用户向网络指定的DNS服务器发起一个域名请求时，通常情况下会有本地由此DNS服务器向上级的DNS服务器发送迭代查询请求；如果该DNS服务器没有要查询的信息，则会进一步向上级DNS服务器发送迭代查询请求，直到获得准确的查询结果为止。

## 13.2 安装Bind服务程序

BIND（Berkeley Internet Name Domain，伯克利因特网名称域）服务是全球范围内使用最广泛、最安全可靠且高效的域名解析服务程序。加上chroot（俗称牢笼机制）扩展包，以便有效地限制bind服务程序仅能对自身的配置文件进行操作，以确保整个服务器的安全。

bind服务程序

> 主配置文件（/etc/named.conf）：只有58行，而且在去除注释信息和空行之后，实际有效的参数仅有30行左右，这些参数用来**定义bind服务程序的运行**。
>
> 区域配置文件（/etc/named.rfc1912.zones）：用来**保存域名和IP地址对应关系的所在位置**。类似于图书的目录，对应着每个域和相应IP地址所在的具体位置，当需要查看或修改时，可根据这个位置找到相关文件。
>
> 数据配置文件目录（/var/named）：该目录用来保存域名和IP地址真实对应关系的**数据配置文件**。

```
[root@linuxprobe ~]# yum install bind-chroot
[root@linuxprobe ~]# vim /etc/named.conf
  11 listen-on port 53 { any; };
  17 allow-query { any; };
```

区域配置文件服务类型有三种，分别为hint（根区域）、master（主区域）、slave（辅助区域），其中常用的master和slave指的就是主服务器和从服务器。

<img src="C:\Users\方也\AppData\Roaming\Typora\typora-user-images\image-20200321201215538.png" alt="image-20200321201215538" style="zoom:67%;" />

### 13.2.1 正向解析实验

**第1步**：编辑区域配置文件

```
[root@linuxprobe ~]# vim /etc/named.rfc1912.zones
zone "linuxprobe.com" IN {
type master;
file "linuxprobe.com.zone";
allow-update {none;};
};
```

**第2步**：编辑数据配置文件

```
[root@linuxprobe ~]# cd /var/named/
[root@linuxprobe named]# ls -al named.localhost
-rw-r-----. 1 root named 152 Jun 21 2007 named.localhost
[root@linuxprobe named]# cp -a named.localhost linuxprobe.com.zone
[root@linuxprobe named]# vim linuxprobe.com.zone
[root@linuxprobe named]# systemctl restart named
```

**第3步**：检验解析结果。

```
[root@linuxprobe ~]# systemctl restart network
[root@linuxprobe ~]# nslookup //检测能否从DNS服务器中查询到域名与IP地址的解析记录
> www.linuxprobe.com
Server: 127.0.0.1
Address: 127.0.0.1#53
Name: www.linuxprobe.com
Address: 192.168.10.10
> bbs.linuxprobe.com
Server: 127.0.0.1
Address: 127.0.0.1#53
Name: bbs.linuxprobe.com
Address: 192.168.10.20
```

### 13.2.2 反向解析实验

反向解析的作用是将用户提交的IP地址解析为对应的域名信息，它一般用于对某个IP地址上绑定的所有域名进行整体屏蔽，屏蔽由某些域名发送的垃圾邮件。也可以针对某个IP地址进行反向解析，大致判断出有多少个网站运行在上面

**第1步**：编辑区域配置文件。定义zone（区域）时应该要把IP地址反写

```
[root@linuxprobe ~]# vim /etc/named.rfc1912.zones
zone "linuxprobe.com" IN {
type master;
file "linuxprobe.com.zone";
allow-update {none;};
};
zone "10.168.192.in-addr.arpa" IN {
type master;
file "192.168.10.arpa";
};
```

**第2步**：编辑数据配置文件。IP地址仅需要写主机位

<img src="C:\Users\方也\AppData\Roaming\Typora\typora-user-images\image-20200321201844890.png" alt="image-20200321201844890" style="zoom: 67%;" />

```
[root@linuxprobe named]# cp -a named.loopback 192.168.10.arpa
[root@linuxprobe named]# vim 192.168.10.arpa
[root@linuxprobe named]# systemctl restart named
```

**第3步**：检验解析结果。

```
[root@linuxprobe ~]# nslookup
```

## 13.3 部署从服务器

在DNS域名解析服务中，从服务器可以从主服务器上获取指定的区域数据文件，从而起到备份解析记录与负载均衡的作用

**第1步**：在主服务器的区域配置文件中允许该从服务器的更新请求，即修改allow-update {允许更新区域信息的主机地址;};参数，然后重启主服务器的DNS服务程序。

```
[root@linuxprobe ~]# vim /etc/named.rfc1912.zones
zone "linuxprobe.com" IN {
type master;
file "linuxprobe.com.zone";
allow-update { 192.168.10.20; };
};
zone "10.168.192.in-addr.arpa" IN {
type master;
file "192.168.10.arpa";
allow-update { 192.168.10.20; };
};
[root@linuxprobe ~]# systemctl restart named
```

**第2步**：在从服务器中填写主服务器的IP地址与要抓取的区域信息，然后重启服务。

```
[root@linuxprobe ~]# vim /etc/named.rfc1912.zones
zone "linuxprobe.com" IN {
type slave;
masters { 192.168.10.10; };
file "slaves/linuxprobe.com.zone";
};
zone "10.168.192.in-addr.arpa" IN {
type slave;
masters { 192.168.10.10; };
file "slaves/192.168.10.arpa";
};
[root@linuxprobe ~]# systemctl restart named
```

**第3步**：检验解析结果。

```
[root@linuxprobe ~]# cd /var/named/slaves
[root@linuxprobe slaves]# ls 
192.168.10.arpa linuxprobe.com.zone
[root@linuxprobe slaves]# nslookup
```

## 13.4 安全的加密传输

**第1步**：在主服务器中生成密钥。dnssec-keygen命令用于生成安全的DNS服务密钥，其格式为“dnssec-keygen [参数]”，

表13-3                    dnssec-keygen命令的常用参数

| 参数 | 作用                                                         |
| ---- | ------------------------------------------------------------ |
| -a   | 指定加密算法，包括RSAMD5（RSA）、RSASHA1、DSA、NSEC3RSASHA1、NSEC3DSA等 |
| -b   | 密钥长度（HMAC-MD5的密钥长度在1~512位之间）                  |
| -n   | 密钥的类型（HOST表示与主机相关）                             |

```
//生成一个主机名称为master-slave的128位HMAC-MD5算法的密钥文件
[root@linuxprobe ~]# dnssec-keygen -a HMAC-MD5 -b 128 -n HOST master-slave
Kmaster-slave.+157+46845
[root@linuxprobe ~]# ls -al Kmaster-slave.+157+46845.*
```

**第2步**：在主服务器中创建密钥验证文件。

```
[root@linuxprobe ~]# cd /var/named/chroot/etc/
[root@linuxprobe etc]# vim transfer.key
key "master-slave" {
algorithm hmac-md5;
secret "1XEEL3tG5DNLOw+1WHfE3Q==";
};
[root@linuxprobe etc]# chown root:named transfer.key
[root@linuxprobe etc]# chmod 640 transfer.key
[root@linuxprobe etc]# ln transfer.key /etc/transfer.key
```

**第3步**：开启并加载Bind服务的密钥验证功能。

```
[root@linuxprobe ~]# vim /etc/named.conf
 9 include "/etc/transfer.key";
 18 allow-transfer { key master-slave; };
```

**第4步**：配置从服务器，使其支持密钥验证。

```
[root@linuxprobe ~]# cd /var/named/chroot/etc
[root@linuxprobe etc]# vim transfer.key
key "master-slave" {
algorithm hmac-md5;
secret "1XEEL3tG5DNLOw+1WHfE3Q==";
};
[root@linuxprobe etc]# chown root:named transfer.key
[root@linuxprobe etc]# chmod 640 transfer.key
[root@linuxprobe etc]# ln transfer.key /etc/transfer.key
```

**第5步**：开启并加载从服务器的密钥验证功能

```
[root@linuxprobe etc]# vim /etc/named.conf
 9 include "/etc/transfer.key";
  43 server 192.168.10.10
 44 {
 45 keys { master-slave; };
 46 }; 
```

**第6步**：DNS从服务器同步域名区域数据。

```
[root@linuxprobe ~]# systemctl restart named
[root@linuxprobe ~]# ls /var/named/slaves/
 192.168.10.arpa  linuxprobe.com.zone
```

## 13.5 部署缓存服务器

DNS缓存服务器（Caching DNS Server）是一种不负责域名数据维护的DNS服务器。简单来说，缓存服务器就是把用户经常使用到的域名与IP地址的解析记录保存在主机本地，从而提升下次解析的效率。DNS缓存服务器一般用于经常访问某些固定站点而且对这些网站的访问速度有较高要求的企业内网中

**第1步**：配置系统的双网卡参数。

**第2步**：在bind服务程序的主配置文件中添加缓存转发参数。

```
[root@linuxprobe ~]# vim /etc/named.con
17 forwarders { 210.73.64.1; }; //上级DNS服务器地址指的是获取数据配置文件的服务器
```

**第3步**：重启DNS服务，验证成果

## 13.6 分离解析技术

<img src="C:\Users\方也\AppData\Roaming\Typora\typora-user-images\image-20200321202847673.png" alt="image-20200321202847673" style="zoom:50%;" />

**第1步**：修改bind服务程序的主配置文件，把第11行的监听端口与第17行的允许查询主机修改为any。

```
[root@linuxprobe ~]# vim /etc/named.conf
 51 zone "." IN {
 52 type hint;
 53 file "named.ca";
 54 };
```

**第2步**：编辑区域配置文件。

```
[root@linuxprobe ~]# vim /etc/named.rfc1912.zones
```

**第3步**：建立数据配置文件。

```
[root@linuxprobe ~]# cd /var/named
[root@linuxprobe named]# cp -a named.localhost linuxprobe.com.china
[root@linuxprobe named]# cp -a named.localhost linuxprobe.com.american
[root@linuxprobe named]# vim linuxprobe.com.china
```

**第4步**：重新启动named服务程序，验证结果