[TOC]

# vmstat 

功能：报告虚拟内存统计信息

语法： vmstat [options] [delay [count]]

-s 显示内存统计数据

vmstat [delay]后面直接给一个数字，表示每多少秒刷新一次,或者vmstat delay conut，显示多少个 多少次

例如 vmstat 2 3  两秒刷新一次，显示三次

后面都是可选项：

```
[root@localhost ~]# vmstat
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 1  0      0 1240732   4184 455972    0    0     1     1   41   40  0  0 100  0  0
```

各字段的意思：

* procs

  r：表示等待运行的进程的个数，cpu上等待运行的任务队列长度

  b：处于不可中断睡眠态的进程个数；被阻塞的任务队列长度

  > 如果这个队列很长，意味着等待io的时间太大了，io能力有限

* memory，内存段

  swpd：交换内存的使用总量，0表示没启用

  > 如果该数值过大，意味着你的物理内存开始不够用了

  free：空闲的物理内存总量

  buffer：用于buffer的内存总量（缓冲）

  cache：用于cache的内存总量（缓存）

* swap段

  si：数据进入swap中的速率，单位kb/s

  so：数据离开swap的速率，单位kb/s  

  > 判断物理内存是否不够用的依据
  >
  > 1.swap是否启用了
  >
  > 2.si，so的活动是否频繁，so的速率比较重要

* io

  bi：从块设备读入数据到内存的速率，单位kb/s

  bo：保存数据到块设备的速率，单位kb/s

* system

  in：inerupts：中断产生的速率，每秒多少个

  > 中断：一种通知机制，io设备需要和cpu交互时，设备产生一个中断信号，提醒cpu，我要传送数据给你了，cpu为了能知道是谁发的信号，使用不同的线路，针脚加以区分

  cs：context switch 上下文切换的速率，进程被内核调度的速率，每秒多少次

* cpu：

  us：用户空间程序占据cpu的百分比

  sy：内核空间程序占据cpu的百分比

  id：idie 空闲的

  wa： wait 等待io完成的

  st： stolen 被虚拟化偷走的CPU百分比

# tcpdump命令

功能，抓取网络上传输的数据包信息

注意：只有root用户才能使用该命令抓取数据包

tcpdump命令，可以分为 选项，过滤表达式和输出信息三个部分

* 选项部分：

-i：interface ，指定抓取的目标网络接口，也就是指定到哪块往卡上抓取

-nn：表示不要将端口号转成协议名称，例如，碰到80端口时，不要将其转换成为“http”

-X：表示将抓取到的协议头和包内容全部显示出来（会分别以十六进制和ASCII码同时显示）

示例：

```
[root@localhost ~]# tcpdump -i ens33 -nn -X -c 1
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on ens33, link-type EN10MB (Ethernet), capture size 262144 bytes
22:07:33.823182 IP 172.16.0.106.22 > 172.16.0.104.60574: Flags [P.], seq 2618411041:2618411253, ack 2314209566, win 320, length 212
	0x0000:  4510 00fc 7d33 4000 4006 63c6 ac10 006a  E...}3@.@.c....j
	0x0010:  ac10 0068 0016 ec9e 9c11 c821 89f0 091e  ...h.......!....
	0x0020:  5018 0140 59e1 0000 0000 00b0 77b2 94d7  P..@Y.......w...
	0x0030:  f072 9161 4d60 2e8f 765e ffd6 1197 5a19  .r.aM`..v^....Z.
	0x0040:  35d4 a42d bc35 9bf0 8828 c229 b4c8 f0df  5..-.5...(.)....
	0x0050:  7a17 5963 e4b9 2789 3c62 7247 057b 09ce  z.Yc..'.<brG.{..
	0x0060:  0af2 ea91 d1fa aae0 3148 72aa b716 ed9e  ........1Hr.....
	0x0070:  d9f9 0298 0598 abc2 e64c 5525 35e9 1585  .........LU%5...
	0x0080:  2864 9663 8009 7522 1e5e 05ed 2668 aa60  (d.c..u".^..&h.`
	0x0090:  b19f cca6 ece6 a749 d345 b7c6 2978 ae45  .......I.E..)x.E
	0x00a0:  7160 e5ea c86f d53d 306e db04 bea9 d48b  q`...o.=0n......
	0x00b0:  6b69 c94f 7ad0 29c8 abb4 087f 48c8 cdcd  ki.Oz.).....H...
	0x00c0:  21ce 0011 f90a 32c3 1d5d 1fb2 0b28 c731  !.....2..]...(.1
	0x00d0:  f858 8481 f595 31cb d323 f560 81fd fcff  .X....1..#.`....
	0x00e0:  6496 7eeb f0f6 5a00 5fd0 9de8 c458 ce74  d.~...Z._....X.t
	0x00f0:  46c2 1bd2 2984 6f1a 40e8 042d            F...).o.@..-
