# chapter 4 Vim编辑器与Shell命令脚本

## 4.1 Vim文本编辑器

> 命令模式：控制光标移动，可对文本进行复制、粘贴、删除和查找等工作。
>
> 输入模式：正常的文本录入
>
> 末行模式：保存或退出文档，以及设置编辑环境。

切换方法：

![image-20200311100951832](C:\Users\方也\AppData\Roaming\Typora\typora-user-images\image-20200311100951832.png)

常用命令：

| 命令 | 作用                                               |
| ---- | -------------------------------------------------- |
| dd   | 删除（剪切）光标所在整行                           |
| 5dd  | 删除（剪切）从光标处开始的5行                      |
| yy   | 复制光标所在整行                                   |
| 5yy  | 复制从光标处开始的5行                              |
| n    | 显示搜索命令定位到的下一个字符串                   |
| N    | 显示搜索命令定位到的上一个字符串                   |
| u    | 撤销上一步的操作                                   |
| p    | 将之前删除（dd）或复制（yy）过的数据粘贴到光标后面 |

| 命令（末行模式） | 作用                                 |
| ---------------- | ------------------------------------ |
| :w               | 保存                                 |
| :q               | 退出                                 |
| :q!              | 强制退出（放弃对文档的修改内容）     |
| :wq!             | 强制保存退出                         |
| :set nu          | 显示行号                             |
| :set nonu        | 不显示行号                           |
| :命令            | 执行该命令                           |
| :整数            | 跳转到该行                           |
| :s/one/two       | 将当前光标所在行的第一个one替换成two |
| :s/one/two/g     | 将当前光标所在行的所有one替换成two   |
| :%s/one/two/g    | 将全文中的所有one替换成two           |
| ?字符串          | 在文本中从下至上搜索该字符串         |
| /字符串          | 在文本中从上至下搜索该字符串         |

### 4.1.1 编写简单文档

### 4.1.2 配置主机名称

修改vim /etc/hostname 内容名称

### 4.1.3 配置网卡信息

```
# cd /etc/sysconfig/network-scripts/
# ls 
# vim ifcfg-etho
```

### 4.1.4 配置Yum仓库

```
# cd /etc/yum.repos.d/
# vim rhel7.repo
```

>**[rhel-media]** ：Yum软件仓库唯一标识符，避免与其他仓库冲突。
> **name=linuxprobe**：Yum软件仓库的名称描述，易于识别仓库用处。
> **baseurl=file:///media/cdrom**：提供的方式包括FTP（ftp://..）、HTTP（http://..）、本地（file:///..）。
> **enabled=1**：设置此源是否可用；1为可用，0为禁用。
> **gpgcheck=1**：设置此源是否校验文件；1为校验，0为不校验。
> **gpgkey=file:///media/cdrom/RPM-GPG-KEY-redhat-release**：若上面参数开启校验，那么请指定公钥文件地址。

## 4.2 编写Shell脚本

Shell脚本命令工作方式：

> 交互式（Interactive）：用户每输入一条命令就立即执行。
>
> 批处理（Batch）：由用户事先编写好一个完整的Shell脚本，一次性执行脚本中诸多的命令。

### 4.2.1 编写简单的脚本

```
# vim example.sh
# bash example.sh   //bash解释器执行
# chmod u+x example.sh
# ./example.sh		//输入完整路径方式执行
```

### 4.2.2 接受用户参数

$0对应的是当前Shell脚本程序的名称，$#对应的是总共有几个参数，$*对应的是所有位置的参数值，$?对应的是显示上一次命令的执行返回值，而$1、$2、$3……则分别对应着第N个位置的参数值

![image-20200311150914819](C:\Users\方也\AppData\Roaming\Typora\typora-user-images\image-20200311150914819.png)

```
# vim Example.sh
#!/bin/bash
echo "当前脚本名称为$0"
echo "总共有$#个参数，分别是$*。"
echo "第1个参数为$1，第5个为$5。"
```

### 4.2.3 判断用户的参数

条件测试语法：成立返回0，否则返回其他随机数字

![image-20200311151928898](C:\Users\方也\AppData\Roaming\Typora\typora-user-images\image-20200311151928898.png)

条件测试语句4种：

- 文件测试语句

- 逻辑测试语句

- 整数值比较语句

- 字符串比较语句

1. 文件测试：指定条件来判断文件是否存在或权限是否满足等情况的运算符

