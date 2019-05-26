[TOC]

# ngx_stream系列模块

首先看一段官网的配置示例

```
stream {
    upstream backend {
        hash $remote_addr consistent;

        server backend1.example.com:12345 weight=5;
        server 127.0.0.1:12345            max_fails=3 fail_timeout=30s;
        server unix:/tmp/backend3;
    }

    upstream dns {
       server 192.168.0.1:53535;
       server dns.example.com:53;
    }

    server {
        listen 12345;
        proxy_connect_timeout 1s;
        proxy_timeout 3s;
        proxy_pass backend;
    }

    server {
        listen 127.0.0.1:53 udp reuseport;
        proxy_timeout 20s;
        proxy_pass dns;
    }

    server {
        listen [::1]:12345;
        proxy_pass unix:/tmp/stream.socket;
    }
}
```

接下来介绍上述所涉及到的指令及相关模块；

## Module ngx_stream_core_module

* 常用指令：

  1. **stream** { ... }

     使用位置，独立于其他任何上下文，新建一个新的上下文，供其他相关指令使用

  2. **server** `{...}`

     使用位置：stream上下文中，注意，这里和http相关模块中的server不是同一个

     设置服务器相关配置

  3. **listen** `address:port`

     使用位置：server上下文中

     指定监听的地址和端口，可以使用*通配所有地址，例如listen *：80监听本机所有地址的80端口，默认是tcp端口，可以添加udp 指定udp端口，端口可用类似100-200形式监听一个范围

## Module ngx_stream_proxy_module

* 常用指令

  1. **proxy_pass** `address[:port]`;

     使用位置：server上下文中

     指定后端服务器地址和端口,地址也可以是由upstream定义的组名

  2. proxy_connect_timeout 1s;

     使用位置stream、server

     定义客户机与本机建立连接的超时时长

  3. proxy_timeout 3s;

     定义客户机与代理服务器多长时间内没有传数据，则断开连接

## Module ngx_stream_upstream_module

* 常用指令

  1. **upstream** `name` { ... }

     使用位置：stream上下文中

     定义一个服务器组，和ngx_http_upstream_module中的用法差不多
     
  2. server `HOSTNAME[:PORT]`
  
     使用位置：upstream上下文中，定义后端服务器，此server非彼server，和前面的server有所不同
  
     参数：
  
     > weight=number：指定服务器权重，不指定时，默认为1
     > max_fails=number：指定健康状态检测失败多少次后，摘除该服务器
     > fail_timeout=#s：指定健康状态检测请求的超时时长，多长时间未响应判定为失败
     > backup：指定服务器为备用服务器，其他所有服务器都出现故障才会启用，一般用于say sorry；
     > 其他常用参数
     > max_conns=number：指定服务器能够承载的最大并发连接数，nginx最多同时调度多少请求给服务器
     > down：认为关闭服务器，实现灰度发布时可以用到
  
  3. least_conn：指定调度算法为lc最少连接算法进行调度， 当server拥有不同的权重时，自动改为wlc加权最少连接算法进行调度，当所有后端主机的连接数相同时，则使用wrr进行调度
  
  4. ip_hash：源地址hash算法，绑定源地址，将同一个源ip地址的请求绑定至同一个upstream server
  
  5. hash key[consistent]：基于指定的key的hash表事先请求调度，此处的key可以是文本、变量或者二者的组合；其中consistent参数，指定使用**一致性的hash算法
  
     示例：
  
     ```
     hash $request_uri consistent
     hash $remote_addr
     hash $cookie_name
     ```
  
  6. keepalive CONNETTIONS：指定每个worker进程可以和后端服务器保持的长连接的数量
     示例
  
     ```
     upstream memcached_backend {
        server 127.0.0.1:11211;
        server 10.0.0.2:11211;
     
        keepalive 32;
     }
     ```
  
     
