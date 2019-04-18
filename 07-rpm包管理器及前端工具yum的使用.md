[TOC]

# rpm包管理器

## rpm包和RPM

rpm包：什么是rpm包呢？

为了方便用户的使用，软件开发商在固定的硬件平台与操作系统平台上将程序源代码编译成为二进制格式目标代码，然后将这个软件的所有相关代码文件和该软件所提供的所有文件信息，以及检测系统与依赖关系的脚本等一系列文件，打包成为一个特殊格式的软件安装文件，分发给用户安装，这个特殊格式的安装文件，目前比较主流的有两种格式，一种是deb格式安装包（主要用于Debian系列的发行版上），另外一种，就是咱们的主角，rpm格式的安装包，也就是rpm包！

* rpm包的通用格式：

  name-VERSION-release.arch.rpm

  name：名字

  VERSION：版本号，一般包括主版本号和次版本号用点号隔开

  release：rpm的打包次数，当一个软件发现某些bug，或者更新时，就需要重新编译打包！

  arch：使用的平台，例i386、i586、i686、noarch、x86_64

  rpm：扩展名！

一个程序包，一般会提供很多功能，但是有些用户使用的却只是其中的一个功能，而不得不安装整个程序包，这个怎么办呢？ 答案是，将一个程序包拆分成多个子包，整个程序由一个主程序包和众多提供各功能的子包组成！

* 子包的通用格式：

  name-function-VERSION-release.arch.rpm

  相对于主包，就多了一个function：

  例如常见的

  devel（development）：开发组件

  utils：工具组件

  libs：库文件		

而所谓程序包管理器（rpm），全称是ReadHat Package Manager，顾名思义，该软件管理器是由Red Hat公司研发出来的，**RPM是以一种数据库记录的方式，来将你所需要的程序包，安装到你的linux系统中的一套软件管理机制！**它只能用于安装rpm格式的程序包，安装的时候，会检测rpm所提供的依赖信息，如果当前系统不满足，则报错，不予安装（当然你可以强制安装，但是不保证能顺利运行），如果满足，则安装，并将该包提供的所有信息，整个写入RPM的数据库中（/var/lib/rpm/目录下的所有文件），以便于将来的查询，验证，卸载操作！

一般使用rpm安装的软件，配置文件会放置在/etc目录下，执行文件放置在/usr/bin目录下，程序使用的动态函数库放置在/usr/lib或者/usr/lib64目录下，帮助文档放置在/usr/share/man目录下，还有些说明文件放置在/usr/share/doc目录下！

总结：RPM的优点

* RPM包内包含已经编译好的程序与配置文件等数据，用户无需重新编译
* 安装前会检查硬盘容量、系统版本、等，可避免文件被错误的安装
* RPM包提供软件版本信息，依赖属性检查、软件用途说明、软件所含文件等信息，便于了解查询软件
* RPM管理的方式，使用数据库记录RPM文件的相关信息，便于升级，卸载，查询和验证

## rpm工具的使用

只有管理员才有权限安装软件哦！

### 安装：

rpm [-i|--install] [install-options] PACKAGE_FILE ……

* install-options：   

  -h: hash marks输出进度条：每个#表示2%的进度：

  --test：测试安装，检查并报告依赖关系及冲突关系

  --nodeps： 忽略依赖关系

  --replacepkgs： 重新安装，该操作不会替换掉配置文件

  --nosignature：不检查包签名信息，即不检查来源合法性

  --nodigest：不检查包完整性信息：

  --noscripts：不执行rpm包自带的脚本

  > rpm包可自带四类脚本，分别是：
  >
  > preinstall：安装过程开始之前运行的脚本，%pre    使用--nopre单独禁用
  >
  > postinstall： 安装过程完成后运行的脚本，%post    使用--nopost单独禁用
  >
  > preuninstall：卸载过程真正开始执行之前运行的脚本：%preun    使用--nopreun单独禁用
  >
  > postuninstall：卸载过程完成之后运行的脚本，%postun    使用--nopostun单独禁用

  
