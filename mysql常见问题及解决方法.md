1. 在centos6中，安装好mysql-server，启动失败，错误日志：

   > 190422 20:45:11  InnoDB: Started; log sequence number 0 44233
   > 190422 20:45:11 [ERROR] Fatal error: Can't open and lock privilege tables: Table 'mysql.host' doesn't exist
   > 190422 20:45:11 mysqld_safe mysqld from pid file /var/run/mysqld/mysqld.pid ended

   提示无法打开权限表，mysql.host表不存在：

   **解决方法**

   运行mysql_install_db， 以重新安装mysql.host等相关依赖

