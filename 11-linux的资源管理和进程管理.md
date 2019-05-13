[TOC]

# 进程管理

## 相关概念

**进程（process）：** 是一个执行中的程序文件的一个的副本，意思就是，同一个程序文件，可以发起为多个进程！

**调用（call）：**所谓调用，就是调用别人事先编写好的函数或者功能模块

> 系统调用发生在内核空间中，库调用发生在用户空间，库调用可能是系统调用的二次封装，也可能是独立的功能模块

cpu上的指令分为两类，普通指令（环3上的指令）和特权指令（环0上的指令），特权指令主要完成硬件的管理工作，决不允许程序自己随意调用，除了内核。 如果某个程序需要用到特权指令，就需要向内核发起系统调用

> 每一个程序的执行流程，都是自上而下的顺序选择执行的，将指令放到CPU上执行，如果应用程序需要完成特权操作时，就直接调用内核提供的系统调用，接下来程序本身就停下来了，等待内核将你调用的功能放到CPU上执行，调用返回后，程序再继续运行，所以整个代码执行过程中，会在执行用户代码和执行内核代码这两者之间不停转换（称之为模式切换），而进程终止的时候，称之为**软中断**，运行用户代码的时候，表示整个操作系统运行在用户模式（用户空间）下，一旦发起系统调用，整个操作系统就运行在内核模式下（内核空间）

linux是一个多任务的操作系统，当CPU只有一个的时候，进程又有多个的时候，内核就负责对CPU的资源进行时间分片，以及对进程实现调度，根据进程的优先级，将进程轮流调度到CPU上执行，内核还要负责将每个进程保存于CPU寄存器中的中间信息（当前状态）保存下来，这个过程叫**保存现场**，而对应的调度之前的进程到CPU中执行时，就需要**恢复现场**，这个切换过程，每一次切换都需要执行一段内核代码

每个进程的中间信息（元数据），结构都差不多，所以内核为进程创建一个结构体，这个结构体叫task struct，每个进程的中间信息都存储于一个结构体中，而存储位置不连续时，为了便于寻找，基于链表存储，多个任务的task struct组成的链表，称之为task list。

进程内存：进程是不可能直接访问硬件的，所以进程的内存空间都是虚拟内存空间，所谓虚拟内存空间，是内核会为每一个进程虚拟出一个理想的运行环境，告诉它，在当前系统上，除了内核，只有一个应用程序在运行，所以所有内存空间都为你所用。

内核将内存划分为一个个小的单元，一般来说，这个单元的固定大小为4K，而称这个单元为Page Frame：页框，用于存储页面数据，所谓页面数据，就是能够存在页框中的数据，就是页面数据，存储Page。而进程不断的创建和销毁，会造成内存中出现大量分散的Page Frame，所以，内核还负责将其组织成为线性的连续的内存空间提供给新的进程，这就是所谓的线性内存空间！

IPC机制：Inter Process Communication，进程间通信机制

> 在同一主机上的进程间通信的方法，shm：shared memory，共享内存、signal、semerphor等
>
> 跨主机间的进程通信方法：
>
> 1. rpc：remote procecure call，远程过程调用
> 2. socket：套接字

任何进程想摆脱内核的控制，就意味着要去运行特权指令了，而运行特权指令，会触发软中断，cpu会立即唤醒内核，内核将接管一切！（没有漏洞的情况）

进程优先级：分为140个级别：
>1-99：实时优先级：很少手动处理
>100-139：静态优先级：数字越小，优先级越高
>Nice值：-20-+19，分别对应于100-139这个范围
>
>内核为了加快调度速度，将进程队列分为140个，每个优先级，放在一个队列中，所以，每次调度，只需要扫描着140个队列中的第一个进程，每个队列都有两个，一个运行队列，一个过期队列，运行过的进程就被放置在过期队列中，当运行队列运行完毕后，过期队列切换为运行队列，运行队列切换为过期队列，然后重复执行操作

**LINUX进程类型：**

1. 守护进程：daemon，跟终端无关，在系统引导过程中启动的进程
2. 用户进程（前台进程）：用户通过终端启动的进程，跟终端相关