### 升级：

  升级或安装：

  rpm [-U|--upgrade] [install-options] PACKAGE_FILE……

  仅升级（如果指定包不存在，则略过）：

  rpm [-F|--freshen] [install-options] PACKAGE_FILE……

  * install-options：   

    -h: hash marks输出进度条：每个#表示2%的进度：

    --test：测试安装，检查并报告依赖关系及冲突关系

    --nodeps： 忽略依赖关系

    --replacepkgs： 重新安装，该操作不会替换掉配置文件

    --nosignature：不检查包签名信息，即不检查来源合法性

    --nodigest：不检查包完整性信息：

    --noscripts：不执行rpm包自带的脚本

    > rpm包可自带四类脚本，分别是：
    >
    > preinstall：安装过程开始之前运行的脚本，%pre    使用--nopre单独禁用
    >
    > postinstall： 安装过程完成后运行的脚本，%post    使用--nopost单独禁用
    >
    > preuninstall：卸载过程真正开始执行之前运行的脚本：%preun    使用--nopreun单独禁用
    >
    > postuninstall：卸载过程完成之后运行的脚本，%postun    使用--nopostun单独禁用

    --oldpackage： 降级软件包

    --force：强制升级

  注意：

  1. 不要对内核进行升级操作：linux支持多内核共存，因此可直接安装新版内核即可！
  2. 如果某源程序包的配置文件安装后，曾被修改过，升级时，新版本的程序提供的同一个配置文件不会覆盖原有版本的配置文件，而是把**新版本的配置文件重命名（filename.rmpnew）后保留**

### 卸载：

rpm [-e|--erase] [--altmatches] [--nodeps] [--noscripts] [--test] PACKAGE_NAME ……

​	--allmatches：卸载所有匹配指定名称的各版本

​	--nodeps：忽略依赖关系

​	--noscripts： 不执行卸载时执行的脚本（包括卸载前和卸载后）

​	--test： 测试卸载，dry run模式，空跑一遍，并不会真卸载！

### 查询

rpm [-q|--query] [select-options] [query-options]

* select-options

  -a，--all：查询所有已经安装过的包

  -f FILE： 查询指定的文件由哪个程序包安装生成

  --whatprovides CAPABILITY:查询指定的CAPABILITY由哪个程序包提供

  示例：

  ```bash
  [root@localhost rpm]# rpm -q --whatprovides /usr/sbin/httpd
  httpd-2.4.6-88.el7.centos.x86_64
  [root@localhost rpm]# rpm -q --whatprovides /bin/ls
  coreutils-8.22-21.el7.x86_64
  ```

  --whatrequires  CAPABILITY: 查询指定的capbility被哪个包所依赖

  -p，--package PACKAGE_FILE:对未安装的程序包执行查询

* query-options

  --changelog： 查询rpm包的changlog，可以查看软件包的历史更新及其更新的特性！

  -l，--list：程序包安装生成的所有文件列表

  -i，--info：程序包相关的信息，版本号，大小，所属的包组等

  -c；--configfiles: 查询指定的程序提供的配置文件

  -d，--docfiles：查询指定的程序包提供的文档

  -provides：列出指定的程序包提供的所有CAPBILITY

  -R，--requires：查询指定的程序包的依赖关系

  --scripts：查看程序包自带的脚本片段

### 校验：

rpm [-V|--verify] [select-options] [verify-options]

包来源合法性验证和完整性验证

这个是一个超有用的功能，可以有效防止黑客入侵哟

-V：

-Va：例出目前系统上可能被修改过的文件（通过和rpm数据库中的校验值对比）

-Vp： 后面跟软件包名：列出该包中可能被修改过的文件

-Vf：显示某个文件是否被修改过

以上内容，知识将系统中的文件和rpm包所提供的校验信息做比对，如果rpm包在安装前就被人动了手脚，则检验不出来，如果要检验rpm包来源的合法性，需要安装rpm包发行商的公钥！ 这个话题，暂且不谈！

### RPM数据库重建:

选项：

--initdb:初始化数据库，当前无任何数据可实时化创建一个新的：当前有时不执行任何操作

--rebuilddb:重新构建，通过读取当前系统上所有已经安装过得程序包进行重新创建

# yum工具介绍和使用

yum工具是RPM的前端管理工具，它的功能是自动解决RPM属性依赖的问题，所以yum需要依赖于RPM而存在

它通过检查rpm包的表头信息，也就是依赖关系，而后自动处理之，缺什么装什么！（当然，前提是软件仓库中有这个软件），所以，我们使用yum，首先得需要一个yum仓库的服务器！也就是软件源！ 一般，这个软件源有系统发行商提供！一般CentOS系统默认的就有提供！ 但是该源是国外的！ 最好自行修改成为国内的镜像站，例如阿里的，163的等等，这样软件安装的速度会快上不少！

yum的工作过程：yum客户端-->访问配置文件指定的服务器-->下载软件仓库元数据至本地缓存区域-->分析是否存在用户请求的包名-->分析依赖关系-->查询本地已安装的安装包-->安装尚未安装的程序

软件仓库存储了该仓库的所有程序包和其依赖关系

下载下来的元数据文件一般不删除，下次使用时，只请求服务器的元数据特征码，进行比对，如果有更新，才重新下载元数据文件，否则直接使用本地元数据文件进行分析

注意：yum仓库，首先得是一个文件服务器：支持ftp，http，nfs，file等

## yum源的配置

yum的配置文件：

/etc/yum.conf：该文件为所有仓库提供公共配置（仓库可以配置不止一个）

