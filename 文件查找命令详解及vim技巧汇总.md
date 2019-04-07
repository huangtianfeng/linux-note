[TOC]

# find：文件查找命令

实时查找，精确查找，速度略慢

语法：find [options] [查找路径] [查找条件] [处理动作]

查找路径默认是当前所在工作目录路径，当然也可以自己手动指定

前面的option咱先忽略过去，用的不多，以后有机会回来补充

## 常用的查找条件

### 根据文件名称，支持使用glob通配符

> -name PATTERN：根据文件名查找
>
> -iname PATTERN：忽略大小写
>
> -regex PATTERN：支持使用正则表达式查找

### 根据文件类型查找

-type TYPE

常用的type有

> c	字符设备文件
>
> b	块设备文件
>
> d	目录文件
>
> f	普通文件
>
> p	管道文件
>
> s	套接字文件
>
> l	符号链接文件

### 根据文件大小查找

> -size n
>
> 常用的单位有：k、M、G
>
> 注意k是小写的，M,G是大写的！
>
> 其他单位还有b，c，w
>
> 1b=512byte
>
> 1c=1byte
>
> 1w=2byte
>
> find命令统计大小的规则有点诡异，以后有时间再补充说明

### 根据文件属主属组查找

-user/uid 用户名/用户id：查找属组为指定用户的文件

-group/gid 组名/组id：查找属组为指定用户的文件

-nouser：查找没有属主的文件，一般在删除某用户后，原本属于该用户的文件会成为没有属主的文件

-nogroup：查找没有属组的文件

### 根据文件时间戳查找

-atime [+|-]#：根据atime查询

-mtime [+|-]#：根据mtime查询

-ctime [+|-]#：根据ctime查询

以上，时间单位都是天，+表示从当前时间算，多少天之前修改过，-表示多少天之内修改过！

-amin [+|-]

-mmin [+|-]

-cmin [+|-]

同上，只不过时间单位改为分

示例：查找var目录下，最近五分钟内修改过的文件

```bash
[root@localhost /]# find /var -mmin -5 | xargs -i ls -ld {}
drwx------. 2 root root 29 4月   7 21:33 /var/lib/rsyslog
-rw-------. 1 root root 124 4月   7 21:33 /var/lib/rsyslog/imjournal.state
-rw-------. 1 root root 959700 4月   7 21:33 /var/log/audit/audit.log
-rw-------. 1 root root 102130 4月   7 21:33 /var/log/cron
-rw-------. 1 root root 159328 4月   7 21:33 /var/log/messages
```



### 根据文件权限查找

-perm [/|-]MODE

-perm /MODE 表示，只要有一类用户中的某一位权限满足要求即可

示例，查找/etc目录下，其他人拥有执行权限的所有文件

```bash
[root@localhost /]# find /etc/httpd/  -perm /001 | xargs -i ls -ld {} 
drwxr-xr-x. 6 root root 104 4月   1 20:23 /etc/httpd/
drwxr-xr-x. 2 root root 37 4月   3 22:41 /etc/httpd/conf
drwxr-xr-x. 2 root root 117 4月   3 21:14 /etc/httpd/conf.d
```

-perm -MOD表示，文件的权限需要囊括所有指定的权限才行

示例：查找/etc/httpd目录下，所有人都有读权限，并且属组拥有读执行权限的文件

```
[root@localhost /]# find /etc/httpd -perm -454 | xargs -i ls -ld {}
drwxr-xr-x. 6 root root 104 4月   1 20:23 /etc/httpd
drwxr-xr-x. 2 root root 37 4月   3 22:41 /etc/httpd/conf
drwxr-xr-x. 2 root root 117 4月   3 21:14 /etc/httpd/conf.d
drwxr-xr-x. 2 root root 165 4月   3 21:26 /etc/httpd/conf.modules.d
lrwxrwxrwx. 1 root root 19 2月  27 22:40 /etc/httpd/logs -> ../../var/log/httpd
lrwxrwxrwx. 1 root root 29 2月  27 22:40 /etc/httpd/modules -> ../../usr/lib64/httpd/modules
lrwxrwxrwx. 1 root root 10 2月  27 22:40 /etc/httpd/run -> /run/httpd
drwxr-xr-x. 2 root root 20 4月   1 20:27 /etc/httpd/date
```

### 查找条件组合

逻辑与，或，非；

-a： 与运算，当有多个查找条件，不加任何逻辑运算符时，默认进行与运算，需要满足所有条件

-o：或运算，满足条件之一即可

