# Chapter20 使用LNMP架构部署动态网络环境

LNMP动态网站部署架构是一套由Linux+ Nginx + MySQL + PHP组成的动态网站系统解决方案，具有免费、高效、扩展性强且资源消耗低等优良特性。

## 20.1 源码包程序

> 源码包的可移植性非常好，几乎可以在任何Linux系统中安装使用，而RPM软件包是针对特定系统和架构编写的指令集，必须严格地符合执行环境才能顺利安装（即只会去“生硬地”安装服务程序）。
>
> 使用源码包安装服务程序时会有一个编译过程，因此可以更好地适应安装主机的系统环境，运行效率和优化程度都会强于使用RPM软件包安装的服务程序。也就是说，可以将采用源码包安装服务程序的方式看作是针对系统的“量体裁衣”

**第1步**：下载及解压源码包文件。

```
# tar xzvf FileName.tar.gz
# cd FileDirectory
```

**第2步**：编译源码包代码。

```
# ./configure --prefix=/usr/local/program
```

**第3步**：生成二进制安装程序。

```
# make
```

**第4步**：运行二进制的服务程序安装包。

```
# make install
```

**第5步**：清理源码包临时文件

```
# make clean
```

## 20.2 LNMP动态网站架构

```
# yum install -y apr* autoconf automake bison bzip2 bzip2* compat
# cd /usr/local/src
# tar xzvf cmake-2.8.11.2.tar.gz
# cd cmake-2.8.11.2/
# ./configure
# make
# make install
```

### 20.2.1 配置MySQL服务

```
# cd ..
# useradd mysql -s /sbin/nologin
# mkdir -p /usr/local/mysql/var
# chown -Rf mysql:mysql /usr/local/mysql
# rm -rf /etc/my.cnf
# cd /usr/local/mysql
# ./scripts/mysql_install_db --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/var
# ln -s my.cnf /etc/my.cnf 
# cp ./support-files/mysql.server /etc/rc.d/init.d/mysqld
# chmod 755 /etc/rc.d/init.d/mysqld
# vim /etc/rc.d/init.d/mysqld 
# service mysqld start
# chkconfig mysqld on
# vim /etc/profile
[root@linuxprobe mysql]# mkdir /var/lib/mysql
[root@linuxprobe mysql]# ln -s /usr/local/mysql/lib/mysql /usr/lib/mysql
[root@linuxprobe mysql]# ln -s /tmp/mysql.sock /var/lib/mysql/mysql.sock
[root@linuxprobe mysql]# ln -s /usr/local/mysql/include/mysql /usr/include/mysql
# mysql_secure_installation 
```

### 20.2.2 配置Nginx服务

Nginx是一款相当优秀的用于部署动态网站的轻量级服务程序，它最初是为俄罗斯门户站点而开发的，因其稳定性、功能丰富、占用内存少且并发能力强而备受用户的信赖

Nginx服务程序的稳定性源自于采用了分阶段的资源分配技术，降低了CPU与内存的占用率

> 解决相关的软件依赖关系

```
[root@linuxprobe ~]# cd /usr/local/src
[root@linuxprobe src]# tar xzvf pcre-8.35.tar.gz 
[root@linuxprobe src]# cd pcre-8.35
[root@linuxprobe pcre-8.35]# ./configure --prefix=/usr/local/pcre
[root@linuxprobe pcre-8.35]# make
[root@linuxprobe pcre-8.35]# make install 
```

openssl软件包是用于提供网站加密证书服务的程序文件，在安装该程序时需要自定义服务程序的安装目录

```
[root@linuxprobe pcre-8.35]# cd /usr/local/src
[root@linuxprobe src]# tar xzvf openssl-1.0.1h.tar.gz
[root@linuxprobe src]# cd openssl-1.0.1h
[root@linuxprobe openssl-1.0.1h]# ./config --prefix=/usr/local/openssl
[root@linuxprobe openssl-1.0.1h]# make
[root@linuxprobe openssl-1.0.1h]# make install 
```

将这个目录添加到PATH环境变量中，并写入到配置文件中，最后执行source命令以便让新的PATH环境变量内容可以立即生效：

```
[root@linuxprobe pcre-8.35]# vim /etc/profile
 74 export PATH=$PATH:/usr/local/mysql/bin:/usr/local/openssl/bin
```

zlib软件包是用于提供压缩功能的函数库文件。

