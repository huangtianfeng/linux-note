[TOC]

# linux系统的启动流程

linux系统主要由两部分组成：

内核+根文件系统

* 内核

  内核是一个工作在计算机硬件之上通用软件程序，负责对硬件的管理，以及实现以下功能

1. 进程管理
> 包括进程的创建，调度，销毁等等一系列操作，其中最关键的为进程调度
2. 内存管理
> 将内存抽象为虚拟的线性地址形式，营造出每一个进程单独运行在系统上的虚拟视角
3. 网络协议栈
> 是当前系统上诸多进程使用的公共功能
4. 文件系统
> 文件系统在内核中实现，工作在内核空间，一个运行中的操作系统大体上可以分为两层：
> 内核空间（模式）和用户空间（模式）
> 用户空间用来运行各种各样的应用程序，表现为进程或线程。
>
> 内核空间主要运行内核代码，这些代码通常是特权级操作，代码通过系统调用向用户空间的进程输出，进程需要执行特权操作时，就发起系统调用请求，由内核代为操作，任何进程都有可能需要发起系统调用
5. 驱动程序
> 任何一种硬件，都需要驱动程序，一些常见的硬件驱动被编译进内核中，一些不常见的硬件，其驱动程序则需要另行安装，驱动程序一般有硬件厂家提供
6. 安全功能
> 例如各种提供加密解密的栈

站在硬盘的角度，内核一定是存放在硬盘上的某个分区上的，该分区被称之为启动分区，对于windows而言，也被称之为活动分区，对于linux而言，又被命名为boot分区（挂载在根文件系统的/boot目录下）

* 根文件系统：

遵循FHS，有特定目录结构的分区，才能够作为根分区或者说根文件系统来使用，仅有结构还不够，其上还需要有各种各样所需的基本文件，其中有一个最关键的程序文件为init程序，站在动态视角来看，用户空间的所有进程都是由该程序创建的！

**注意：内核是脱离根文件系统，可以理解为它是一个负责挂载和管理根文件系统的虚拟机**

内核在启动之前，它只是一个磁盘上的静态文件，所谓启动，一定是将其加载至内存中，运行为进程或者线程，那么在内核启动之前，就需要在计算机启动时，有一个工作于内核启动前的程序，负责将内核启动起来，该程序会在后面的启动流程中介绍到，内核启动后，将接管整个计算机的硬件控制权。

内核的组成部分：

1. 核心文件 

   > 存放位置：/boot/vmlinuz-VERSION-release （z表示该文件是经过压缩的，release是编译者自行加进去的）
   >
   > 注意：我们文件系统中的内核文件是经过发行商编译好的二进制文件，www.kernel.org中提供的内核是源码文件

2. 模块文件

   > 存放位置：/lib/modules/Version-release/目录下（和内核版本同名的目录名），如果安装有多个内核，这里可以有多个目录
   >
   > 该目录下有许多辅助性的文件，最主要的是kernel目录，其下又有许多子目录，包括文件系统，驱动，加密解密组件，与平台相关的特有代码，内存管理功能，网络功能等等相关模块

内核要装载根时，需要先装载根所在磁盘的磁盘驱动程序，但是驱动又在根目录下的/lib/modules目录下，这事就有点麻烦了，不过，问题总是有解决办法的，这里的解决办法就是，将驱动程序编译进内核，这个办法，面向个人时，的确很好，但是，如果是系统发行商，需要面对的是各种各样的用户，这就意味着，需要将市面上所有常见的硬盘驱动编译进内核，而每个特定用户又只会使用到一种驱动，这样内核就太臃肿了，有点得不偿失，怎么办呢？   答案是，借助于中间临时系统，内核启动后，去加载一个临时根文件系统，该文件系统下的/lib/modules目录下放的是当前计算机专用的磁盘驱动，这个文件系统只适用于当前设备，它是在安装操作系统时，扫描当前设备的磁盘型号，为当前设备量身定制的，这个文件我们称之为基于内存的磁盘设备：ramdisk，它把内存中的某一段空间当磁盘用，挂载根文件系统后，临时根文件系统将自动让位。

临时根文件系统不是必须的，当我们知道自己磁盘的型号，以及自行编译内核，将其驱动编译进内核，就可以不需要ramdisk

ramdisk

对centos5来讲，该文件在/boot/目录下，名为initrd-VERSION-release.img
   > 用来创建该文件的工具程序：mkinitrd

对centos6、7来讲，该文件在/boot目录下，名为initramfs-VERSION-release.img
   > 用来创建该文件的工具程序：dracut、mkinitrd

