[TOC]

# 数据库系统简介！

所谓数据库，就是数据集合！

excel表格，相信大家都用过！ 一个表格由行和列组成！其实，数据库存储数据，也就是基于这种表格的形式来存储数据的！ 所以，一般数据库中存储的数据，都遵循着一定的结构！例如：

| 学号   |      | 姓名   | 性别 | 分数 |
| ------ | ---- | ------ | ---- | ---- |
| N37001 |      | 宋江   | 男   | 60   |
| N37002 |      | 卢俊义 | 男   | 80   |
| N37003 |      | 吴用   | 男   | 100  |
| N37004 |      | 公孙胜 | 男   | 20   |

像这样的一个表格，在数据库中，我们也就称之为表（有时，我们也称之为一个关系），在这个表格中，存储着四行数据，每一行数据，我们称之为一个元组；而每一列，我们称之为一个字段或者一个属性！ 

一个数据库，由一个或多个表组成！

## 数据库的设计范式

我们在创建一个数据库时，需要对表的结构进行定义，数据库的设计范式是数据库设计所需要满足的规范，满足这些规范的数据库是简洁的、结构明晰的，同时，不会发生插入（insert）、删除（delete）和更新（update）操作异常

**数据库的设计范式就是：一张数据表的表结构所符合的某种设计标准的级别**

* 第一范式（1NF）

   所谓第一范式（1NF）是指数据库表的每一列都是不可分割的基本数据项，同一列中不能有多个值，即实体中的某个属性不能有多个值或者不能有重复的属性。如果出现重复的属性，就可能需要定义一个新的实体，新的实体由重复的属性构成，新实体与原实体之间为一对多关系。在第一范式（1NF）中表的每一行只包含一个实例的信息。简而言之，第一范式就是无重复的列。

这里只对第一范式做一下简单的介绍，还有第二范式、第三范式、第四范式、第五范式、 有兴趣的朋友可以自行百度！

## 常见概念介绍

* 表：为了满足范式设计要求，将一个数据集拆分为多个字段

* 约束：想数据表插入的数据要遵循的限制规则；

  > * 主键（primary key）：一个或多个字段的组合，填入主键中的数据必须不同于已存在的数据；且不能为空（一个表只能有一个主键）
  > * 外键：设置为外键的字段，能插入的数据，取决于另一个表中主键中的数据
  > * 唯一键（unique key）：一个或多个字段的组合，填入其中的数据不能重复，但是可以为空
  > * 检查性约束： 取决于表达式的要求

* 索引：将表中的某一个字段或者多个字段抽取出来，单独将其组织成为一个特殊的数据结构中；以方便检索

  > 常用的索引类型有：树形和hash
  >
  > 注意：索引的存在，有利于读请求，但是不利于写请求

  

# MariaDB简介

MariaDB是一个关系型数**据库管理系统**

所谓关系型数据库，就是采用关系模型（通常指二维表格模型）来组织数据的数据库

数据库能将大量结构一致的数据，组织成行和列的表装形式，进行存放！

而数据库管理系统，它的功能是将下层的文件系统向上封装，再提供一个统一的调用接口，以更加方便的形式，实现对数据库的增删改查等等一些列操作！ MariaDB就是这么一个管理软件！ 它也是基于C/S架构，分为客户端和服务器端

MariaDB是MySQL数据库的一个分支，由MySQL原开发团队研发维护！

# MariaDB服务端的安装和配置

在CentOS 7 中，只需要通过yum 安装两个包，mariadb-server 和mariadb，分别是mariadb的服务器端和客户端程序

修改一下配置文件/etc/my.cnf，在mysql配置快中，添加以下指令

```bash
skip_name_resolve=ON
```

该指令的作用在于，客户端通过账号登录系统的时候，跳过反解ip地址的操作！  记住这个功能就好

接下来就可以启动了

```bash
systemctl restart mariadb.service
```

启动完成后，使用ss -tnl会发现，已经监听在3306端口！

现在，我们要做的，就是使用MariaDB的客户端程序mysql，来连接MariaDB的服务器了！

# MariaDB的客户端程序mysql

