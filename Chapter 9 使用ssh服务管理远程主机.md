# Chapter 9 使用ssh服务管理远程主机

[TOC]

## 9.1 配置网卡服务

### 9.1.1 配置网卡参数

```
# nmtui
# vi /etc/sysconfig/network-scripts/ifcfg-eth0:0 //单IP地址运行
DEVICE=eth0:0
TYPE=Ethernet
BOOTPROTO=static
ONBOOT=yes
IPADDR=<IPv4地址1>
NETMASK=<IPv4掩码>
GATEWAY=<IPv4网关>
# systemctl restart network
```

### 9.1.2 创建网络会话

RHEL系统默认使用NetworkManager来提供网络服务，这是一种动态管理网络配置的守护进程，能够让网络设备保持连接状态。可以使用nmcli命令来管理Network Manager服务。

```
# nmcli connection show
//RHEL7系统支持网络会话功能，允许用户在多个配置文件中快速切换
# nmcli connection add con-name company ifname eno16777736 autoconnect no type ethernet ip4 192.168.10.10/24 gw4 192.168.10.1
# nmcli connection add con-name house type ethernet ifname eno16777736
# nmcli connection up house 
```

### 9.1.3 绑定两块网卡

```
# vim /etc/sysconfig/network-scripts/ifcfg-eno16777736
# vim /etc/sysconfig/network-scripts/ifcfg-eno33554968
# vim /etc/sysconfig/network-scripts/ifcfg-bond0
```

让Linux内核支持网卡绑定驱动。常见的网卡绑定驱动有三种模式—mode0、mode1和mode6。

> mode0（平衡负载模式）：平时两块网卡均工作，且自动备援，但需要在与服务器本地网卡相连的交换机设备上进行端口聚合来支持绑定技术。
>
> mode1（自动备援模式）：平时只有一块网卡工作，在它故障后自动替换为另外的网卡。
>
> mode6（平衡负载模式）：平时两块网卡均工作，且自动备援，无须交换机设备提供辅助支持。

im文本编辑器创建一个用于网卡绑定的驱动文件，使得绑定后的bond0网卡设备能够支持绑定技术（bonding）；同时定义网卡以mode6模式进行绑定，且出现故障时自动切换的时间为100毫秒。
```
# vim /etc/modprobe.d/bond.conf
alias bond0 bonding
options bond0 miimon=100 mode=6
# systemctl restart network
```

## 9.2 远程控制服务

### 9.2.1 配置sshd服务

SSH（Secure Shell）是一种能够以安全的方式提供远程登录的协议

sshd是基于SSH协议开发的一款远程管理服务程序。sshd服务的配置信息保存在/etc/ssh/sshd_config,2种安全验证方法

> 基于口令的验证—用账户和密码来验证登录；
>
> 基于密钥的验证—需要在本地生成密钥对，然后把密钥对中的公钥上传至服务器，并与服务器中的公钥进行比较；该方式相较来说更安全。

表9-1                 sshd服务配置文件中包含的参数以及作用

| 参数                              | 作用                                |
| --------------------------------- | ----------------------------------- |
| Port 22                           | 默认的sshd服务端口                  |
| ListenAddress 0.0.0.0             | 设定sshd服务器监听的IP地址          |
| Protocol 2                        | SSH协议的版本号                     |
| HostKey /tc/ssh/ssh_host_key      | SSH协议版本为1时，DES私钥存放的位置 |
| HostKey /etc/ssh/ssh_host_rsa_key | SSH协议版本为2时，RSA私钥存放的位置 |
| HostKey /etc/ssh/ssh_host_dsa_key | SSH协议版本为2时，DSA私钥存放的位置 |
| PermitRootLogin yes               | 设定是否允许root管理员直接登录      |
| StrictModes yes                   | 当远程用户的私钥改变时直接拒绝连接  |
| MaxAuthTries 6                    | 最大密码尝试次数                    |
| MaxSessions 10                    | 最大终端数                          |
| PasswordAuthentication yes        | 是否允许密码验证                    |
| PermitEmptyPasswords no           | 是否允许空密码登录（很不安全）      |

```
# ssh 192.168.10.10
# exit
# vim /etc/ssh/sshd_config 
# systemctl restart sshd
# systemctl enable sshd
```

### 9.2.2 安全密钥验证

密钥即是密文的钥匙，有私钥和公钥之分。在传输数据时，如果担心被他人监听或截获，就可以在传输前先使用公钥对数据加密处理，然后再行传送。这样，只有掌握私钥的用户才能解密这段数据，

```
第1步：在客户端主机中生成“密钥对”。
# ssh-keygen
第2步：把客户端主机中生成的公钥文件传送至远程主机：
# ssh-copy-id 192.168.10.10
第3步：对服务器进行设置，使其只允许密钥验证，拒绝传统的口令验证方式。
# vim /etc/ssh/sshd_config
 78 PasswordAuthentication no
# systemctl restart sshd 
第4步：在客户端尝试登录到服务器，此时无须输入密码也可成功登录。
# ssh 192.168.10.10
```

### 9.2.3 远程传输命令

scp（secure copy）是一个基于SSH协议在网络之间进行安全传输的命令，其格式为“scp [参数] 本地文件 远程帐户@远程IP地址:远程目录”。

表9-2                       scp命令中可用的参数及作用

| 参数 | 作用                     |
| ---- | ------------------------ |
| -v   | 显示详细的连接进度       |
| -P   | 指定远程主机的sshd端口号 |
| -r   | 用于传送文件夹           |
| -6   | 使用IPv6协议             |

```
# echo "Welcome to LinuxProbe.Com" > readme.txt
# scp /root/readme.txt 192.168.10.20:/home //文件从本地复制到远程主机
# scp 192.168.10.20:/etc/redhat-release /root //远程主机上的文件下载到本地主机
```

## 9.3 不间断会话服务

screen是一款能够实现多窗口远程控制的开源服务程序，为了解决网络异常中断或为了同时控制多个远程终端窗口而设计的程序。用户还可以使用screen服务程序同时在多个远程会话中自由切换

> **会话恢复**：即便网络中断，也可让会话随时恢复，确保用户不会失去对远程会话的控制。
>
> **多窗口**：每个会话都是独立运行的，拥有各自独立的输入输出终端窗口，终端窗口内显示过的信息也将被分开隔离保存，以便下次使用时依然能看到之前的操作记录。
>
> **会话共享**：当多个用户同时登录到远程服务器时，便可以使用会话共享功能让用户之间的输入输出信息共享。

### 9.3.1 管理远程会话

可以用-S参数创建会话窗口；用-d参数将指定会话进行离线处理；用-r参数恢复指定会话；用-x参数一次性恢复所有的会话；用-ls参数显示当前已有的会话；用-wipe参数把目前无法使用的会话删除

```
# screen -S backup  //-S参数创建会话窗口；
# screen -ls		//-ls参数显示当前已有的会话
可以直接使用screen命令执行要运行的命令，这样在命令中的一切操作也都会被记录下来，当命令执行结束后screen会话也会自动结束。
# screen vim memo.txt
# screen -r linux	//-r参数恢复指定会话
```

### 9.3.2 会话共享功能

screen具有会话共享、分屏切割、会话锁定等实用的功能

<img src="C:\Users\方也\AppData\Roaming\Typora\typora-user-images\image-20200314115741095.png" alt="image-20200314115741095" style="zoom:67%;" />

```
clientA
# ssh 192.168.10.10
# screen -S linuxprobe
clientB
# ssh 192.168.10.10
# screen -x
```