一个是基与ram的**磁盘**，一个是基于ram的**文件系统**

> 说到这两者的区别，就要提一个linux内核的特性
>
> linux内核的特性之一：使用缓冲（buffer）和缓存（cache）来加速对磁盘上的文件访问
>
> ramdisk把内存当磁盘用，导致ramdis还会被缓冲和缓存一次，所以换乘了ramfs

# 启动流程

1. POST：加电自检：

   > 用于实现该功能的代码，存放于主板上的一个芯片中，我们称之为ROM，最有代表性的是CMOS芯片，其中有一个基本输入输出系统：BIOS（basic input and output system）是只读的，没有外部手段干预，是无法修改的，x86的cpu在硬件设计上，一但通电，就会自动去主板上的某个存储位置的某个地方，读取并运行，所以cpu能够访问的地址空间有ROM+RAM

2. Boot Sequence：引导过程

   > 按次序（次序存放于BIOS中）查找各引导设备，第一个有引导程序的设备即为本次启动要用到的设备
   >
   > 引导程序就是bootloader：引导加载器，它是一个程序，安装在硬盘上的引导程序，注意bootloader无法驱动逻辑设备，比如说lvm，raid等设备，所以内核文件一定不能放在逻辑卷上，只能放在基本磁盘分区上
   >
   > 对windows来说bootloader叫ntloader
   >
   > 对linux而言，有很多种实现，如：
   >
   > LILO：LInux LOader： 各种安卓手机上，使用的就是该程序引导的
   >
   > GRUB：Grand Uniform BootLoader
   >
   > > GRUB 0.X：也叫Grub Legacy
   > >
   > > GRUB 1.X：完全重写，和前者相去胜远，叫Grub2
   >
   > 引导程序主要用于提供一个菜单界面，允许用户选择要启动的系统或者不同的内核版本，把用户选定的内核装载到RAM中的特定空间中，解压，展开，而后把整个系统的控制权移交给内核

   对于MBR（Master Boot Record）类的系统来讲：bootloader存放于前446字节中

   GRUB采用了一种精巧的设计方式，把程序分成了两端，第一阶段放在原来的bootloader所在处，1st stage，第二阶段，放在磁盘分区（partition）上，也就是/boot/grub/目录下，2st stage

   第一阶段的功能不再是加载内核，而是加载第二阶段的代码，第二阶段由于是存放在磁盘中，所以能提供更为复杂的功能和操作，甚至能直接提供一个交互式命令行接口给用户，第二阶段再负责去加载内核

   第一阶段和第二阶段中间，还有一个1.5阶段，叫filesystem driver，以后再解释

