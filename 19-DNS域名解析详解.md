[TOC]

# DNS域名解析详解

在我们日常浏览网页中，DNS解析是不可缺少的一环，用户在浏览器输入一个网址，计算机会将该地址解析成为ip地址，而后根据ip地址寻找到对应的服务器，请求服务！

那么，将网址转换为IP地址，是如何实现的呢？ 这就是本文的重点内容！

**DNS是Domain Name System的简写，域名系统，他是Internet上一项非常重要的服务，也是一个分布式的数据库，其作用是将域名和ip地址相互映射，使人们能够更加方便的访问互联网**

所谓的分布式数据库，就是将一个大的数据库，分割成n多小的数据库，统一管理

## 域名

拿www.baidu.com 来举例，我们称之为一个FQDN(Fully Qualified Domain Name)，翻译过来就是完全限定域名，它由n段由.号隔开的字符串组成，**自右向左**分别是顶级域、二级域、主机名！简化一下，FQDN由主机名和域名组成。

* 顶级域（TLD：Top-Level Domain）

  * 组织域： 常见的组织域有 .com,.net,.ort,.gov,.edu等等。

  * 国家域：大部分国家都有自己的国家域，例如lq，cn等，cn就是中国的国家域

  顶级域直接由根域管理

* 二级域

  二级域需要向所在地区的顶级域代理申请，如果你想申请一个以.com结尾的域名，那么你就要想你所在地区的域名代理商申请一个子域，这个子域不能和其他.com顶级域的子域重名，必须具备唯一性，中国著名的两个域名代理商是“万网”和“新网” ，大家如果想注册域名，搜索一下这两个网站即可

* 三级域

  这个一般都是免费的，申请到二级域后，三级域随便你怎么玩，因为每一个上级域，都负责管理其子域，你拿到二级域的管理权后，其下的子域，都是你的！该位置也可以是主机名，也还可以有四级域，五级域 


域名申请成功后，你需要一个服务器来负责解析该域内的子域，或者说主机，这个服务器就是DNS服务器。

## DNS服务器

早期的域名解析，是全靠计算机本地的一个hosts文件完成的，我们把要解析的域名和ip地址的对应关系手动编辑保存至这个文件中（现在这个文件也依旧存在），后来由于互联网的膨胀式发展，导致域名数量太多，所以引入了DNS这个服务，它是一个分布式的数据库，将互联网中所有的域名和ip地址对应关系保存至数据库中，供用户查询！

* 域名解析过程：

  当你在浏览器中输入一个网址后，计算机解析的过程如下

  hosts文件-->本地DNS缓存-->DNS服务器（这里分为两种情况，一种是你请求的域名，刚好是该服务器负责的域，那么它将直接返回结果，如果不是，那么它会帮你找到根域，层层迭代最后找到结果返回给你！）而后基于ip地址访问目标服务器！

  **注意**：DNS服务器又分为两种，一种是缓存DNS服务器，它不负责任何域的解析，只负责帮你迭代查询，而后将结果缓存到服务器上，当你下次再次请求解析时，就直接返回结果给你！

  另外一种就是负责特定域的解析工作的服务器了！

* 返回结果的分类：

  根据是否查询到答案，分为肯定答案和否定答案，否定答案表示没有查询对应的映射关系，否定答案也会被缓存下来，该缓存的生命周期（TTL：Time to Live）由服务器中的数据库定义

  根据由何种服务器返回的结果，分为权威答案和非权威答案，权威答案表示由直接负责特定域解析的服务器返回的结果，其他缓存服务器返回的结果则属于非权威答案！

* DNS服务器的主从关系

  负责同一个域解析的服务器，可以有多台，其中一台为主服务器，其他的均为从服务器，二者的数据库必须保持一致，为了达到该目的，数据库就有了序列号的概念，从服务器每隔多长时间（这个时间称为刷新时间），会请求主服务器对比序列号，如果序列号不同，则同步数据库！当然这还不够，主服务器在有数据改动的时候，也会通知从服务器同步数据。

  注意：从服务器只能从主服务器或者其他从服务器中同步数据，没有写权限，只接受查询，不可更改内容！

  当从服务器联系不上主服务器时，每隔多久重新联系一次，这个时间称为重试时间（retry），联系不上后，多久时间放弃联系，这个时间称为过期时间、最后从服务器一旦联系不上主服务器，那么它将停止提供服务。

  主从服务器的同步方式还分为**全量传送和增量传送**  顾名思义，增量传送表示只同步更改过得内容，而全量传送，一般只会在刚建立从服务器的时候使用一次