1 packet captured
2 packets received by filter
0 packets dropped by kernel
```

-e：输出链路层的头部信息，包括MAC地址，以及网络层的协议

-c： 指定抓取数据包的数量，默认会持续抓取

-D： 列出可以抓取的接口列表

-f 　将外部的网络地址以数字的形式打印出来

-l 	将输出修改为行缓冲，表示一旦遇到换行符，就将缓冲区的内容输出，以避免需要将数据进行后续处理的时候，出错

-t：取消时间戳的显示

-v：显示更加详细的信息，包括ttl值，总长度等等

-F：指定一个保存有过滤条件的文件，从其中读取过滤条件

-w：流量保存，后面指定文件名即可

-r：流量回访，将使用-w选项保存下来的数据包重新回放出来，并且还可以使用其他选项和过滤条件

> 流量保存就是把抓到的网络包存储到磁盘上，保存下来，为以后使用
>
> 流量回访就是把历史上的某一时段的流量，重新模拟回访出来，用于流量分析



* 过滤表达式部分

  tcpdump不仅支持单个条件过滤表达式，还支持同时接受多个过滤表达式，当过滤表达式含有shell通配符时，需要用单引号强引用，以免被shell解释

  过滤表达式大体分为三种，类型、方向和协议

  注意：协议过滤不支持应用层协议，只支持应用层以下的，但是可以通过端口号来实现，/etc/service文件中保存了几乎所有常见应用层协议和使用的端口号的对应关系

  基于源ip和目标ip过滤，使用src和dst，例如

  ```
  [root@localhost ~]# tcpdump -i ens33 'dst 172.16.0.104'
  [root@localhost ~]# tcpdump -i ens33 'src 172.16.0.104'
  ```

  大多数情况，单引号可以省略

  基于主机名过滤，可以使用 host，例如

  ```
  [root@localhost ~]# tcpdump -i ens33 -nn -c 1 host 172.16.0.106
  ```

  基于网络段过滤，可使用net，例如

  ```
  [root@localhost ~]# tcpdump -i ens33 -nn -c 1 net 172.16
  ```

  基于端口号范围可以使用portrange，例如：

  ```
  [root@localhost ~]# tcpdump -i ens33 -nn -c 1 portrange 6000-8000
  ```

  基于单个端口号过滤可以使用port，例如

  ```
  root@localhost ~]# tcpdump port 22  -i ens33 -nn -c 1 
  root@localhost ~]# tcpdump port ssh  -i ens33 -nn -c 1 
  ```

  这里的协议和端口号的关系保存于/etc/services文件中，如果在文件中修改了对应关系，那么在该命令中也会生效

  还有。过滤条件可以用and或者or连接起来的哦！例如，抓取22号端口上，并且源地址是172.16.0.104的包

  ```
  [root@localhost ~]# tcpdump port 22 and src 172.16.0.104 -i ens33 -nn -c 1 
  tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
  listening on ens33, link-type EN10MB (Ethernet), capture size 262144 bytes
  22:53:26.611195 IP 172.16.0.104.60574 > 172.16.0.106.22: Flags [.], ack 2622681141, win 2051, length 0
  1 packet captured
  1 packet received by filter
  0 packets dropped by kernel
  ```

  也可以这样：抓取ens33网卡上，本机和www.baidu.com，或者本机和www.souhu.com的网络包

```
[root@localhost ~]# tcpdump 'host 172.16.0.106 and (baidu.com or souhu.com)' -i ens33
```

## 内容解读

我们来看一下，使用tcpdump抓取到的内容

```
[root@localhost ~]# tcpdump  -X -c 1 -i ens33
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on ens33, link-type EN10MB (Ethernet), capture size 262144 bytes
22:42:14.405662 IP localhost.localdomain.ssh > 172.16.0.104.hri-port: Flags [P.], seq 80314465:80314677, ack 3344250488, win 274, length 212
	0x0000:  4510 00fc cadc 4000 4006 161d ac10 006a  E.....@.@......j
	0x0010:  ac10 0068 0016 0d6f 04c9 8061 c755 3678  ...h...o...a.U6x
	0x0020:  5018 0112 59e1 0000 0000 00b0 3b55 52d5  P...Y.......;UR.
	0x0030:  a8c0 4f03 548c 789a a513 97a7 8ad4 b6f7  ..O.T.x.........
	0x0040:  7aee 3041 9f66 ab06 3061 689d 5bff 468b  z.0A.f..0ah.[.F.
	0x0050:  1089 ed30 f8d7 d391 7fbb aae0 c70c 7b1b  ...0..........{.
	0x0060:  d330 3120 6901 6cc0 ac75 029e a841 0e4f  .01.i.l..u...A.O
	0x0070:  155c c60a ec2e 73ce ee98 02dd 9f33 b5ae  .\....s......3..
	0x0080:  3c9b 579a 93a2 2e0d cb41 33d9 a422 e7e7  <.W......A3.."..
	0x0090:  529a fbcc d8d1 fdca 3c83 499e 9c6f d21c  R.......<.I..o..
	0x00a0:  30ad a4fa 624f 0add c770 1172 c14c 0d2f  0...bO...p.r.L./
	0x00b0:  17ac 578b 6d28 ec5c f12b a708 436e 8cc9  ..W.m(.\.+..Cn..
	0x00c0:  49f0 01b5 3080 4795 2205 2bb9 31c6 25f5  I...0.G.".+.1.%.
	0x00d0:  7937 fb94 5403 28e6 ab50 c1d0 0583 d830  y7..T.(..P.....0
	0x00e0:  db3e 4efd cbb8 125b dda1 bb37 5911 51d3  .>N....[...7Y.Q.
	0x00f0:  3983 1b0c a3c0 2449 5916 2f01            9.....$IY./.
```

我们从第一行开始解读，第一行的大意就是贴心的告诉你使用-v或者-vv可以查看更加详细的信息

第二行 监听在ens33网卡上，网络类型是以太网，能够抓取的单个包的大小限制，此处为262144，可以使用-s选项调整

第三行，源ip加端口到目标ip加端口，后面部分，是应用层协议的内容，有兴趣可以去了解一下DNS，FTP，HTTP等协议的格式

接下来的内容最左侧的一列，我们先不管，大概就是序号把，中间的四个十六进制数一组的，是网络层ip报文的真正内容，其中包含了ip报文首部和ip报文的数据部分，我们知道，ip报文的数据部分就包含了传输层的tcp或者udp协议的首部和数据，而tcp协议的数据又是上层协议的内容，所以，解读这些内容，最起码对网络层的协议格式和传输层的协议格式了如指掌，笔者暂时还做不到，但是，可以翻书对着解读啊，哈哈！ 上面的每个十六进制数对应4bit，一组也就是2字节！ 把他拆开来看就好了，例如，ip报文首部一共二十字节，也就是前十组十六进制数所对应的数据，最前面4bit表示协议版本，这里的值是4，也就是ipv4协议，接下来的4位是首部长度，单位是32bit（4byte），这里的值是5，也就表示该报文首部大小是5*4byte=20字节，然后我们看到第十组的后两位十六进制数，对应的是第20字节，标识的是上层协议，这里的06表示上层协议是tcp协议，没错吧！ 接下来的内容，大家去对着看吧，然后我们跳过而是字节，看看接下来的就是TCP协议了的首部内容了，对应到上面第二行的第三组0016，该值表示源端口，0x0016对应十进制的22号端口，刚好是我们使用的ssh协议的端口，没错吧！ 其他内容，按照各报文格式解读就可以了！