注意：也可以把前台启动的进程送往后台，以守护模式运行

**进程状态**：

进程载入内存后，给其分配CPU时间，就能够运行，或者说，内核将其调度到CPU上，就能运行，所以，进程还分为几个不同的状态

1. 运行态：running，数据指令都在内存中，也已经分配CPU时间正在运行

2. 就绪态：ready，数据指令都在内存中，但没有费培CPU时间，或者说没有被调度

3. 睡眠态

   > 可中断睡眠：interruptable，所谓可中断睡眠，就是cpu时间用完之后，处于等待调度状态的进程，它随时可以被内核调度到CPU，随时可以被叫醒，所以称之为可中断睡眠，
   >
   > 不可中断睡眠：uninterruptable：不可中断睡眠，是一个进程需要读取数据，而数据又不再内存中，而是在磁盘上，需要发起系统调用，请求内核将数据载入到内存中，这个载入过程，我们称之为一次IO，相对于CPU的运行速度来说，IO是很慢的，所以，内核可以先调度其他进程至CPU，在等待IO的过程中，进程被唤醒是没用的，因为数据还没载入完成，所以，我们将这种状态的进程，定义为不可中断睡眠

4. 停止态（stopped）：暂存于内存中，但是不会被调度执行，除非手动启动之

5. 僵死态：（zombie）：进程是被父进程创建的，子进程任务完成后，就需要关闭所有打开的文件，清理相关数据等，然后等待父进程将其内存空间回收，这个等待过程，就是僵死态

   > 而僵尸进程，就是父进程挂掉之前，没有处理其子进程，所产生的孤儿进程，这些进程没有父进程，在挂掉后，没有父进程负责回收内存，所以就成为了僵尸进程，常驻内存中！ 一般来说，一个有经验的程序员不会让这种情况出现，会在父进程销毁之前，将子进程处理掉，或者找一个新的父进程作为其父进程，例如init进程！

根据进程占用CPU多还是占用的IO多，还可以分为CPU密集形的进程（CPU-Bound）和IO密集形的进程（IO-Bound）

一般交互式的进程就是IO密集型的，非交互式的进程就是CPU密集型的

**进程创建**：init负责接受一切用户空间的进程的请求，然后统一向内核发起系统调用，它也是系统启动后，第一个启动的程序

对linux系统来讲，进程存在所谓的父子关系，父子关系保存于task struct中

进程都由其父进程创建，父进程通过fork()系统调用，fork自身，当子进程需要修改进程数据时，通过写实复制（CoW）机制，创建出一段新的内存空间，该内存空间即为子进程

**线程**

轻量级的进程，线程是进程的子单位，一个进程中可以生成多个线程，线程可以在不同CPU核心上并行运行， 但是内核在调度进程和线程时，是一视同仁的！

# 进程管理命令

内核通过/proc/ 目录，将内核的状态信息输出给用户，其中包含两种参数
内核参数：可设置其值从而调整内核运行特性的参数：可设置的位于/proc/sys/目录下（拥有写权限的才能修改）
状态变量：其用于输出内核中统计信息或状态信息，仅用于查看

为了统一linux一切接文件的哲学思想，各参数都被模拟成文件系统乐行，参数本身被模拟成文件，位置被模拟成路径，或者说目录！

每个进程在/proc目录下，都有一个以pid同名的目录，该目录中存放了该进程的各种相关状态信息，该目录下的信息很难读懂，所以就有了如下很多进程查看工具，帮我们在该目录下抽取出相关信息，予以显示

