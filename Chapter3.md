# Chapter 3 . 管道符、重定向与环境变量

[TOC]



## 3.1 输入输出重定向

输入重定向是指把文件导入到命令中。输出重定向则是指把原本要输出到屏幕的数据信息写入到指定文件中。

 标准输入重定向（STDIN，文件描述符为0）：默认从键盘输入，也可从其他文件或命令中输入。

标准输出重定向（STDOUT，文件描述符为1）：默认输出到屏幕。

错误输出重定向（STDERR，文件描述符为2）：默认输出到屏幕。

| 符号                             | 作用                                                         |
| :------------------------------- | :----------------------------------------------------------- |
| 命令 < 文件                      | 将文件作为命令的标准输入                                     |
| 命令 << 分界符                   | 从标准输入中读入，直到遇见分界符才停止                       |
| 命令 < 文件1 > 文件2             | 将文件1作为命令的标准输入并将标准输出到文件2                 |
| 命令 > 文件                      | 将标准输出重定向到一个文件中（清空原有文件的数据）           |
| 命令 2> 文件                     | 将错误输出重定向到一个文件中（清空原有文件的数据）           |
| 命令 >> 文件                     | 将标准输出重定向到一个文件中（追加到原有内容的后面）         |
| 命令 2>> 文件                    | 将错误输出重定向到一个文件中（追加到原有内容的后面）         |
| 命令 >> 文件 2>&1或命令 &>> 文件 | 将标准输出与错误输出共同写入到文件中（追加到原有内容的后面） |

e.g.  # man bash > readme.txt

#echo "Hello, world" > readme.txt    //覆盖写入

#echo "Hello, world" >> readme.txt  // 追加写入

```
[root@linuxprobe ~]# ls -l linuxprobe > /root/stderr.txt 
[root@linuxprobe ~]# ls -l linuxprobe 2> /root/stderr.txt 
-rw-r--r--. 1 root root 0 Mar  1 13:30 linuxprobe
```

```
[root@linuxprobe ~]# ls -l xxxxxx > /root/stderr.txt    //stdout
cannot access xxxxxx: No such file or directory
[root@linuxprobe ~]# ls -l xxxxxx 2> /root/stderr.txt  //stderr
[root@linuxprobe ~]# cat /root/stderr.txt 
ls: cannot access xxxxxx: No such file or directory
```

```
# wc -l < readme.txt   //输入重定向
```

## 3.2 管道命令符

“命令A | 命令B”  == 把前一个命令原本要输出到屏幕的标准正常数据当作是后一个命令的标准输入”

``` 
# grep "/sbin/nologin" /etc/passwd | wc -l
# ls -l /etc/ | more
# echo "linuxpro" | passwd --stdin root   //2次密码确认
# echo "Content" | mail -s "Subject" linuxpro //邮件内容标题打包发送
# mail -s "Readme" root@linuxprobe.com << over//一直输入内容，直到输入了的分界符时
```

“命令A | 命令B | 命令C”

## 3.3 命令行通配符

（*）代表匹配零个或多个字符，（?）代表匹配单个字符，[0-9]代表匹配0～9之间的单个数字的字符，[abc]则是代表匹配a、b、c三个字符中的任意一个字符。

```
# ls -l /dev/sda*
# ls -l /dev/sda?
# ls -l /dev/sda[0-9]
# ls -l /dev/sda[135]   //匹配135数字
```

##  3.4 常用的转义字符

>反斜杠（\）：使反斜杠后面的一个变量变为单纯的字符串。
>
>单引号（''）：转义其中所有的变量为单纯的字符串。
>
>双引号（""）：保留其中的变量属性，不进行转义处理。
>
>反引号（``）：把其中的命令执行后返回结果。

```
# PRICE = 5
# echo "Price is $PRICE"
# echo "Price is $$PRICE"  //$$进程ID
# echo "Price is \$$PRICE" 
# echo `uname -a`
```

## 3.5 重要的环境变量

> 变量名称一般都是大写

命令在Linux中的执行分为4个步骤:

- 判断用户是否以绝对路径或相对路径的方式输入命令（如/bin/ls）
- 检查命令是否别名命令（alias 别名=命令”， “unalias 别名”）
- Bash解释器判断的是内部命令还是外部命令（“type命令名称”来判断）
- 系统在多个路径中查找用户输入的命令文件（**PATH**：由多个路径值组成的变量，每个路径值之间用冒号间隔）

```
# PATH=$PATH:/root/bin   //增加路径值
```

| 变量名称     | 作用                             |
| ------------ | -------------------------------- |
| HOME         | 用户的主目录（即家目录）         |
| SHELL        | 用户在使用的Shell解释器名称      |
| HISTSIZE     | 输出的历史命令记录条数           |
| HISTFILESIZE | 保存的历史命令记录条数           |
| MAIL         | 邮件保存路径                     |
| LANG         | 系统语言、语系名称               |
| RANDOM       | 生成一个随机数字                 |
| PS1          | Bash解释器的提示符               |
| PATH         | 定义解释器搜索用户执行命令的路径 |
| EDITOR       | 用户默认的文本编辑器             |

变量使用

```
# mkdir /home/workdir
# WORKDIR=/home/workdir   //局部变量
# export WOEKDIR       		//全局变量
```

