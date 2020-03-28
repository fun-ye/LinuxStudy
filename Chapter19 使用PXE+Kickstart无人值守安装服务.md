# Chapter19 使用PXE+Kickstart无人值守安装服务

[TOC]

## 19.1 无人值守系统

<img src="C:\Users\方也\AppData\Roaming\Typora\typora-user-images\image-20200325102822878.png" alt="image-20200325102822878" style="zoom:67%;" />

PXE（Preboot eXecute Environment，预启动执行环境）是由Intel公司开发的技术，可以让计算机通过网络来启动操作系统，主要用于在无人值守安装系统中引导客户端主机安装Linux操作系统。

Kickstart是一种无人值守的安装方式，其工作原理是预先把原本需要运维人员手工填写的参数保存成一个ks.cfg文件，当安装过程中需要填写参数时则自动匹配Kickstart生成的文件。

## 19.2 部署相关服务程序

### 19.2.1 配置DHCP服务程序

```
# yum install dhcp
# vim /etc/dhcp/dhcpd.conf
# systemctl restart dhcpd
# systemctl enable dhcpd
```

### 19.2.2 配置TFTP服务程序

TFTP是一种非常精简的文件传输服务程序，它的运行和关闭是由xinetd网络守护进程服务来管理的。xinetd服务程序会同时监听系统的多个端口，然后根据用户请求的端口号调取相应的服务程序来响应用户的请求

```
# yum install tftp-server
# vim /etc/xinetd.d/tftp
# systemctl restart xinetd
# systemctl enbale xinetd
# firewall-cmd --permanent --add-port=69/udp
# firewall-cmd-cmd --reload
```

### 19.2.3 配置SYSLinux服务程序

SYSLinux是一个用于提供引导加载的服务程序。

```
# yum install syslinux
```

### 19.2.4 配置VSFtpd服务程序

### 19.2.5 创建KickStart应答文件

使用PXE + Kickstart部署的是一套“无人值守安装系统服务”，而不是“无人值守传输系统光盘镜像服务”，因此还需要让客户端主机能够一边获取光盘镜像，还能够一边自动帮我们填写好安装过程中出现的选项。

```
# cp ~/anaconda-ks.cfg /var/ftp/pub/ks.cfg
# chmod +r /var/ftp/pub/ks.cfg
# vim /var/ftp/pub/ks.cfg 
```

## 19.3 自动部署客户机