```
[root@localhost ~]# ls /proc
1      14  26   296  46   478    63     732    74666  778  818        cmdline      fs          kpageflags  pagetypeinfo  sysrq-trigger
10     16  277  3    469  479    686    733    748    8    826        consoles     interrupts  loadavg     partitions    sysvipc
11     18  278  33   47   48     7      734    749    801  856        cpuinfo      iomem       locks       sched_debug   timer_list
12     19  279  34   470  480    727    741    750    803  9          crypto       ioports     mdstat      schedstat     timer_stats
12224  2   280  35   471  5      72712  742    751    806  914        devices      irq         meminfo     scsi          tty
12225  20  281  36   472  50     72719  74348  752    808  96         diskstats    kallsyms    misc        self          uptime
12226  21  284  44   473  556    72721  74605  753    809  acpi       dma          kcore       modules     slabinfo      version
12437  22  286  445  474  57092  728    74632  754    811  asound     driver       keys        mounts      softirqs      vmallocinfo
12443  23  293  446  475  574    729    74638  755    812  buddyinfo  execdomains  key-users   mpt         stat          vmstat
12680  24  294  456  476  589    730    74639  756    813  bus        fb           kmsg        mtrr        swaps         zoneinfo
13     25  295  457  477  614    731    74642  757    814  cgroups    filesystems  kpagecount  net         sys
```

以上是笔者liux系统中的/proc目录下的内容，可以看到，有许多纯数字命名的目录，每一个目录，代表一个进程，该目录下，有该进程的一系列状态信息，我们看一眼

```
[root@localhost ~]# ls /proc/74642/
ls: 无法读取符号链接/proc/74642/exe: 没有那个文件或目录
attr        cmdline          environ  io         mem         ns             pagemap      sched      stack    task
autogroup   comm             exe      limits     mountinfo   numa_maps      patch_state  schedstat  stat     timers
auxv        coredump_filter  fd       loginuid   mounts      oom_adj        personality  sessionid  statm    uid_map
cgroup      cpuset           fdinfo   map_files  mountstats  oom_score      projid_map   setgroups  status   wchan
clear_refs  cwd              gid_map  maps       net         oom_score_adj  root         smaps      syscall
```

该目录下的信息，想要完全读懂，还是有一定难度的，我就不一一解释了，只要知道，每个进程的所有状态信息，内核都以这种文件格式输出信息，供用户查看。

而在/proc/sys目录下，有许多可调整的内核参数，常见的有

> /proc/sys/net/core/rmem_max： 指定TCP连接的最大接受缓冲（窗口），单位字节。
>
> /proc/sys/net/core/wmem_max：指定TCP练级的的最大发送缓冲（窗口），单位字节。
>
> /proc/sys/net/core/rmem_default：默认的接受窗口大小。
>
> /proc/sys/net/core/wmen_default：默认的发送窗口大小。
>
> /proc/sys/net/ipv4/ip_local_port_range：用于设定向外连接的端口范围。
>
> /proc/sys/net/ipv4/tcp_max_syn_backlog：用于设定SYN报文的队列长度
>
> 
>
> 

#### pstree命令：以树状结构显示运行中的进程

基本没什么复杂用法

