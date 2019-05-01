[TOC]

# 文件系统及相关概念介绍

## 相关概念介绍

### 文件系统格式

操作系统中，根据不同的存储形式，文件系统格式有很多分类！

linux中，最常见的格式有ext系列（2,3,4），xfs，9660以及最新的btrfs文件系统

创建文件系统，便是我们常说的格式化。 格式化分为两种，低级格式化和高级格式化，低级格式化指在对磁盘分区前，进行磁道的划分、 高级格式化，是在分区之后，为分区创建文件系统，也就是本章节要重点介绍的内容。

所谓的创建文件系统，首先将分区划分为固定大小的block，然后划分出元数据区，和数据区，元数据区，用来存放inode，数据区，用来存放实际数据；

对于较大的磁盘，系统会将其划分成多个块组，每个块组都有独立的元数据区和数据区， 为了便于管理，还要有一个超级块，用来存放每个块组的位置等相关信息

### 元数据

每个文件都有一组元数据，所谓的元数据，我将其理解为一个文件的索引，目录， 它包含了对应文件的文件大小，文件类型，属主，属组，权限，数据块指针以及三个时间戳，最近访问时间（access time），最近改动时间（change time）和最近更改时间（modification time）等内容！

其中modification time，表示最近更改文件内容的时间

change time 表示的则是最近更改元数据的时间，该时间不允许手动修改，它会随着创建链接，手动修改时间戳，修改文件等操作一同更新

**inode**

用来存储元数据的固定数据结构，一般来说，在创建文件系统的时候，就将其数量固定（这意味着，一个文件系统的文件数量也有最大限制，即使空间还有容量，也无法存储文件），用的时候，只需要向该节点中填充对应数据即可，其中，数据块指针存储位置的大小，决定了单个文件的最大上限

众多inode组成inode表，每个inode都有编号，而位图索引，由二进制数组成，每个数标识着对应的inode是否空闲

**综上所述，一个文件在文件系统中，由inode和数据块两部分组成**

**元数据查看，可以使用stat命令，后跟文件名即可**

### 目录

目录是一个特殊的文件，一个数据块，其中存放着文件和inode之间的对应关系（注意，文件名是存放在目录中的，不是元数据）

我们通常找一个文件的过程大概是这样的，列入，我们找/etc/passwd 文件，我们从根目录开始，首先找到/etc目录，在该数据块中找到passwd文件名所对应到的inode号，而后去找这个inode，最后根据这个inode中的数据块指针，找到passwd文件存放的真正位置，说起来可能有点绕，上图