| 操作符 | 作用                           |
| ------ | ------------------------------ |
| -d     | 测试文件是否为**目录**类型     |
| -e     | 测试文件是否**存在**           |
| -f     | 判断是否为**一般**文件         |
| -r     | 测试当前用户是否有权限**读取** |
| -w     | 测试当前用户是否有权限**写入** |
| -x     | 测试当前用户是否有权限**执行** |

```
# [ -d /etc/fstab ]
# echo $?    //$?显示上一条命令执行后的返回值
# [ -f /etc/fstab ]
# [ -e /etc/fstab ] && echo "Exist"
# [ $USER = root ] || echo "user"  //||，表示当前面的命令执行失败后才会执行它后面的命令
```

2. 整数比较运算符

   | 操作符 | 作用           |
   | ------ | -------------- |
   | -eq    | 是否等于       |
   | -ne    | 是否不等于     |
   | -gt    | 是否大于       |
   | -lt    | 是否小于       |
   | -le    | 是否等于或小于 |
   | -ge    | 是否大于或等于 |

```
# [ 10 -gt 10 ]
# echo $?
# FreeMem=`free -m | grep Mem: | awk '{print $4}'` //awk保留第四列
# [ FreeMem -lt 1024 ] && echo "Insufficent Memory"
```

3. 字符串比较运算符
   

判断测试字符串是否为空值，或两个字符串是否相同,某个变量是否未被定义

| 操作符 | 作用                   |
| ------ | ---------------------- |
| =      | 比较字符串内容是否相同 |
| !=     | 比较字符串内容是否不同 |
| -z     | 判断字符串内容是否为空 |

```
# [ -z $String ]
```

## 4.3 流程控制语句

### 4.3.1 if条件测试语句

![image-20200311161239447](C:\Users\方也\AppData\Roaming\Typora\typora-user-images\image-20200311161239447.png)

```
# vim chkhost.sh
#!/bin/bash
ping -c 3 -i 0.2 -W 3 $1 &> /dev/null
if [ $? -eq 0 ]
then
echo "Host $1 is On-line."
else
echo "Host $1 is Off-line."
fi
```

### 4.3.2 for条件循环语句

![image-20200311162235820](C:\Users\方也\AppData\Roaming\Typora\typora-user-images\image-20200311162235820.png)

```
[root@linuxprobe ~]# vim Example.sh
#!/bin/bash
read -p "Enter The Users Password : " PASSWD
for UNAME in `cat users.txt`    //`` 命令行执行后返回值
do
id $UNAME &> /dev/null			//"id 用户名"查看用户信息
if [ $? -eq 0 ]
then
echo "Already exists"
else
useradd $UNAME &> /dev/null		//  /dev/null  回收机制
echo "$PASSWD" | passwd --stdin $UNAME &> /dev/null
if [ $? -eq 0 ]
then
echo "$UNAME , Create success"
else
echo "$UNAME , Create failure"
fi
fi
done
```

### 4.3.3 while循环语句

<img src="C:\Users\方也\AppData\Roaming\Typora\typora-user-images\image-20200311164506303.png" alt="image-20200311164506303" style="zoom:80%;" />

### 4.3.4 case条件测试语句

<img src="C:\Users\方也\AppData\Roaming\Typora\typora-user-images\image-20200311164730908.png" alt="image-20200311164730908" style="zoom: 80%;" />

```
[root@linuxprobe ~]# vim Checkkeys.sh
#!/bin/bash
read -p "请输入一个字符，并按Enter键确认：" KEY
case "$KEY" in
[a-z]|[A-Z])
echo "您输入的是 字母。"
;;
[0-9])
echo "您输入的是 数字。"
;;
*)
echo "您输入的是 空格、功能键或其他控制字符。"
esac
```

## 4.4 计划任务服务程序

>一次性计划任务：今晚11点30分开启网站服务。
> 长期性计划任务：每周一的凌晨3点25分把/home/wwwroot目录打包备份为backup.tar.gz。

```
# at 23:30			//一次性任务
at > systemctl restart httpd
# at -l
# atrm 1
# echo "systemctl restart httpd" | at 23:30
```

- 长期性计划

  创建、编辑计划任务的命令为“crontab -e”，查看当前计划任务的命令为“crontab -l”，删除某条计划任务的命令为“crontab -r”。

<img src="C:\Users\方也\AppData\Roaming\Typora\typora-user-images\image-20200311170109293.png" alt="image-20200311170109293" style="zoom:80%;" />

```
[root@linuxprobe ~]# crontab -e
no crontab for root - using an empty oneS
```