#### ps命令

  查看执行ps命令这一刻的所有进程状态

  语法：ps [options]

  选项有三种风格

  >unix风格： 选项前必须带“-”
  >BSD风格：选项前有些选项必须不带“-” 有些选项前面又必须带“-” 否则就报错
  >GNU long风格：两个“--” 引导
  >
  >总之，下面介绍的选项，严格区分是否有-符号就好了，加-和不加-，表示的意思可能天差地别！

  启动进程的两种方式：

  1. 系统启动过程中自动启动，与终端无关的进程
  2. 用户通过终端启动，与终端相关的进程

  当我们通过终端启动进程时，终端提供一个shell壳程序，在当前shell下启动的所有进程，都是该shell的子进程，那么，一旦连接断开，或者终端关闭，我们启动的进程都将被动自动终止，所以，有时候，我们需要一个进程长时间运行在系统中是，就需要使用其他方法，将进程与shell的关系剥离

  常用选项：

  a：显示所有与终端相关的进程

  x：显示所有与终端无关的进程

  u：以用户为中心来组织进程的状态信息显示

  示例: 

  ```bash
  [root@localhost ~]# ps -ax
     PID TTY      STAT   TIME COMMAND
       1 ?        Ss     0:04 /usr/lib/systemd/systemd --switched-root --system --deserialize 22
  ```

  其中。STAT表示进程状态，TIME表示累计占用CPU的时间，COMMAND表示由哪个命令启动，带中括号的是内核线程

  ```bash
  [root@localhost ~]# ps aux
  USER        PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
  root          1  0.0  0.2 191024  3940 ?        Ss   5月05   0:04 /usr/lib/systemd/systemd --switched-root --system --deserialize 22
  root          2  0.0  0.0      0     0 ?        S    5月05   0:00 [kthreadd]
  ```

  VSZ：表示占用的虚拟内存集，占用的虚拟内存的大小

  > 虚拟内存和物理内存存在一种映射关系，我们知道，内核会为每个进程虚拟出一个理想的运行空间，当前系统只有进程和内核分配内存空间，所以，每个进程都有一个理想的运行环境，我们就称之为虚拟内存，虚拟内存所占用的大小和物理内存所占用的大小是有区别的，例如，我们常说的共享库，这个库文件载入到物理内存中，只需要载入一份，放到一段内存空间中，然后每个需要用到的进程，在其虚拟内存中，都将该内存空间，映射一份为自己的，这就出现了，一个物理空间，对应多个虚拟空间！导致虚拟内存总值是要大于实际物理内存的！  ，这样说并不精确，差不多理解就行了！

  RSS：Resident Size 常驻内存集

  > 常驻内存集，就是坚决不能放到交换内存当中去的数据！

  STAT：状态

  > 基于BSD风格所表述的状态
  >
  > R:runing 运行态
  >
  > S：可中断睡眠态
  >
  > D：不可中断睡眠
  >
  > T：stopped 停止态
  >
  > Z：zonbie：僵死态
  >
  > +：前台进程
  >
  > l：表示是一个多线程进程
  >
  > N：表示低优先级进程
  >
  > <：表示是一个高优先级进程
  >
  > s：session leader：会话领导者，

-e：显示所有进程，包括与终端相关和无关的

-f：显示完整格式的进程信息2

例如：

```bash
[root@localhost ~]# ps -ef
UID         PID   PPID  C STIME TTY          TIME CMD
root          1      0  0 5月05 ?       00:00:04 /usr/lib/systemd/systemd --switched-root --system --deserialize 22
root          2      0  0 5月05 ?       00:00:00 [kthreadd]
```

0号进程是存在的，但是在启动完systemd后，该进程就被终止了

其中C 表示占用的CPU百分比

STIME表示该进程的启动时间

TIME表示累计运行时间

CMD表示启动此进程的命令

-F：显示更加完整的状态信息

```bash
[root@localhost ~]# ps -F
UID         PID   PPID  C    SZ   RSS PSR STIME TTY          TIME CMD
root      12821  12819  0 28893  2148   0 00:09 pts/0    00:00:00 -bash
root      13490  12821  0 38840  1868   0 22:07 pts/0    00:00:00 ps -F
```

PSR表示运行在哪个CPU上

-H：以层级结构显示进程的相关信息

```bash
[root@localhost ~]# ps -eFH
UID         PID   PPID  C    SZ   RSS PSR STIME TTY          TIME CMD
root          1      0  0 47756  3940   1 5月05 ?       00:00:04 /usr/lib/systemd/systemd --switched-root --system --deserialize 22
root        554      1  0  9769  4564   0 5月05 ?       00:00:01   /usr/lib/systemd/systemd-journald
root        584      1  0 11059  1832   1 5月05 ?       00:00:01   /usr/lib/systemd/systemd-udevd
root        777      1  0 13877   896   1 5月05 ?       00:00:00   /sbin/auditd
polkitd     800      1  0 134641 12264  0 5月05 ?       00:00:00   /usr/lib/polkit-1/polkitd --no-debug

```

o：自定义要显示的字段列表，以逗号分隔

示例：ps -axo pid,comand

常用组合

ps -eo field1，field2

ps axo field1，field2

常见的field：

pid，ni（nice值），pri（priority优先级），rtprio（real time priority，实时优先级），psr，pcpu，stat，comm，tty，ppid



#### pgrep和pkill命令：查询进程或向进程发送信号

  语法：pgrep [options] pattern

  常用选项：

  -u uid：显示指定用户的进程

  -U uid：显示以指定用户启动的进程（进程运行时，用户是可以切换的）

  -t terminal：查看于指定的终端相关的进程

  -l ： 显示进程名

  -a：显示完整格式的进程名（显示包括启动时的参数列表）

  -P pid：显示指定进程的所有子进程（pid只能使用id号）

