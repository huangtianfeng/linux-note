[TOC]
# 通过ansible的roles功能批量部署zabbix3.0监控系统
以下roles，除了安装zabbix-web这一段，适应于centos6和7，zabbix-web的配置文件，由于6的httpd版本较低，有些指令无效，需要自行修改诸多指令
## zabbix-server端的安装过程
1. 在和playbook文件同级的目录下创建roles所需目录：
```
[root@node1 roles]# mkdir -pv ./zabbix/{files,handlers,tasks,templates,vars}
mkdir: 已创建目录 "./zabbix"
mkdir: 已创建目录 "./zabbix/files"
mkdir: 已创建目录 "./zabbix/handlers"
mkdir: 已创建目录 "./zabbix/tasks"
mkdir: 已创建目录 "./zabbix/templates"
mkdir: 已创建目录 "./zabbix/vars"
```
2. 准备工作，通过模板文件分别为centos6和centos7系统提供yum文件，指向zabbix官方仓库 http://repo.zabbix.com/zabbix/3.0/rhel/release/x86_64/和提供依赖包的仓库 http://repo.zabbix.com/non-supported/rhel/7/$basearch/
```
[root@node1 roles]# cd ./zabbix/templates/
[root@node1 templates]# vim yum.repo.j2
[zabbix]
name=Zabbix Official Repository - $basearch
baseurl=http://repo.zabbix.com/zabbix/3.0/rhel/{{ansible_distribution_major_version}}/$basearch/
enabled=1
gpgcheck=0

[zabbix-non-supported]
name=Zabbix Official Repository non-supported - $basearch
baseurl=http://repo.zabbix.com/non-supported/rhel/{{ansible_distribution_major_version}}/$basearch/
enabled=1
gpgcheck=0
```
3. 提供将模板文件复制到目标主机的task
```
[root@node1 tasks]# cat create_repos.yml 
- name: create repos
  template: src=yum.repo.j2 dest=/etc/yum.repos.d/zabbix.repo backup=yes
```
4. 安装数据库并启动之
```
[root@node1 tasks]# cat install_mysql.yml
- name: install mysql
  yum: name=mysql-server state=present
  when: ansible_distribution_major_version=="6"
- name: install mariadb
  yum: name=mariadb-server state=present
  when: ansible_distribution_major_version=="7"
- name: start mysql
  service: name=mysqld state=started
  when: ansible_distribution_major_version=="6"
- name: start mariadb
  service: name=mariadb state=started
  when: ansible_distribution_major_version=="7"
```
5. 安装zabbix-server-mysql和zabbix-get
```
[root@node1 tasks]# cat install_zabbix_server.yml
- name: install zabbix-server-mysql
  yum: name=zabbix-server-mysql state=present
- name: install zabbix-get
  yum: name=zabbix-get
```
6. 手动创建数据库zabbix，并创建一个zabbix用户并授权
```
[root@node1 tasks]# cat create_mysql_user.yml
- name: create_database
  shell: mysql -uroot -h127.0.0.1 -e "CREATE DATABASE IF NOT EXISTS zabbix"
- name: create_user
  shell: mysql -uroot -h127.0.0.1 -e "GRANT ALL ON zabbix.* TO 'zabbixtian'@'192.168.0.%' IDENTIFIED BY '507666715'"
```
7. 解压sql脚本，并导入，生成所需表
```
[root@node1 tasks]# cat mysql_script.yml
- name: scripts
  shell: gunzip -c /usr/share/doc/zabbix-server-mysql-3.0.28/create.sql.gz > /root/data/create.sql
- name: import
  shell: mysql -uroot -h127.0.0.1 zabbix</root/data/create.sql
```
8. 提供zabbix-server的配置文件，修改数据库指向，登录用户和口令等相关配置，并启动之(因为都是用的固定的用户名端口，server端也只需要一台，所以使用文件，而没有使用模板）
```
[root@node1 tasks]# cat start_zabbix_server.yml 
- name: config zabbix server
  copy: src=zabbix_server.conf dest=/etc/zabbix/zabbix_server.conf backup=yes
- name: start server
  service: name=zabbix-server state=started
```
9. 安装zabbix-web并启动
```
[root@node1 tasks]# cat install_zabbix_web.yml
- name: intall zabbxi-web
  yum: name=httpd,php,php-mysql,php-mbstring,php-gd,php-bcmath,php-ldap,php-xml,zabbix-web,zabbix-web-mysql
- name: config
  template: src=zabbix.conf.j2 dest=/etc/httpd/conf.d/zabbix.conf backup=yes
- name: start
  service: name=httpd state=started
```
10. 将以上文件，按照顺序，在main.yml文件中，使用include指令包含，如下所示
```
[root@node1 playbook]# cat ./roles/zabbix/tasks/main.yml 
- include: create_repos.yml
- include: install_mysql.yml
- include: install_zabbix_server.yml
- include: create_mysql_user.yml
- include: mysql_script.yml
- include: start_zabbix_server.yml
- include: install_zabbix_web.yml
```
11. 编辑playbook文件，使用roles指令调用角色名（roles的目录名）
```
[root@node1 playbook]# cat zabbix-server.yml 
---
- hosts: 192.168.0.104
  remote_user: root

  roles: 
    - role: zabbix
```

## zabbix-agent端的安装过程
1. 提供zabbix-agent角色目录
```
[root@node1 playbook]# mkdir ./roles/zabbix-agent/{files,tasks,templates,vars,handlers} -pv
mkdir: 已创建目录 "./roles/zabbix-agent"
mkdir: 已创建目录 "./roles/zabbix-agent/files"
mkdir: 已创建目录 "./roles/zabbix-agent/tasks"
mkdir: 已创建目录 "./roles/zabbix-agent/templates"
mkdir: 已创建目录 "./roles/zabbix-agent/vars"
mkdir: 已创建目录 "./roles/zabbix-agent/handlers"
```
2. 提供yum仓库文件，此处我们调用server端已经提供的模板文件
```
[root@node1 tasks]# cat 01-create_repos.yml
- name: create repos
  template: src=/root/playbook/roles/zabbix/templates/yum.repo.j2 dest=/etc/yum.repos.d/zabbix.repo backup=yes
```
3. 安装zabbix-agent,提供配置文件，并启动之
```
[root@node1 tasks]# cat 02-install_zabbix_agent.yml
- name: install agent
  yum: name=zabbix-agent state=present 
- name:  configuration
  template: src=zabbix_agentd.conf.j2 dest=/etc/zabbix/zabbix_agentd.conf backup=yes
  notify: restart server
- name: start agent
  service: name=agent state=started
```
4. 在handlers目录提供上一步所需handler
```
[root@node1 tasks]# cat ../handlers/main.yml
- name: restart server
  service: name=agentd state=restarted
```
5. 提供第三步所需要的模板文件，按需修改如下指令
> Server=192.168.0.104： 授权哪台主机允许获取本机信息
> ServerActive=192.168.0.104：授权允许向哪台主机主动发送本机信息
> Hostname=：在zabbix监控界面被识别的主机名，这里为了图方便，就不设置了，默认会自动获取本机主机名

6. 提供main文件，包含tasks
```
[root@node1 tasks]# cat main.yml
- include: 01-create_repos.yml
- include: 02-install_zabbix_agent.yml
```
7. 编辑playbook文件
```
[root@node1 playbook]# cat zabbix-agent.yml
- hosts: zabbix-agent
  remote_user: root
 
  roles: 
    - role: zabbix-agent
```
8. ansilbe-playbook  zabbix-agent.yml
至此，zabbix的服务端和代理端都安装完毕，接下来我们尝试配置zabbix的自动发现功能，将当前所有主机纳入监控。

## 通过discovery功能，配置监控主机
1. 设置发现规则：通过ssh端口扫描发现主机
![插图24-1](https://github.com/huangtianfeng/pictures/blob/master/linux%E5%9B%BE%E5%BA%93/%E6%8F%92%E5%9B%BE24-1.png)
2. 设置执行动作：将扫描到的主机添加进监控，并链接至内置模板OS linux
![插图24-2](https://github.com/huangtianfeng/pictures/blob/master/linux%E5%9B%BE%E5%BA%93/%E6%8F%92%E5%9B%BE24-2.png)
![插图24-3](https://github.com/huangtianfeng/pictures/blob/master/linux%E5%9B%BE%E5%BA%93/%E6%8F%92%E5%9B%BE24-3.png)
![插图24-4](https://github.com/huangtianfeng/pictures/blob/master/linux%E5%9B%BE%E5%BA%93/%E6%8F%92%E5%9B%BE24-4.png)
4. 启动发现功能并检查
![插图24-5](https://github.com/huangtianfeng/pictures/blob/master/linux%E5%9B%BE%E5%BA%93/%E6%8F%92%E5%9B%BE24-5.png)

![插图24-6](https://github.com/huangtianfeng/pictures/blob/master/linux%E5%9B%BE%E5%BA%93/%E6%8F%92%E5%9B%BE24-6.png)
可以看到，所有主机均已被监控

项目完成

