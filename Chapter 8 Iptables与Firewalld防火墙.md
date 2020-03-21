# Chapter 8 Iptables与Firewalld防火墙

[TOC]

## 8.1 防火墙管理工具

防火墙策略可以基于流量的源目地址、端口号、协议、应用等信息来定制，然后防火墙使用预先定制的策略规则监控出入的流量，若流量与某一条策略规则相匹配，则执行相应的处理，反之则丢弃。

<img src="C:\Users\方也\AppData\Roaming\Typora\typora-user-images\image-20200313221631790.png" alt="image-20200313221631790" style="zoom:67%;" />

## 8.2 Iptables

### 8.2.1 策略与规则链

iptables服务把用于处理或过滤流量的策略条目称之为规则，多条规则可以组成一个规则链，而规则链则依据数据包处理位置的不同进行分类，具体如下：

> 在进行路由选择前处理数据包（PREROUTING）；
>
> 处理流入的数据包（INPUT）；
>
> 处理流出的数据包（OUTPUT）；
>
> 处理转发的数据包（FORWARD）；
>
> 在进行路由选择后处理数据包（POSTROUTING）。

ACCEPT（允许流量通过）、REJECT（拒绝流量通过）、LOG（记录日志信息）、DROP（拒绝流量通过）.就DROP来说，它是直接将流量丢弃而且不响应；REJECT则会在拒绝流量后再回复一条“您的信息已经收到，但是被扔掉了”信息，从而让流量发送方清晰地看到数据被拒绝的响应信息。

### 8.2.2 基本的命令参数

iptables命令可以根据流量的源地址、目的地址、传输协议、服务类型等信息进行匹配，一旦匹配成功，iptables就会根据策略规则所预设的动作来处理这些流量。

> 防火墙策略规则的匹配顺序是**从上至下**的，因此要把较为严格优先级较高的策略规则放到前面

表8-1                      iptables中常用的参数以及作用

| 参数        | 作用                                         |
| ----------- | -------------------------------------------- |
| -P          | 设置默认策略                                 |
| -F          | 清空规则链                                   |
| -L          | 查看规则链                                   |
| -A          | 在规则链的末尾加入新规则                     |
| -I num      | 在规则链的头部加入新规则                     |
| -D num      | 删除某一条规则                               |
| -s          | 匹配来源地址IP/MASK，加叹号“!”表示除这个IP外 |
| -d          | 匹配目标地址                                 |
| -i 网卡名称 | 匹配从这块网卡流入的数据                     |
| -o 网卡名称 | 匹配从这块网卡流出的数据                     |
| -p          | 匹配协议，如TCP、UDP、ICMP                   |
| --dport num | 匹配目标端口号                               |
| --sport num | 匹配来源端口号                               |

```
# iptables -F		//-F参数清空已有的防火墙规则链
# iptables -L		//-L参数查看已有的防火墙规则链
# iptables -P INPUT DROP //INPUT规则链的默认策略设置为拒绝,默认策略拒绝动作只能是DROP
# iptables - I INPUT icmp -j ACCEPT  //向INPUT链中添加允许ICMP流量进入
# iptables -D INPUT 1		//删除INPUT规则链
# iptables -INPUT ACCEPT	//把默认策略设置为允许
//将INPUT规则链设置为只允许指定网段的主机访问本机的22端口，拒绝来自其他所有主机的流量：
//允许动作放到拒绝动作前面  防火墙动作从上到下
# iptables -I INPUT -s 192.168.10.0/24 -p tcp --dport 22 -j ACCEPT
# iptables -A INPUT -p tcp --dport 22 -j REJECT
# ssh 192.168.10.10
// 向INPUT规则链中添加拒绝所有人访问本机12345端口的策略规则：
# iptables -I INPUT -p tcp --dport 12345 -j REJECT
# iptables -I INPUT -p udp --dport 12345 -j REJECT
// 向INPUT规则链中添加拒绝192.168.10.5主机访问本机80端口（Web服务）
# iptables -I INPUT -p tcp -s 192.168.10.5 --dport 80 -j REJECT
// 向INPUT规则链中添加拒绝所有主机访问本机1000～1024端口
# iptables -A INPUT -p tcp --dport 1000:1024 -j REJECT
# iptables -A INPUT -p udp --dport 1000:1024 -j REJECT
# service iptables save    //防火墙策略永久生效，还要执行保存命令
```

 ## 8.3 Firewalld