3. Kernel

   > grub加载完内核后，内核开始自身初始化
   >
   > 1. 探测可识别到的所有硬件设备
   > 2. 加载各种硬件驱动程序（有可能借助于ramdisk加载跟文件系统所在磁盘的驱动）
   > 3. 切换根，以只读方式挂载根文件系统（防止内核bug把文件系统干掉了）
   > 4. 运行用户空间的第一个应用程序：/sbin/init

   init程序的类型（从5-7迭代了三次）

   CentOS 5 及其之前的： SysV init

   > 配置文件：/etc/initab

   CentOS 6 Upstart:SysV init的改进版

   > 配置文件：/etc/initab（该文件基本被废弃）
   >
   > 用的更多的是/etc/init/*.conf文件

   CentOS 7：Systemd

   > 配置文件：/usr/lib/systemd/system/目录下和
   >
   > /etc/systemd/system/目录下

总结一下：内核级别的系统初始化流程：POST-->BootSequence（BIOS)-->BootLoader(MBR)-->Kernel(ramdisk)-->rootfs(readonly)-->/sbin/init

centos5的init程序是最经典的，centos6仿照centos5的init，centos7也兼容5和6的方式！

接下来以centos5的init程序说明：SysV init

# CentOS-5 的init程序

## 运行级别

操作系统启动后，有init程序负责系统的初始化工作，启动一系列特定应用程序，驱动程序等等，在一般模式中，我们需要启动的程序是很多的，例如，我们在windows中，有时候会出现，安装一个新的显卡驱动后，出现开机黑屏的现象，这个时候，就可以进入到安全模式（维护模式），在该模式下，初始化时，许多应用程序不会启动，或者只会启动最原始系统自带的驱动程序，在该模式下，把之前安装的有问题的显卡驱动卸载掉就可以了！ 这里所谓的安全模式，和我们平常开机所启动的操作系统，所处的就是不同的运行级别，所谓级别的不同，就是初始化启动的程序有所不同。

运行级别的设定，是为了系统的运行或维护等目的而设定的一种机制

对linux而言，运行级别一共有七个级别0-7，级别时可以切换的

0：关机，shutdown

1：单用户模式，single user， 单用户指的是root用户，无需认证，维护模式

2：多用户模式，multi user，不是一个完整的模式，会启动网络功能，但不会启动NFS；仍然是一种维护模式

3：多用户模式，multi user，这是一个完全功能模式，但是不会启动图形界面，只有文本界面

4：预留模式：目前无特别使用目的，但习惯同3级别对待

5：多用户模式，multi user，完全功能模式，但它是图形界面，会自动启动图形接口

6：重启模式：reboot

可以用作默认级别的是3和5，完成级别切换，直接使用init加上一个数字即可

级别查看，使用who -r命令，或者runlevel命令，示例：

```bash
[root@localhost /]# runlevel
N 3
```

第一位表示前一次（最近一次）的级别，后一位表示当前级别，N表示最近一次运行的级别

## 配置文件：/etc/initab

该文件决定了init在整个初始化过程中，要做哪些事！ （待补充）

# CentOS7的systemd

systemd：是一个系统启动和服务器守护进程管理器，负责在系统启动或运行时，激活系统资源，服务器进程和其他进程

新特性：

1. 系统引导时实现服务并行启动

   > 有依赖关系，也可以并行启动，

2. 按需激活进程，再次之前让进程处于半活动状态

3. 能做系统状态快照

4. 基于依赖关系定义的服务控制逻辑

   > 自动启华的服务依赖关系管理；当一个服务启动时，需要依赖于某服务，那么启动该服务的同时，如果依赖的服务未启动，会自动启动！

5. 采用socket式于D-Bus总线式激活机制

核心概念：unit

unit表示不同类型的systemd对象，通过配置文件进行标识和配置；文件中主要包含了系统服务，监听socket、保存的系统快照以及其他与init相关的信息

unit由其相关的配置文件进行标识，识别和配置，文件中主要包含了与系统服务相关的，与某个服务监听的socket相关的、或者与快照相关的以及其他与init相关的信息

这些配置文件主要保存在
**/usr/lib/systemd/system**：每个服务最主要的启动脚本设置，类似于之前的/etc/init.d/
/run/systemd/system：系统执行过程中所产生的服务脚本，比上面目录有限运行。
**/etc/systemd/system**： 管理员建立的执行脚本，类似于/etc/rcN.d/Sxx的功能，比上面目录优先级高

这些目录下的文件，每个文件称之为一个unit文件，以后缀名分类

常见的unit类型：

1. Service unit： 文件扩展名为.service ,用于定义系统服务，类似于/etc/init.d下的服务脚本，扮演了以前的服务脚本的角色；
2. Target unit：文件扩展名为.target，主要用于模拟实现“运行级别”；
3. Device unit：文件扩展名为.device，用于定义内核识别的设备；
4. Mount unit：.mount，用于定义文件系统挂载点；
5. Socket unit：.socket, 标识进程间通信用到的socket文件
6. Snapshot unit：.snapshot，用于管理系统快照；
7. Swap unit：.swap，用于标识swap设备
8. Automount unit：.automount,用于文件系统自动挂载
9. Path unit：.path，用于定义文件系统中的一文件或目录

关键特性：

1. 基于socket的激活机制：socket与程序是分离的；
2. 基于bus的激活机制；
3. 基于device的激活机制；
4. 基于Path的激活机制；
5. 系统快照：能够保存各unit的当前状态信息于持久存储设备中；
6. 向后兼容SysV init脚本；/etc/init.d/

注意：

systemctl 的命令是固定不变的

非由systemd启动的服务，systemctl无法与之通信，无法控制该服务

## 管理系统服务

CentOS 7 通过service类型的unit文件来管理系统服务

systemctl命令：

语法：systemctl [option] COMMAND [NAME...]

#### service unit的管理（与CentOS 6的对比）

