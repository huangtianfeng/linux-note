[TOC]

# Nginx实现反向代理

正向代理，即代理客户端向服务器发起请求，类似于iptables中的源地址转换

反向代理：即代理服务器接收客户端的请求，类似于iptables中的目标地址转换

我们知道，DNAT 和SNAT都是工作在iso七层模型中的传输层，通过修改报文的IP地址和端口号实现的

而Nginx是一个应用层的协议，所以它工作于osi模型的应用层，而非传输层；因此它能实现的的功能，相较于前者而言要丰富的多，但是同时，效率要低一些！ 很容易想明白，层次越高，所需要操作的步骤越多，所以层次越低效率越高，层次越高效率月底。

Nginx和DNAT/SNAT的区别还有，DNAT/SNAT转发时，仅修改了传输层和网络层的报文内容，地址，其他都没变，相当于ABC三个人对话，B在中间仅发挥传达的作用，而Nginx在代理的时候，是可以对报文的主体内容进行修改的，相当于ABC三人对话，Nginx表演B的角色，他夹在A和C中间，做一个中间人，它的作用就不仅仅是传达了，它需要按照自己的理解，将一方说的话，转述给另外 一方。

实现起来非常容易，只需要在配置文件中添加如下指令即可

```
server{
	listen 80;
	server_name www.tian.com;
	location / {
		proxy_pass http://172.16.0.100:80
	}
}
```



proxy_pass指令由ngx_http_proxy_module模块提供

## ngx_http_proxy_module模块

1. proxy_pass

   格式：**proxy_pass** URL;

   用于将请求转发至后端服务器，或者说，又nginx代为向后端服务器发起请求，而后得到后端服务器响应的内容，再将其响应给客户端

   如上所述，用法很简单，不过值得注意的一点是，

   如果在主机名后面添加路径，那么被匹配到的路径，会被替换为指定的路径，例如

   ```
   location /tian/ {
   	proxy_pass http://172.16.0.100:80/test/
   }
   ```

   那么，在访问/tian/index.html这个资源时，会请求到172.16.0.100主机的/test/index.html的内容，不知道这样表达的够不够清楚，就是该location的路径会被替换为指定的路径！

   如果proxy_pass指令后面，不指定路径，那么location的路径会被原封不同的传递给后端主机。

   所以，当使用正贼表达式匹配路径时，就不能再该指令中指定路径！

   ```
   location ~* \.(jpg|ipeg) {
   	proxy_pass http://172.16.0.100/test/
   }
   ```

   上述这样的行为是不被允许的。

2. proxy_set_header

   格式： roxy_set_header  field*` `*value

   用于修改客户端发往服务器端的报文首部（nginx收到客户端请求后，转发给后端服务器的那段报文）

   常用用法：

   proxy_set_header X-Real-IP $remote_addr：将客户单真实地址添加进请求报文，默认情况下，客户端对服务器主机时不可见的，服务器主机只知道代理服务器的ip地址信息，所以，如果不加该参数，在服务器主机上，无法分析ip地址相关信息

   proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for；该指令会创建一个X-Forwarded-For的首部，并将响应报文的源地址添加进去（这里之所以用的不是后端服务器主机地址，是因为，代理服务器可能不止一个，所以，该首部的值，也可能是一长串地址）

## ngx_http_headers_module模块

1. add_header 

   语法：add_header name value [always]

   用于给发往客户端的响应报文添加自定义首部，或者修改指定首部的值

   示例：

   ```
   add__header X-Via $server_addr;
   add_header X-Accel $server_name;
   ```

   

# nginx的缓存组织管理方式

nginx作为代理服务器时，可以打开缓存功能，将后端服务器的部分内容缓存到本地，当一个资源被重复请求时，直接由代理服务器响应，不必转发至后端服务器，从而在降低后端服务器的负载的同时，提升用户的体验（响应速度变快）

nginx的缓存分为两个部分：

* 一部分为存放在内存中的hash表，它保存了url字符串所对应的hash值，当一个新的请求到来时，通过查询该表，以判断本地是否有缓存，有则直接在本地磁盘中取出缓存内容，构建响应报文，没有则转发至后端服务，得到响应后，将内容缓存下来
* 一部分存放在磁盘中，通过hash码命名来组织一个至多三级的子目录，从右至作截取hash码，hash码是十六进制的，因此，一级子目录，截取一个hash码时，则能够得到十六个子目录（目录时按需创建的，不是一次性全部创建出来），二级子目录截取一个hash码，再得到16个子目录，此时就有256个子目录了，如果三级子目录再截取两个hash码，将得到65536个子目录，这个数目已经足够存放缓存内容了，即使少了，也可以增加截取的hash码！

缓存空间需要先定义后使用，需要定义好内存中哪段内存存放hash表，磁盘的哪个位置存放缓存内容

## ngx_http_proxy_module模块

