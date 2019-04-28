[TOC]

# 计划任务

linux的计划任务可以分为两类！ 

一类是在指定时间执行一次的任务，例如，我想在今天晚上十二点重启一次机器

一类是需要基于每一分钟，每一天，每一周，每一个月，每一年的某个时间点，周期性执行的，例如，我想每天的晚上十二点，例如，我想每天晚上十二点备份一下某个文件！

这两类任务的实现方法有所不同！

## 单次执行的任务计划

### at命令

用法：  at [option]... TIME 该命令执行后，进入编辑模式，编辑完成后，使用ctrl+d提交

其中，TIME的格式有如下写法：

> HH:MM:[YYYY-mm-dd]
>
> noon：中午十二点
>
> midnight：午夜十二点
>
> teatime：下午四点
>
> tomorrow：二十四小时后
>
> now+#：多久之后，#的单位可以是minutes、hours、days、weeks

进入交互式模式后，每行一个命令即可！ 编辑完提交！ atd守护进程就会在你指定的时间，执行你编辑的操作命令！如果想进行比较复杂的操作可编写一个脚本，而后在这里指定时间启动脚本即可！

常用选项：

> -l：查看任务队列，也可以使用atq命令查看
>
> -f： -f /PATH/TO/SOMEFILE 从指定文件中读取作业，而非交互式输入： 该文件直接输入要执行的命令，每条命令一行即可
>
>  -d #  删除第#号任务，相当于atrm 
>
> -c # 查看指定任务的具体内容（内容中包含执行环境的一系列环境变量）

注意：at有自己的运行环境，所以有些命令可能需要指定绝对路径，不能只给一个简单的命令名

示例，指定一个计划，在今晚十二点执行关机操作：

```bash
[root@localhost rpm]# systemctl start atd.service
[root@localhost rpm]# at midnight
at> poweroff   
at> <EOT>
job 2 at Fri Apr 19 00:00:00 2019
```

注意：at工具，依赖于atd服务，所以安装后，需要将atd服务启动起来方可使用！

### batch命令

该命令和at命令用法基本一致，知识不需要再加时间参数，它会自行选择一个空闲的时间，执行用户所编辑的命令

# 周期性计划任务

为了完成周期性任务计划的目的，需要一个服务程序始终不间断的去监听查看对应的时间点是否满足条件去执行的，该服务名为 cronie（也是主程序包），该程序包提供了crond守护进程及相关辅助工具，在指定周期性计划任务之前，请先确保系统上的crond服务正常运行

周期性计划任务分为两类，root用户的和普通用户的！

其中系统计划任务需要编辑配置文件/etc/crontab： 主要用来实现系统自身的维护

注意，该文件仅为管理员编写任务计划，支持指定使用哪个用户执行任务！

而普通用户，使用crontab命令编辑，使用ctrl+d提交即可！普通用户的配置文件放置在/var/spool/cron/USERNAME

## 配置文件

格式：

```
# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed
```

/var/spool/cron/USERNAME 文件中的，就不需要指明username了！ 其他都是一样的

> user-name：运行任务的用户身份
>
> command to be executed： 任务
>
> \* \* \* \* \*:定义周期性时间,星号表示匹配范围内的任意时间点，所以如果间给一个任务的时间定义为五个信号，表示每月每日每时每分都执行一次任务！

时间表示法：

* 特定值

  直接给定时间点内有效范围内的值

  注意，day of week 和 day of month 一般不要同时使用

* \*

  不指定值，用默认的\*号代替，有效取值范围内的所有值，表示每...

  > 3 * * * *  表示每小时执行一次，每小时的第3分钟
  >
  > 3 4 * * 5  每周五的四点三分执行一次
  >
  > 5 6 7 * *   每月七号的六点5分执行一次
  >
  > 7 8 9 10 *  每年10月9日8点7分执行一次

* 离散取值，在一个时间点上行，使用逗号隔开多个值

  > 9 8 * * 3,7    每周3和周日的8点9分执行一次
  >
  > 8 8,20 * * 3,7，每周三和周日的 八点过八分和二十点过八分执行一次，注意第一位不能为*，如果为\* 就成了没周三和周日的八点和二十点内，没分钟执行一次了！

* 连续取值： -

  在时间点上使用-符号连接开始和结束时间

  > 0 9-18 * * 1-5  工作日白天九点到十八点每小时执行

* 在指定时间点上，定义步长/#:  #即步长

  > */5 * * * *  每五分钟执行一次

  应该很好理解！

注意：指定的时间点不能被步长整除时，其意义将不复存在

## crontab命令

