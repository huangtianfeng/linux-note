# 三种存储类型

我们的存储设备有许多类型，例如：SATA,ASA,IDE,SCSI,USB等等，这些其实也都是一种协议，一种低端的存取发送协议，由硬件芯片完成，这些类型的硬盘直接接在主机总线上，所以也称之为DAS(Direct Attached Storage)：直接附加存储，与之相对应的，还有网络附加存储NAS（Network Attached Storage）！

* DAS：在底层，接口以block形式存在，可以被分区，格式化，单独挂载

> 常见设备（协议）有：SATA,ASA,IDE,SCSI,USB

* NAS：是一种网络共享的存储空间，将别人共享出来的目录或者文件系统，挂载至本地，就像使用本地文件系统一样，但是，他的接口不是block接口，而是文件接口，只有block接口才能被分区格式化，所以，NAS不能被分区格式化，只能直接挂载使用！

>  常见设备（协议）：CIFS（samba）、NFS（Network File System）	

* SAN：Storage Area Network，存储区域网，将原本在总线上传输的数据，使用tcp/ip协议封装起来，让其在网络上传输，以达到远程存储的目的（也就是隧道技术的初级模型吧），这样的设备，通过网络传输存取数据，底层提供的接口类型是block，所以能够被分区，格式化并单独挂载！

> 常见设备（协议）：ISCSI(IP-SAN)、FCSAN、FCoE

# FTP

是一种应用层的协议，file transfer protocol（文件传输协议），它是一种非常古老的协议，主要实现在个主机之间完成文件传送！监听于21tcp端口用于接受命令连接，同时也会使用20号端口，用于主动模式向客户端发起请求（目前来说，大概已经废弃了）

ftp协议的连接分为两类，一类是命令连接，一类是数据连接！（一个完整的信道为一个连接！）

命令连接，即客户端向服务器端发起下载，上传等请求！ 服务器端给与响应，并告知客户端，新打开一个随机端口的端口号，用于数据传输！ 随后，等待客户端发起数据连接请求！

数据连接：顾名思义，专门用来传输前面命令连接所请求的数据！经过tcp三次握手，即可开始传输，传输完毕后自动断开！数据连接又有两种模式，主动模式（PSRT）和被动模式(PASV)，主动模式，服务器端的20号tcp端口，向客户端发起命令连接的端口号后的第一个可用端口发起数据连接！一般，这种模式会被客户端的防火墙挡掉，所以难以实现。 而被动模式，服务器端接收到命令请求后，打开一个随机端口，并等待客户端的链接！

ftp有两种传输机制，文本模式，和二进制模式！ 由ftp自行选择！

## ftp协议的实现

* 服务器端程序：

  windows：serv-u、IIS、filezilla……

  linux：wuftpd、proftpd、filezilla、pureftpd、vsftpd……

* 客户端程序：

  windows：ftp、filezilla、cuteFTP……

  linux：lftp、ftp、filezilla、gftp……

  大多数浏览器也都是一个ftp协议的客户端！

我们这里主要讲vsftp

# vsftpd服务的搭建

vsftpd：very secure ftpd 非常安全的ftpd

ftp是一个很古老，很不安全的协议，所以当下的互联网世界，我们一般是不建议使用的，如果非要用，那就用vsftpd，看名字就知道，是一个非常安全的ftpd服务！

和httpd服务一样基于URL被访问，格式为：[SCHEME://username:passwd@HOST:PORT/path/to/file](scheme://username:passwd@HOST:PORT/path/to/file)

路径映射：基于用户的家目录，每个用户的的url映射到当前用户的家目录，匿名访问时，自动映射为ftp用户的家目录！（因为vsftpd进程是以ftp用户的身份发起运行的）

服务安装很简单，centos系统自带的光盘，就提供这个服务程序，也可以通过yum安装。

> yum install vsftpd -y

安装完毕后，就是配置文件了！

注意：vsftp服务对根目录（/var/ftp/)要求其他用户不能有写权限，且属主属组（root）不能更改，一改，服务就启动不了了！

想创建一个目录，允许其他人上传，可以在该目录下新建一个子目录，修改其权限！

## 配置文件介绍

* 主配置文件：/etc/vsftpd/vsftpd.conf

  格式： 指令=值（必须顶格写，不能有任何多余的空格！）

  使用man vsftpd可查看帮助文档！

  常用指令：

  > * 匿名用户指令
  >
  > anon_enable=YES  是否启用匿名登录
  >
  > anon_upload_enable=YES 是否启用匿名用户的上传权限
  >
  > anon_mkdir_write_enable=YES 是否启用匿名用户的呃创建目录权限
  >
  > anon_other_write_enable=YES 是否启用匿名用户的其他写权限，包括删除文件，目录等等
  >
  > anon_umask=022 	匿名用户上传文件的权限码
  >
  > * 系统用户指令
  >
  > local_enable=YES     是否允许系统用户登录（/etc/passwd文件下的用户
  >
  > local_umask=022        系统用户上传的文件权限掩码
  >
  > write_enable=YES     系统用户是否有写权限
  >
  > chroot_list_enable=YES	禁锢某些用户于家目录中（而后使用下面指令，指出用户列表保存位置）
  >
  > chroot_list_file=/etc/vsftpd/chroot.list   禁锢列表文件中存在的用户于其家目录中，需要先移除用户对家目录的写权限
  >
  > chroot_local_user=YES	禁锢所有系统用户于家目录中，禁锢完后，连接后，使用pwd将显示为当前目录为根目录！
  >
  > userlist_enable=YES	启用/etc/vsftpd/user_list文件来控制可登录用户
  >
  > userlist_deny=YES|NO	默认是YES！ YES意味着黑名单，NO意味着白名单！