使用rpm -ql mariadb 可以看到，该程序包提供的主程序，叫mysql

由于MariaDB和mysql暧昧的关系，很多时候，大家把他们俩都称为mysql，所以，就不要纠结，为什么MariaDB的客户端程序叫mysql了！   

## 使用方法

语法格式； mysql [options] db_name

常用选项有

> -u	指明使用哪个用户连接服务端；默认为root
>
> -h	指明mysql服务器地址；
>
> -p	指明用户密码，不建议在命令行中直接给出，而是加-p后 回车进入密码输入界面输入密码
>
> -P；--port=端口号；mysql服务器监听的端口；默认为3306/tcp
>
> -S；--socket=路径： 套接字文件路径
>
> -e：--execute=“sql命令”连接至服务器并让其执行命令后返回结果并退出

注意：

1. mysql服务器端的用户账号由两个部分组成：'USERNAME@HOST'，其中HOST表示，该用户允许从哪些主机远程连接服务器！HOST的表示方式，支持使用通配符：

   > %	匹配任意长度的任意字符
   >
   > _	匹配任意单个字符
   >
   > 例如：user@172.16.0.0/16 或者使用user@172.16.%.% 表示user这个用户，允许通过172.16这个网段中的主机远程连接服务器

   

2. 客户端连接服务器端时，服务器会试图反解客户端的ip地址为主机名；而服务器对主机名和ip地址的授权，是分开的，比如说，授权了user用户可以通过192.168.0.1主机远程连接服务器端，但是，user连接的时候，服务器端会试图将192.168.0.1的主机名反解出来，这个时候，user用户的源地址变成了主机名，服务器端将拒绝user用户的登录。所以，为了避免这种情况的发生，要么在配置文件中设置跳过反解这个过程，要么授权的时候，将对应主机名也一并授权，一般我们采用前一种方式！

使用mysql -uroot 登录后，会出现如下界面

```bash
[root@localhost ~]# mysql -uroot
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 8
Server version: 5.5.60-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> 
```

表示已经连接至服务器，接下来我们就可以输入sql命令来进行数据库的相关操作了！

关于创建用户的相关操作，会在后文详细说明！

# mysql的基础操作

连接上服务器后，可以通过help命令查看帮助

根据命令的运行方式，或者说运行环境，又分为客户端命令和服务器端命令

## 客户端命令

所谓客户端命令，我觉得理解成客户端的控制字符比较好，它的作用就是修改本地运行命令时的环境！我们一起来看一下客户端的一些常用命令及其功能，大概就能意会了

> \u 	切换默认库
>
> \q	 退出
>
> \c 	取消执行的命令，
>
> \d 	修改语句结束符，
>
> \g 	语句结束标记
>
> \G	语句结束标记，纵向显示，有时候显示的内容会超出屏幕边界使用这个即可
>
> \!	 执行shell命令
>
> \\. 	后接一个脚本路径，装载并运行sql脚本，注意，该脚本需要放在mysql用户有权限读到的路径

客户端的命令很简单！ 接下来的服务器端命令，才是重点；

## 服务器端命令

所谓服务器端命令，就是需要送往服务器端执行并返回结果的命令！包括对数据库的增删查改等等一系列操作！

获取服务器端命令的帮助，使用 help contents获取命令类别；help CMDTYPE 可获取指定类别的命令列表，使用help 指定命令，即可获取该命令的详细帮助文档

注意： 命令是不区分大小写的！ 但是约定俗称，服务器端命令使用大写！ 你非要用小写，也没多大关系

我们先来看看命令分类列表：

```mysql
mysql> help contents
You asked for help about help category: "Contents"
For more information, type 'help <item>', where <item> is one of the following
categories:
   Account Management
   Administration
   Compound Statements
   Data Definition
   Data Manipulation
   Data Types
   Functions
   Functions and Modifiers for Use with GROUP BY
   Geographic Features
   Help Metadata
   Language Structure
   Plugins
   Procedures
   Storage Engines
   Table Maintenance
   Transactions
   User-Defined Functions
   Utility
```

