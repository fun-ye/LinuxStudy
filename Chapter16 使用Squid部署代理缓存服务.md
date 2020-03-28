# Chapter16 使用Squid部署代理缓存服务

[TOC]

## 16.1 代理缓存服务

Squid是[Linux系统](https://www.linuxprobe.com/)中最为流行的一款高性能代理服务软件，通常用作Web网站的前置缓存服务，能够代替用户向网站服务器请求页面数据并进行缓存

Squid服务程序具有配置简单、效率高、功能丰富等特点，它能支持HTTP、FTP、SSL等多种协议的数据缓存，可以基于访问控制列表（ACL）和访问权限列表（ARL）执行内容过滤与权限管理功能

正向代理模式，是指让用户通过Squid服务程序获取网站页面等资源，以及基于访问控制列表（ACL）功能对用户访问网站行为进行限制，在具体的服务方式上又分为标准代理模式与透明代理模式

**标准正向代理模式**是把网站数据缓存到服务器本地，提高数据资源被再次访问时的效率，但是用户在上网时必须在浏览器等软件中填写代理服务器的IP地址与端口号信息，否则默认不使用代理服务。而**透明正向代理模式**的作用与标准正向代理模式基本相同，区别是用户不需要手动指定代理服务器的IP地址与端口号

<img src="C:\Users\方也\AppData\Roaming\Typora\typora-user-images\image-20200323202622924.png" alt="image-20200323202622924" style="zoom:50%;" />![image-20200323202804615](C:\Users\方也\AppData\Roaming\Typora\typora-user-images\image-20200323202804615.png)

![image-20200323202804615](C:\Users\方也\AppData\Roaming\Typora\typora-user-images\image-20200323202804615.png)

## 16.2 配置Squid服务程序

```
# yum install squid
```

表16-2                 常用的Squid服务程序配置参数以及作用

| 参数                                       | 作用                                  |
| ------------------------------------------ | ------------------------------------- |
| http_port 3128                             | 监听的端口号                          |
| cache_mem 64M                              | 内存缓冲区的大小                      |
| cache_dir ufs /var/spool/squid 2000 16 256 | 硬盘缓冲区的大小                      |
| cache_effective_user squid                 | 设置缓存的有效用户                    |
| cache_effective_group squid                | 设置缓存的有效用户组                  |
| dns_nameservers IP地址                     | 一般不设置，而是用服务器默认的DNS地址 |
| cache_access_log /var/log/squid/access.log | 访问日志文件的保存路径                |
| cache_log /var/log/squid/cache.log         | 缓存日志文件的保存路径                |
| visible_hostname linuxprobe.com            | 设置Squid服务器的名称                 |

## 16.3 正向代理

### 16.3.1 标准正向代理

```
启动默认标准正向代理
# systemctl restart squid
# systemctl enable squid
# vim /etc/squid/squid.conf
http_port 10000
# semanage port -l |grep squid_port_t
# semanage port -a -t squid_port_t -p tcp 10000
# semanage port -l | grep squid_port_t
```

### 16.3.2 ACL访问控制

Squid服务程序的ACL是由多个策略规则组成的，它可以根据指定的策略规则来允许或限制访问请求，而且策略规则的匹配顺序与防火墙策略规则一样都是由上至下

### 16.3.3 透明正向代理

在透明代理模式中，用户无须在浏览器或其他软件中配置代理服务器地址、端口号等信息，而是由DHCP服务器将网络配置信息分配给客户端主机。

## 16.4 反向代理

```
# vim /etc/squid/squid.conf
```

