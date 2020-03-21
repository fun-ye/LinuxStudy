# Chapter 7 使用RAID与LVM磁盘阵列技术

## 7.1 RAID磁盘冗余列阵

[TOC]

1. RAID 0

RAID 0技术把多块物理硬盘设备（至少两块）通过硬件或软件的方式串联在一起，组成一个大的卷组，并将数据依次写入到各个物理硬盘中。

<img src="C:\Users\方也\AppData\Roaming\Typora\typora-user-images\image-20200312204519684.png" alt="image-20200312204519684" style="zoom: 67%;" />

2. RAID 1

把两块以上的硬盘设备进行绑定，在写入数据时，是将数据同时写入到多块硬盘设备上（可以将其视为数据的镜像或备份）。当其中某一块硬盘发生故障后，一般会立即自动以热交换的方式来恢复数据的正常使用。

<img src="C:\Users\方也\AppData\Roaming\Typora\typora-user-images\image-20200312204643055.png" alt="image-20200312204643055" style="zoom:67%;" />

3. RAID 5

RAID5技术是把硬盘设备的数据奇偶校验信息保存到其他硬盘设备中。RAID 5磁盘阵列组中数据的奇偶校验信息并不是单独保存到某一块硬盘设备中，而是存储到除自身以外的其他每一块硬盘设备上，这样的好处是其中任何一设备损坏后不至于出现致命缺陷；

<img src="C:\Users\方也\AppData\Roaming\Typora\typora-user-images\image-20200312205031946.png" alt="image-20200312205031946" style="zoom:50%;" />

4. RAID 10

   RAID 10技术是RAID 1+RAID 0技术的一个“组合体”。RAID 10技术需要至少4块硬盘来组建，其中先分别两两制作成RAID 1磁盘阵列，以保证数据的安全性；然后再对两个RAID 1磁盘阵列实施RAID 0技术，进一步提高硬盘设备的读写速度。

<img src="C:\Users\方也\AppData\Roaming\Typora\typora-user-images\image-20200312205250170.png" alt="image-20200312205250170" style="zoom:50%;" />

### 7.1.1 部署磁盘阵列

mdadm命令用于管理Linux系统中的软件RAID硬盘阵列---"mdadm [模式] <RAID设备名称> [选项] [成员设备名称]"

| 参数 | 作用               |
| ---- | ------------------ |
| -a   | 检测设备名称       |
| -n   | 指定设备数量       |
| -l   | 指定RAID级别       |
| -C   | 创建一个RAID阵列卡 |
| -v   | 显示过程           |
| -f   | 模拟设备损坏       |
| -r   | 移除设备           |
| -Q   | 查看摘要信息       |
| -D   | 查看详细信息       |
| -S   | 停止RAID磁盘阵列   |

```
# mdadm -Cv /dev/md0 -a yes -n 4 -l 10 /dev/sdb /dev/sdc /dev/sdc /dev/sde
# mkfs.ext4 /dev/md0
# mount /dev/md0 /RAID
```

-v参数显示创建的过程

### 7.1.2 损坏磁盘列阵及修复

```
# mdadm /dev/md0 -f /dev/sdb
# umount /RAID
# mdadm /dev/md0 -a /dev/sdb
```

### 7.1.3 磁盘列阵+备份盘

```
# mdadm -Cv /dev/md0 -n 3 -l 5 -x 1 /dev/sdb /dev/sdc /dev/sde   //-x备份盘
# mkfs.ext4 /dev/md0 			//格式化为ext4文件格式
# echo "/dev/md0 /RAID ext4 defaults 0 0" >> /etc/fstab  //挂载永久关联
# mkdir /RAID
# mount -a 
# mdadm /dev/md0 -f /dev/sdb
# mdadm -D /dev/md0
```

##  7.2 LVM逻辑管理器

LVM技术是在硬盘分区和文件系统之间添加了一个逻辑层，它提供了一个抽象的卷组，可以把多块硬盘进行卷组合并。这样一来，用户不必关心物理硬盘设备的底层架构和布局，就可以实现对硬盘分区的动态调整。

<img src="C:\Users\方也\AppData\Roaming\Typora\typora-user-images\image-20200312212055485.png" alt="image-20200312212055485" style="zoom: 80%;" />

卷组建立在物理卷之上，一个卷组可以包含多个物理卷，而且在卷组创建之后也可以继续向其中添加新的物理卷。逻辑卷是用卷组中空闲的资源建立的，并且逻辑卷在建立后可以动态地扩展或缩小空间。