| 功能     | CentOS-6           | CentOS-7                     |功能描述|
| :------: | ------------------ | ---------------------------- | -------- |
| 启动     | service NAME start | systemctl start NAME.service ||
| 停止     | service NAME stop | systemctl stop NAME.service ||
| 重启     | service NAME restart | systemctl restart NAME.service |                                                      |
| 查看状态 | service NAME status | systemctl stop NAME.status                   |                                                      |
| 条件式重启 | service NAME condrestart | systemctl try-restart NAME.service |如果此前启动了，重启，如果没启动，那就算了|
| 重载或重启服务 |  | systemctl reload-or-restart NAME.service |重载：重新读取配置文件，支持重载就重载，不支持就重启|
| 重载或条件式重启服务 |  | systemctl reload-or-try-restart NAME.service ||
| 设置服务开机自启 | chkconfig NAME on | systemctl enable NAME.service ||
| 禁止服务开机自启 | chkconfig NAME off | systemctl disable NAME.service ||
| 查看某服务是否能开机自启 | chkconfig --list NAME | systemctl is-enable NAME.service ||
| 禁止手动和自动启动，将脚本文件遮挡 |  | systemctl mask NAME.service ||
| 取消某服务设定的禁止启动 |  | systemctl unmask NAME.service ||
| 查看某服务当前激活与否的状态 |  | systemctl is-active NAME.service ||
| 查看所有已激活的服务 |  | systemctl list-units --t service |加上--all或-a 可以显示所有的包括未激活的service|
| 查看服务的依赖关系 |  | systemctl list-dependencies NAME.service ||
| 杀死进程 | | systemctl kill unitname ||

#### 服务状态：

systemctl list-unit-files --type service --all 显示状态

> loaded:	 Unit配置文件已处理，已经加载到内存中
>
> active（running）: 一次或多次持续处理的运行
>
> active（exited）:成功完成一次性的配置
>
> active（waiting）：运行中，等待一个事件
>
> inactive：不运行
>
> enabled：开机启动
>
> disabled：开机不启动
>
> static：开机不启动，但可以被另一个服务激活

#### target units管理

运行级别：

0==>runlevel0.target/poweroff.target    关机

1==>runlevel1.target/rescue.target    救援模式

2==>runlevel2.target/multi-user.target

3==>runlevel3.target/multi-user.target

4==>runlevel4.target/multi-user.target

5==>runlevel5.target/graphical.target

6==>runlevel6.target/reboot.target

级别切换：init N==>systemctl isolate NAME.target

查看级别：runlevel == systemctl list-units --t target

查看所有级别：systemctl list-units -t target -a

获取默认运行级别：systemctl get-default

修改默认运行级别：修改/etc/inittab/文件==》systemctl set-default NAME.target（0和6不应该当成默认）

切换至救援模式（级别1）：systemctl rescue

切换至emergency模式：systemctl emergency

其他常用命令：

> 关机：systemctl halt、systemctl poweroff
>
> 重启：systemctl reboot
>
> 挂起：systemctl suspend
>
> 创建快照：systemctl hlbernate
>
> 快照并挂起：systemctl hibernate-sleep

## service units文件格式

设置定义启动就是在/etc/systemd/system目录下创建对应的文件链接至/usr/lib/systemd/system目录下对应的文件

service units文件通常有三部分组成，Unit、Service、Install

[Unit]：定义与unit类型无关的通用选项；用于提供unit的描述信息，unit行为及依赖关系

> Description：描述信息，意义性描述，使用systemctl status时可以看到
>
> After：定义unit的启动次序，表示当前unit应该晚于哪些unit启动，其功能与Before相反
>
> Wants: 依赖到的其他unit（弱依赖，被依赖的units无法激活时，不影响当前unit的激活）
>
> Requles：依赖到的其他units（强依赖，表示被依赖的units无法激活时，当前unit一定不能激活）
>
> Conflicts：定义units间的冲突关系

[Service]（跟类型相关的，也可以是Target）：与特定类型相关的专用选项：此处为service类型

> Type：用于定义影响ExecStart及相关参数功能的unit进程启动类型
>
> > simple：由execstart指明的程序就是服务的主进程，默认type就是simple
> >
> > forking：表示由execstart启动的进程，生成的其中一个子进程成为主进程
> >
> > notify：类似于simple，
> >
> > 了解两个就可以了，用的最多的还是simple
>
> EnvironmentFile：环境配置文件，为Exec指令提供一些变量
>
> **ExecStart**：指明启动unit要运行的命令或脚本；
>
> **ExecStop**：指明停止unit要运行的命令或脚本
>
> **Restart**：意外终止时自动重启要运行的命令或脚本

[Install]：定义由"systemctl enable"以及"systemctl disable"命令在实现服务启用或禁用时用到的一些选项

>  Alias，别名
>
> RequiredBy：被哪些units所依赖（强依赖）
>
> WantedBy：被哪些units所依赖（弱依赖）

注意：对于新创建的unit文件或修改了的unit文件，要通知systemd重载此配置文件

\#systemctl daemon-reload

