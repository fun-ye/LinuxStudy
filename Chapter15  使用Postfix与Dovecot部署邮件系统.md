#  Chapter15  使用Postfix与Dovecot部署邮件系统

[TOC]

## 15.1 电子邮件系统

> **简单邮件传输协议（Simple Mail Transfer Protocol，SMTP）**：用于发送和中转发出的电子邮件，占用服务器的25/TCP端口。
>
> **邮局协议版本3（Post Office Protocol 3）**：用于将电子邮件存储到本地主机，占用服务器的110/TCP端口。
>
> **Internet消息访问协议版本4（Internet Message Access Protocol 4）**：用于在本地主机上访问邮件，占用服务器的143/TCP端口。

## 15.2 部署基础的电子邮件系统

一个最基础的电子邮件系统肯定要能提供发件服务和收件服务，为此需要使用基于SMTP协议的Postfix服务程序提供发件服务功能，并使用基于POP3协议的Dovecot服务程序提供收件服务功能

<img src="C:\Users\方也\AppData\Roaming\Typora\typora-user-images\image-20200322202849518.png" alt="image-20200322202849518" style="zoom: 67%;" />

**第1步**：配置服务器主机名称，需要保证服务器主机名称与发信域名保持一致：

```
# vim /etc/hostname
```

**第2步**：清空iptables防火墙默认策略，并保存策略状态，避免因防火墙中默认存在的策略阻止了客户端DNS解析域名及收发邮件

```
# iptables -F
# service iptables save
```

**第3步**：为电子邮件系统提供域名解析

```
# cat /etc/named.conf
```

### 15.2.1 配置Postfix服务程序

```
# yum install postfix
# systemctl disable iptables
```

Postfix服务程序主配置文件（/etc/ postfix/main.cf）

表15-1                Postfix服务程序主配置文件中的重要参数

| 参数            | 作用                     |
| --------------- | ------------------------ |
| myhostname      | 邮局系统的主机名         |
| mydomain        | 邮局系统的域名           |
| myorigin        | 从本机发出邮件的域名名称 |
| inet_interfaces | 监听的网卡接口           |
| mydestination   | 可接收邮件的主机名或域名 |
| mynetworks      | 设置可转发哪些主机的邮件 |
| relay_domains   | 设置可转发哪些网域的邮件 |

```
# vim /etc/postfix/main.cf
```

### 15.2.2 配置Dovercot服务程序

Dovecot是一款能够为Linux系统提供IMAP和POP3电子邮件服务的开源服务程序，安全性极高，配置简单，执行速度快，而且占用的服务器硬件资源也较少# su - boss

```
# yum install dovecot
# vim /etc/dovecot/dovecot.conf
# vim /etc/dovecot/conf.d/10-mail.conf //配置邮件格式与存储路径。
# su - boss
# exit
$ mkdir -p mail/.imap/INBOX
# systemctl restart dovecot 
# systemctl enable dovecot
```

### 15.2.3 **客户使用电子邮件系统**

## 15.3 设置用户别名邮箱

用户别名功能是一项简单实用的邮件账户伪装技术，可以用来设置多个虚拟信箱的账户以接受发送的邮件，从而保证自身的邮件地址不被泄露，还可以用来接收自己的多个信箱中的邮件

```
# su -bin
# mail
# cat /etc/aliases
```