再补充三点

每个DNS服务器接受到解析请求时，如果不是自己负责的域，那么它将直接去找根节点服务器，而非上级

根节点服务器全球一共十一台，瑞典一台，日本一台，剩下的均在美国！ 根节点的服务器安全等级比白宫高，因为十一台根节点服务器一旦挂掉，全球互联网将陷入瘫痪。

什么是域和区域？

域是一个逻辑概念，看不见摸不着，区域才是物理概念，一个域由正向解析区域和反向解析区域组成！ 大致理解就成

## 区域数据库文件

数据库中存放数据，我们称之为资源记录RR（Resource Record）

**语法格式：name	[TTL]	IN	RR_TYPE	value**

该记录又分为A、AAAA、PTR、SOA、NS、CNAME和MX几种不同的类型

* SOA（Start Of Authority）起始授权记录

  格式：

  name：当前区域名称：例如magedu.com

  value：由多部分组成

  1. 当前区域名称（也可以是DNS服务器名）

  2. 当前管理员的邮箱，注意@符号需要用.号代替

  3. （一串数字，主从协调属性的定义以及否定答案的TTL） 注意要用括号括起来

     例如：

     > magedu.com.	86400	IN	SOA	magedu.com.	admin.magedu.com.(
     >
     > 201701080	;序列号，不超过十位
     >
     > 2H	;refresh刷新时间
     >
     > 10M	；retry重试时间
     >
     > 1W	；expire过期时间
     >
     > 1D	；negative answer ttl否定答案生命周期
     >
     > )
     >
     > 注意：“；”后面的是注释信息，M,H,D,W分别表示分，时，天，周

  注意：起始授权记录一个区域有且只能有一条，且必须为第一条

* NS（Name Service) 域名服务记录，标明该区域的DNS服务器

  格式：

  name：当前区域名

  value：当前区域某DNS服务器的主机名，可以有多个

  例如：

  > magedu.com.	86400	IN	NS	ns1.magedu.com
  >
  > magedu.com.	86400	IN	NS	ns2.magedu.com

* MX邮件交换器

  格式：

  name：当前区域的区域名称

  value：当前区域某邮件交换器的主机名，MX可以有多条记录，每条记录的value前需要有一个数字，表示优先级，数字越小，优先级越高，范围1-99

  例：

  > magedu.com	IN	MX	10  mx1.magedu.com
  >
  > magedu.com	IN	MX	20  mx2.magedu.com

* A(IPv4 address)

  格式：

  name：某FQDN，例：www.magedu.com

  value：某IPv4地址：

  例如：

  > www.magedu.com	IN	A	1.1.1.1
  >
  > bbs.magedu.com	IN	A	1.1.1.1
  >
  > bbs.magedu.com	IN	A	1.1.1.2

  注意：一个主机可以有多个ip，一个ip也可以有多个主机

* AAAA：ipv4的位数是32位，ipv6的地址是128位，也就是四个ipv4，所以你懂的，格式和上面一样，就不多说了

* PIR（Point）指针，或者说反向记录

  格式

  name：IP地址，有特定格式，例如1.2.3.4的ip地址需要些成4.3.2.1.in-addr.arpa

  value：某FQDN

  例如：

  > 4.3.2.1.in-addr arpa	IN	PIR	www.magedu.com

* CNAME:别名

  格式：

  name：FQDN格式的别名

  value：FQDN格式的正式名

  例如

  > web.magedu.com	IN	CNAME	www.magedu.com

注：

1. TTL可以从全局继承
2. @符号可以表示当前区域的名称
3. 相邻的两条记录的名字如果相同，后面的可以省略
4. MX,NS等类的记录，value为一个FQDN，此FQDN应该有一个A记录（只是应该，不是必须）

## Bind的安装和配置

Berkeley Internet Name Domain的简写，该工具目前由ISC组织代为维护，站点：ISC.org

Bind是DNS协议的一种实现，其运行的进程名为named

