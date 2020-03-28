# Chapter14 使用DHCP动态管理主机地址

[TOC]

## 14.1 动态主机地址管理协议

动态主机配置协议（DHCP）是一种基于**UDP**协议且仅限于在**局域网**内部使用的网络协议，主要用于大型的局域网环境或者存在较多移动办公设备的局域网环境中，其主要用途是为局域网内部的设备或网络供应商**自动分配IP地址**等参数。

<img src="C:\Users\方也\AppData\Roaming\Typora\typora-user-images\image-20200322201049984.png" alt="image-20200322201049984" style="zoom:50%;" />

> **作用域**：一个完整的IP地址段，DHCP协议根据作用域来管理网络的分布、分配IP地址及其他配置参数。
>
> **超级作用域**：用于管理处于同一个物理网络中的多个逻辑子网段。超级作用域中包含了可以统一管理的作用域列表。
>
> **排除范围**：把作用域中的某些IP地址排除，确保这些IP地址不会分配给DHCP客户端。
>
> **地址池**：在定义了DHCP的作用域并应用了排除范围后，剩余的用来动态分配给DHCP客户端的IP地址范围。
>
> **租约**：DHCP客户端能够使用动态分配的IP地址的时间。
>
> **预约**：保证网络中的特定设备总是获取到相同的IP地址。

## 14.2 部署dhcpd服务程序

dhcpd是Linux系统中用于提供DHCP协议的服务程序

```
# yum install dhcp
# cat /etc/dhcp/dhcpd.conf
# DHCP Server Configuration file.
# see /usr/share/doc/dhcp*/dhcpd.conf.example
# see dhcpd.conf(5) man page
```

<img src="C:\Users\方也\AppData\Roaming\Typora\typora-user-images\image-20200322201500826.png" alt="image-20200322201500826" style="zoom:50%;" />

表14-1            dhcpd服务程序配置文件中使用的常见参数以及作用

| 参数                               | 作用                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| ddns-update-style 类型             | 定义DNS服务动态更新的类型，类型包括： none（不支持动态更新）、interim（互动更新模式）与ad-hoc（特殊更新模式） |
| allow/ignore client-updates        | 允许/忽略客户端更新DNS记录                                   |
| default-lease-time 21600           | 默认超时时间                                                 |
| max-lease-time 43200               | 最大超时时间                                                 |
| option domain-name-servers 8.8.8.8 | 定义DNS服务器地址                                            |
| option domain-name "domain.org"    | 定义DNS域名                                                  |
| range                              | 定义用于分配的IP地址池                                       |
| option subnet-mask                 | 定义客户端的子网掩码                                         |
| option routers                     | 定义客户端的网关地址                                         |
| broadcast-address 广播地址         | 定义客户端的广播地址                                         |
| ntp-server IP地址                  | 定义客户端的网络时间服务器（NTP）                            |
| nis-servers IP地址                 | 定义客户端的NIS域服务器的地址                                |
| hardware 硬件类型 MAC地址          | 指定网卡接口的类型与MAC地址                                  |
| server-name 主机名                 | 向DHCP客户端通知DHCP服务器的主机名                           |
| fixed-address IP地址               | 将某个固定的IP地址分配给指定主机                             |
| time-offset 偏移差                 | 指定客户端与格林尼治时间的偏移差                             |

## 14.3 自动管理IP地址

表14-4              dhcpd服务程序配置文件中使用的参数以及作用

| 参数                                        | 作用                                         |
| ------------------------------------------- | -------------------------------------------- |
| ddns-update-style none;                     | 设置DNS服务不自动进行动态更新                |
| ignore client-updates;                      | 忽略客户端更新DNS记录                        |
| subnet 192.168.10.0 netmask 255.255.255.0 { | 作用域为192.168.10.0/24网段                  |
| range 192.168.10.50 192.168.10.150;         | IP地址池为192.168.10.50-150（约100个IP地址） |
| option subnet-mask 255.255.255.0;           | 定义客户端默认的子网掩码                     |
| option routers 192.168.10.1;                | 定义客户端的网关地址                         |
| option domain-name "linuxprobe.com";        | 定义默认的搜索域                             |
| option domain-name-servers 192.168.10.1;    | 定义客户端的DNS地址                          |
| default-lease-time 21600;                   | 定义默认租约时间（单位：秒）                 |
| max-lease-time 43200;                       | 定义最大预约时间（单位：秒）                 |
| }                                           | 结束符                                       |

## 14.4 分配固定IP地址

```
# tail -f /var/log/messages
# vim /etc/dhcp/dhcpd.conf 
# systemctl restart dhcpd
```

