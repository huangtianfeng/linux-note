[TOC]

# http协议介绍

http协议，是构建在TCP/IP协议之上的一个无连接状态的应用层协议

它基于c/s架构，分为服务端和客户端

客户端就是我们常见的各种浏览器，例如google，Firefox等……

而服务端，有许多实现方式，httpd就是其中的一种

* 一次完整的http请求处理过程

  1. 建立或处理连接：接受或拒绝请求
  2. 接受请求
  3. 处理请求：对请求报文解析，获取客户端请求的资源及请求方法等相关信息
  4. 访问资源：获取请求报文中请求的资源
  5. 构建响应报文
  6. 发送响应报文
  7. 记录日志

* 接受请求的模型：

  并发访问响应的模型：

  1. 单进程I/O模型：启动一个进程处理用户请求；这意味着一次只能处理一个请求，多个请求被串行响应
  2. 多进程I/O模型：并行启动多个进程，每个进程响应一个请求
  3. 复用的I/O模型：一个进程响应n个请求；
     1. 多线程模式：一个进程生成n个线程，一个线程处理一个请求；
     2. 时间驱动（event-driven）；一个进程直接响应n个请求；
  4. 复用的多进程I/O模型：启动多个进程，每个进程生成n个线程；

# httpd程序的安装和配置

AFS:著名的开源组织，旗下产品众多，httpd就是其中之一

httpd文档：httpd.apache.org

主程序包：httpd

工具包：httd-tools

帮助文档：httpd-manual（安装该程序后，会在/etc/httpd/conf.d/目录下创建一个配置文件，定义了访问帮助手册的URL）

httpd功能特性：CGI（common gateway interface）公共网关接口、虚拟主机、反向代理、负载均衡、路径别名、丰富的用户认证机制、支持第三方模块等等

httpd安装好后，就可以直接启动使用了，当然，这知识最原始的状态！ 想要丰富的功能，还需要通过自己修改定义配置文件，修改配置文件之前，建议先将配置文件备份一份！

### 配置文件位置：

核心配置文件：/etc/httpd/conf/httpd.conf