> * **proxy_cache_path** `path [levels=levels] [`use_temp_path`=`on`|`off`] `keys_zone`=`*name*`:`*size*` [`inactive`=`*time*`] [`max_size`=`*size*`] [`manager_files`=`*number*`] [`manager_sleep`=`*time*`] [`manager_threshold`=`*time*`] [`loader_files`=`*number*`] [`loader_sleep`=`*time*`] [`loader_threshold`=`*time*`] [`purger`=`on`|`off`] [`purger_files`=`*number*`] [`purger_sleep`=`*time*`] [`purger_threshold`=`*time*`];
>
> levels=1:1:1	指定磁盘目录的组织等级
>
> keys_zone=pcache：缓存名字，调用时使用的名字
>
> * **proxy_cache** *zone* | off;  调用缓存功能
> * **proxy_cache_key** *string*; 指定内存中hash表的键值
>   默认proxy_cache_key $scheme$proxy_host$request_uri;
> * **proxy_cache_methods** `GET` | `HEAD` | `POST` ...;指定查找缓存的请求方法，默认为GET和HEAD
> * **proxy_cache_valid** [code ...] time;定义响应码需要缓存的时间
> * **proxy_cache_use_stale** `error` | `timeout` | `invalid_header` | `updating` | `http_500` | `http_502` | `http_503` |`http_504` | `http_403` | `http_404` | `http_429` | `off` ...;定义后端服务器为什么状态时，可以使用缓存响应，默认为off，表示服务器不正常时，不能使用缓存响应
> * **proxy_connect_timeout** `*time*`;定义面向服务器一侧的连接超时时间，默认60，规定不能超过75秒
> * **proxy_read_timeout** *time*;定义两次重传之间的间隔时间，大概就是接受请求的超时时间
> * **proxy_send_timeout** *time：*向服务器端发送请求的超时时长
>

## ngx_http_upstream_module模块

用于将后端服务器分组，以实现反向代理的同时实现负载均衡

常用指令：

1. upstream name {...}

   定义后端服务器组；引入一个新的上下文，只能用于http{}上下文中；其默认的调度方法是wrr;

   Example：

   ```
   upstream backend {
       server backend1.example.com weight=5;
       server 127.0.0.1:8080       max_fails=3 fail_timeout=30s;
       server unix:/tmp/backend3;
   
       server backup1.example.com  backup;
   }
   
   其中：
   server为upstream上下文中的指令，用于定义服务器地址和相关的参数
   地址格式为：
   IP[:PORT]
   HOSTNAME[:PORT]
   UNIX:/PATH/TO/SOME_SOCK_FILE
   参数：
   weight=number：指定服务器权重，不指定时，默认为1
   max_fails=number：指定健康状态检测失败多少次后，摘除该服务器
   fail_timeout=#s：指定健康状态检测请求的超时时长，多长时间未响应判定为失败
   backup：指定服务器为备用服务器，其他所有服务器都出现故障才会启用，一般用于say sorry；
   其他常用参数
   max_conns=number：指定服务器能够承载的最大并发连接数，nginx最多同时调度多少请求给服务器
   down：认为关闭服务器，实现灰度发布时可以用到
   ```

   upstream中的其他常用指令

   > * least_conn：指定调度算法为lc最少连接算法进行调度， 当server拥有不同的权重时，自动改为wlc加权最少连接算法进行调度，当所有后端主机的连接数相同时，则使用wrr进行调度
   >
   > * ip_hash：源地址hash算法，绑定源地址，将同一个源ip地址的请求绑定至同一个upstream server
   >
   > * hash key[consistent]：基于指定的key的hash表事先请求调度，此处的key可以是文本、变量或者二者的组合；其中consistent参数，指定使用**一致性的hash算法**
   >
   >   示例：
   >
   >   hash $request_uri consistent
   >
   >   hash $remote_addr
   >
   >   hash $cookie_name
   >
   > * keepalive CONNETTIONS：指定每个worker进程可以和后端服务器保持的长连接的数量
   >   示例
   >
   >   ```
   >   upstream memcached_backend {
   >       server 127.0.0.1:11211;
   >       server 10.0.0.2:11211;
   >   
   >       keepalive 32;
   >   }
   >   ```

   * 一致性的hash算法

     当我们基于hash算法进行负载均衡调度时，其实可以不用查表就直接调度，使用的方法是，将hash值对一个固定的数进行取模运算，一般这个数为2^32，计算后，得到的值会在0到232-1之间的任意数，我们将这个区间的值，抽象成为一个封闭的圆环，每个值对应圆环上的一个点，然后将所有后端服务器的ip地址的hash值对2^32取模，得到圆环上的一个点，而后，在客户端发起请求时，对请求报文的某个键值也同样的取得hash值后对2^32进行取模，得到一个圆环上的点，而后按照顺时钟方向寻找最近一台服务器，该服务器就是需要被调度到的服务器。而为了尽量使得服务器在圆环上的位置均匀分布，很多时候，我们需要给服务ip地址做hash计算时，加上一下随机数，计算出多个虚拟位置，而非单个虚拟位置。

     

