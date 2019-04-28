php是通过服务器端脚本编程语言，主要 用于web开发以实现动态web页面，也是最早实现将脚本嵌入HTML源码文档中的服务器脚本语言之一

#  LAMP简介

所谓LAMP即Linux+Apache+MySQL+Php的组合！

Apache HTTP Server（简称Apache）是Apache软件基金会的一个开放源码的软件，也就是我们常说的httpd服务！

我们平常访问一个网页时，一般可以获取两种资源，一种是静态资源，一种是动态资源

* 静态资源：指的是不用经过额外处理，服务器直接响应给客户端的数据

* 动态资源：指的是一个程序文件，用户请求时，服务器端需要将程序载入内存中，运行一下，将得到的结果返回给服务器端！

其中静态资源，通过Apache服务就可以直接响应给客户端了，而动态资源，需要结合php解释器运行，当php程序需要访问数据库时，还需要和数据库进行交互！

php和httpd结合的两种方式！

1. 将php编译成为httpd的一个模块，只有动态资源交由php模块运行！静态资源，由httpd直接响应
2. fpm模式，将php独立出一个服务（FastCGI process manager：fpm)，httpd通过fastcgi协议，将客户端的php资源请求，转发给fpm服务器运行，运行完后返回运行结果给httpd，而后返回给客户端！





# php基于模块化的方式，安装LAMP

安装LAMP很简单！

1. 安装httpd，php，php-mysql，mysql-server

   其中httpd提供web页面，php提供httpd的php模块，php-mysql提供php程序连接mysql数据库的模块

2. 看一眼配置文件

   php程序包提供的文件

   ```bash
   [root@localhost ~]# rpm -ql php
   /etc/httpd/conf.d/php.conf
   /etc/httpd/conf.modules.d/10-php.conf
   /usr/lib64/httpd/modules/libphp5.so
   /usr/share/httpd/icons/php.gif
   /var/lib/php/session
   [root@localhost ~]# cat /etc/httpd/conf.d/php.conf | grep "^[^#]"
   <FilesMatch \.php$>
       SetHandler application/x-httpd-php	以.php结尾的文件，启动x-httpd-php这个程序处理，也就是调用libphp5.so模块！
   </FilesMatch>
   AddType text/html .php	添加以.php结尾的文件为text/html文件类型
   DirectoryIndex index.php	添加一个主页文件index.php
   php_value session.save_handler "files"
   php_value session.save_path    "/var/lib/php/session"
   
   ```

   php-mysql程序包提供的文件

   ```bash
   [root@localhost ~]# rpm -ql php-mysql
   /etc/php.d/mysql.ini
   /etc/php.d/mysqli.ini
   /etc/php.d/pdo_mysql.ini
   /usr/lib64/php/modules/mysql.so
   /usr/lib64/php/modules/mysqli.so
   /usr/lib64/php/modules/pdo_mysql.so
   ```

   该程序包上面三个配置文件的内容，就是分别激活启用下面三个模块！以实现php能够和mysql连接

3. 启动服务httpd服务：

   ```
   systemctl start httpd.service
   ```

   至此，我们的httpd+php就配置好了！

4. 配置mysql，添加如下选项（这里的选项和php没啥关系，仅用来方便用户的远程登录操作）

   vim /etc/my.cnf 在该文件的mysql模块下添加跳过主机解析选项！

   ```
   skip-name-resolve=ON
   ```

   启动mysql就ok了！

5. 之前我们已经安装了php-mysql模块，现在我们可以直接测试一下，php能否连接到mysql

   编辑一个简单的测试文件！

   ```bash
   cat /var/www/html/test.php < EOF
   <?php
           $conn = mysql_connect('127.0.0.1','root','');
           if ($conn)
                   echo "OK";
           else
                   echo "nO";
   ?>
   EOF
   ```

   注意：iptables和selinux务必都关掉，笔者叕被selinux折腾了半小时，莫名其妙的不知道哪里出错了！

   还有，这里连接mysql使用的ip地址是本机回环地址，不要用网卡地址，mysql默认是不允许远程连接的！

6. 接下来我们就可以使用浏览器，访问一下，我们的linux主机地址下的test.php文件了！

