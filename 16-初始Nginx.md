[TOC]



# NGINX介绍

是一个master/worker模型，由主进程生成子进程，子进程负责响应用户请求

master的作用

1. 装载配置文件
2. 启动和管理workers进程
3. 非停机更新（平滑升级），用户请求不终端就能将其版本升级

worker进程负责处理和响应用户请求

缓存相关的进程：

> Cache loader：载入缓存对象
>
> Cache manager：管理缓存对象

Nginx的特性：

异步，时间驱动和非阻塞

并发请求处理（网络IO）：通过kevent/epoll/select,/dev/poll等机制实现

文件IO：高级IO sendfile，异步，mmap（内存映射）等机制

Nginx是高度模块化的，主程序只有一个核心框架，其他功能都是由模块实现，近期版本才支持动态装载和卸载

模块分类：

> 核心模块：core module
>
> 标准模块：
>
> > HTTP modules:
> >
> > > standart HTTP modules
> > >
> > > optional HTTP modules
> >
> > Mail modules
> >
> > Stream modules
> >
> > 传输层代理
>
> 3rd party modules：第三方模块

nginx的公用：

1. 静态的web资源服务器；（图片服务器，或js/css/html/txt等静态资源服务器）
2. 结合FastCGI/uwSGI/SCGI等协议反代动态资源请求，php不能模块化的编译进Nginx，只能将其工作为fpm
3. http/https协议的反向代理
4. imap4/pop3

# nginx的安装

* 创建nginx的yum仓库，指向nginx的官网仓库，然后安装

* 或者使用epel源中的nginx

* 源码编译安装： 

  1. 先装上Development Tools和Server Platform Development两个包组
  2. 安装pcre-devel，openssl0devel，zlib-devel等相关依赖包
  3. 添加nginx用户
  4. ./configure .....
  5. make && make install

  

  配置文件的组成部分：

* 主配置文件：/etc/nginx/nginx.conf