在这里边，我们只需要重点关注Data Definition（DDL：数据定义） 和Data Manipulation（DML：数据操作）两类即可，毕竟我们只需要能够简单使用，不必钻太深

### DDL：Data Definition

用于实现定义数据，管理数据库组件，例如数据库本身，表，索引，视图，用户，存储过程等

#### 数据库管理

##### 创建数据库

命令格式：CREATE {DATABASE | SCHEMA} [IF NOT EXISTS] db_name [create_specification] ...

> IF NOT EXISTS： 该选项用于写脚本时使用，表示，当创建失败时，不报错，只警告
>
> create_specification:
>
> [DEFAULT] CHARACTER SET [=] charset_name	设定仓库使用的默认字符集
>
> [DEFAULT] COLLATE [=] collation_name	设定仓库的默认排序规则！

示例：创建一个名为tian的数据库：

```mysql
MariaDB [(none)]> CREATE DATABASE tian
    -> ;
Query OK, 1 row affected (0.00 sec)

MariaDB [(none)]> SHOW DATABASES
    -> ;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| test               |
| tian               |
+--------------------+
5 rows in set (0.00 sec)

```

注意，笔者在这里输入完服务端命令后，老师忘记加分号，所以，出现了以上情况！ 回车后，换行等待继续输入。

其实这里的数据库，就是/var/lib/mysql路径下的一个目录！我们来看一眼

```bash
[root@localhost ~]# ll /var/lib/mysql/
总用量 28700
-rw-rw----. 1 mysql mysql    16384 4月  11 18:38 aria_log.00000001
-rw-rw----. 1 mysql mysql       52 4月  11 18:38 aria_log_control
-rw-rw----. 1 mysql mysql 18874368 4月  11 18:38 ibdata1
-rw-rw----. 1 mysql mysql  5242880 4月  11 18:38 ib_logfile0
-rw-rw----. 1 mysql mysql  5242880 4月   3 21:36 ib_logfile1
drwx------. 2 mysql mysql     4096 4月   3 21:36 mysql
srwxrwxrwx. 1 mysql mysql        0 4月  11 18:39 mysql.sock
drwx------. 2 mysql mysql     4096 4月   3 21:36 performance_schema
drwx------. 2 mysql mysql        6 4月   3 21:36 test
drwx------. 2 mysql mysql       20 4月  12 21:29 tian
```

创建数据库，其实也可以自己手动在该目录下创建一个目录，所以说，数据库中的数据库名是否区分大小写，取决于文件系统是否区分大小写！ 

##### 修改数据库

命令语法：ALTER {DATABASE | SCHEMA} [db_name]  alter_specification ...

>  alter_specification
>
> ​	[DEFAULT] CHARACTER SET [=] charset_name	设定仓库使用的默认字符集
>
> ​	[DEFAULT] COLLATE [=] collation_name	设定仓库的默认排序规则！

##### 删除数据库

语法格式：DROP {DATABASE | SCHEMA} [IF EXISTS] db_name

删除了可就恢复不过来了，在回车前，请确保自己在做什么！ 建议要删除的时候，使用mv将对应的数据库目录移走！

##### 查询数据库

语法格式：SHOW {DATABASES | SCHEMAS} [LIKE 'pattern' | WHERE expr]

支持使用通配符

**WHERE子句，用于提取满足标准的元组（记录/行）**

示例：查询名字以t开头的数据库

```mysql
MariaDB [(none)]> SHOW DATABASES LIKE 't%';
+---------------+
| Database (t%) |
+---------------+
| test          |
| tian          |
+---------------+
2 rows in set (0.00 sec)
```

注意：通配符：

%表示匹配任意长度任意字符

_ 表示匹配单个任意字符

#### 表管理

在说创建表之前，需要先了解一下字段的数据类型，和字段数据修饰符

##### 数据类型