* 常用程序包

  bind：提供DNS服务器程序和几个常用的测试工具named-checkconf，named-checkzone等

  bind-utils：bind客户端程序集，例如dig，nslookup，host等

  bind-libs：提供被bind和*.utls包中的程序共同用到的库文件

  bind-chroot：选装：为安全目的，实现将named程序运行在一个沙箱中

  

* 配置文件：

  /etc/named.conf：主配置文件，文件中可以使用include关键字将其他文件包含进来

  文件格式：

  ![named主配置文件](https://github.com/huangtianfeng/pictures/blob/master/linux%E5%9B%BE%E5%BA%93/%E6%8F%92%E5%9B%BE19-1.png)

  其中options表示全局配置段、logging表示日志配置段、zone表示区域配置段（由本级负责解析的区域）

  该文件每个配置语句必须以分号结尾，支持c语言格式的注释风格

* /var/named/   该目录下存放解析文件库，一般文件名为ZONE_NAME.zone

#### 正反解析区域的配置流程

bind安装完成后，默认即可作为缓存服务器使用，前提是它能访问互联网（需要能够找到根节点），如果没有专门负责解析的区域，可直接启动之

1. 修改主配置文件/etc/named.conf，将全局配置段的监听地址改为能与外部通信的ip地址

   ![bind配置文件1](https://github.com/huangtianfeng/pictures/blob/master/linux%E5%9B%BE%E5%BA%93/%E6%8F%92%E5%9B%BE19-2.png)

2. 学习时，建议关闭dnssec功能，改成no

3. 在/etc/named.rfc1912.zones添加区域解析区域

   例：

   ```
   zone "tianfeng.com" IN {
   				type maseter;
   				file "tianfeng.com.zone";
   };
   ```

4. 在/var/named/目录下创建和上一步file后同名的区域数据库文件，格式前面已经说过了

   例：

   ```
   $TTL 3600
   @	IN	SOA	tianfen.com.	admin.tianfeng.com. (
   		2019032401
   		10M
   		5M
   		2H
   		1H
   )
   www	IN	A	192.168.0.2
   bbs	IN	A	192.168.0.3
   mx	IN	A	192.168.0.4
   @	IN	NS	ns1.tianfeng.com
   ns1.tianfeng.com	IN	A	192.168.0.5
   192.168.0.6	IN	PTR	www.tianfeng.com
   
   ```

   此处填写FQDN的位置，会自动补全/etc/named.rfc1912.conf文件中定义的区域名，也可以在文件头加$ORIGIN 定义自动补全的后缀（即域名）。 注意：此文件编辑完后应当将其他人的权限设置为0

5. 检查主配置文件和区域库文件语法，没问题的话重启named服务

   ```
   named-checkconf
   named-checkzone tianfeng.com /var/named/tianfeng.com.zone
   systemctl restart named.service
   ```

   也可以使用rndc reload 重载配置文件，效果一样

    至此，一个DNS服务器就搭建完成

   * 反向解析区域搭建同理，只是需要注意几点
     1. 在/etc/named.rfc1912.conf中定义区域时，ip地址有特定格式，例如192.168.1网段应该写成1.168.192.in-addr.arpa
     2. 区域解析库文件中，主要用于定义PTR记录，前面的主要用于定义A记录

**问题** 笔者在这一步遇到一个问题，目前还未解决，在自己的虚拟和物理机上，搭建named服务，只能解析本地主机和自己配置的域，解析不了外网，但是，在云实例上却没有问题，暂时怀疑是本地网络问题！ （路由器我都拆了，直接拨号上网，都解决不了，一个头两个大：wq）

#### 主从服务器搭建

值得注意的是，这里的主从，是基于区域的主从，不是整个域的主从！

就拿上面配置的tianfeng.com 正向解析区域为例，这里为这个正向解析区域配置一个从服务器，分为两个阶段

* 在从服务器上

  安装bind程序，完成主配置文件修改后，定义区域

```
zone "tianfeng.com" IN {
    type slave;
    file "slave/tianfeng.com.zone";
};
```

注意这里和前面的不同，type类型是slave，file后面是相对于/var/named/目录的相对路径！

* 在主服务器上

  1. 确保在区域数据库中，为每一台从服务器都配置了一个NS记录，并且该NS记录对应的还应该有一个A记录，指向真正的从服务器ip地址，例如，现在，我应该在主服务器中的tianfeng.com.zone区域数据库文件中添加两行（假设上面从服务器的ip地址是192.168.0.100）

     ```
     ns	IN	NS	ns1.tianfeng.com
     ns1	IN	A	192.168.0.100
     ```

     这里的ns1主机名可以随便取，只要保证ip地址真正指向你的从服务器，还有：**主从服务器的时间必须同步** 可以使用ntpdate命令将两个主机连到一个时间服务器以保证时间的同步

  2. 重启主从服务器就可以了！此时，正常情况下，从服务器应该已经将数据库文件同步到/var/named/slave目录下

#### 子域授权

* 流程
  1. 在区域数据库中添加子域解析服务器的NS记录
  2. 在子服务器中配置区域解析记录和资源记录文件
  3. 检查语法，重启服务

假如现在，我要在tianfeng.com这个域下再添加一个opt.tianfeng.com的三级域！并且有192.168.0.101这台主机负责解析子域

1. 在负责解析tianfeng.com域的资源记录中，添加如下两行

   ```
   opt.tianfeng.com	IN	NS	ns1.opt.tianfeng.com
   ns1.opt.tianfeng.com	IN	A	192.168.0.101
   ```

   **注意：在有主从服务器的情况下，任何时候修改资源记录文件，都必须更新序列号**

2. 在192.168.0.101主机上编辑配置文件/etc/named.rfc1912.conf文件，添加如下内容

   ```
   zone "opt.tianfeng.com" IN {
       type master;
       file "opt.tianfeng.com.zone";
   };
   ```

3. 创建/var/named/opt.tianfeng.com资源记录文件

   ```
   $TTL 3600
   @	IN	SOA	@	admin.opt.tianfeng.com (
   					2019032401
   					10M
   					3M
   					1D
   					1D )
   www	IN	A	192.168.220
   bbs	IN	A	192.168.221
   mx	IN	A	192.168.222
   ```

4. 检查语法，并重启主服务器，和子域服务器！

   这里又有一个非常非常非常值得注意的地方，务必要将主配置文件中的仅允许本机访问选项关闭，不然会出现子域服务器无法解析父域，父域服务器无法解析子域的问题，如下图所示

   ![bind主配置文件注意点](https://github.com/huangtianfeng/pictures/blob/master/linux%E5%9B%BE%E5%BA%93/%E6%8F%92%E5%9B%BE19-3.png)

   这个值修改成any，或者指定ip

#### bind中的安全相关配置

acl：访问控制列表： 

可以在主配置文件中，将一个或多个地址合并成为一个名称的集合，随后便可以通过该名称统一调用，类似于给变量赋值后调用变量名

定义格式：

```
acl acl_name {
    IP;
    网段/掩码长度；
}
```

bind中内置有四个acl，其实前面我们已经见过了

* none：空地址，没有任何主机的意思
* any；none的反义词，任何主机
* localhost：仅指代本机
* localnet：包括本机所在的ip所属的网络

常用的访问控制选项：

* allow-query {}:允许查询的主机，定义子域的时候，就需要修改此值
* allow-transfer {}:允许向哪些主机做区域传送，默认是所有，但是应该改为仅向从服务器传送，不然所有人都可以传送去，公司内部的拓扑图就很容易外泄，这很危险
* allow-recursion {}：允许哪些主机向当前DNS服务器发起递归查询请求（什么是递归，随后我就稍加解释，不保证正确性）
* allow-update {}：DDNS，允许动态更新区域数据库文件中的内容（这一条我暂时也不太懂，欢迎留言）

这些选项，可以在主配置文件中，也可以在区域配置文件中（/etc/named.rfc1912.conf)



最后还有一个bind view 视图功能，它的功能是解析分离，同一个主机名，根据来自不同ip的请求，解析出不同的结果；

例如：在/etc/named.rfc1912.conf 添加如下内容

```
view local {
	match-cliets {192.168.0.0/24;};
	zone "tianfeng.com" IN {
        type master;
        file "tianfeng.com/local";
	};
};
view net {
    match-cliets {any;};
    zone "tianfeng.com" IN {
        type master;
        file "tianfeng.com/net";
    };
};
```

然后创建两个不同的资源记录文件，local和net

这表示，来自192.168.0.0/24网络中的解析请求，将返回local数据库中的答案

否则，返回net文件中的答案