附加配置文件：/etc/httpd/conf.d/*.conf

模块相关的配置文件：/etc/httpd/conf.modules.d/*.conf（2.2的模块配置都在核心配置文件当中定义）

模块文件路径：/usr/lib64/http/modules

* 对于httpd2.2而言，配置文件分为三段：分别是

  section1: Global Environment 定义全局环境配置

  section2：Main Server Configuration中心主机配置

  section3：Virtual Host 虚拟主机配置

  section2和section3不能并存，后面会介绍到的

* 对于httpd2.4而言，没有严格的区分

## 常用配置介绍

配置文件中，主要由 指令+指令值的格式进行定义，指令有许多许多，只需要掌握一些常用的指令即可，官方文档中对指令都有详细的说明介绍，需要的时候可以进行查阅

### 端口监听

在主配置文件中，使用Listen IP:PORT定义即可，IP地址可以胜率，表示监听本机全部ip

注意：在不影响主配置文件的前提下，自定义的配置尽量在/etc/httpd/conf.d/目录下新建配置文件进行配置

### 配置块

所谓的配置快，就是用一个标签，将一段配置内容包裹封装起来，被封装起来的内容，仅对配置快定义的区域生效，不在配置快中的指令，将全局有效！

例如：

> <IfModule worker.c>
> StartServers         4
> MaxClients         300
> MinSpareThreads     25
> MaxSpareThreads     75
> ThreadsPerChild     25
> MaxRequestsPerChild  0
> </IfModule>

这就是一个配置快，至于它表示什么意思，且别着急，后文会有详细介绍

这样的配置快，有时候我们也称之为容器

### 保持连接设定

我们知道，http协议，是建立在TCP/IP协议之上的一个应用层协议，这意味着，客户端和服务器端之间的通信，需要经过TCP/IP协议的三次握手建立连接，和四次挥手，断开连接，而http协议，是没有链接状态的，它的一次请求和响应构成一个事物，如果每次请求和响应，都需要进行tcp协议的建立链接断开链接，未免也太麻烦，太耗时了，所以为了避免频繁建立，断开连接，我们通过所谓的保持连接（建立链接，请求一个资源完毕后，不断开链接），而是基于连接时长和请求资源数两个维度来进行限制，列入，规定，一次tcp链接后，最多可以保持连接多长时间，在这个时间段内，最多可以请求多少个资源，两个条件任意一个满足，则断开连接！

* 使用指令

  KeepAlive  On|Off： 定义保持连接功能是否开启，默认是关闭的

  KeepAliveTimeout  15：定义保持连接的最长时间

  MaxKeepAliveRequest 100：定义最大请求资源数

**当修改完配置文件后，最好使用httpd -t 命令来检查一下配置文件的语法是否有错误** 

**注意：httpd2.4的KeepAliveTimeout的值，可以是毫秒级别的，加上ms单位即可，默认单位是秒** 

### MPM

Multi-Processing Modules 多路处理模块之意

它定义了httpd服务程序响应客户端请求的方式

* prefork：二级模型，主程序预先创建多个子进程，每个子进程响应一个请求
* worker：三级模型，主程序创建多个子进程，子进程中又包含多个线程，每个线程响应一个请求
* event：二级模型，主程序创建多个子进程，每个子进程基于事件驱动，响应多个请求，一般用不上！

该模块有三个，对应三种响应模型

对于httpd2.2而言，MPM模块被静态编译进主程序，如果要切换模块，编辑/etc/sysconfi/httpd配置文件，将对应的变量启用后重启服务即可（修改默认的主程序），默认是prefork

对于httpd2.4而言，MPM模块是可以被动态装载的，如果需要切换模块，编辑/etc/httpd/conf.modules.d/00-mpm.conf文件，修改LoadModule将对应的模块装载进来即可

#### prefork模块的配置

对于2.4而言，默认配置文件中么有各模块的配置指令，应该是由默认值的！

如果想配置，可以在/etc/httpd/conf.modules.d/00-mpm.conf文件下，修改模块后写一个配置块

> <IfModule mpm_prefork_module>
> StartServers       8	启动后创建多少个空闲子进程
> MinSpareServers    5	最小空闲子进程数
> MaxSpareServers   20	最大空闲子进程数
> ServerLimit      256	允许处于活跃状态的最大进程数
> MaxClients       256	最大允许启动的子进程数
> MaxRequestsPerChild  4000	每个子进程最多响应多少个请求后被杀死
> </IfModule>

对于2.2而言，指令是一样的，但是在/etc/httpd/conf/httpd.conf文件中，默认会有配置块，只需修改即可

#### worker模块的配置

同上，2.4需要自行添加，2.2只需要修改

> <IfModule worker.c>
> StartServers         4	启动后创建多少个子进程
> MaxClients         300	最大允许活动的线程数
> MinSpareThreads     25	最小空闲线程数
> MaxSpareThreads     75	最大空闲线程数
> ThreadsPerChild     25	每个进程的线程数
> MaxRequestsPerChild  0	每个线程最大响应多少个请求后杀死，0表示没限制
> </IfModule>

有意思的是，2.2的默认值如上所示，启动后创建四个进程，每个进程拥有25个线程，也就是说一共创建100个线程，但是最大空闲线程数却只有75，所以，启动完后，还会杀掉一个进程！   奇葩的设定！ 

#### event模块的配置

基本没人用，就不介绍了！

以上三种模块，用的最多的还是prefork模块了！

### 定义Main Server和virtual Server

主服务器和虚拟服务器，又称之为中心主机和虚拟主机

中心主机，单台物理主机仅能为单个站点提供服务

虚拟主机，单台物理主机可以同时为多个站点提供服务

#### 中心主机

* 中心主机的定义

  我理解的中心主机，就是配置文件中，所有的指令，都是用来定义中心主机的！ **最重要的指令是DocumentRoot**:指定URL的起始位置

  ServerName： 标识主机用来服务于谁，未定义时，会试图反解本地主机名，解析失败会报错，但是并不影响服务的正常启动

  当然，还包括其他各种模块加载，访问控制配置快等等等等

* 中心主机的配置

  未启用虚拟主机时，配置文件中的所有配置都属于中心主机的配置

#### 虚拟主机

* 虚拟主机的定义

  对于2.2来说，虚拟主机不能和中心主机混用，所以，定义了虚拟主机，需要将中心主机禁用，禁用的方法是，将中心主机的DocumentRoot指令注释掉！2.4就没有这个要求了，好像启用虚拟主机后，中心主机自动失效！

  虚拟主机，都使用配置块封装起来！未被封装起来的指令，将对所有虚拟主机生效（不知道有没有理解错）

* 虚拟主机的配置方法：

  每个虚拟主机都定义在一个配置快中，在中心主机中的所有指令，都可以在虚拟主机中使用

  格式大体如下：

  > <VirtualHost IP:PORT>
  >
  > ​	ServerName 主机名（FQDN）
  >
  > ​	DocumentRoot "URL起始路径"
  >
  > ​		<Directory "URL起始路径">	访问控制
  >
  > ​			Options None
  >
  > ​			AllowOverride None
  >
  > ​			Require all granted	允许所有人访问
  >
  > ​		</Directory>
  >
  > ​	CustomLog "日志存储相对路径" combined   
  >
  > </VirtualHost>	

  注意：日志设置中的combined是日志格式，需要提前定义，存储路径是相对于ServerRoot指令指定的路径

* 虚拟主机的区分方式

  由于虚拟主机都在同一台主机上，所以在访问时，我们需要加以区分，区分的方式有三种，基于ip，基于端口，基于（主机名）FQDN

  基于ip时，需要给网卡配置多个ip，公网ip的代价还是挺大的，用的少

  基于端口时：httpd的默认端口时80，而其他非众所周知的端口，访问时需要自行指定，也很麻烦，基本也不会有送到

  基于主机名：也就是基于FQDN（根据请求的报文，判断FQDN）的方式，是最常见，也是最方便的！

  ** 注意：httpd-2.2使用基于FQDN的虚拟主机时，需要实现定义如下指令

  > NameVirtualHost IP:PORT
  >
  > 表示在指定的ip端口下，才能使用基于FQDN的虚拟主机
  >
  > 2.4没这个限制

虚拟主机基于不同的区分方式，可以混合用，直接网上加就可以了！



### 站点访问控制

访问控制的对象可以是目录，文件以及URL路径

针对目录使用如下配置块封装

> <Directory "">
>
> …………
>
> </Directory>
>
> 或者
>
> <Directory Match"">
>
> …………
>
> </DirectoryMatch>

* 该配置块中，Options指令的常用值（该选项定义了目录下的资源被访问时的特性

  > all：启用除了MultiViews属性的所有选项
  >
  > indexes：允许索引，这表示没有主页时，直接将该目录下的所有文件列出来，这是很危险的
  >
  > FollowSymLinks：默认URL陆金霞有链接文件时，所指向的文件目录如果未被授权，则无法访问，使用该选项，能允许链接文件被直接访问
  >
  > none：表示没有任何特性

针对文件使用如下配置块封装

> <File "">	支持使用通配符
>
> …………
>
> </File >
>
> 或者
>
> <FileMatch "PATTERN">	支持使用正则表达式
>
> …………
>
> </FileMatch>

针对URL路径使用如下配置块封装

> <Location "">	支持使用通配符
>
> …………
>
> </Location>
>
> 或者
>
> <LocationMatch "PATTERN">	支持使用正则表达式
>
> …………
>
> </Location>

#### 基于源地址实现访问控制

* 对于2.2而言

  允许所有来源主机可访问，在对应的配置块中使用如下指令

  > Order allow，deny
  >
  > Allow from all

  拒绝所有

  > Order deny,allow
  >
  > Deny from all

  允许部分

  > Order allow,deny
  >
  > Deny from 192.168.0.107
  >
  > Allow from 192.168

  此处表示，拒绝192.168.0.107的ip访问，但是允许该网段内的其他用户访问

  注意：Order allow，deny，表示未被允许的，全部拒绝访问

  ​		Order deny，allow 则表示未被拒绝的，全部允许访问

* 对于2.4而言

  允许所有主机可访问

  > Require all granted

  拒绝所有主机访问

  > Require all denied

  允许部分

  > <Requireall>
  >
  > ​	Require not ip 192.168.0.107
  >
  > ​	Require ip 192.168
  >
  > </Requireall>

  **2.4版本的允许部分主机，需要用Requireall或者RequireAny配置段封装。**

  **RequireAny配置快，设置白名单，不能包含否认指令，httpd2.4协议中，默认没有被明确授权的，就表示拒绝访问！**

  **如果想允许部分用户访问的同时，还拒绝部分用户访问，那么需要用RequireAll配置块**

#### 基于用户的访问控制

http协议内置有一个协议认证的方式

客户端向服务器端发起请求后，服务器端会反悔一个状态码为401的认证窗口

服务器端填入账号密码后再次发起请求，认证通过后，才会将请求的资源响应给客户端

认证的密码格式有两种，一种是明文的basic，一种是消息摘要认证，使用单项加密算法，但是需要客户端浏览器支持才行，其实这两种认证方式用的都很少，网站一般都是表单认证，不是协议认证，这里只简单说一说

配置快

> <Directory "">
>
> ​	Options none
>
> ​	AllowOverride none
>
> ​	AuthType basic
>
> ​	AuthnName "string"
>
> ​	AuthUserFile "用户密码路径"
>
> ​	Require valid-user	也可以使用Require user 用户名：
>
> </Directory "">

用户密码文件创建

使用htpasswd命令：

语法：htpasswd [options] 用户名

常用选项：

-c 在没有密码文件时，在指定路径创建一个密码文件

-s 使用sha算法加密

-b 批量添加用户

-m 使用md5加密算法

-D 删除指定用户

##### 基于组账号进行认证

手动创建一个文件，定义组名和所包含的用户，格式如下

> 组名：用户名 1  用户名2  用户名3  …………

组名与用户名之间用冒号隔开，用户名之间使用空格隔开即可

而后定义配置块

> <Directory "">
>
> Options none
>
> AllowOverride none
>
> Authtype basic
>
> AuthName "string"
>
> AuthUserFile "文件路径"
>
> AuthGroupFile "文件路径"
>
> Require group 组名  组名  组名 ……
>
> </Derectory>

### 日志的设定方法

* 日志格式

  在httpd中，可以事先使用LogFormat指令，定义日志格式并指定格式名，而后设定日志保存路径时，在后面调用

  例如，在httpd默认的主配置文件中，有如下指令

  ```
   LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined
      LogFormat "%h %l %u %t \"%r\" %>s %b" common
  
  ```

  其中各格式符表示的含义如下

  | 格式符         | 含义                                                         |
  | -------------- | ------------------------------------------------------------ |
  | %h             | 客户端主机的IP地址                                           |
  | %l             | 远程登录的用户名，一般为-                                    |
  | %u             | 认证登录的用户名，没有则为-                                  |
  | %t             | 发起请求的时间                                               |
  | %r             | 请求报文的首行（请求方法，URL，协议版本                      |
  | %s             | 状态码，2开头表成功，4开头表客户端出错，5开头表服务端出错，加>号，表示最终请求的状态，而非第一次请求的状态码 |
  | %b             | 响应报文的大小（不包括报文首部）单位字节                     |
  | %{referen}i    | 固定格式，Referen是请求报文中的一个值，表示从哪个页面引用过来的，末尾的i表示，记录该变量值 |
  | %{User-Agent}i | 客户端的浏览器类型                                           |
  | ...            | ...                                                          |

  最后面的combined和common则分别表示这两个格式的值！

* 错误日志的设定

  > ErrorLog "logs/error_log"

* 访问日志的设定

  >  CustomLog "logs/access_log" combined

  这里的combined就是前面定义的格式

### 定义路径别名

所谓定义路径别名，就是将一个URL和文件系统中的一个路径建立映射关系

一台主机定义好DocumentRoot后，想要访问该路径以外的文件，应该就只有使用链接文件和定义别名的方式了

别名的设定需要加载alias_modules, 而后按照以下格式设定别名即可

> alias	URL	文件路径

对于httpd2.4来说，权限是很严格的，所有没有明确授权访问的路径，皆会被拒绝

对于httpd2.2来说，就宽松很多了，只需要指定路径后就可以访问了！



### 设定默认字符集

AddDefaultCharSet	utf-8

utf字符集，是全球统一字符编码

常见的中文字符集有：GBK，GB2312，GB18030