![文件查找](https://github.com/huangtianfeng/pictures/blob/master/linux%E5%9B%BE%E5%BA%93/%E6%96%87%E4%BB%B6%E6%9F%A5%E6%89%BE.png)

注意，根目录下是存放有其下各目录和inode号的对应关系的，所以从根目录便可直接找到etc目录的inode！

### 软链接和硬连接

有了上面的概念，软链接和硬连接就很好理解了，所谓的软链接其实就是在某个目录下创建一个文件名和inode的映射关系，将其关联到你想指向的文件的inode上面

而硬连接，是创建一个新的inode，它和你想指向的文件，指向同一个数据块

说起来又有点绕了，老规矩，以passwd文件为例，建它的软链接和硬连接，上图

![软链接和硬连接](https://github.com/huangtianfeng/pictures/blob/master/linux%E5%9B%BE%E5%BA%93/%E8%BD%AF%E9%93%BE%E6%8E%A5%E5%92%8C%E7%A1%AC%E8%BF%9E%E6%8E%A5.png)

这个图的硬链接应该是指向inode，这里说明一下，有时间再来改一下！

软链接文件的inode中，存放数据块指针的位置，存放的是一个路径，当源文件被删除时，软链接即失效，但是如果创建另外一个和该路劲同名的文件，那么这个软链接又将重新生效

硬连接，是直接指向数据块的inode号的，及时源文件被删除，还是可以通过硬连接访问！

说到这里，提一下我们的删除文件和移动文件，大家有没有发现，在平常生活中，我们删除文件和移动文件的速度是非常快的，其实它只是把inode清空了，源文件还是在硬盘上的，通过特定手段还是可以恢复的！ 而移动文件（在同一个文件系统中），本质上就是删除原有目录下的文件名和inode对应关系，再重新在另一个目录下创建一个新的对应关系！（也就是一个硬连接）

注意： 硬连接，是直接指向磁盘数据块的，所以，硬连接不能跨文件系统，还有，作用对象不能是目录（大概是为了防止互相硬连接倒置无限循环）

软链接是指向一个路径的，所以，它可以跨文件系统，也可以对目录进行链接



# 磁盘的相关操作

## 磁盘分区

* fdisk工具：交互式分区管理工具

  fdisk -l DEVICE 可以列出指定磁盘设备的分区情况

  执行fdisk DEVICE 会提供一个交互式接口，管理分区，有许多子命令，常用命令有

  > n	创建新分区
  >
  > d	删除已有分区
  >
  > t	修改分区类型
  >
  > l	产看所有分区id
  >
  > w	保存并退出
  >
  > q	不保存并退出
  >
  > m	查看帮助信息
  >
  > p	显示现有分区信息

  注意：在交互式模式中，所有操作均在内存中进行，直到使用w保存到磁盘上才生效！ q退出，修改即无效

  **在已经分区并且已经挂其中某个分区的设备上创建分区时，内核可能无法直接识别；**

  **可以执行 partx -a 设备名（例/dev/sda）或者 kpartx -af 设备名  来使内核强制重新识别**

  **此命令可能不能一次成功，需要执行两次或以上**

  **cat /proc/partitions 可查看内核已识别的所有分区**

示例：创建一个10G大小的分区：

```bash
[root@localhost ~]# fdisk /dev/sdb
欢迎使用 fdisk (util-linux 2.23.2)。

更改将停留在内存中，直到您决定将更改写入磁盘。
使用写入命令前请三思。

Device does not contain a recognized partition table
使用磁盘标识符 0x75064cc3 创建新的 DOS 磁盘标签。

命令(输入 m 获取帮助)：p

磁盘 /dev/sdb：53.7 GB, 53687091200 字节，104857600 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节
磁盘标签类型：dos
磁盘标识符：0x75064cc3

   设备 Boot      Start         End      Blocks   Id  System

命令(输入 m 获取帮助)：n  
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): p
分区号 (1-4，默认 1)：1
起始 扇区 (2048-104857599，默认为 2048)：      
将使用默认值 2048
Last 扇区, +扇区 or +size{K,M,G} (2048-104857599，默认为 104857599)：+10G  
分区 1 已设置为 Linux 类型，大小设为 10 GiB
```



## 分区格式化

各种文件系统，有各自不同的格式化工具：

mkfs 工具的使用

mkfs是一个集合命令，可以使用mkfs.分区类型，来创建不同的文件系统，我们可以使用mkfs. 命令补全看一下

```bash
[root@localhost ~]# mkfs.
mkfs.btrfs   mkfs.cramfs  mkfs.ext2    mkfs.ext3    mkfs.ext4    mkfs.minix   mkfs.xfs    
```

也可是使用mkfs -t选项，指明格式类型，根据不同的类型，该命令会去调用以上的命令完成创建

使用语法：mkfs [-t fstype] [option] device ，对于mkfs命令，就不多做介绍了

#### ext系列文件系统的管理工具：

* mkfs.ext2（ext3/ext4)：创建文件系统

  使用语法：mkfs.xfs [options] device

  后接设备名即可：

* mke2fs：创建文件系统

  使用语法：mke2fs [options] device

  常用选项：

  > -t {ext2|ext3|ext4}:指明创建文件系统类型
  >
  > -b{1024|2048|6096} 指明文件系统的块大小，默认4k
  >
  > -L LABEL  指明卷标：
  >
  > -j  创建有日志功能的文件系统（ext3） ext2时代用来创建ext3系统的！ 
  >
  > -i # 指定每多少个字节对应一个inode，inode与字节的比率
  >
  > -N #  直接指明要创建的inode的数量
  >
  > -m # 指定预留空间，百分比数值，不需要加百分号，直接指定一个数字即可
  >
  > -O [^]FEATURE： 以指定的特性来创建目标文件系统！ 加^ 表示关闭此特性！

* e2lable：卷标查看与设定

  查看： e2lable device

  设定：e2lable device LABLE

* tune2fs：查看或修改ext系列文件系统的某些属性（并不是全部）

  **块大小，一旦格式化完成，就确定了，不能修改**

  用法格式：tune2fs [options] device

  常用选项：

  > -l：列出超级块中的内容
  >
  > -j：将ext2升级为ext3，无损升级
  >
  > -L：设定或修改卷标
  >
  > -m #：调整预留给管理员的空间百分比
  >
  > -O [^]feather: 开启或关闭某种特性
  >
  > -o [^]feather_options 调整默认挂载选项，多个用逗号分开即可

* dumpe2fs：显示ext系列文件系统的属性信息

  用法格式：dumpe2fs [options] device

  常用选项：不加选项时，会显示整个文件系统的每个块组的相关详细信息（包括超级块和各块组的信息）

* 文件系统检测的工具

  硬进程意外终止欧哲系统崩溃等原因导致定稿操作非正常终止时，可能会造成文件损坏，此时，应检测并修复文件系统；建议将文件系统卸载后进行

  * ext系列文件系统的专用工具：e2fsck：

    使用语法：e2fsck [option] device 

    常用选项：

    > -y： 对所有问题自动回答yes
    >
    > -f：强制执行，及时文件系统处于clean状态，也要强制进行检测

  * 通用工具：fsck

    使用语法：fsck [options] device

    常用选项：

    > -t fstype： 指明文件系统类型，和mkfs一样的，调用fsck.  命令
    >
    > -a：不用跟用户交互，而自动修复所有错误，不建议使用，修复一般是把损坏的文件删除！
    >
    > -r：交互式修复

* 文件系统查看查找工具：blkid

  用法：

  blkid device：查看某文件系统的信息

  blkid -L LABEL：根据label定位设备

  blkid -U UUID：根据UUID定位设备

示例：将前面创建的分区格式化为ext4文件系统，将block大小设置为2048，预留空间20%，卷标为MYDATA

```bash
[root@localhost ~]# mke2fs -L MYDATA -m 20 -b 2048 /dev/sdb1
mke2fs 1.42.9 (28-Dec-2013)
文件系统标签=MYDATA
OS type: Linux
块大小=2048 (log=1)
分块大小=2048 (log=1)
Stride=0 blocks, Stripe width=0 blocks
655360 inodes, 5242880 blocks
1048576 blocks (20.00%) reserved for the super user
第一个数据块=0
Maximum filesystem blocks=542113792
320 block groups
16384 blocks per group, 16384 fragments per group
2048 inodes per group
Superblock backups stored on blocks: 
	16384, 49152, 81920, 114688, 147456, 409600, 442368, 802816, 1327104, 
	2048000, 3981312

Allocating group tables: 完成                            
正在写入inode表: 完成                            
Writing superblocks and filesystem accounting information: 完成   
```

接下来设置一下自动挂载：



## 文件系统的挂载

在linux中，一个磁盘如果想要进行数据的存取，它首先要将其分区（非必要）并格式化为linux系统所支持的文件系统类型，然后关联到根下面的一个目录，被关联到的目录，称之为挂载点，而后，计算机就可以通过该目录（挂载点）对磁盘进行访问，存取数据

#### 挂载命令mount

语法：mount [-nrw] [-t vfstype] [-o options]  device dir

不加任何参数时，显示所有已经挂载的文件系统及相关信息

* 常用选项

  -a 重读/etc/fstab配置文件，并挂载！

  -n 默认情况下，设备挂载的操作会同步到/etc/mtab文件中，此选项，将禁用此特性

  -r 只读挂载

  -w 读写挂载

  -L 使用lable方式指明设备

  -U 使用UUID的方式指明设备

  -t 指明要挂载的文件系统类型，一般可省略

  -o，指明挂载选项

  > sync/nosync    同步/异步
  >
  > atime/noatime    访问文件或目录的同时，是否更行访问时间戳
  >
  > diratime/nodiratime        访问目录时，是否同步更新其访问时间
  >
  > remount            重新挂载
  >
  > acl     激活acl(access control list)访问控制列表
  >
  > auto/noauto： 是否允许使用mount -a 自动挂载
  >
  > suid/nosuid：是否允许该文件系统使用suid，sgid特殊权限
  >
  > exec/noexec：是否允许此文件系统上拥有可执行的二进制程序文件
  >
  > user/nouser：是否允许该文件系统被普通用户挂载
  >
  > defaults：默认值：rw,suid,dev,exec,auto,nouser,async

示例，之前格式化为ext4的分区挂载至/mydata目录，要求挂载时静止程序自动运行，且不更新文件的访问时间戳，并设置开机自动挂载

```bash
[root@localhost ~]# mke2fs -L MYDATA -m 20 -b 2048 /dev/sdb1
mke2fs 1.42.9 (28-Dec-2013)
文件系统标签=MYDATA
OS type: Linux
块大小=2048 (log=1)
分块大小=2048 (log=1)
Stride=0 blocks, Stripe width=0 blocks
655360 inodes, 5242880 blocks
1048576 blocks (20.00%) reserved for the super user
第一个数据块=0
Maximum filesystem blocks=542113792
320 block groups
16384 blocks per group, 16384 fragments per group
2048 inodes per group
Superblock backups stored on blocks: 
	16384, 49152, 81920, 114688, 147456, 409600, 442368, 802816, 1327104, 
	2048000, 3981312

Allocating group tables: 完成                            
正在写入inode表: 完成                            
Writing superblocks and filesystem accounting information: 完成   
[root@localhost ~]# mount -t ext4 -o remount,noexec,noatime,nodiratime /dev/sdb1 /mydata
[root@localhost ~]# mount | grep /dev/sdb1
/dev/sdb1 on /mydata type ext4 (rw,noexec,noatime,nodiratime,seclabel)
```

```fstab
#
# /etc/fstab
# Created by anaconda on Thu Dec 27 05:29:44 2018
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/centos-root /                       xfs     defaults        0 0
UUID=3cd699bf-f7a2-4de3-81b6-f2f608043415 /boot                   xfs     defaults        0 0
/dev/mapper/centos-home /home                   xfs     defaults        0 0
/dev/mapper/centos-swap swap                    swap    defaults        0 0
/dev/sdb1       /mydata         ext4    noexec,noatime,nodiratime       0 0 
```

最后向/etc/fstab文件中添加最后一行保存退出即可实现开机自动挂载

## swap文件系统介绍

swap 交换分区文件系统，它能在内存不足的时候，把一部分硬盘空间虚拟成内存，将内存中，使用频率相对较少的数据，放到交换分区中，从而腾出内存空间！  在需要使用到交换分区中的数据时，再将其交换到内存中来！

* swap的创建：linux中，swap交换分区，必须使用独立的文件系统，swap文件系统的system id 必须为82

  格式化工具：mkswap [options] device

  常用选项：-L 指明卷标、-f 强制执行

  swap的启动和关闭方法：使用swapon/swapoff指定分区名即可！

示例，创建一个大小为1G的swap分区，并启用

```bash
[root@localhost ~]# fdisk /dev/sda
欢迎使用 fdisk (util-linux 2.23.2)。

更改将停留在内存中，直到您决定将更改写入磁盘。
使用写入命令前请三思。


命令(输入 m 获取帮助)：p

磁盘 /dev/sda：21.5 GB, 21474836480 字节，41943040 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节
磁盘标签类型：dos
磁盘标识符：0x000438dd

   设备 Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048     4196351     2097152   83  Linux
/dev/sda2         4196352    33572863    14688256   8e  Linux LVM
/dev/sda3        33572864    35670015     1048576   82  Linux swap / Solaris
/dev/sda4        35670016    41943039     3136512    5  Extended
/dev/sda5        35672064    37769215     1048576   83  Linux

命令(输入 m 获取帮助)：n
All primary partitions are in use
添加逻辑分区 6
起始 扇区 (37771264-41943039，默认为 37771264)：
将使用默认值 37771264
Last 扇区, +扇区 or +size{K,M,G} (37771264-41943039，默认为 41943039)：+1G 
分区 6 已设置为 Linux 类型，大小设为 1 GiB

命令(输入 m 获取帮助)：T
分区号 (1-6，默认 6)：6
Hex 代码(输入 L 列出所有代码)：82
已将分区“Linux”的类型更改为“Linux swap / Solaris”
命令(输入 m 获取帮助)：w
The partition table has been altered!

Calling ioctl() to re-read partition table.

WARNING: Re-reading the partition table failed with error 16: 设备或资源忙.
The kernel still uses the old table. The new table will be used at
the next reboot or after you run partprobe(8) or kpartx(8)
正在同步磁盘。

```

现在swap分区已经创建出来了，接下来我们将其格式化,别忘记使用w保存退出

```bash
[root@localhost ~]# partx -a /dev/sda
partx: /dev/sda: error adding partitions 1-5
[root@localhost ~]# partx -a /dev/sda
partx: /dev/sda: error adding partitions 1-6
[root@localhost ~]# mkswap /dev/sda6
正在设置交换空间版本 1，大小 = 1048572 KiB
无标签，UUID=101bde6f-c2ed-480f-a609-43f79cc875f1
[root@localhost ~]# swapon
NAME      TYPE      SIZE USED PRIO
/dev/dm-1 partition   4G   0B   -1
[root@localhost ~]# swapon /dev/sda6
[root@localhost ~]# swapon
NAME      TYPE       SIZE USED PRIO
/dev/dm-1 partition    4G   0B   -1
/dev/sda6 partition 1024M   0B   -2

```

如上所示，分区完毕后，别忘记执行两次partx -a 使内核识别新分区，然后格式化，最后启用

## GPT和MBR磁盘分区表！

### MBR（MS-DOS）分区表

MBR；master boot record 

它使用磁盘的第一个扇区（512字节）来存放主引导程序和记录分区表信息

主引导程序，占据该扇区的446个字节，系统启动的必备程序之一！ 往后会介绍到的，在这里有个概念就好；

分区表信息：占据该扇区的64个字节！ 这64个字节，分为四组，能够记录四组分区信息，这也是为什么使用mbr分区表分区的磁盘，最多只能拥有四个主分区的原因，并且，由于每组仅有16个字节记录分区信息，所以，单块磁盘的容量也被限制在2.2TB 左右！  

扩展分区：所谓扩展分区，只是指定 一个范围，它会在扩展分区指定的范围内找到另一个位置，用来存放分区信息，而后，我们就可以在这个扩展分区中，创建所谓的逻辑分区了！ 这个逻辑分区的分区信息，就保存在该扩展分区中的！ 并不是直接保存在磁盘的第一个扇区的分区表中！ 

**总结一下mbr磁盘分区的特性：**

**磁盘默认的分区表仅能记录四组分区信息**

**分区的最小单位是柱面**

**单个磁盘最大容量2.2TB**

**MBR仅有一个用来存放分区信息的扇区，如果损坏，很难恢复**

### GPT分区表

GPT：GUID partition table

gpt磁盘分区表，将整个磁盘以LBA（logical block address，逻辑区块地址）来规划，默认每个LBA的大小是512字节！ 前34个LBA用来存放主分区记录和其他相关信息，并且，在磁盘的最后，也有34个LBA用来作为前34个LBA的备份，以达到提高磁盘数据安全性的目的！

LAB从0开始编号！这里，我们暂时只要知道，0号LBA用来作为MBR的兼容区块，里边存放的内容和MBR中的第一个扇区内容基本差不多！1号LBA 记录了分区表本身的位置和最后32个LBA的位置，还有分区表的校验码等等！

从LBA2-LBA33 用来记录分区信息！，没个LBA记录四组，每组使用128bit几率起始LBA和结束LBA编号！ 所以，GPT磁盘分区表的单块磁盘容量达到了惊人的2^64*512字节！ 具体是多少，有兴趣的可以算一算！ 并且，分区的个数，也达到了之前的32倍！ 也就是128个！

LVM:逻辑卷管理器（logical volume manager）

逻辑卷管理器，能将底层不同的磁盘设备，或者说分区，组合成一个大磁盘，以方便用户的使用！它还提供弹性收缩的功能！ 可以自由调整逻辑卷的大小！ 



# LVM

### LVM的几个核心概念：PV，VG，LV，PE，及其管理工具

* PV：physical volume

  物理卷：也就是我们底层的磁盘设备，或者说磁盘分区

  管理工具

  > ​    pvs：显示pv简要信息
  >
  > ​    pvdisplay：显示pv详细信息
  >
  > ​    pvcreate /dev/DEVICE：创建pv

* VG：volume group

  卷组：所谓的卷组，就是LVM组合起来的大磁盘了！

  管理工具：

  >vgs：显示vg简要信息
  >
  >vgdisplay：显示vg详细信息
  >
  >vgcreate 【-s #[kKmMgGtTpPeE]】VolumeGroupName PhysicaDevicePath 【PhysicalDevicePath。。。】创建VG，-s指定创建的卷组大小！
  >
  >vgextend VGname PhysicalDevicePath 【PhysicalDevicePath。。。】扩展卷组
  >
  >​    vgreduce VGname PhysicalDevicePath 【PhysicalDevicePath。。。】缩减卷组，缩减先需要先使用pvmove命令，将要所见到呃卷组中的数据，移动到其他卷组

* LV：logical volume

  逻辑卷：大磁盘创建好后，我们就需要在大磁盘上进行分区了！ 这个可以对应到磁盘的分区上

  管理工具：

  > lvs:显示lv简要信息
  >
  > lvdisplay：显示lv详细信息
  >
  > lvcreate -L #[mMgGtT] -n NAME VGname：创建lv
  >
  > lvextend -L [+]#[mMgGtT] /dev/VGname/LVname    扩展lv
  >
  > lvreduce -L [-]#[ mMgGtT] /dev/VGname/LVname    缩减lv

  lv在/dev目录下的文件名为：

  ​        /dev/dm-#

  为了方便记忆，会有两个链接文件指向这个设备文件，他们分别是：

  ​    /dev/mapper/VGname-LVname

  ​    /dev/VGname/LVname

* PE：physical extent

  物理扩展快： 这个是什么呢？ 想想成为我们磁盘中的block大小就可以了！他是lvm大磁盘中的最小存储单位！

LVM 的实现过程，就是将底层的物理卷组合成为卷组，而后在卷组上划分出逻辑卷，而这个逻辑卷，就是我们可以格式化存储数据的分区了！

### lvm的简单创建过程

准备工作，我们先在不同的磁盘上划分出三个分区来！注意，lvm的物理卷，需要将其设置为8e（lvm）

```bash
[root@localhost ~]# fdisk /dev/sda
欢迎使用 fdisk (util-linux 2.23.2)。

更改将停留在内存中，直到您决定将更改写入磁盘。
使用写入命令前请三思。


命令(输入 m 获取帮助)：p

磁盘 /dev/sda：21.5 GB, 21474836480 字节，41943040 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节
磁盘标签类型：dos
磁盘标识符：0x000438dd

   设备 Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048     4196351     2097152   83  Linux
/dev/sda2         4196352    33572863    14688256   8e  Linux LVM
/dev/sda3        33572864    35670015     1048576   82  Linux swap / Solaris
/dev/sda4        35670016    41943039     3136512    5  Extended
/dev/sda5        35672064    37769215     1048576   83  Linux
/dev/sda6        37771264    39868415     1048576   82  Linux swap / Solaris
命令(输入 m 获取帮助)：t  
分区号 (1-6，默认 6)：6
Hex 代码(输入 L 列出所有代码)：8e    
已将分区“Linux swap / Solaris”的类型更改为“Linux LVM”
```

将sda上之前的一个swap分区拿来做实验

```bash
磁盘 /dev/sdb：53.7 GB, 53687091200 字节，104857600 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节
磁盘标签类型：dos
磁盘标识符：0x75064cc3

   设备 Boot      Start         End      Blocks   Id  System
/dev/sdb1            2048    20973567    10485760   8e  Linux LVM
/dev/sdb2        20973568    25167871     2097152   8e  Linux LVM
```

再在/dev/sdb上创建两个分区！ 并将格式改为linux lvm 现在三个底层物理卷就准备好了！

先创建三个pv

```bash
[root@localhost ~]# pvcreate /dev/sda6 /dev/sdb1 /dev/sdb2
  Physical volume "/dev/sda6" successfully created.
  Physical volume "/dev/sdb1" successfully created.
  Physical volume "/dev/sdb2" successfully created.
```

然后将/dev/sdb1,2 创建为一个卷组，指明卷组名为test

```bash
[root@localhost ~]# vgcreate test /dev/sdb1 /dev/sdb2
  Volume group "test" successfully created
```

在test卷组中创建名为testlv的lv, 大小1G

```bash
[root@localhost ~]# lvcreate -L 1G -n testlv test
  Logical volume "testlv" created.
```

将testlv格式化为ext4文件系统(使用lvdisplay可以查看其信息)

```bash
[root@localhost ~]# lvdisplay
  --- Logical volume ---
  LV Path                /dev/test/testlv
  LV Name                testlv
  VG Name                test
  LV UUID                V8sTiq-ZUgT-Grcl-0NKt-UbrT-Oh3V-mh1G2s
  LV Write Access        read/write
  LV Creation host, time localhost.localdomain, 2019-04-14 23:11:57 +0800
  LV Status              available
  # open                 0
  LV Size                1.00 GiB
  Current LE             256
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:3
[root@localhost ~]# mke2fs /dev/test/testlv
mke2fs 1.42.9 (28-Dec-2013)
文件系统标签=
OS type: Linux
块大小=4096 (log=2)
分块大小=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
65536 inodes, 262144 blocks
13107 blocks (5.00%) reserved for the super user
第一个数据块=0
Maximum filesystem blocks=268435456
8 block groups
32768 blocks per group, 32768 fragments per group
8192 inodes per group
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376

Allocating group tables: 完成                            
正在写入inode表: 完成                            
Writing superblocks and filesystem accounting information: 完成

```

至此，我们的逻辑卷就创建完成了，lv的详细信息我们有已经看过了，我们来看看pv，vg的详细信息

```bash
[root@localhost ~]# pvdisplay
  --- Physical volume ---
  PV Name               /dev/sdb1
  VG Name               test
  PV Size               10.00 GiB / not usable 4.00 MiB
  Allocatable           yes 
  PE Size               4.00 MiB
  Total PE              2559
  Free PE               2303
  Allocated PE          256
  PV UUID               9uUPyD-CL1K-wkVO-kW6U-TSdK-SXLv-KBP4Oq
   
  --- Physical volume ---
  PV Name               /dev/sdb2
  VG Name               test
  PV Size               2.00 GiB / not usable 4.00 MiB
  Allocatable           yes 
  PE Size               4.00 MiB
  Total PE              511
  Free PE               511
  Allocated PE          0
  PV UUID               ihp2aO-0VQG-e2m4-z8zZ-uQgf-C2j6-NTgHUy
[root@localhost ~]# vgdisplay
  --- Volume group ---
  VG Name               test
  System ID             
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  2
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               0
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               11.99 GiB
  PE Size               4.00 MiB
  Total PE              3070
  Alloc PE / Size       256 / 1.00 GiB
  Free  PE / Size       2814 / 10.99 GiB
  VG UUID               yEdjIV-Ev0Q-7ZAw-CF2X-pSkr-z4Sp-uYyjEQ

```

我们可以看到，我们的卷组大小为12G

我们先来实现以下lv的扩展！ 把之前的/dev/sda6加到卷组来，然后把lv大小扩展到12.5G(因为我的/dev/sda6只有1g大小)

```bash
[root@localhost ~]# vgextend test /dev/sda6
  Volume group "test" successfully extended
[root@localhost ~]# vgs test
  VG   #PV #LV #SN Attr   VSize   VFree  
  test   3   1   0 wz--n- <12.99g <11.99g
[root@localhost ~]# lvextend -L 12.5G /dev/test/testlv
  Size of logical volume test/testlv changed from 1.00 GiB (256 extents) to 12.50 GiB (3200 extents).
  Logical volume test/testlv successfully resized.
[root@localhost ~]# lvs /dev/test/testlv
  LV     VG   Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  testlv test -wi-a----- 12.50g                                                    
```

到这里，其实还没完，我们的lv虽然扩展了，但是我们的文件系统，，还没有！

```bash
[root@localhost ~]# mount /dev/test/testlv /mydata/
[root@localhost ~]# df -h /mydata/
文件系统                 容量  已用  可用 已用% 挂载点
/dev/mapper/test-testlv 1008M  1.3M  956M    1% /mydata
[root@localhost ~]# resize2fs /dev/test/testlv
resize2fs 1.42.9 (28-Dec-2013)
Filesystem at /dev/test/testlv is mounted on /mydata; on-line resizing required
old_desc_blocks = 1, new_desc_blocks = 1
The filesystem on /dev/test/testlv is now 3276800 blocks long.
[root@localhost ~]# df -h /mydata/
文件系统                 容量  已用  可用 已用% 挂载点
/dev/mapper/test-testlv   13G  2.5M   12G    1% /mydata

```

至此，一个文件系统的扩展才算完成！

接下来，我们再试试缩减！我们把/dev/sdb1,2都砍掉，把文件系统缩减至0.5G，缩减前，需要先卸载文件系统

扩展，我们是自下而上的，收缩，我们需要自上而下：

先文件系统，然后逻辑卷，在到卷组，之后，物理卷就可以去领盒饭了！

来，我们做一遍：

```bash
[root@localhost ~]# umount /mydata/
[root@localhost ~]# e2fsck -f /dev/test/testlv
e2fsck 1.42.9 (28-Dec-2013)
第一步: 检查inode,块,和大小
第二步: 检查目录结构
第3步: 检查目录连接性
Pass 4: Checking reference counts
第5步: 检查簇概要信息
[root@localhost ~]# resize2fs /dev/test/testlv 500m
resize2fs 1.42.9 (28-Dec-2013)
Resizing the filesystem on /dev/test/testlv to 128000 (4k) blocks.
The filesystem on /dev/test/testlv is now 128000 blocks long.
[root@localhost ~]# lvreduce -L 900m /dev/test/testlv
  WARNING: Reducing active logical volume to 900.00 MiB.
  THIS MAY DESTROY YOUR DATA (filesystem etc.)
Do you really want to reduce test/testlv? [y/n]: y
  Size of logical volume test/testlv changed from 12.50 GiB (3200 extents) to 900.00 MiB (225 extents).
  Logical volume test/testlv successfully resized.
[root@localhost ~]# pvmove /dev/sdb2
  /dev/sdb2: Moved: 0.00%
  /dev/sdb2: Moved: 57.33%
  /dev/sdb2: Moved: 100.00%
  [root@localhost ~]# vgreduce test /dev/sdb2
  Removed "/dev/sdb2" from volume group "test"
[root@localhost ~]# pvmove /dev/sdb1
  /dev/sdb1: Moved: 0.89%
  /dev/sdb1: Moved: 57.33%
  /dev/sdb1: Moved: 100.00%
[root@localhost ~]# vgreduce test /dev/sdb1
  Removed "/dev/sdb1" from volume group "test"
[root@localhost ~]# vgdisplay  test
  --- Volume group ---
  VG Name               test
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  16
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               0
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               1020.00 MiB
  PE Size               4.00 MiB
  Total PE              255
  Alloc PE / Size       225 / 900.00 MiB
  Free  PE / Size       30 / 120.00 MiB
  VG UUID               yEdjIV-Ev0Q-7ZAw-CF2X-pSkr-z4Sp-uYyjEQ
[root@localhost ~]# mount /dev/test/testlv /mydata/
[root@localhost ~]# df -h /mydata/
文件系统                 容量  已用  可用 已用% 挂载点
/dev/mapper/test-testlv  492M  780K  471M    1% /mydata

```

步骤是，卸载文件系统，强制检测文件系统，缩减文件系统为500M ，缩减卷组为900M，分别移除sdb2，和sdb1的数据，而后将其移出test卷组！ 最后可以看到，我们的卷组大小1020m， 文件系统，471m  卷组的数值出入有点大，大概是因为基于pe大小的来划分的，所以出现了些误差把！暂且不管他！

写到这里，lvm就讲的差不多了！想到再补充！

休息了！写博客着实挺累的！



最后附带几个脚本练习：

1：编写脚本计算/etc/paswwd文件中第10个用户和第20个用户id号之和

```shell
[root@localhost scripts]# cat sum.sh
#!/bin/bash
id1=$(head -n 10 /etc/passwd | tail -n 1 | cut -d: -f3)
id2=$(head -n 20 /etc/passwd | tail -n 1 | cut -d: -f3)
let sum=$id1+$id2
echo "$id1+$id2=$sum"
```

2： 编写脚本，通过命令行参数，传入一个用户名，判断id号是技术还是偶数

```shell
[root@localhost scripts]# cat test.sh 
#!/bin/bash
id=$(grep ^$1: /etc/passwd  | cut -d: -f3)
let i=id%2
if [[ $i -eq 1 ]];then
	echo id is odd number
	else 
	echo id is even number
fi
```