#### pidof命令：根据进程名，获取其pid号
#### top命令：动态查看当前系统的进程信息

  ```bash
  top - 21:48:25 up 2 min,  2 users,  load average: 0.58, 0.39, 0.16
  Tasks: 123 total,   2 running, 121 sleeping,   0 stopped,   0 zombie
  %Cpu(s):  0.2 us,  0.2 sy,  0.0 ni, 99.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
  KiB Mem :  1865308 total,  1495924 free,   106816 used,   262568 buff/cache
  KiB Swap:  4194300 total,  4194300 free,        0 used.  1574904 avail Mem 
     PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND                                                                        
   12718 root      20   0  162012   2252   1564 R   0.3  0.1   0:00.19 top                                                                            
       1 root      20   0  125472   3828   2560 S   0.0  0.2   0:02.23 systemd                                                                        
       2 root      20   0       0      0      0 S   0.0  0.0   0:00.01 kthreadd                                                                       
       3 root      20   0       0      0      0 S   0.0  0.0   0:00.07 ksoftirqd/0                                                                 
  ```

  其中，第一行显示了，当前系统时间，已运行时间，登录用户数，和平均负载，过去一分钟，五分钟和15分钟的平均负载，uptime命令也能显示该行信息

  第二行，已启动的进程总数及处于各状态的进程数量

  第三行，cpu占用百分之比

  > us：表示用户空间的进程所占用的百分比
  >
  > sy：表示内核空间的内核占用的CPU百分比
  >
  > ni：表示nice值调整所额外占用到的cpu百分比
  >
  > id：表示空闲百分比
  >
  > wa：表示等待io完成所消耗的时间百分比
  >
  > hi：表示处理硬件中断所消耗的CPU百分比
  >
  > si：表示处理软中断所消耗的CPU百分比
  >
  > st：表示被虚拟机或者虚拟化程序偷走的CPU百分比

  第四行和第五行：以kb为单位显示物理内存的占用情况，buff/cache表示用户缓冲和缓存的内存空降，他们是可以被回收回来使用的，所以free空间，应该是free时间+buff/cache所占用的内存空间，avail mem表示可用的内存空间！

  最后各个字段的意思分别是：进程号，所属用户，优先级，nice值，虚拟内存集，常驻内存集，共享内存空间，当前状态，CPU占用百分比，内存占用百分比，运行（占用CPU）时长，启动命令

  交互式命令：

  > M:更具占据内存的百分比排序，P：根据占据CPU百分比排序，T：累计占用CPU时间进行排序
  >
  > l：关闭或开启uptime（第一行）信息
  >
  > t：关闭或开启tasks及cpu信息
  >
  > m：关闭或开启内存信息
  >
  > q：退出
  >
  > s：键入后，调整属性频率
  >
  > k：终止指定的进程，输入后，再输入pid

  常用选项：

  > -d # ：指定刷新时间间隔，默认为3秒
  >
  > -b：以批次方式显示
  >
  > -n #：指定显示多少批次后自动退出

#### 小结

衡量CPU工作状态的参数：

> us：用户空间占用CPU时间的百分比
>
> sy：内核空间占用CPU时间的百分比
>
> ni：nice值调整额外占用的CPU时间百分比
>
> id：CPU空闲时间占用百分比
>
> hi：处理硬件中断消耗的CPU占用百分比
>
> si：处理软中断消耗的CPU占用的百分比
>
> cs：上下文切换所占用的百分比
>
> st：被虚拟化程序分配走的CPU百分比

衡量进程占用内存的参数

> VSZ：虚拟内存集
>
> RSS：常驻内存集
>
> SHM：共享内存集

#### htop命令：

安装：需要从epel源安装，运行后，其显示界面如下

![进程管理插图1](F:\Users\tian\Pictures\linux图库\进程管理插图1.png)

其中，上方分别表示1号，二号CPU的使用情况，thr表示线程数量，其他的和top命令大同小异，只是界面更加美观，可以使用F1查看帮助：