firewalld支持动态更新技术并加入了区域（zone）的概念。区域就是firewalld预先准备了几套**防火墙策略集合**（策略模板），用户可以根据**生产场景**的不同而选择合适的策略集合，从而实现防火墙策略之间的**快速切换**。

表8-2                   firewalld中常用的区域名称及策略规则

| 区域     | 默认规则策略                                                 |
| -------- | ------------------------------------------------------------ |
| trusted  | 允许所有的数据包                                             |
| home     | 拒绝流入的流量，除非与流出的流量相关；而如果流量与ssh、mdns、ipp-client、amba-client与dhcpv6-client服务相关，则允许流量 |
| internal | 等同于home区域                                               |
| work     | 拒绝流入的流量，除非与流出的流量相关；而如果流量与ssh、ipp-client与dhcpv6-client服务相关，则允许流量 |
| public   | 拒绝流入的流量，除非与流出的流量相关；而如果流量与ssh、dhcpv6-client服务相关，则允许流量 |
| external | 拒绝流入的流量，除非与流出的流量相关；而如果流量与ssh服务相关，则允许流量 |
| dmz      | 拒绝流入的流量，除非与流出的流量相关；而如果流量与ssh服务相关，则允许流量 |
| block    | 拒绝流入的流量，除非与流出的流量相关                         |
| drop     | 拒绝流入的流量，除非与流出的流量相关                         |

### 8.3.1 终端管理工具

firewall-cmd是firewalld防火墙配置管理工具的CLI（命令行界面）版本。设置的策略只有在系统重启后才能自动生效。策略默认为Runtime模式，随系统重启失效。要永存，则使用Permanen模式

> 如果想让配置的策略立即生效，需要手动执行firewall-cmd --reload命令。

表8-3                  firewall-cmd命令中使用的参数以及作用

| 参数                          | 作用                                                 |
| ----------------------------- | ---------------------------------------------------- |
| --get-default-zone            | 查询默认的区域名称                                   |
| --set-default-zone=<区域名称> | 设置默认的区域，使其永久生效                         |
| --get-zones                   | 显示可用的区域                                       |
| --get-services                | 显示预先定义的服务                                   |
| --get-active-zones            | 显示当前正在使用的区域与网卡名称                     |
| --add-source=                 | 将源自此IP或子网的流量导向指定的区域                 |
| --remove-source=              | 不再将源自此IP或子网的流量导向某个指定区域           |
| --add-interface=<网卡名称>    | 将源自该网卡的所有流量都导向某个指定区域             |
| --change-interface=<网卡名称> | 将某个网卡与区域进行关联                             |
| --list-all                    | 显示当前区域的网卡配置参数、资源、端口以及服务等信息 |
| --list-all-zones              | 显示所有区域的网卡配置参数、资源、端口以及服务等信息 |
| --add-service=<服务名>        | 设置默认区域允许该服务的流量                         |
| --add-port=<端口号/协议>      | 设置默认区域允许该端口的流量                         |
| --remove-service=<服务名>     | 设置默认区域不再允许该服务的流量                     |
| --remove-port=<端口号/协议>   | 设置默认区域不再允许该端口的流量                     |
| --reload                      | 让“永久生效”的配置规则立即生效，并覆盖当前的配置规则 |
| --panic-on                    | 开启应急状况模式                                     |
| --panic-off                   | 关闭应急状况模式                                     |

```
# firewall-cmd --get-default-zone  //查看firewalld服务当前所使用的区域：
# firewall-cmd --get-zone-of-interface=eno16777728 //查询eno16777728网卡在firewalld服务中的区域
//firewalld服务中eno16777728网卡的默认区域修改为external，并在系统重启后生效
# firewall-cmd --permanent --zone=external --change-interface=eno16777728
# firewall-cmd --set-default-zone=public //firewalld服务的当前默认区域设置为public
//应急状况模式
# firewall-cmd --panic-on
# firewall-cmd --panic-off
//查询public区域是否允许请求SSH和HTTPS协议的流量：
# firewall-cmd --zone=public --query-service=ssh
yes
# firewall-cmd --zone=public --query-service=https
no
//把firewalld服务中请求HTTPS协议的流量设置为永久允许，并立即生效
# firewall-cmd --zone=public --add-service=https
# firewall-cmd --permanent --zone=public --add-service=https
# firewall-cmd --reload
//把firewalld服务中请求HTTP协议的流量设置为永久拒绝，并立即生效：
# firewall-cmd --permanent --zone=public --remove-service=http 
# firewall-cmd --reload
//访问8080和8081端口的流量策略设置为允许
# firewall-cmd --zone=public --add-port=8080-8081/tcp 
```