### 7.2.1 部署逻辑卷

| 功能/命令 | 物理卷管理 | 卷组管理  | 逻辑卷管理 |
| --------- | ---------- | --------- | ---------- |
| 扫描      | pvscan     | vgscan    | lvscan     |
| 建立      | pvcreate   | vgcreate  | lvcreate   |
| 显示      | pvdisplay  | vgdisplay | lvdisplay  |
| 删除      | pvremove   | vgremove  | lvremove   |
| 扩展      |            | vgextend  | lvextend   |
| 缩小      |            | vgreduce  | lvreduce   |

```
第1步：让新添加的两块硬盘设备支持LVM技术。
# pvcreate /dev/sdb /dev/sdc
第2步：把两块硬盘设备加入到storage卷组中，然后查看卷组的状态。
# vgcreate storge /dev/sdb /dev/sdc
第3步：切割出一个约为150MB的逻辑卷设备。
# lvcreate -n vo -l 37 storge	//-l 37可以生成一个大小为37×4MB=148MB的逻辑卷
# lvcreate -n vo -L 150M storge	//-L 150M生成一个大小为150MB的逻辑卷
第4步：把生成好的逻辑卷进行格式化，然后挂载使用。
# mkfs.ext4 /dev/storge/vo
# mkdir /linuxpro
# mount /dev/storage/vo /linuxpro
第5步：查看挂载状态，并写入到配置文件，使其永久生效。
# df -h
# echo "/dev/storage/vp /linuxpro ext4 defaults 0 0" >> /etc/fstab
```

### 7.2.2 扩容逻辑卷

扩展前请一定要记得卸载设备和挂载点的关联。

```
# umount /linuxpro
第1步：把上一个实验中的逻辑卷vo扩展至290MB。
# lvextend -L 290M /dev/storage/vo
第2步：检查硬盘完整性，并重置硬盘容量。
# e2fsck -f /dev/storage/vo
第3步：重新挂载硬盘设备并查看挂载状态。
# mount -a
# df -h
```

### 7.2.3 缩小逻辑卷

```
# umount /linpro
第1步：检查文件系统的完整性。
# e2fsck -f /dev/storage/vo
第2步：把逻辑卷vo的容量减小到120MB。
# resizefs /dec/storage/vo 120M
第3步：重新挂载文件系统并查看系统状态。
# mount -a
# df -h
```

### 7.2.4 逻辑卷快照

LVM还具备有“快照卷”功能，该功能类似于虚拟机软件的还原时间点功能

> 快照卷的容量必须等同于逻辑卷的容量；
>
> 快照卷仅一次有效，一旦执行还原操作后则会被立即自动删除。

```
首先查看卷组的信息。
# vgdisplay
重定向往逻辑卷设备所挂载的目录中写入一个文件。
# echo "wcel" > /linuxpro/readme.txt
# ls -l /linuxpro
```
```
第1步：使用-s参数生成一个快照卷，使用-L参数指定切割的大小
# lvcreate -L 120M -s -n SNAP /dev/storage/vo
	Logical volume "SNAP" created

第2步：在逻辑卷所挂载的目录中创建一个100MB的垃圾文件，然后再查看快照卷的状态。可以发现存储空间占的用量上升了。
# dd if=/dev/zero of=/linuxprobe/files count=1 bs=100M
第3步：为了校验SNAP快照卷的效果，需要对逻辑卷进行快照还原操作。在此之前记得先卸载掉逻辑卷设备与目录的挂载。
# umount /linuxpro
# lvconvert --merge /dev/storage/SNAP
第4步：快照卷会被自动删除掉，并且刚刚在逻辑卷设备被执行快照操作后再创建出来的100MB的垃圾文件也被清除了。
# mount -a
```

### 7.2.5 删除逻辑卷

提前备份好重要的数据信息，然后依次删除逻辑卷、卷组、物理卷设备，这个顺序不可颠倒。
```
第1步：取消逻辑卷与目录的挂载关联，删除配置文件中永久生效的设备参数。
# umount /linuxpro
# vim /etc/fstab
第2步：删除逻辑卷设备，需要输入y来确认操作。
# lvremove /dev/storage/vo
第3步：删除卷组，此处只写卷组名称即可，不需要设备的绝对路径。
# vgremove storage
第4步：删除物理卷设备。
# premove /dev/sdb /dev/sdc
```

