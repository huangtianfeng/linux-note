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

![文件查找](F:\Users\tian\Pictures\linux图库\文件查找.png)

注意，根目录下是存放有其下各目录和inode号的对应关系的，所以从根目录便可直接找到etc目录的inode！

### 软链接和硬连接

有了上面的概念，软链接和硬连接就很好理解了，所谓的软链接其实就是在某个目录下创建一个文件名和inode的映射关系，将其关联到你想指向的文件的inode上面

而硬连接，是创建一个新的inode，它和你想指向的文件，指向同一个数据块

说起来又有点绕了，老规矩，以passwd文件为例，建它的软链接和硬连接，上图

![软链接和硬连接](F:\Users\tian\Pictures\linux图库\软链接和硬连接.png)

这个图的硬链接应该是指向inode，这里说明一下，懒得改了！

软链接文件的inode中，存放数据块指针的位置，存放的是一个路径，当源文件被删除时，软链接即失效，但是如果创建另外一个和该路劲同名的文件，那么这个软链接又将重新生效

硬连接，是直接指向数据块的inode号的，及时源文件被删除，还是可以通过硬连接访问！

说到这里，提一下我们的删除文件和移动文件，大家有没有发现，在平常生活中，我们删除文件和移动文件的速度是非常快的，其实它只是把inode清空了，源文件还是在硬盘上的，通过特定手段还是可以恢复的！ 而移动文件（在同一个文件系统中），本质上就是删除原有目录下的文件名和inode对应关系，再重新在另一个目录下创建一个新的对应关系！（也就是一个硬连接）

注意： 硬连接，是直接指向磁盘数据块的，所以，硬连接不能跨文件系统，还有，作用对象不能是目录（大概是为了防止互相硬连接倒置无限循环）

软链接是指向一个路径的，所以，它可以跨文件系统，也可以对目录进行链接



## 磁盘的相关操作

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