常用交互式命令：

> u：选择查看指定用户的进程，键入后将列出用户列表供选择
>
> H：关闭或显示用户线程数
>
> K：隐藏或显示内核线程数，默认是不显示的
>
> P,M,T：根据CPU，内存或者CPU累积时间进程排序
>
> F6 > : 使用指定字段进程排序
>
> c：标记处一个进程和其所有子进程
>
> a：设置CPU的affinity：默认情况，一个进程可以运行到任何CPU核心上，该命令能将选定进程绑定到指定CPU核心上
>
> l：显示选定进程所打开的文件，注意，它依赖于lsof工具，需要实现安装！
>
> s：跟踪选定进程所发起过的系统调用
>
> t：以层级关系显示各进程状态

常用选项：

> -d #：指定刷新时间间隔
>
> -u UserName：仅显示指定用户的进程
>
> -s CLOUME：以指定字段进行排序

#### vmstat命令：报告虚拟内存统计数据

语法：vmstat [option] [delay[count]]

delay:指定间隔时间，不指定delay时，静态显示一次，不刷新

count：指定刷新次数，不指定count时，将以delay的间隔不停的刷新下去

```bash
[root@localhost ~]# vmstat
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 1  0      0 1280488   4184 425740    0    0     3     1   26   56  0  0 100  0  0
```

各个字段代表的意思：

> * procs
>
>   r：处于等待运行的进程个数，CPU上等待运行的任务队列长度
>
>   b：处于不可中断睡眠态的进程个数，被阻塞的任务队列的长度
>
> * memory
>   swpd：交换内存的使用总量，0表示每启用
>
>   free：空闲的物理内存总量
>
>   buffer：用于buffer的内存总量（缓冲）
>
>   cache：用于cache的内存总量（缓存）
>
> * swap
>   si：swap in，进入swpa中的数据速率，单位是kb/s，站在swap的角度，叫换进
>
>   so：swap out ，数据离开swap的速率，单位是kb/s，叫换出
>
> * io
>
>   bi：block in，从块设备读入数据到内存的速率，单位kb/s
>
>   bo：block out，从内存保存数据至块设备的速率，单位kb/s
>
> * system
>
>   in：interrupts，中断产生的速率，每秒钟多少个
>
>   cs：context switch，上下文切换速率
>
> * CPU
>
>   us：用户空间程序占用CPU的百分比
>
>   sy：讷河空间程序占用CPU的百分比
>
>   id：空闲CPU的百分比
>
>   wa：等待io完成消耗的CPU百分比
>
>   st：被虚拟化程序分配走的CPU百分比

常用选项：

> -s：显示内存统计数据
>
> ```
> [root@localhost ~]# vmstat -s
>       1865308 K total memory
>        154848 K used memory
>        227728 K active memory
>        157764 K inactive memory
>       1280504 K free memory
>          4184 K buffer memory
>        425772 K swap cache
>       4194300 K total swap
>             0 K used swap
>       4194300 K free swap
>         14129 non-nice user cpu ticks
>            11 nice user cpu ticks
>         11185 system cpu ticks
>      11200105 idle cpu ticks
>          2496 IO-wait cpu ticks
>             0 IRQ cpu ticks
>           241 softirq cpu ticks
>             0 stolen cpu ticks
>        368704 pages paged in
>        159983 pages paged out
>             0 pages swapped in
>             0 pages swapped out
>       2922161 interrupts
>       6194759 CPU context switches
>    1557236737 boot time
>         13383 forks
> ```
>
> 

#### pmap命令：显示报告进程的内存映射表

pmap跟上pid即可！

pmap [option] pid

常用选项：

> -x：显示更为详细的信息

另一种查看方式：cat  /proc/PID/maps

#### glances命令：跨平台的监控工具

需要通过epel源安装：

常用选项：

> -b：以byte为单位显示网卡数据速率
>
> -d：关闭磁盘I/O模块
>
> -m：关闭mount模块
>
> -n：关闭network模块
>
> -t #：指明刷新时间间隔
>
> -1：每颗CPU的统计数据单独显示
>
> -o {HTML|CSV}： 指定导出的格式
>
> -f /path/to/somedir/： 指明保存目录（不是文件名，是目录名）