* 字符型：

    > 1. 定长字符型
    >
    >    CHAR(#)；不区分字符大小写；#指定字节长度，最大不超过256
    >
    >    BINARY(#)；区分字符大小写；#同上
    >
    > 2. 变长字符型
    >
    >    VARCHAR(#)；不区分字符大小写；#同上
    >
    >    VARBINARY(#)；区分字符大小写；#同上

* 数值型：

    > 1. 精确数值型
    >
    >    INT：整形数值
    >
    >    INT的五个变种：TINYINT、SMALLINT、MEDIUMINT、INT、LONGINT
    >
    > 2. 近似数值型（浮点型）
    >
    >    FLOAT：单精度浮点型
    >
    >    DOUBLE：双精度浮点型

* 日期时间型：

  > 日期型：DATE
  >
  > 时间型：TIME
  >
  > 日期时间型：DATETIME
  >
  > 年份型：YEAR(2)、YEAR（4）
  >
  > 时间戳型：TIMESTAMP(从1970年开始计时所经过的秒数)

* 对象存储型：

  存储长篇文章时，放在字段中是不理智的！可以另外找一片区域，单独存放一段数据，在字段中使用一个指正指向该数据的起始位置

  > TEXT：不区分字符大小写
  >
  > BLOB：区分字符大小写
  >
  > TEXT和BLOB还有以下变种：
  >
  > TINYTEXT、SMALLTEXT、MEDIUMTEXT、TEXT、LONGTEXT
  >
  > TINYBLOB、SMALLBLOB、MEDIUMBLOB、BLOB、LONGBLOB

* 内置类型

    > 集合型：SET
    >
    > 枚举型：ENUM
    >
    > 集合型，表示取值范围只能在给定的集合的**组合范围**内
    >
    > 枚举型，表示取值范围只能是给定的字符串中的一个

##### 数据类型的修饰符

* NOT NULL  非空

  NULL    允许为空

  AUTO_INCREMENT;自动增长

  DEFAULT  value；默认值

  PRIMARY KEY；主键（意味着，唯一、非空）

  UNIQUE KEY；唯一键，可以为空

##### 创建表

语法格式：CREATE [TEMPORARY] TABLE [IF NOT EXISTS] tbl_name
    (create_definition,...)
    [table_options]
    [partition_options]

也可使用CREATE [TEMPORARY] TABLE [IF NOT EXISTS] tbl_name { LIKE old_tbl_name | (LIKE old_tbl_name) }来复制其他表的结构！

> * create_definition : 该内容的格式是	（字段名 字段定义，字段名 字段定义，……）
>
> 注意字段名和属性之间用空格分隔，不同字段，使用逗号分隔
>
> 字段定义，就是由上面所介绍的数据类型，以及修饰符所构成！
>
> 这里只介绍其简单用法，后面的table_options partition_options 可自行查看帮助也了解！
>
> 主键，唯一键，外键，索引等，也可以使用如下格式，在括号中单独指定
>
> > 指定主键：PRIMARY KEY (con1，col2……）
> >
> > 指定唯一键：UNIQUE KEY(con1，col2……）
> >
> > 指定外键：FOREIGN KEY (column)
> >
> > 指定用于索引的字段：KEY|INDEX [index_name]  (col1,col2……)

> table_options
>
> ENGINE [=] engine_name 定义表的引擎
>
> CHARACTER SET [=] charset_name；设定字符集
>
> COLLATE [=] collation_name;设定排序规则
>
> SHOW ENGINES 可以查看支持的存储引擎，所谓引擎，暂时可以理解为表的类型
>
> SHOW TABLE STATUS [LIKE 'tbl_name] [WHERE clause] \G   查看所有表的状态信息，\G垂直显示

示例：

* 在数据库tian中，创建一个名为grade的表，定义三个字段，id、name、gender，id设置为主键，整型数值，name为char类型，gender为枚举类型，要么为M(男)，要么为F（女）

```MYSQL
MariaDB [(none)]> \u tian
Database changed
MariaDB [tian]> CREATE TABLE grade ( id INT PRIMARY KEY AUTO_INCREMENT, name CHAR NOT NULL, grander ENUM('F','M') );
Query OK, 0 rows affected (0.21 sec)

MariaDB [tian]> DESC grade;
+---------+---------------+------+-----+---------+----------------+
| Field   | Type          | Null | Key | Default | Extra          |
+---------+---------------+------+-----+---------+----------------+
| id      | int(11)       | NO   | PRI | NULL    | auto_increment |
| name    | char(1)       | NO   |     | NULL    |                |
| grander | enum('F','M') | YES  |     | NULL    |                |
+---------+---------------+------+-----+---------+----------------+
3 rows in set (0.04 sec)


```

创建之前，先使用\u 将默认仓库切换至tian，就和切换挡墙工作路径类似；创建完毕后，可以使用DESC命令或者SHOW COLUMN FORM 表名，查看该表的信息，如上所示：这里有个失误，忘记指定数据类型的长度，到后面写到修改表，再来改过来！

* 另外一种创建方法

  ```mysql
  MariaDB [tian]> CREATE TABLE grande3(id TINYINT NOT NULL AUTO_INCREMENT,name CHAR(60) NOT NULL,grander ENUM('F','M'),PRIMARY KEY(id),UNIQUE KEYI(name),INDEX(id,name));
  Query OK, 0 rows affected (0.11 sec)
  MariaDB [tian]> DESC grande3;
  +---------+---------------+------+-----+---------+----------------+
  | Field   | Type          | Null | Key | Default | Extra          |
  +---------+---------------+------+-----+---------+----------------+
  | id      | tinyint(4)    | NO   | PRI | NULL    | auto_increment |
  | name    | char(60)      | NO   | UNI | NULL    |                |
  | grander | enum('F','M') | YES  |     | NULL    |                |
  +---------+---------------+------+-----+---------+----------------+
  3 rows in set (0.00 sec)
  ```

##### 修改表

语法格式：

ALTER [ONLINE | OFFLINE] [IGNORE] TABLE tbl_name
    [alter_specification [, alter_specification] ...]
    [partition_options]

alter_specification

> * 添加字段：
>
>   ADD [COLUMN] (col_name column_definition,...)
>
> * 添加索引：
>
>   ADD {INDEX|KEY} [index_name] [index_type] (index_col_name,...) [index_option] ...
>
> * 添加主键：
>
>    ADD [CONSTRAINT [symbol]] PRIMARY KEY [index_type] (index_col_name,...) [index_option] ...
>
> * 添加唯一键：
>
>   ADD UNIQUE [INDEX|KEY] [index_name] [index_type] (index_col_name,...) [index_option] ...
>
> * 修改字段属性：
>
>    ALTER [COLUMN] col_name {SET DEFAULT literal | DROP DEFAULT}
>
> * 修改字段名，字段属性，FIRST|AFTER可指定修改后的字段放在哪个字段之前或之后
>
>   CHANGE [COLUMN] old_col_name new_col_name column_definition
>           [FIRST|AFTER col_name]
>
> * 修改字段属性：
>
>    MODIFY [COLUMN] col_name column_definition
>           [FIRST | AFTER col_name]
>
> * 取消各种键（约束）：
>
>   DROP [COLUMN] col_name
>
>   DROP PRIMARY KEY
>
>   DROP {INDEX|KEY} index_name
>
>   DROP FOREIGN KEY fk_symbol

示例，我们前面创建了一个grade的表，但是没有指定字段中数据类型的长度，现在我们来改一改：

```MYSQL
MariaDB [tian]> DESC grade;
+---------+---------------+------+-----+---------+----------------+
| Field   | Type          | Null | Key | Default | Extra          |
+---------+---------------+------+-----+---------+----------------+
| id      | int(11)       | NO   | PRI | NULL    | auto_increment |
| name    | char(1)       | NO   |     | NULL    |                |
| grander | enum('F','M') | YES  |     | NULL    |                |
+---------+---------------+------+-----+---------+----------------+
3 rows in set (0.01 sec)

MariaDB [tian]> ALTER TABLE grade MODIFY name CHAR(60) NOT NULL  ; 
Query OK, 0 rows affected (0.22 sec)               
Records: 0  Duplicates: 0  Warnings: 0

MariaDB [tian]> DESC grade;
+---------+---------------+------+-----+---------+----------------+
| Field   | Type          | Null | Key | Default | Extra          |
+---------+---------------+------+-----+---------+----------------+
| id      | int(11)       | NO   | PRI | NULL    | auto_increment |
| name    | char(60)      | NO   |     | NULL    |                |
| grander | enum('F','M') | YES  |     | NULL    |                |
+---------+---------------+------+-----+---------+----------------+
3 rows in set (0.00 sec)

```

##### 查看表

查看仓库中的表列表

SHOW [FULL] TABLES [{FROM | IN} db_name]
    [LIKE 'pattern' | WHERE expr]

查看某个表的字段属性：

SHOW COLUMNS FROM table_name；

示例：查看名为tian的数据库中有几个表，分别是什么，以及查看表名为grade的字段详细属性

```mysql
MariaDB [tian]> SHOW TABLES IN tian;
+----------------+
| Tables_in_tian |
+----------------+
| grade          |
| grade2         |
| grande3        |
| old            |
+----------------+
4 rows in set (0.00 sec)

MariaDB [tian]> SHOW COLUMNS FROM grade;
+---------+---------------+------+-----+---------+----------------+
| Field   | Type          | Null | Key | Default | Extra          |
+---------+---------------+------+-----+---------+----------------+
| id      | int(11)       | NO   | PRI | NULL    | auto_increment |
| name    | char(60)      | NO   |     | NULL    |                |
| grander | enum('F','M') | YES  |     | NULL    |                |
+---------+---------------+------+-----+---------+----------------+
3 rows in set (0.00 sec)
```

##### 删除表：

语法格式：

DROP [TEMPORARY] TABLE [IF EXISTS]
    tbl_name [, tbl_name] ...

示例：删除数据库tian中的grade2，grade3以及old；

```mysql
MariaDB [tian]> DROP TABLE grade2,grade3,old;
ERROR 1051 (42S02): Unknown table 'grade3'
MariaDB [tian]> DROP TABLE grade2,grande3,old;
ERROR 1051 (42S02): Unknown table 'grade2,old'
MariaDB [tian]> SHOW TABLES IN tian;
+----------------+
| Tables_in_tian |
+----------------+
| grade          |
+----------------+
1 row in set (0.00 sec)
```

打错字了，引以为戒！报了两次错误

#### 索引管理

##### 创建索引

使用语法：

CREATE [ONLINE|OFFLINE] [UNIQUE|FULLTEXT|SPATIAL] INDEX index_name
    [index_type]
    ON tbl_name (index_col_name,...)
    [index_option] ...

简化一下，就是CREATE INDEX 索引名 ON 哪个表（的哪(些）个字段）

例如：在grade表中创建一个索引，由id+name字段组成

```mysql
MariaDB [tian]> CREATE INDEX suoyin ON grade(id,name);
Query OK, 0 rows affected (0.68 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

##### 查看索引

使用语法：

SHOW {INDEX | INDEXES | KEYS}
    {FROM | IN} tbl_name
    [{FROM | IN} db_name]
    [WHERE expr]

简化一下就是：SHOW INDEX IN 哪个表 IN 哪个数据库 WHERE 字段过滤！（提取满足条件的行）

简单示例：查看数据库中tian中的grade表的索引信息：

```mysql
MariaDB [tian]> SHOW INDEX IN grade;
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| Table | Non_unique | Key_name | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment |
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| grade |          0 | PRIMARY  |            1 | id          | A         |           0 |     NULL | NULL   |      | BTREE      |         |               |
| grade |          1 | suoyin   |            1 | id          | A         |           0 |     NULL | NULL   |      | BTREE      |         |               |
| grade |          1 | suoyin   |            2 | name        | A         |           0 |     NULL | NULL   |      | BTREE      |         |               |
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
3 rows in set (0.00 sec)
```

##### 删除索引：

语法格式：

DROP [ONLINE|OFFLINE] INDEX index_name ON tbl_name

这个就太简单了吧： 示例，将前面创建的名为suoyin的索引，删除：

```mysql
MariaDB [tian]> DROP INDEX suoyin ON grade;
Query OK, 0 rows affected (0.13 sec)
Records: 0  Duplicates: 0  Warnings: 0
```



### DML:Data Manipulation

数据操纵命令：常用的四个是：INSERT、DELETE、UPDATE、SELECT

##### 插入行：INSERT

语法格式：

INSERT [INTO] tbl_name [(col1,...)] {VALUES|VALUE} (val1,...)(...)...

示例：在grade表中，插入一行数据，id为1，name为tom，grander为M：

```mysql
MariaDB [tian]> INSERT grade (id,name,grander) VALUES ('1','tom','M');
Query OK, 1 row affected (0.09 sec)

MariaDB [tian]> SHOW TABLE grade;
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MariaDB server version for the right syntax to use near 'grade' at line 1
MariaDB [tian]> SELECT * FROM grade;
+----+------+---------+
| id | name | grander |
+----+------+---------+
|  1 | tom  | M       |
+----+------+---------+
1 row in set (0.00 sec)

```

**注意，这里的字段名是可省略的，省略时，后面需要给出所有字段值！**

**字段为空，使用NULL ，不能加引号,字符串必须加单引号或者双引号**

##### 删除行：DELETE

语法格式：

DELETE FROM tbl_name [WHERE where_condition] [ORDER BY ...] [LIMIT row_count]

示例：我们先添加一行id为2，name为Jerry，grander为M的行！而后将name为tom的行删除

```mysql
MariaDB [tian]> INSERT grade VALUES('2','jerry','F');
Query OK, 1 row affected (0.10 sec)

MariaDB [tian]> SELECT * FROM grade;
+----+-------+---------+
| id | name  | grander |
+----+-------+---------+
|  1 | tom   | M       |
|  2 | jerry | F       |
+----+-------+---------+
2 rows in set (0.00 sec)

MariaDB [tian]> DELETE FROM grade WHERE name='tom';
Query OK, 1 row affected (0.11 sec)

MariaDB [tian]> SELECT * FROM grade;
+----+-------+---------+
| id | name  | grander |
+----+-------+---------+
|  2 | jerry | F       |
+----+-------+---------+
1 row in set (0.00 sec)

```

**注意：必须加where，不然会删除所有行，这个rm -rf 基本上性质差不多了！**

##### 查询行：SELECT

语法格式：

SELECT select_expr,... FROM tbl_name WHERE clause

查询哪个字段（可以用通配符：* 表示所有字段）FROM 哪个表，WHERE 满足哪个条件

这是最简单的用法，还有许多可选项，如要详细了解，可自行查看帮助手册：

示例：查询grade表中，id为2的行：

```mysql
MariaDB [tian]> SELECT * FROM grade WHERE id=2;
+----+-------+---------+
| id | name  | grander |
+----+-------+---------+
|  2 | jerry | F       |
+----+-------+---------+
1 row in set (0.05 sec)
```

还可以使用AS将字段名改为指定的别名显示，

例如，查询grade表中，id为2的行的name，和grander字段，将其字段名改为test1，test2予以显示；

```mysql
MariaDB [tian]> SELECT name AS test1,grander AS test2  FROM grade WHERE id=2;
+-------+-------+
| test1 | test2 |
+-------+-------+
| tom   | F     |
+-------+-------+
1 row in set (0.00 sec)
```



WHERE clause 用于指明筛选条件；条件可以使用操作符，>、<、>=、<=、=、!=，组合条件 and or not

还可以使用LIKE 'PATTERN'  通配符：% 任意长度的任意是字符、 _ 任意单个字符；

RLIKE ‘PATTERN’ 使用RLIKE时，PATTERN支持使用正则表达式

需要匹配内容为空的字段时，不能使用以上的操作符，而需要使用，IS NULL/IS NOT NULL

##### 数据更新：UPDATE

语法格式：

UPDATE table_reference SET col_name1={expr1|DEFAULT} [, col_name2={expr2|DEFAULT}] ...

​    [WHERE where_condition]

​    [ORDER BY ...]

​    [LIMIT row_count]

简单用法：UPDATE 哪个表 SET 字段名=字段值

示例，将grade表中，id为2的行的name改为tom；

```
MariaDB [tian]> UPDATE grade SET name='tom' WHERE id=2;
Query OK, 1 row affected (0.10 sec)
Rows matched: 1  Changed: 1  Warnings: 0

MariaDB [tian]> SELECT * FROM grade;
+----+------+---------+
| id | name | grander |
+----+------+---------+
|  2 | tom  | F       |
+----+------+---------+
1 row in set (0.00 sec)
```

**注意；该命令，不加WHERE子句时，会对所有行进行更新，很危险！**

## mysql的用户创建与授权

### 创建用户：

语法格式：

CREATE USER user_specification
    [, user_specification] ...

user_specification:
    user
    [
        IDENTIFIED BY [PASSWORD] 'password'
      | IDENTIFIED WITH auth_plugin [AS 'auth_string']
    ]

简化后就是： CREATE USER '用户名'@'允许远程登录的主机' IDENTIFIED BY '用户密码'

示例：创建一个名为test的用户，密码为test，允许从172.16网段的所有主机远程登录：

```mysql
MariaDB [tian]> CREATE USER 'test'@'172.16.%' IDENTIFIED BY 'test';
Query OK, 0 rows affected (0.06 sec)
```

### 用户授权

GRANT
    priv_type [(column_list)]
      [, priv_type [(column_list)]] ...
    ON [object_type] priv_level
    TO user_specification [, user_specification] ...
    [REQUIRE {NONE | ssl_option [[AND] ssl_option] ...}]
    [WITH with_option ...]

简化一下，就是 GRANT 哪些权限 ON 哪个数据库库或者哪个数据库的哪个表 TO 授权给哪个用户

权限类型，常见的有，ALL、INSERT、DELETE、SELECT、UPDATE

示例：给test用户授权，只允许查看grade表中的数，不能修改，删除等操作：

```mysql
MariaDB [(none)]> GRANT SELECT ON tian.grade TO 'test'@'172.16.%' IDENTIFIED BY 'test';
Query OK, 0 rows affected (0.10 sec)
```

然后我从另一台主机使用test登录该服务器，执行INSERT操作：

```mysql
mysql> INSERT grade VALUES('1','tian','M');
ERROR 1142 (42000): INSERT command denied to user 'test'@'172.16.0.107' for table 'grade'
mysql> select * FROM grade;
+----+------+---------+
| id | name | grander |
+----+------+---------+
|  2 | tom  | F       |
+----+------+---------+
1 row in set (0.00 sec)
```

可以看到，INSERT被拒绝了！但是SELECT是可以的！

### 权限回收

语法格式：

收回授权

REVOKE priv_type,... ON db_name.tbl_name FROM 'user'@'host'

示例，现在我收回test账户的SELECT权限：

```mysql

MariaDB [(none)]> REVOKE SELECT ON tian.grade FROM 'test'@'172.16.%';
Query OK, 0 rows affected (0.10 sec)
```

现在我再用另外一台主机，重复之前的查询操作；

```
mysql> INSERT grade VALUES('1','tian','M');
ERROR 1142 (42000): INSERT command denied to user 'test'@'172.16.0.107' for table 'grade'
mysql> select * FROM grade;
+----+------+---------+
| id | name | grander |
+----+------+---------+
|  2 | tom  | F       |
+----+------+---------+
1 row in set (0.00 sec)

mysql> select * FROM grade;
ERROR 1142 (42000): SELECT command denied to user 'test'@'172.16.0.107' for table 'grade'
```

前面能查询成功，是没收回权限之前，后面执行同样的查询操作，被拒绝了！



注意：MariaDB 服务进程启动时，会读取mysql库中的所有授权表至内存中；

1。GRANT或REVOKE命令等执行的权限操作保存于表中，MariaDB刺客一般会自动重读授权表，权限修改会立即生效

2,，其他方式实现的权限修改，要想生效，必须手动运行FLUSH PRIVILEGES命令即可；



最后：加固mysql服务器，在安装完成后，运行mysql_secure_installation命令；

会让你设置root用户密码，删除测试库test等操作！