crontab [-u user] [-l | -r] ： 都是可选项，默认进入交互式模式，对于普通用户来说，直接输入时间点和执行任务即可！

常用选项：

> -l：列出自己的所有任务
>
> -e：编辑模式（能够自行检测语法）
>
> -r：移除所有任务，即删除/var/spool/cron/USERNAME文件,想单个删除，使用-e编辑文件即可，修改立即生效
>
> -i：在使用-r选项时，提示用户确认，避免出错
>
> -u user：root用户可为指定用户编辑任务

注意： 运行结果默认以邮件通知当前用户：如果拒绝接受邮件

​            1：COMMAND > /dev/null

​            2:  COMMAND &> /dev/null

定义COMMAND时，如果命令中需要用到%，需要对其转义，或者加单引号

## 示例

1. 每12小时备份并压缩/etc目录至/backup目录中，保存文件名称格式为"etc-年-月-日-时-分.tar.gz"

```bash
[root@localhost cron]# vim /etc/crontab
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed
0 0 * * * root tar -zcf /backup/etc-$(date "+%Y-%m-%d-%H-%M").tar.gz /etc/
```



# 压缩和归档工具

常见的几种压缩/解压工具及文件格式

| 压缩工具 | 解压工具 | 文件格式 |
| -------- | -------- | -------- |
| gzip     | gunzip   | .gz      |
| bzip2    | bunzip   | .bz2     |
| xz       | unxz     | .xz      |
| zip      | unzip    | .zip     |

## gzip

gzip/gunzip/zcat    压缩/解压缩/直接查看压缩过的文件内容

不加任何参数，默认将文件压缩/解压后删除源文件

用法：gzip [options]...  file...

常用选项     

> -d：解压缩，相当于gunzip
>
> -#：指定压缩比，默认6，数字越大，压缩比越大，范围1-9 建议不修改
>
> -c：将压缩结果输出至标准输出： gzip -c FILE > /PATH/TO/SOMEFILE.gz

## bzip2

bzip2/bunzip2/bzcat：压缩/解压缩/直接查看压缩文件的内容

用法： bzip2 [options]...  file...

常用选项：

> -d：解压缩，相当于bunzip2
>
> -#：指定压缩比，默认6
>
> -k：keep，保留源文件，这里就不用重定向了

## xz

xz/unxz/xzcat：压缩/解压缩/直接查看压缩文件的内容

用法：xz [options]... file...

常用选项：

> -d：解压缩，相当于unxz
>
> -#：指定压缩比，默认6
>
>  -k：keep，保留源文件

以上工具都不具备压缩目录的功能，如果想压缩目录，需要先将目录打包，归档工具就上场了：

## 归档工具 tar

功能，将目录打包成为一个文件

用法： tar [OPTION]... FILE....

常用选项：

> -c 创建归档
>
> -x 展开归档
>
> -f 指定保存文件路劲及文件名
>
> -z： 归档后或展开归档后指定gzip工具进行压缩或者解压，解压时可以省略，会自动识别
>
>  -j：归档后或展开归档后使用bzip2工具压缩或者解压
>
>  -J：归档后或展开归档后使用xz工具压缩或者解压

日常用法：

>  **-zcf /PATH/TO/SOMEFILE.tar.gz FILE...**
>
> **-zxf /PATH/TO/SOMEFILE.tar.gz FILE...**
>
> -cf /PATH/TO/SOMEFILE.tar FILE... 创建归档
>
> -xf  /PATH/TO/SOMEFILE.tar   展开归档
>
> -xf  /PATH/TO/SOMEFILE.tar -C /PATH/TO/SOMEDIR   展开到指定目录
>
> -tf /PATH/TO/SOMEFIEL.tar  查看归档文件的文件列表





最后附一个脚本练习：

写一个脚本，列出一下菜单给用户，输入对应序号，执行对应操作

1）disk：打印磁盘信息

2）mem：打印内存信息

3）cpu：打印cpu信息

q）quit：退出脚本

执行完毕后，继续等待用户操作，知道输入q退出脚本！

```shell
[root@localhost scripts]# cat ./info.sh
#!/bin/bash

while true;do
	echo -e "please enter a number \n1):disk\n2):mem\n3):cpu\nq):quit"
	read  -p "please enter" value
	if [[ $value -eq 1 ]]
		then
			fdisk -l | grep "/dev.*"
		elif [[ $value -eq 2 ]]
		then
			free
		elif [[ $value -eq 3 ]] 
		then
			lscpu
		elif [[ $value == q ]]
		then
		echo "bye";exit
		else 
		echo -e "input error,please re enter\n\n" 
fi
done

```

