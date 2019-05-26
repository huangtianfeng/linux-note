[TOC]

LVS-dr（dr：director routing，直接路由）:通过将请求报文重新封装一个MAC首部，将其转发至后端服务器，转发完成后，响应报文将直接由后端rs响应给客户端，因为是基于MAC地址进行的转发，所以，请求报文的源ip地址和目标ip地址都不会变，由于请求报文的源ip和目标ip皆未发生改变，所有，后端的每个rs都需要配置双网卡，其中一块网卡配置和VS的ip地址VIP一致，以达到接受转发过来的报文的目的，而响应报文，也需要经由ip地址为VIP的网卡，发送出去，为了避免ip地址冲突，还需要对rs机器的arp协议的参数进行限制

> echo 2 >  /proc/sys/net/ipv4/conf/all/arp_announce :禁止网卡向非直接连接的网络进行通报
>
> echo 1 > /proc/sys/net/ipv4/conf/all/arp_ignore：禁止响应目标地址为非直接连接的网卡ip的广播请求

# 准备拓扑图

![插图15-1](F:\Users\tian\Pictures\linux图库\插图15-1-1558524909783.png)

准备四台虚拟机，都使用桥接模式即可

# 配置流程

1. 配置vs，添加两块网卡，一块桥接到宿主机，地址设置为172.16.1.50，一块设置为仅主机模式，地址设置为172.16.1.100，其中还设置了一个172.16.0.50用于ssh连接，下文中，所有172.16.0网段的地址都仅用于ssh连接

   设置完后如下所示：

   ```bash
   [root@node1 ~]# ip addr 
   1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
       link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
       inet 127.0.0.1/8 scope host lo
          valid_lft forever preferred_lft forever
       inet6 ::1/128 scope host 
          valid_lft forever preferred_lft forever
   2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
       link/ether 00:0c:29:cf:62:9d brd ff:ff:ff:ff:ff:ff
       inet 172.16.0.50/24 scope global ens33
          valid_lft forever preferred_lft forever
       inet 172.16.1.50/16 scope global ens33
          valid_lft forever preferred_lft forever
   3: ens37: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
       link/ether 00:0c:29:cf:62:a7 brd ff:ff:ff:ff:ff:ff
       inet 172.16.1.100/16 scope global ens37
          valid_lft forever preferred_lft forever
   ```

2. 安装ipvsadm程序包，并添加地址为172.16.1.50的lvs服务，通过wlc算法，将请求转发至后端主机，172.16.1.101和172.16.1.102，并且将vs的核心转发功能打开

   ```bash
   [root@node1 ~]# ipvsadm -A -t 172.16.1.50:80 -s dr
   No such device or address
   
   [root@node1 ~]# ipvsadm -A -t 172.16.1.50:80 -s wlc
   [root@node1 ~]# ipvsadm -a -t 172.16.1.50:80 -r 172.16.1.101 -g
   [root@node1 ~]# ipvsadm -a -t 172.16.1.50:80 -r 172.16.1.102 -g
   [root@node1 ~]# ipvsadm -ln
   IP Virtual Server version 1.2.1 (size=4096)
   Prot LocalAddress:Port Scheduler Flags
     -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
   TCP  172.16.1.50:80 wlc
     -> 172.16.1.101:80              Route   1      0          0         
     -> 172.16.1.102:80              Route   1      0          0         
   [root@node1 ~]# echo 1 > /proc/sys/net/ipv4/ip_forward
   
   ```

   注意第一个报错，-s应该指明调度算法，不是指明lvs类型

3. 配置rs，ip地址设置为对应的172.16.1.101和172.16.1.102,这个过程就不贴了，配置完后是这样的

   RIP:172.16.1.101

   ```bash
   [root@localhost ~]# ip addr l 
   1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
       link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
       inet 127.0.0.1/8 scope host lo
          valid_lft forever preferred_lft forever
       inet 172.16.1.50/32 scope global lo
          valid_lft forever preferred_lft forever
       inet6 ::1/128 scope host 
          valid_lft forever preferred_lft forever
   2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
       link/ether 00:0c:29:08:4c:af brd ff:ff:ff:ff:ff:ff
       inet 172.16.1.101/16 scope global ens33
          valid_lft forever preferred_lft forever
       inet 172.16.0.222/24 scope global ens33
          valid_lft forever preferred_lft forever
   
   ```

   RIP:172.16.1.102

   ```bash
   [root@localhost ~]# ip addr l
   1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN 
       link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
       inet 127.0.0.1/8 scope host lo
       inet 172.16.1.50/32 scope global lo
       inet6 ::1/128 scope host 
          valid_lft forever preferred_lft forever
   2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
       link/ether 00:0c:29:12:fb:8b brd ff:ff:ff:ff:ff:ff
       inet 172.16.1.102/16 scope global eth0
       inet 172.16.0.223/24 scope global eth0
   
   ```

4. 在rs1和rs2上限制arp的发送和响应级别，在两台rs主机上分别执行以下命令

   ```
   [root@localhost ~]# echo 2 > /proc/sys/net/ipv4/conf/all/arp_announce 
   [root@localhost ~]# echo 1 > /proc/sys/net/ipv4/conf/all/arp_ignore 
   ```

5. 至此，所有配置完成，在另外一个172.16.1网段的虚拟机，访问172.16.1.50测试。

   ```
   [root@localhost ~]# for i in {1..10}; do curl 172.16.1.50; done
   <h1>this is 172.16.1.101</ht>
   <h1> This is 172.16.1.102 </h2>
   <h1>this is 172.16.1.101</ht>
   <h1> This is 172.16.1.102 </h2>
   <h1>this is 172.16.1.101</ht>
   <h1> This is 172.16.1.102 </h2>
   <h1>this is 172.16.1.101</ht>
   <h1> This is 172.16.1.102 </h2>
   <h1>this is 172.16.1.101</ht>
   <h1> This is 172.16.1.102 </h2>
   ```

   