配置文件格式：

> [main]： 固定值
>
> cachedir=/var/cache/yum/$basearch/$releasever：指定yum的缓存数据存储位置
>
> keepcache=0： 指定安装完成后是否保留程序包，0为不保留
>
> debuglevel=2:
>
> logfile=/var/log/yum.log：日志文件保存位置
>
> exactarch=1： 当安装已经安装的包时，是否会报错，1表示不报错，0表示会报错
>
> obsoletes=1：
>
> gpgcheck=1: 是否检验来源合法性，可以被单独的仓库配置中的选项覆盖！
>
> plugins=1:是否启用插件，1为允许
>
> installonly_limit=5
>
> bugtracker_url=http://bugs.centos.org/set_project.php?project_id=19&ref=http://bugs.centos.org/bug_report_page.php?category=yum
>
> distroverpkg=centos-release

大致了解一下就好！一般也很少来配置

/etc/yum.repos.d/*.repo： 该目录下，所有以.repo结尾的文件，都被yum.comf配置文件包含进来！为每一个特定的仓库提供配置选项，配置文件格式如下所示：

必要配置项：

> [repositoryID]：自定义仓库ID，例如base，update，extras等，这个ID在众多仓库中，必须唯一，不能重复
>
> name=some name for this repository：自定义仓库名
>
> baseurl=url://path/to/repository：仓库服务器的url，reposdata所在的路径，就是url需要指向的路径
>
> enabled=[1|0]：是否启用该仓库！

可选配置项：

> gpgcheck=[0|1]： 是否检验rpm包来源的合法性，也就是/etc/pki/rpm-gpg/目录下的一个公钥，解密数字签名，比对校验
>
> gpgkey=/path/to/file： 指明公钥的位置，使用默认值就好，一般指向/etc/pki/rpm-gpg/下的一个文件（一般是CentOS发行商的公钥，因为rpm包一般都是由发行商制作的！）
>
> cost=： 设定仓库的优先级，默认一千，越小优先级越高

常用的一般就是上面这些选项了！

yum的repo配置文件中还有可以使用的变量：

> $releasever：当前OS的发行版的主版本号：
>
> $arch：平台
>
> $basearch：基础平台
>
> $YUM0-$YUM9： 自定义变量

## yum工具的使用

语法格式：yum [options] [command] [package ...]

常用OPTIONS：（注意，选项的优先级大于配置文件）

> --nogpgcheck：禁止进行gpg check：
>
> -y：自动回答为"yes"
>
> -q： 静默模式
>
> --disablerepo=repoidglob：临时禁用此处指定的repo
>
> --enablerepo=repoidglob：临时启用此处指定的repo
>
> --noplugins：禁用所有插件

command：

* 显示仓库列表：yum repolist [all|enabled|disabled]

* 显示程序列表：yum list

  支持使用通配符：yum list [all|glob_exp1] [glob_exp2] [……]

  支持查询可用的，已安装的，可升级的：yum list [all|glob_exp1] [glob_exp2] [……]

* 安装程序包：yum install package1 [package2] [...]

* 重新安装程序包：yum reinstall package1 [package2] [...]

* 升级程序包：yum update package1 [package2] [...]

* 降级：yum downgrade package1 [package2] [...]

* 检查可用升级：yum check-update

* 卸载程序包：yum remove | erase package1 [package2] [...]

* 查看程序包：yum information：* info [...]

* 查看指定的特性（可以是某文件）是由哪个程序包所提供：yum provides | whatprovides feature1 [feature2] [...]

* 清理本地缓存：yum clean [ packages | metadata | expire-cache | rpmdb | plugins | all ]

* 构建缓存：yum makecache

* 搜索：yum search string1 [string2] [...]：以指定的关键字搜索程序包名及summary信息

* 查看指定包所依赖的capabilities：yum deplist package1 [package2] [...]

* 查看yum事物历史：

  history    [info|list|packages-list|packages-info|summary|addon-info|redo|undo|rollback|new|sync|stats]

* 包组管理的管理命令

  > grouplist [hidden] [groupwildcard] [……]：列出包组
  >
  > groupinstall group1 [group2] [……]：安装包组
  >
  > groupupdate group1 [group2] [……]：升级包组
  >
  > groupremove group1 [group2] [……]：移除包组
  >
  > groupinfo group1 [……]：包组信息查看

## createrepo命令：仓库元数据生成

用法： createrepo [options] <directory>

常用options：

> -o,--outputdir： 指明元数据输出位置
>
> --basedir=路径：当元数据和包文件不再一个目录下是，可以使用该选项，指定包所在的目录路径！

一般，我们只需简单的指定目录即可，选线先不管那么多！

## 实例演示

1. 自行配置一个yum仓库源，以搜狐镜像站为例，地址是<http://mirrors.163.com>

我们可以在原有的配置文件上加，也可以直接自行新建一个配置文件，这里，我们把原有的配置文件移除，然后自行新建一个！

```bash
[root@localhost yum.repos.d]# mv centos.repo{,.bak}
[root@localhost yum.repos.d]# vim souhu.repo
[souhu]
name=souhumirrors
baseurl=http://mirrors.163.com/centos/6.10/os/x86_64/
enable=1
gpgcheck=1
```

注意，配置完成后，需要先将缓存清理，然后构建缓存才能正常使用哦！

```bash
[root@localhost yum.repos.d]# yum makecache
已加载插件：fastestmirror, refresh-packagekit, security
Determining fastest mirrors
souhu                                                                                                                                                                                     | 3.7 kB     00:00     
souhu/group_gz                                                                                                                                                                            | 242 kB     00:00     
souhu/filelists_db                                                                                                                                                                        | 6.4 MB     00:00     
souhu/primary_db                                                                                                                                                                          | 4.7 MB     00:00     
souhu/other_db                                                                                                                                                                            | 2.8 MB     00:00     
元数据缓存已建立
[root@localhost yum.repos.d]# yum repolist
已加载插件：fastestmirror, refresh-packagekit, security
Loading mirror speeds from cached hostfile
仓库标识                                                                                            仓库名称                                                                                                状态
souhu                                                                                               souhumirrors                                                                                            6,713
repolist: 6,713
```

看，这样就可以了！ 现在我们尝试安装一下ftp程序

```bash
[root@localhost yum.repos.d]# yum install ftp -y
已加载插件：fastestmirror, refresh-packagekit, security
设置安装进程
Loading mirror speeds from cached hostfile
解决依赖关系
--> 执行事务检查
---> Package ftp.x86_64 0:0.17-54.el6 will be 安装
--> 完成依赖关系计算

依赖关系解决

=================================================================================================================================================================================================================
 软件包                                         架构                                              版本                                                    仓库                                              大小
=================================================================================================================================================================================================================
正在安装:
 ftp                                            x86_64                                            0.17-54.el6                                             souhu                                             58 k

事务概要
=================================================================================================================================================================================================================
Install       1 Package(s)

总下载量：58 k
Installed size: 95 k
下载软件包：
ftp-0.17-54.el6.x86_64.rpm                                                                                                                                                                |  58 kB     00:00     
运行 rpm_check_debug 
执行事务测试
事务测试成功
执行事务
  正在安装   : ftp-0.17-54.el6.x86_64                                                                                                                                                                        1/1 
  Verifying  : ftp-0.17-54.el6.x86_64                                                                                                                                                                        1/1 

已安装:
  ftp.x86_64 0:0.17-54.el6                                                                                                                                                             
  

完毕！

```



2. 利用createrepo命令将本地光盘制作成为一个yum源，并配置为名为local的仓库！

```
[root@localhost /]# mkdir /media/local
[root@localhost /]# mount /dev/sr0 /media/local
mount: block device /dev/sr0 is write-protected, mounting read-only
[root@localhost /]# cd /media
[root@localhost media]#  createrepo -o . -v --basedir=/media/local/ /media/local
Saving Primary metadata
Saving file lists metadata
Saving other metadata
Generating sqlite DBs
Starting other db creation: Thu Apr 18 12:40:42 2019
Ending other db creation: Thu Apr 18 12:40:52 2019
Starting filelists db creation: Thu Apr 18 12:40:52 2019
Ending filelists db creation: Thu Apr 18 12:41:02 2019
Starting primary db creation: Thu Apr 18 12:41:03 2019
Ending primary db creation: Thu Apr 18 12:41:18 2019
Sqlite DBs complete
[root@localhost media]# ls
local  repodata
[root@localhost repodata]# vim /etc/yum.repos.d/souhu.repo 
[local]
name=localmirrors
baseurl=/media/
enable=1
gpgcheck=0
[root@localhost media]# yum repolist
已加载插件：fastestmirror, refresh-packagekit, security
Loading mirror speeds from cached hostfile
仓库标识                                                                                            仓库名称                                                                                                状态
local                                                                                               localmirrors                                                                                            3,243
souhu                                                                                               souhumirrors                                                                                            6,713
repolist: 9,956

```

注意： 因为我们的光盘是只能只读挂载的，所以我把元数据输出到另外的位置，并用basedir指明光盘位置！

至此，就ok了，可以尝试一下使用自制的yum源安装ftp、openssl、curl、wget、tcpdump等软件，安装过程我就不贴出来了，最后，我发现，系统的光盘镜像自带了repodata目录！  这就很尴尬了，所以，编辑配置文件，直接指向这个光盘挂载路径就好了，我前面基本白做了！