```
[root@linuxprobe pcre-8.35]# cd /usr/local/src
[root@linuxprobe src]# tar xzvf zlib-1.2.8.tar.gz 
[root@linuxprobe src]# cd zlib-1.2.8
[root@linuxprobe zlib-1.2.8]# ./configure --prefix=/usr/local/zlib
[root@linuxprobe zlib-1.2.8]# make
[root@linuxprobe zlib-1.2.8]# make install
```

创建一个用于执行Nginx服务程序的账户

```
[root@linuxprobe zlib-1.2.8]# cd ..
[root@linuxprobe src]# useradd www -s /sbin/nologin
```

-prefix参数用于定义服务程序稍后安装到的位置，--user与--group参数用于指定执行Nginx服务程序的用户名和用户组。在使用参数调用openssl、zlib、pcre软件包时，请写出软件源码包的解压路径，而不是程序的安装路径：

```
[root@linuxprobe src]# tar xzvf nginx-1.6.0.tar.gz 
[root@linuxprobe src]# cd nginx-1.6.0/
[root@linuxprobe nginx-1.6.0]# ./configure --prefix=/usr/local/nginx --without-http_memcached_module --user=www --group=www --with-http_stub_status_module --with-http_ssl_module --with-http_gzip_static_module --with-openssl=/usr/local/src/openssl-1.0.1h --with-zlib=/usr/local/src/zlib-1.2.8 --with-pcre=/usr/local/src/pcre-8.35
[root@linuxprobe nginx-1.6.0]# make
[root@linuxprobe nginx-1.6.0]# make install
```

启动Nginx服务程序以及将其加入到开机启动项中，也需要有脚本文件。

```
[root@linuxprobe nginx-1.6.0]# vim /etc/rc.d/init.d/nginx
[root@linuxprobe nginx-1.6.0]# chmod 755 /etc/rc.d/init.d/nginx
[root@linuxprobe nginx-1.6.0]# /etc/rc.d/init.d/nginx restart
Restarting nginx (via systemctl):                          [  OK  ]
[root@linuxprobe nginx-1.6.0]# chkconfig nginx on
```

### 20.2.3 配置PHP服务

PHP（Hypertxt Preprocessor，超文本预处理器）是一种通用的开源脚本语言，发明于1995年，它吸取了C语言、Java语言及Perl语言的很多优点，具有开源、免费、快捷、跨平台性强、效率高等优良特性，是目前Web开发领域最常用的语言之一

部署将近十个用于搭建网站页面的软件程序包，然后才能正式安装PHP程序。

```
[root@linuxprobe nginx-1.6.0]# cd ..
[root@linuxprobe src]# tar zxvf yasm-1.2.0.tar.gz
[root@linuxprobe src]# cd yasm-1.2.0
[root@linuxprobe yasm-1.2.0]# ./configure
[root@linuxprobe yasm-1.2.0]# make
[root@linuxprobe yasm-1.2.0]# make install
...
```

在开始编译php源码包之前，先定义一个名为LD_LIBRARY_PATH的全局环境变量，该环境变量的作用是帮助系统找到指定的动态链接库文件，这些文件是编译php服务源码包的必须元素之一。编译php服务源码包时，除了定义要安装到的目录以外，还需要依次定义配置php服务程序配置文件的保存目录、MySQL数据库服务程序所在目录、MySQL数据库服务程序配置文件所在目录，以及libpng、jpeg、freetype、libvpx、zlib、t1lib等服务程序的安装目录路径，并通过参数启动php服务程序的诸多默认功能：

```
[root@linuxprobe t1lib-5.1.2]# cd ..
[root@linuxprobe src]# tar -zvxf php-5.5.14.tar.gz
[root@linuxprobe src]# cd php-5.5.14
[root@linuxprobe php-5.5.14]# export LD_LIBRARY_PATH=/usr/local/libgd/lib
[root@linuxprobe php-5.5.14]# ./configure --prefix=/usr/local/php --with-config-file-path=/usr/local/php/etc --with-mysql=/usr/local/mysql --with-mysqli=/usr/local/mysql/bin/mysql_config --with-mysql-sock=/tmp/mysql.sock --with-pdo-mysql=/usr/local/mysql --with-gd --with-png-dir=/usr/local/libpng --with-jpeg-dir=/usr/local/jpeg --with-freetype-dir=/usr/local/freetype --with-xpm-dir=/usr/ --with-vpx-dir=/usr/local/libvpx/ --with-zlib-dir=/usr/local/zlib --with-t1lib=/usr/local/t1lib --with-iconv --enable-libxml --enable-xml --enable-bcmath --enable-shmop --enable-sysvsem --enable-inline-optimization --enable-opcache --enable-mbregex --enable-fpm --enable-mbstring --enable-ftp --enable-gd-native-ttf --with-openssl --enable-pcntl --enable-sockets --with-xmlrpc --enable-zip --enable-soap --without-pear --with-gettext --enable-session --with-mcrypt --with-curl --enable-ctype 
[root@linuxprobe php-5.5.14]# make
[root@linuxprobe php-5.5.14]# make install
```

