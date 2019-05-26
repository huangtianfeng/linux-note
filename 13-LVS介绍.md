[TOC]

# LVS

LVS：全称Linux Virtual Server，是一个负载集群的实现！ 工作于内核中netfilter的input链上，是input链上的一个功能，它将来自客户端的请求，通过input链中的LVS框架（该框架叫ipvs，工作于input链的钩子之上的一个框架），判断是否为集群主机，如果是，则强行扭转其传输路径，直接送往postrouting链，以送往后端主机。

## LVS集群类型中的术语

客户端地址：CIP

lvs调度器向客户端提供服务的地址：VIP

lvs调度器和后端主机通信的地址：DIP

每个后端主机的地址：RIP

lvs调度器：VS，有时候也称之为Virtual Server，Directory，Dispatcher，Balancer

后端主机：RS，有时候也称之为Real Server，upstream server，backend server

## LVS集群的类型

lvs-nat：修改请求报文的目标ip；类似于iptables的DNAT功能，但是它是一个多目标ip的DNAT；

> 客户端请求报文到达调度器后，目标ip将根据调度算法，改为后端的某个RIP，后端主机响应，所有后端主机应该将默认网关指向调度器，以实现调度器将其源地址改为VIP
>
> 1. RIP和DIP必须在同一个ip网络，且应该使用私网地址；
> 2. 请求报文和响应报文都需要经过director，director容易成为性能的瓶颈
> 3. 支持端口映射，可以修改请求报文的目标port，和iptables原理一样。
> 4. vs必须是linux系统，rs可以是任何系统；

lvs-dr：通过为请求报文重新封装一个MAC首部进行转发，源MAC是DIP所在的接口的MAC，目标MAC是某个被挑选出来的RS的RIP所在的接口的MAC地址

> 各RS必须配置使用一个和VS一样的VIP，将其用于响应报文的源ip地址。保证与请求报文的目标地址一致
>
> 注意：
>
> 1. ，因为每个RS都需要配置一个相同的VIP，所以需要确保前端路由器将目标ip地址为VIP的请求报文发往director；
>
> 方法1：在前端网关做静态mac地址绑定，不实用；
>
> 方法2：在RS上使用arptables工具，用的比较少；
>
> 方法3：在RS上修改内核参数（arp_announce和arp_lgnore），以限制arp通告以及应答级别；
>
> 向每一个网络通告本机所有的网卡的ip和MAC
>
> * arp_announce：限制通告级别：
>
> arp_announce=1：尽量避免向非直接连接网络进行通告

> arp_announce=2：必须避免向非直接连接的网络进行通告
>
> arp_announce=0： 默认值，表示吧本机上的所有接口的所有信息向每个接口上的网络进行通告
>
> * arp_ignore：限制响应级别：
>
> arp_ignore=0：默认值，表示可以使用本地任意接口上配置的任意地址进行响应
>
> arp_ignore=1：仅在请求的目标ip配置在本地主机的接收到请求报文接口上时，才给与响应；

> 2. RS的DIP可以是公网地址，也可以是私网地址，使用公网地址时，需要保证RIP和DIP在同一网络中；
>
> 3. RS的网关不能指向VS，以确保响应报文不会再经过Director；
>
> 4. RS和Director必须在同一个物理网络，不能经过路由器，因为是基于mac地址调度，经过路由器，mac地址就会被修改；
>
> 5. 因为响应高报文不经过Director，所以不支持端口映射；

lvs-tun：在原请求ip报文之外新加一个ip首部；可以理解为一种隧道逻辑

> 转发方式：不修改请求报文的ip首部，而是在源ip报文之外在封装一个IP首部，将报文发往挑选出的目标RS；RS直接响应给客户端
>
> 1. DIP,VIP,RIP都应该是公网地址
> 2. RS的网关不能，也不可能指向DIR
> 3. 请求报文要经过Director，响应报文不会
> 4. 不支持端口映射
> 5. RS的OS得支持隧道功能

lvs-fulinat：修改请求报文的源ip和目标ip