- 把原本访问本机888端口的流量转发到22端口，要且求当前和长期均有效：

> 流量转发命令格式为firewall-cmd --permanent --zone=**<区域>** --add-forward-port=port=<源端口号>:proto=**<协议>**:toport=**<目标端口号>**:toaddr=**<目标IP地址>**

```
# firewall-cmd --permanent  --zone=public --add-forward-port=port=888:proto=tcp:toport=22:toaddr=192.168.10.10
# firewall-cmd --reload
# ssh -p 888 192.168.10.10	//访问192.168.10.10主机的888端口：
```

- 在firewalld服务中配置一条富规则，使其拒绝192.168.10.0/24网段的所有用户访问本机的ssh服务（22端口）：

```
# firewall-cmd --permanent --zone=public --add-rich-rule="rule family"="ipv4" 
source address="192.168.10.0/24" service name="ssh reject"
# firewall-cmd --reload
```

### 8.3.2 图形管理工具

> **1：**选择运行时（Runtime）模式或永久（Permanent）模式的配置。
>
> **2**：可选的策略集合区域列表。
>
> **3**：常用的系统服务列表。
>
> **4**：当前正在使用的区域。
>
> **5**：管理当前被选中区域中的服务。
>
> **6**：管理当前被选中区域中的端口。
>
> **7**：开启或关闭SNAT（源地址转换协议）技术。
>
> **8**：设置端口转发策略。
>
> **9**：控制请求icmp服务的流量。
>
> **10**：管理防火墙的富规则。
>
> **11**：管理网卡设备。
>
> **12**：被选中区域的服务，若勾选了相应服务前面的复选框，则表示允许与之相关的流量。
>
> **13**：firewall-config工具的运行状态。

<img src="C:\Users\方也\AppData\Roaming\Typora\typora-user-images\image-20200313232930162.png" alt="image-20200313232930162" style="zoom:67%;" />

SNAT（Source Network Address Translation，源网络地址转换）

<img src="C:\Users\方也\AppData\Roaming\Typora\typora-user-images\image-20200313233219630.png" alt="image-20200313233219630" style="zoom:67%;" />

<img src="C:\Users\方也\AppData\Roaming\Typora\typora-user-images\image-20200313233245398.png" alt="image-20200313233245398" style="zoom:67%;" />

## 8.4 服务的访问控制列表

TCP Wrappers是RHEL 7系统中默认启用的一款流量监控程序，它能够根据来访主机的地址与本机的目标服务程序作出允许或拒绝的操作。

TCP Wrappers服务的防火墙策略由两个控制列表文件所控制，用户可以编辑允许控制列表文件来放行对服务的请求流量，也可以编辑拒绝控制列表文件来阻止对服务的请求流量。控制列表文件修改后会立即生效，系统将会先检查允许控制列表文件，进一步匹配拒绝控制列表文件

表8-4              TCP Wrappers服务的控制列表文件中常用的参数

| 客户端类型     | 示例                       | 满足示例的客户端列表               |
| -------------- | -------------------------- | ---------------------------------- |
| 单一主机       | 192.168.10.10              | IP地址为192.168.10.10的主机        |
| 指定网段       | 192.168.10.                | IP段为192.168.10.0/24的主机        |
| 指定网段       | 192.168.10.0/255.255.255.0 | IP段为192.168.10.0/24的主机        |
| 指定DNS后缀    | .linuxprobe.com            | 所有DNS后缀为.linuxprobe.com的主机 |
| 指定主机名称   | www.linuxprobe.com         | 主机名称为www.linuxprobe.com的主机 |
| 指定所有客户端 | ALL                        | 所有主机全部包括在内               |

在配置TCP Wrappers服务时需要遵循两个原则：

1. 编写拒绝策略规则时，填写的是服务名称，而非协议名称；
2. 建议先编写拒绝策略规则，再编写允许策略规则，以便直观地看到相应的效果

```
# vim /etc/hosts.allow
(sshd:*)	//禁止访问本机sshd服务的所有流 (拒绝策略)
sshd:192.168.10	//放行源自192.168.10.0/24网段(允许策略)
```

