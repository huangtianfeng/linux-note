[TOC]

nat类型：通过目标ip地址转换，实现将请求转发至后端服务器，后端服务器收到的请求，其目标地址是自己，源地址是客户端，它通过将网关设置为vs主机，将报文转发给vs主机后，由vs主机负责将源地址改为客户端地址，所以，为了保证后端rs能够将默认网关指向vs主机，后端主机和vs的DIP必须在同一网段内，且应该使用私网地址。不是必须。

# 准备工作

1. 准备拓扑图

![插图14-1](https://github.com/huangtianfeng/pictures/blob/master/linux%E5%9B%BE%E5%BA%93/%E6%8F%92%E5%9B%BE14-1.png)

之所以用105这个网段，是因为WMware的仅主机模式连接的虚拟DHCP服务器的默认分配的地址是该网段，所以，笔者也就懒得改了

2. 创建四台CentOS虚拟机，版本自选，用来模拟客户机，VS，RS。我这里准备了两台CentOS-6和两台CentOS-7的虚拟机，6用来当RS，7用来做客户端和VS

# 配置流程

1. 为一台CentOS-6虚拟机 添加两块网卡，模拟VS，一块为桥接模式，一块为仅主机模式，桥接模式的网卡地址设置为172.16.0.100，仅主机模式的网卡地址设置为192.168.105.100

   ![插图14-2](https://github.com/huangtianfeng/pictures/blob/master/linux%E5%9B%BE%E5%BA%93/%E6%8F%92%E5%9B%BE14-2.png)

   忘了我的172.16.0.100地址已经被占用，这里为了方便，VIP 改为了172.16.0.200。

   ```
   [root@localhost ~]# ip addr l 
   1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN 
       link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
       inet 127.0.0.1/8 scope host lo
       inet6 ::1/128 scope host 
          valid_lft forever preferred_lft forever
   2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
       link/ether 00:0c:29:d1:70:6d brd ff:ff:ff:ff:ff:ff
       inet 172.16.0.216/16 brd 172.16.255.255 scope global eth0
       inet6 fe80::20c:29ff:fed1:706d/64 scope link 
          valid_lft forever preferred_lft forever
   3: eth1: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN qlen 1000
       link/ether 00:0c:29:d1:70:77 brd ff:ff:ff:ff:ff:ff
   [root@localhost ~]# ip addr add 172.16.0.200 dev eth0
   ```

   为了不影响远程连接，我就直接添加一个地址，原先的地址保留了！

   这里我们看到，eth1还没有启动起来，我们先给他一个配置文件，然后启动网卡

   ```bash
   [root@localhost ~]# cat > /etc/sysconfig/network-scripts/ifcfg-eth1 << eof
   DEVICE=eth1
   TYPE=Ethernet
   ONBOOT=yes
   NM_CONTROLLED=yes
   BOOTPROTO=dhcp
   eof
   [root@localhost ~]# ip link set eth0 up
   ```

   再看看我们的网络接口情况：

   ```
   [root@localhost ~]# ip addr l 
   1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN 
       link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
       inet 127.0.0.1/8 scope host lo
       inet6 ::1/128 scope host 
          valid_lft forever preferred_lft forever
   2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
       link/ether 00:0c:29:d1:70:6d brd ff:ff:ff:ff:ff:ff
       inet 172.16.0.216/16 brd 172.16.255.255 scope global eth0
       inet 172.16.0.200/32 scope global eth0
       inet6 fe80::20c:29ff:fed1:706d/64 scope link 
          valid_lft forever preferred_lft forever
   3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
       link/ether 00:0c:29:d1:70:77 brd ff:ff:ff:ff:ff:ff
       inet 192.168.105.131/24 brd 192.168.105.255 scope global eth1
       inet6 fe80::20c:29ff:fed1:7077/64 scope link 
          valid_lft forever preferred_lft forever
   ```

   把eth1 的网卡地址清除重新设置为我们需要的192.168.105.100

   ```bash
   [root@localhost ~]# ip addr flush eth1
   [root@localhost ~]# ip addr add 192.168.105.100/24 dev eth1
   ```

   打开路由转发功能

   ```bash
   [root@localhost ~]# echo 1 > /proc/sys/net/ipv4/ip_forward
   [root@localhost ~]# cat /proc/sys/net/ipv4/ip_forward
   1
   ```

   

2. 按照拓扑图，使用ipvsadm工具，配置集群服务

   ```
   [root@localhost ~]# ipvsadm -A -t 172.16.0.200:80 -s wlc
   [root@localhost ~]# ipvsadm -a -t 172.16.0.200:80 -r 192.168.0.101 -m
[root@localhost ~]# ipvsadm -a -t 172.16.0.200:80 -r 192.168.0.102 -m
   [root@localhost ~]# ipvsadm -l
   IP Virtual Server version 1.2.1 (size=4096)
   Prot LocalAddress:Port Scheduler Flags
     -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
   TCP  172.16.0.200:http wlc
     -> 192.168.0.101:http           Masq    1      0          0         
     -> 192.168.0.102:http           Masq    1      0          0         
   ```
   
3. ok 接下来可以开始搭建RS的服务主机了

   rs都使用仅主机模式，并将ip地址都修改为指定地址 192.168.105.101和192.168.105.102,并且将默认网关改为DIP，这里我们的DIP是192.168.105.100

   RS1:

   ```bash
   [root@node1 ~]# ip addr add 192.168.105.101/24 dev ens33
   [root@node1 ~]# ip route add default via 192.168.105.100
   ```

   RS2:
   
   ```
   [root@node1 ~]# ip addr add 192.168.105.101/24 dev ens33
   [root@node1 ~]# ip route add default via 192.168.105.100
   ```
   
   
   
   因为是仅主机模式，安装httpd服务，需要使用本地光盘镜像
   
   将本地光盘挂载上，创建一个repo仓库，路径指向file:///PATH/FORM/DIR即可，前两个斜杠是标准格式，后一个斜杠表示根目录。
   
4. 到这里，我们就搭建完成了，自行测试了一下，但是..我搭完之后，脑抽的把VS主机给还原镜像了！so...

# 需要注意的问题

1. 确保地址不要错，粗心大意的笔者就因为马马虎虎而浪费了很多时间，出了很多错，要么就是地址写错了把192.168.105.101之类的地址写成了我们常用的192.168.0.....    ,以至于想了半天才发现，地址填错了!  
2. 设置网卡ip地址的时候，注意加子网掩码，不要直接填个地址，笔者也是在这个问题上吃亏了，设置完后摸索半天，才发现，网卡地址的子网掩码不对，一个头两个大。
3. 总结起来就是，如果配置完后，发现访问不了：
   * **检查VS的ip地址以及RS的默认网关设置，RS的默认网关是指向VS的DIP**
   * 检查各网卡地址的掩码长度是否一致
   * ipvsadm查看lvs服务配置是否正确