-not或者！，非运算，条件取反

例如： 查找/var 目录下，其他用户拥有写权限的目录！，并且，最近四天内修改过

```bash
[root@localhost /]# find /var -perm -002 -a -type d -a -mtime -4 | xargs -i ls -ld {}
drwxrwxrwt. 6 root root 4096 4月   4 19:27 /var/tmp
```



## 处理动作

默认不指定处理动作时，会执行print打印操作

常用的动作命令如下所示

-ls：以ls的长格式，显示查找到的文件

-delete：删除查找到的文件

-exec COMMAND {} \;	表示将查找到的文件当做参数，一次性传递给COMMAND命令，{}表示前面查找到的所有文件，语句使用分号结尾，由于分号在bash命令行中有特殊意义，所以使用反斜线\进行转义

## xargs命令（待补充）

取个标题占位，时间有限，想将主要目标完成



---

# locate 文件查找命令

该命令也是一个实现文件查找功能的命令，它和上文的find命令所不同的是，find命令是实时查找文件系统中的文件，而locate命令的查找方式，是在事先已经建立好的一个数据库中，检索文件名，所以，该命令的查找速度会比find命令快上不少，但是因为是在事先建立的数据库中查找，所以查找到的结果，不一定精确，因为当我们在系统上新增，删除，修改某个文件后，并不会实时的将该文件的信息更新至数据库中！并且，locate查找时，给定的查找关键字，会匹配到文件绝对路径中的字符，而非仅匹配文件名，so，总结一下locate命令的特性就是：

**速度快，非事实查找，模糊查找：匹配时，文件名，路径名包含关键词即被匹配到**

* ocate依赖于实现建立好的数据库，所以使用前，需要使用updatedb命令来建立数据库

> 建立数据库需要遍历系统中所有的文件，是一个很耗费系统资源的操作，建议选在在系统资源空闲的是时候进行
>
> updatedb命令一般可以设置为周期性任务计划，定期执行

语法格式：locate [option] PATTERN

默认支持glob风格的通配符

常用选项：

-b 只匹配基名    

-c 统计符合条件的文件数    

-r 基于基本正则表达式

示例：查找整个文件系统中，以httpd结尾的文件

```bash
[root@localhost /]# locate -r .*httpd$
/etc/httpd
/etc/logrotate.d/httpd
/etc/sysconfig/httpd
/usr/lib64/httpd
/usr/libexec/initscripts/legacy-actions/httpd
/usr/sbin/httpd
/usr/share/httpd
/var/cache/httpd
/var/log/httpd
```



---

# vim入门

vim是一个功能非常强大的文本编辑工具，它强大的同时，也就意味着它比市面上绝大部分的文本编辑器的学习曲线要陡峭的多的多的多！所以，笔者在此也只敢将标题定位“入门“

首先我们来说一说vim的模式，该编辑器分为三种模式，命令行模式，插入模式和末行模式！ 

他们分别有什么用呢？ 且听我慢慢道来！

## 命令行模式

当我们在linux系统的命令行中使用vim打开一个文件时，模式会进入到vim的命令行模式中！ 在该模式中，有许许多多的命令，让我们实现对文本的操作，编辑，替换，删除，等等等等，一系列非常丰富的功能！下面先列举一些常用的功能

* 光标移动及翻页

| 操作方式 | 实现功能              |
| -------- | --------------------- |
| h键      | 光标向左移动 一个字符 |
| l键      | 光标向右移动一个字符  |
| j键      | 光标向下移动一个字符  |
| k键      | 光标向上移动一个字符  |
| ctrl+f   | 向上翻页              |
| ctrl+b   | 向下翻页              |
| ctrl+d   | 向下翻半页            |
| ctrl+u   | 向上翻半页            |
| 数字0    | 光标移动到行首        |
| $        | 光标移动到行尾        |
| G        | 移动到文件末行        |
| nG       | 移动到文件的第n行     |
| gg       | 移动到文件首行        |
|          |                       |

注意：光标的移动，前面都可以加一个数字，表示重复执行多少次移动

* 内容查找与替换

  | 操作方式 | 实现功能                                           |
  | -------- | -------------------------------------------------- |
  | /word    | 从光标处开始向文件末尾方向查找字符                 |
  | ？word   | 从光标处开始向文件首部方向查找字符                 |
  | n        | 顺着当前查找方向，将光标定位至下一个查找到的字符   |
  | N        | 当前查找方向的反方向，将光标定位至上一个查找的字符 |

  