C/S模式下运行glances命令：

服务模式：使用glances -s -B IPADDR：指明本机所监听的地址

客户端模式：使用glances -c IPADDR：指明要连接的远程服务器ip地址

#### dstat命令：

常用选项

> -c：显示CPU状态信息
>
> -C #,#...total：指明看哪个CPU核心
>
> -d：查看磁盘相关信息
>
> -D sda,sdb,...,total：指明某一块特定磁盘
>
> -g：显示内存也的换进换出
>
> -i：显示中断相关统计信息
>
> -l：显示平均负载
>
> -m：显示内存相关统计数据
>
> -n：显示网络接口相关统计数据
>
> -N：指明网卡
>
> -P：统计进程数据
>
> -r：显示io请求统计数据
>
> -s：统计swap数据
>
> -t：输出当前时间

> --tcp
>
> --udp
>
> --raw
>
> --socket
>
> --ipc
>
> --top-cpu:显示最占用CPU的进程
>
> --top-io：显示最占用io的进程
>
> --top-mem：显示最占用内存的进程
>
> --top-lantency：显示延迟最大的进程（什么是延迟最大？）

#### kill命令

用于向进程发送信号，以实现对进程进行管理

显示当前系统可用信号： kill -l [signal]

每个信号的表示方式有三种：

1）信号的数字标志
2）信号的完整名称
3）信号的简写名称

向进程发送信号：

kill [-s signal|-SIGNAL] pid

常用信号：
1）SIGHUP：无需关闭进程而让其重读配置文件

2）SIGINT：终止正在运行的进程，相当于ctrl+c

3）SIGQUIT：该信号，我在对ping进程实验时，发现每发送一次型号，该进程并不会结束，而是显示一次统计数据，而后继续运行

```
64 bytes from 172.16.0.104: icmp_seq=57 ttl=128 time=0.180 ms
57/57 packets, 0% loss, min/avg/ewma/max = 0.167/0.238/0.197/0.766 ms
64 bytes from 172.16.0.104: icmp_seq=58 ttl=128 time=0.312 ms
```



9）SIGKILL：强行杀死运行中的进程,立即生效

15）SIGTERM：终止运行中的进程，和SIGKILL不同的是，它允许进程处理好后事，正常退出

18）SIGCONT：调度后台被暂停的进程继续运行

19）SIGSTOP：暂停进程，送往后台

19）SIGSTOP：停止进程，送往后台

#### killall命令

根据进程名杀死进程

killall [-signal] program

# linux系统作业控制

作业分为

前台作业（foreground）：通过终端启动，且启动后会一直占据终端前
后台作业（background）：可以通过终端启动，但启动后即转入后台运行，释放出终端

如何将作业运行于后台？

1. 对于运行中的作业：ctrl+z

   注意：该方式送往后台后，作业会转为停止态

2. 对于尚未启动的作业：

   在命令后加一个&符号

   注意：即便送往后台，该进程还是与终端相关的进程，终端关闭，进程则被动结束，如果希望脱离把送往后台的作业，剥离与终端的关系，使用 nohup COMMAND & 命令即可！ 送往后台后，立即转为与终端无关模式

查看所有的作业：jobs

### 实现作业控制的常用命令：

1. fg [%JOB_NUM]， 将指定的作业调往前台
2. bg [%JOB_NUM]：让送往后台的作业在后台继续运行
3. kill %JOB_NUM：终止指定作业，这里的%JOBnum不能省！

### 调整进程优先级

用户可通过nice值管理的优先级范围：100-139

分别对应于：-20-19

进程启动时，其nice值默认为0，其优先级是120

#### nice命令：以指定的nice值启动并运行命令：

语法：nice [option] [COMMAND[ARGU]...]

常用选项：

> -n NICE:指明nice值

注意：仅管理员可以调底nice值，普通用户只能调高

#### renice 调整运行中进程的nice值

语法：renice [-n] NICE PID... 

查看nice值和优先级：

ps axo pid，nice，priority，comm | grep 关键字