要删除当前默认的配置文件，然后将php服务程序目录中相应的配置文件复制过来：

```
[root@linuxprobe php-5.5.14]# rm -rf /etc/php.ini
[root@linuxprobe php-5.5.14]# ln -s /usr/local/php/etc/php.ini /etc/php.ini
[root@linuxprobe php-5.5.14]# cp php.ini-production /usr/local/php/etc/php.ini
[root@linuxprobe php-5.5.14]# cp /usr/local/php/etc/php-fpm.conf.default /usr/local/php/etc/php-fpm.conf
[root@linuxprobe php-5.5.14]# ln -s /usr/local/php/etc/php-fpm.conf /etc/php-fpm.conf
```

```
[root@linuxprobe php-5.5.14]# vim /usr/local/php/etc/php-fpm.conf
[root@linuxprobe php-5.5.14]# cp sapi/fpm/init.d.php-fpm /etc/rc.d/init.d/php-fpm
[root@linuxprobe php-5.5.14]# chmod 755 /etc/rc.d/init.d/php-fpm
[root@linuxprobe php-5.5.14]# chkconfig php-fpm on
[root@linuxprobe php-5.5.14]# vim /usr/local/php/etc/php.ini
[root@linuxprobe php-5.5.14]# vim /usr/local/nginx/conf/nginx.conf
```

## 20.3 搭建Discuz论坛

```
[root@linuxprobe php-5.5.14 ]# cd /usr/local/src/
[root@linuxprobe src]# unzip Discuz_X3.2_SC_GBK.zip
[root@linuxprobe src]# rm -rf /usr/local/nginx/html/{index.html,50x.html}*
[root@linuxprobe src]# mv upload/* /usr/local/nginx/html/
[root@linuxprobe src]# chown -Rf www:www /usr/local/nginx/html
[root@linuxprobe src]# chmod -Rf 755 /usr/local/nginx/html
```

**第1步**：接受Discuz!安装向导的许可协议。

**第2步**：检查Discuz! X3.2论坛系统的安装环境及目录权限。

**第3步**：选择“全新安装Discuz! X（含UCenter Server）”。

**第4步**：填写服务器的数据库信息与论坛系统管理员信息。

**第5步**：等待Discuz! X3.2论坛系统安装完毕，

## 20.4 选购服务器主机

**虚拟主机**：在一台服务器中划分一定的磁盘空间供用户放置网站信息、存放数据等；仅提供基础的网站访问、数据存放与传输功能；能够极大地降低用户费用，也几乎不需要用户来维护网站以外的服务；适合小型网站。

**VPS（Virtual Private Server，虚拟专用服务器）**：在一台服务器中利用OpenVZ、Xen或KVM等虚拟化技术模拟出多台“主机”（即VPS），每个主机都有独立的IP地址、操作系统；不同VPS之间的磁盘空间、内存、CPU、进程与系统配置完全隔离，用户可自由使用分配到的主机中的所有资源，为此需要具备一定的维护系统的能力；适合小型网站。

**ECS（Elastic Compute Service，云服务器）**：是一种整合了计算、存储、网络，能够做到弹性伸缩的计算服务；使用起来与VPS几乎一样，差别是云服务器是建立在一组集群服务器中，每个服务器都会保存一个主机的镜像（备份），从而大大提升了安全性和稳定性；另外还具备灵活性与扩展性；用户只需按使用量付费即可；适合大中小型网站。

**独立服务器**：这台服务器仅提供给用户一个人使用，其使用方式分为租用方式与托管方式。租用方式是用户将服务器的硬件配置要求告知IDC服务商，按照月、季、年为单位来租用它们的硬件设备。这些硬件设备由IDC服务商的机房负责维护，用户一般需要自行安装相应的软件并部署网站服务，这减轻了用户在硬件设备上的投入，适合大中型网站。托管方式则是用户需要自行购置服务器硬件设备，并将其交给IDC服务供应商进行管理（需要缴纳管理服务费）。用户对服务器硬件配置有完全的控制权，自主性强，但需要自行维护、修理服务器硬件设备，适合大中型网站。