* 辅助配置文件：/etc/vsftpd/ftpusers

  该文件中保存了一个系统用户的列表，表明，在该列表中的所有系统用户，都禁止登录ftp服务！为什么呢，因为ftp协议在传输时是明文的，使用tcpdump工具，一抓包，就能看到登录用户的账号和密码！所以，静止任何系统管理员相关账号登录！

* 用户认证

  linux系统登录的时候，登录提示符是由/bin/login程序提供！ 该程序自身没有认证功能，它调用了pam组件完成的（插入式认证模块：Plugable Authentication Module），pam是一个库，任何程序都可以调用该库！它是高度模块化的认证框架！

  /etc/pam.d/目录下为每一个调用了该库的应用程序都提供了配置文件！

  ```bash
  [root@localhost ~]# cat /etc/pam.d/vsftpd 
  #%PAM-1.0
  session    optional     pam_keyinit.so    force revoke
  auth       required	pam_listfile.so item=user sense=deny file=/etc/vsftpd/ftpusers onerr=succeed
  auth       required	pam_shells.so
  auth       include	password-auth
  account    include	password-auth
  session    required     pam_loginuid.so
  session    include	password-auth
  ```

  将这里的sense=deny改为allow，/etc/vsftpd/ftpusers文件将成为一个白名单，建议使用白名单，而不是黑名单！

  认证可以是一个服务，提供给客户端查询！

* 虚拟用户

  非系统用户（非/etc/passwd）文件中的账号密码！账号密码可存放于mysql，文件，等等一切能存放数据的地方！

  而认证是调用pam认证框架进行的，pam默认只能识别文件系统中的文件，访问mysql需要调用第三方模块，需要自己编译安装。每个应用程序，只要开发时调用了pam的API即可使用pam库进行认证，而pam是一个认证框架，其中有许多模块，每一个程序都要有自己的专用配置文件，用于定义使用哪些模块，到哪里去找认证文件！这个配置文件一般在/etc/pam.d/目录下，和服务同名！

## 如何用mysql存储账号密码进行认证

1. 安装pam模块！ 它依赖于mysql的开发环境（mariadb-devel）以及pam-devel： 任何时候编译和某个软件包结合使用的模块，都需要提供程序的开发环境！

2. 安装开发环境包组：主要使用其中的gcc

   ```bash
   yum groupinstall "Development Tools" "Server Platform Development"
   ```

3. 下载：pam_mysql模块！（github上有）

4. 展开后 ./configure --with-mysql=/usr  --with-pam =/usr --with-pam-mods-dir=/usr/lib64/security/

5. make&&make install，会在/usr/lib64/security目录下生成一个模块！

   注意：虚拟用户，都是映射为一个来宾账号访问的！ 所以还是需要提供一个系统用户，并在配置文件中，允许匿名访问，并指明映射的用户名：添加以下指令即可！

   > pam_service_name=vsftpd.vusers：用来认证的配置文件
   >
   > guest_enable=YES：启用来宾用户
   >
   > guest_username=vuser：映射为vuser用户

6. 配置mysql，加上skip_name_resolve=ON，innodb_file_per_table=ON，log_bin=mysql（启动二进制日志）启动后，授权用户账号，创建数据库，创建表，插入密码时，使用

   ```mysql
   INSERT INTO users(name,password) VALUES ('tom',PASSWORD('mgeedu')),('jerry',PASSWORD('mageedu'));
   ```

7. 创建一个系统用户 指定家目录！ 约定俗称，在子目录创建一个pub目录！，该用户用于充当被映射的匿名用户！

8. 提供pam配置文件：vsftpd.vusers

   > auth required /usr/llib64/security/pam_mysql.so usr=用户名 passwd=密码 host=数据库主机 db=数据库名 table=表名 usercolum=用户名字段名 passwdcolumn=密码字段名 cypt=2（指明加密类型）
   >
   > #意思是，认证时，使用pam_mysql.so这个模块，使用vsftp用户，magedu密码登录到127.0.0.1这个主机的mysql上，在vsftpd这个数据库中的users表中，user字段为name，passwd字段为passwd，使用2这个类型加密（2表示使用mysql内置的passwd函数加密，可以在编译安装的帮助文档中查看）
   >
   > account required /usr/lib64/security/pam_mysql.so usr=用户名 passwd=密码 host=数据库主机 db=数据库名 table=表名 usercolum=用户名字段名 passwdcolumn=密码字段名 cypt=2（指明加密类型）
   >
   > 这个表示账号检查

10. 配置vsftpd，添加第六条写明的配置

11. 用户对自己家目录不能有写权限，所以还要将用户对家目录的写权限拿掉！

12. 虚拟用户的配置文件

    > user_config_dir=/etc/vsftpd/vusers_config/	指明一个目录路径，该路径下为每个用户提供一个配置文件，使用匿名用户指令，控制单个用户的权限！