> 通过同时修改请求报文的源ip地址和目标ip地址进行转发；
>
> 1. VIP是公网地址，RIP和DIP是私网地址，且通常不再同一ip网络；因此，RIP的网关一般不会指向DIR
> 2. RS收到的请求报文源地址是DIP，因此，只能响应给DIP，DIP收到后，根据追踪表，将其源ip和目标ip修改回来
> 3. 请求报文和响应报文都需要经过director
> 4. 支持端口映射；
>
> 注意，该来兴，默认不支持，是作者后来开发出来的，想使用，需要自行编译内核，添加其功能

## LVS的调度算法

### 静态调度算法，仅根据算法本身调度

* RR：RoundRobin：轮询调度，顾名思义，轮询

* WRR：Weighted RR ： 加权轮询，根据服务器性能的不同，为其添加权值，权值越大，被调度到的频率越高，假设后端一共三台rs，rs1的权重为3，rs2的权重为1，rs3的权重为1，那么当vs同时接收到五个请求时，将三个请求调度到rs1，一个请求调度到rs2，一个请求调度到rs3.
* SH：Source Hashing：源地址ip hash，将源地址计算出hash值，保存于内存中，作为键值，一个ip请求过后，保存下来hash值后，随后的访问，都将被调度到第一次到来是别调度的服务器上，这样做的缺点是，当下并不是每个客户端都有独立的公网ip地址，很多客户端都是共享同一个公网ip地址的，so...
* DH：Destination Hashing：目标地址ip hash，和SH恰好想法，保存目标地址，通常用于正向代理服务器的负载均衡器上，将请求同一个ip的请求，调度到一台服务器，以提高缓存命中率。

### 动态调度算法，根据后端服务器的负载，动态调整。

主要根据每个rs当前的负载状态及结合调度算法进行调度

* LC：Least Connection：最少连接调度

  很简单，把新连接调度至当前连接数最少的rs上

  通过active\*256+inactive 计算出rs的连接数，优先调度给该值最小的rs（之所以将活动连接\*256，是为了将inactive的比重降低，甚至忽略不计）

* WLC：Weight  Least Connection：加权最少连接调度

  在前面的基础上，为每个服务器加权，通过(active*256+inactive)/weight 计算出值，有限调度给值最小的rs，从算法可以看出，权值越大，结果将越小，也就越加优先被调度

* SED：最小期望延迟，在WLC的基础上，不考虑非互动连接 通过(active*256+1)/weight

* NQ：never queue：永不排队，在SED的基础上，权值小的rs，有可能面临不会被分配到请求的情况，忙的忙死，闲的闲死。  该方法在SED的基础上，规定，vs接受到新请求后，先查看是否有零连接的rs，如果有则直接调度到连接为零的rs上，否则再根据SED的计算方式计算

* LBLC：基于局部性的最小连接

* LBLCR：带复制的基于局部性的最下连接

# ipvsadm工具的使用

安装ipvsadm程序包；

ipvsadm命令

## 管理集群服务：增、删、改

* 增、改

  ipvsadm -A|E -t|u|f serveice-address [-s scheduler] [-p [timeout]]

* 删

  ipvsadm -D -t|u|f service-address

* 其中，service-address表示VS的ip地址VIP：

  -t|u|f：

  -t：tcp协议的端口，VIP:TCP_PORT

  -u：udp协议的端口

  -f：firewall MARK， 防火墙标记，是一个数字，以后再介绍

  [-s scheduler]：指定集群的调度算法，默认为wlc

## 管理集群上的RS： 增、改、删

* 增、该：

  ipvsadm -a|e -t|u|f service-address -r server-address [-g|i|m] [-w weight]

  lvs类型

  >  -g：gateway，dr类型
  >
  > -i：ipip，tun类型
  >
  > -m：masquerade，nat类型

  -w weight：权重

* 删

  ipvsadm -d -t|u|f service-address -r server-address

* 其中 server-address 表示RIP，格式 rip[:port]，如非必要，不要写端口，以免写错

清空定义的所有内容：ipvsadm -C

## 查看：

ipvsadm -L|l [options]

常用选项：

> --numeric, -n :数字格式显示
>
> --exact：精确显示计数器的值
>
> --stats：统计数据
>
> --rate：速率数据
>
> --connection,-c： 显示连接数

规则保存和重载：

ipvsadm -S 或者ipvsadm-save

> ipvsadm -S -n > /etc/sysconfig/ipvsadm
>
> systemctl stop ipvsadm.service停止时，也会自动保存至上面文件中

ipvsadm -R 或者 ipvsadm-restore