>  include conf.d/*.conf
>
>   fastcgi,uwsgi,scgi等协议相关的配置文件
>
>   mime.types:支持mime类型

* 主程序文件：/usr/sbin/nginx

  Unit File :nginx.service

主配置文件的配置指令：

directive value [value2...];

注意：指令必须以分号结尾，支持使用配置变量，变量又分为内建变量和自定义变量，内建变量由Nginx模块引入，可直接应用，官方documentation有说面，自定义变量可由用户使用set命令定义，格式为set variable_name value;	引用格式为$variable_name

* 主配置文件结构：

  main block：主配置段，也即全局配置段：

  event{...}：事件驱动相关的配置；

  http{...}：http/https洗衣相关的配置段；

  mail{...}

  stream{...}

http协议相关的配置结构

nginx中没有中心主机的概念，只有虚拟主机！

```
http{
...
...:各server的公共配置
server{...}:每个server用于定义一个虚拟主机；
	server{
		...
		listen
		server_name
		root
		alias
		location [OPERATOR] URL{	nginx中，不支持文件系统路径定义，只能基于URL定义
			...
			if CONDITION{...}
						}
		}
	}
```



创建一个虚拟主机：

1. 提供网页目录，网页文件

2.  在/etc/nginx/conf.d/目录下创建一个文件，添加如下指令

   ```
   server{
   	listen 80;
   	server_name www.ilinux.io;
   	root /data/nginx/vhost1;
   }
   ```

3. 使用nginx -t 测试语法



## 配置指令

### main配置段常见的配置指令

#### 正常运行必备的配置：

1. user：定义以谁的身份运行进程
   语法：user USER [GROUP]

2. pid /PATH/TO/PID_FILE；

   指定存储nginx主进程好嘛的文件路径；

3. include file|mask（通配符）；

   指明包含进来的其他配置文件片段；

4. load_module file;

   指明要装载的动态模块

#### 性能优化相关的配置

1. worker_processes number | auto；

   worker进程的数量；通常应该小于当前主机的CPU核心数；auto表示自动设置为当前物理CPU核心数对应的worker数

   一般该数量设置为CPU核心数-1 最佳！当进程数多于CPU核心数时，会有副作用！

   可以将进程绑定到CPU核心上，避免互相切换，提高缓存命中率

2. worker_cpu_affinity cpumask ...;

   worker_cpu_affinity auto：

   也可以使用cpumask指明要绑定的核心

   CPU mask： 假设设备有八颗CPU，那么使用八位二进制表示，从右至左第一位为1时，表示0号核心，第二位为1，表示1号核心...依次类推！

   0号 00000001

   1号 00000010

   2号 00000100

   ...

   例如： 假设我们机器上有八个cpu核心上，并启动四个worker，使用umask的方式将其绑定到后面四颗CPU上

   worker_cpu_affinity 00010000 00100000 01000000 10000000; 即可！

   可以使用ab进行压测，使用watch -n.5 'ps axo comm,pid,psr | grep nginx' 动态查看worker所在的cpu核心

3. worker_priority number;

   指定worker进程的nice值，设定worker进程优先级，取值范围-20,20

4. worker_rlimit_nofile number;

   所有worker进程所能打开的文件数量上限；其值应该大于worker_processes和events中 worker_connections两者的乘积

#### 调试，定位问题的配置

1. daemon on | off

   是否以守护进程方式运行Nginx

2. master_process on | off

   是否以master/worker模型运行nginx；默认为on，如果是off的话，将只启动一个master进程，进行调试

3. error_log file [level]

   指明日志位置，不是使用rsyslog管理的！

事件驱动相关的配置，也属于全局配置段

events{...}

1. worker_connections number;

   每个worker进程所能打开的最大并发连接数量

   其值应该为：worker_processes * worker_connections

2. use method

   指明并发连接请求的处理方法：

   use epoll；

3. accept_mutex on | off

   处理新的链接请求的方法；on以为这由各worker轮流处理新请求，off意味着每个新请求的到达都会通知所有worker进程

### http 协议的相关配置：

#### 与套接字相关的配置

1. server{...}

   配置一个虚拟主机；

   server{

   ​	listen address [:PORT] | PORT；

   ​	server_name SERVER_NAME;

   ​	root  /PATH/TO/DOCUMENT_ROOT；

   }

2. listen PORT | address[:port] | unix:/PATH/TO/SOCKET_FILE

   listen address[:oprt] [default_server] [ssl] [http2 | spdy] [backlog=number] [rcvbuf=size] [sndbuf=size]

   其中：

   default_server：设定为默认虚拟主机

   ssl：限制进程通过ssl连接提供服务

   backlog=number：后援队列长度

   rcvbuf=size：接受缓冲区大小

   sndbuf=size：发送缓冲区大小

3. server_name name ...;

   指明虚拟主机的主机名称；后可跟多个由空白字符分割的字符串；

   支持*通配任意长度的任意字符；server_name *.magedu.com www.magedu.*

   支持~起始的字符做正则表达式模式匹配；sever_name ~^www\d+\\.magedu\\.com$

   匹配机制：（优先级）

   > 1. 首先是字符串精确匹配；
   > 2. 左侧*通配符；
   > 3. 右侧*通配符；
   > 4. 正则表达式

4. tcp_nodelay on | off;

   在keepalived模式下的链接时否启用TCP_NODELAY选项，只有保持连接时，才生效，当请求的资源非常小时，将多个资源合并在一起，一个报文承载多个文件，延迟到达

5. tcp_nopush on|off;

   将响应报文首部，和内容 合并打包成有一个报文发送

6. keepalive_timeout：保持连接的超时时间，默认76s

7. keepalive_requests: 保持连接内，最多接受的请求数，默认100

8. types_hash_max_size：设定保存hash表的最大条目，单位为项

9. sendfile on|off；是否启用sendfile

#### 定义路径相关的配置

1. root path；

   设置web资源路径映射；用于指明用户请求的url所对应的本地文件系统上的文档所在的目录路劲；可出现的位置为：http，server，location，if in location；

2. location [=|~|~*|^~] url {...}

   对url进行访问控制或者其他配置，在一个server中location配置段可以有多个，用于实现从url到文件系统的路径映射；ngnix会根据用户请求的URL来检查定义的所有location，并找出一个最佳匹配，而后应用其配置

   > =：对URL做精确匹配；location / {...}和location=/{...} 的区别在于，前者能匹配以根起始的所有url，后者匹配的仅仅是根
   >
   > ~：对URL做正则表达式模式匹配，区分字符大小写；
   >
   > ~*：对URL做正则表达式匹配，不区分字符大小写
   >
   > ^~：对URL的左半部分做匹配检查，不区分字符大小写，也就是词首锚定
   >
   > 匹配优先级：=,^~,~,~*,不带符号；
   >
   > 同符号，匹配长度越长，优先级越高

   注意，在location中，定义root，会将全局配置中的root定义覆盖，例如，location ^~ /picture/ {root /tmp/}，该指令定义了，当用户访问/picture/目录时，将root切换至/tmp ,注意，只是将根切换为/tmp， 访问/picture/目录下的资源，会被映射为文件系统中/tmp/picture下的资源。理解这里的被映射的关系很重要

3. alias path；

   定义路径别名，文档映射的另一种机制，仅能用于location上下文；

   注意：location中使用root指令和alias指令的意义的不同之处

   1. root， 给定路径对应于location中的/url左侧的/；
   2. alias，给到你个的路径对应于location中的/url/右侧的/；

   加入吧前面的location ^~ /picture/ {root /tmp/}改为location ^~ /picture/ {alias /tmp/}，该指令将会在用户访问/picture/目录时，直接将其映射到/tmp目录下，而不是会在该目录下还存在picture目录

4. index file ...;

   定义默认资源，可用于http，server，location上下文中；

5. error_page code ... [=[response]] url;

   定义指定的状态返回码 需要显示的自定义页面，并且还可以使用=将原来的状态码重命名为其他状态码，达到用户意识不到这个页面不存在的目的，例如

   ```bash
   error_page 404 =200 /notfound.html;
   location = /notfound.html {
   	root /data/nginx/error_pages;
   }
   ```

   当用户访问一个状态码为404，不存在的页面时，nginx会将状态码改为200，并显示/notfound.html资源，后面的location定义了，当访问这个资源的时候，将根映射为另外一个用于保存自定义错误页面的路径，而不是去正常的资源路径下去找该错误页面！

#### 定义客户端请求的相关配置

1. **keepalive_timeout timeout[header_timeout];**

   设定保持连接的超市时长，0表示禁止场链接；默认值为75s

2. **keepalive_requests number；**

   在一次长连接上所允许请求的资源的最大数量，默认为100

3. keepalive_disable none | browser

   对哪种浏览器禁用长连接

4. send_timeout time；

   想客户端发送响应报文的超时时长，此处是指两次写操作之间的间隔时长

5. client_body_buffer_size size；

   用于接受客户端请求报文的body部分的缓冲区大小；默认为16k；超出此大小是，其将被暂存到磁盘上的有client_body_temp_path指令所定义的位置；

6. client_body_temp_path path [level1 [level2[level3]]];

   设定用于存储客户端请求报文的body部分的零食存储路径及子目录结构的数量；

   当一个网站有非常大量带body部分的请求报文需要处理时，会使用哈希计算出每个报文的md5值，暂存于磁盘上，在磁盘上，使用十六进制数命名，组织处一个三级的目录结构，便于检索查找，MD5的单项加密值都是由16进制数组成，所以，我们根据其末尾的十六进制数进行分类组织，例如

   client_body_temp_path /var/tmp/client_body 2 1 1

   该指令表示在/var/tmp/client_body 目录下，创建一个层级结构，其中

   2 表示用2位十六进制数表示一级子目录 00-ff

   1 表示用1位十六进制数表示二级子目录 0-f

   1 表示用以为十六进制数表示三级子目录  0-f

   而后根据报文的body部分的hash值，将其对应存储到三级目录中

#### 对客户端进行限制的相关配置

1. limit_rate rate;

   限制响应给客户端的传输速率，单位是bytes/second，0表示无限制；

2. limit_except method ... {...}

   限制对指定的请求方法之外的其他方法的使用客户端；

   例如：限制除了GET以外的请求方法，都只能在本地局域网段中使用

   limit_except GET{

   ​	allow 192.168.1.0/24;

   ​	deny all;

   }

#### 文件操作优化的配置

1. aio on | off | threads[=pool];

   是否启用aio功能

2. directio size | off

   在linux主机启用O_DIRECT标记，此处以为文件大小等于给定的大小时使用，例如，directio 4m；

3. open_file_cache off；

   open_file_cache max=N [inactive=time];

   nginx可以缓存一下三种信息；

   a. 文件的描述符、文件大小和最近一次的修改时间；

   b. 打开的目录结构；

   c. 没有找到的或者没有权限访问的文件的相关信息

   max=N：可缓存的缓存项上限，达到上限后会使用LRU算法实现缓存管理

   inactive=time：缓存项的非活动时长，在此处指定的时长内，未被命中的或者或命中的次数少于open_file_cache_min_usrs指令所指定的次数的缓存项即为非活动项

4. open_file_cache_valid time；

   缓存项有效性的检查频率，默认为60s；

5. open_file_cache_min_uses number；

   在open_file_cache指令的inactive参数指定的时长内，至少应该被命中多少次方可被归类为活动向；

6. open_file_cache_errors on | of；

   是否缓存查找时发生错误的文件一类的信息；类似于DNS的缓存否定答案

### 各常见模块配置

#### ngx_http_access_module模块

实现基于ip地址的访问控制功能

1. allow address | CIDR | unix:| all;
2. deny address | CIDR | unix: | all;

可在http，server，location，limit_except上下文中使用

#### ngx_http_auth_basic_module模块

实现基于用户的访问控制，使用basic机制进行用户认证

1. auth_basic string | off；

2. auth_basic_user_file file；

   示例：

   ```
   location /admin/{
   	alias /webapps/app1/data/;
   	auth_basic "Admin Area";
   	auth_basic_user_file /etc/nginx/.ngxpasswd;
   }
   ```

   注意：htpasswd命令有httpd-tools包提供

#### ngx_http_stub_status_module模块

用于输出nginx的基本状态信息；

1. stub_status;

   location /ngxstatus {stub_status;} 

   即可通过/ngxstatus访问，当然还需要做访问认证

   输出信息的意义

> Active connections：表示当前的活动连接数
>
> accepts：已经接受的客户端请求的总数
>
> handled：已经处理完成的客户端请求的总数
>
> requests：客户端发来的总的请求书；
>
> Reading：处于读取客户端请求报文首部的连接数
>
> Writing：处于想客户端发送响应报文过程中的连接数；
>
> Waiting：处于等待客户端发出请求的空闲连接数；

#### ngx_http_log_module模块

定义日志格式

1. log_format name string ...；自定义格式

   string可以使用nginx核心模块及其他模块内嵌的变量

2. access_log path [format [buffer=size] [gzip[=level]] [flush=time] [if=condition]];

   access_log off

   访问日志文件路径，格式及相关的缓冲的配置；该指令可以为每个location自定义单独的配置文件

3. open_log_file_cache max=N [inactive=time] [min_users=N] [valid=time]；

   open_log_file_cache off;

   缓存日志文件相关的元数据信息

   max：缓存的最大文件描述符数量

   min_users：在inactive指定的时长内访问大于等于此值方可被当做活动项；

   inactive：非活动时长；

   valid：验证缓存中各缓存项是否为活动项的时间间隔；

#### ngx_http_ssl_module模块

1. ssl on | off；激活https 协议

2. ssl_certificate file;

   当前虚拟主机使用PEM格式的证书文件；

3. ssl_certificate_key file；

   当前虚拟主机上与其证书匹配的私钥文件；

4. ssl_protocols [SSLv2] [SSLv3] [TLSv1] [TLSv1.1] [TLSv1.2];

   支持的ssl协议版本，默认为后面三个；

5. ssl_session_cache off | none | [builtin[:size]] [shared:name:size];

   builtin[:size]：表示使用Opensll内建的缓存，此缓存为么个worker进程私有，也就意味着，当一个请求请求的数据缓存在worker1中后，同样的请求发送给worker2 将没有加速的效果，所以，建议使用共享缓存

   [shared:name:size]：在各worker之间使用一个共享的缓存，该缓存1M大小，即可缓存四千条记录

6. ssl_session_timeout time；

   客户端一侧的链接可以服用ssl session cache中缓存的ssl参数的有效时长；

配置示例：

```
server{
	linsten 443 ssl;
	server_name www.tianfeng.com
	root /vhosts/ssl/htdocs;
	ssl on;
	ssl_certificate /etc/nginx/ssl/nginx.crt;
	ssl_certificate_key /etc/nginx/ssl/nginx.key;
	ssl_session_cache shared:sslcache:20m;
}
```

#### ngx_http_rewrite_module模块

url（重写）重定向；本质上就是查找替换，将用户请求的URL基于regex所描述的模式进行匹配检查，而后完成替换；

1. rewrite regex replacement [flag]

   将用户请求的URL基于regex所描述的模式进行检查，匹配到时将其替换文replacement指定的新的URL；

   示例：

   ```
   rewrite /(.*)\.png$ http://www.ilinux.io/$1.jpg
   rewrite /(.*)$ https://www.ilinux.io/$1;
   ```

   

   注意：如果同一级配置快中存在多个rewrite规则，name会自上而下逐个检查；被某条规则替换完成后，会再次重新自上而下逐个检查，一次，隐含有循环机制；[flag]所表示的标志用于控制此循环机制；

   flag：

   > last：重写完成后停止对当前URL所在l的ocation中后续的其他ngx_http_rewrite_module模块的的重写操作指令进行匹配，执行后续的其他指令
   >
   > break：重写完成后，跳出当前ngx_http+rewrite_module模块的所有操作
   >
   > redirect：重写完成后，以临时重定向方式直接返回重写后生成的新的URL给客户端，由客户端重新发起请求；
   >
   > permanent：重写完成后，以永久重定向方式直接返回重写后申城的新URL给客户端，由客户端重新发起新请求；

   2. return

      停止进程并返回一个状态码

      return code [text];

      return code URL;

      return URL; 

   3. if （condition）{...}

      引入一个新的配置上下文；条件满足时，执行配置快中的配置指令；

      可以在server，location上下文中使用

      condition： 

      比较操作符：

      > ==
      >
      > !=
      >
      > ~:模式匹配，区分字符大小写；
      >
      > ~*：模式匹配，不区分字符大小写；
      >
      > !~：模式不匹配，区分字符大小写
      >
      > !~*：模式不匹配，不区分只读大小写

      文件即目录存在性判断：

      > -e,!-e
      >
      > -f,!-f
      >
      > -d,!-d
      >
      > -x,!-x

   4. rewrite_log  on | off;

      是否开启重写日志；（如果发生重写，是否记录到日志中）

   5. set $variable value;

      用户自定义变量

   ### ngx_http_referer_module模块

   防盗链

   1. valid_referers none | blocked | server_names | string ...;

      定义referer首部的合法可用值；

      none：请求报文首部没有referer首部

      blocked：请求报文的referer首部没有值

      server_name：参数，其可以有值作为主机名或主机名模式；

      ​	arbitrary——string: 直接字符串，但可以使用*做通配符；

      ​	regular expression： 被指定的正则表达式模式匹配到的字符串；要使用~打头，例如~.*\\.tianfeng\\.com;

      配置示例：

      ```
      valid_referers none block server_names *.magedu.com *.mageedu.com magedu.* mageedu.* ~\.magedu\.;
      if($invalid_referer){
      	return 403
      }
      ```

return还可以返回一个url，返回一个主页，或者指定图片资源等

