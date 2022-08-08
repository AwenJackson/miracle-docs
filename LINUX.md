## 安装mysql

查看是否安装mysql

```apl
rpm -qa | grep mysql
```

如果你系统有安装，那可以选择进行卸载:

```apl
rpm -e mysql　　// 普通删除模式
rpm -e --nodeps mysql　　// 强力删除模式，如果使用上面命令删除时，提示有依赖的其它文件，则用该命令可以对其进行强力删除
```

下载mysql启动程序

```apl
wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm
```

使用yum进行安装

```apl
rpm -ivh mysql-community-release-el7-5.noarch.rpm
```

更新yum

```apl
yum update
```

使用yum命令安装mysql

```apl
yum install mysql-server
```

设置权限

```apl
chown -R mysql:mysql /var/lib/mysql/
```

初始化mysql

```apl
mysqld --initialize
```

启动mysql

```apl
systemctl start mysqld
```

查看mysql运行状态

```apl
systemctl status mysqld
```

修改`/etc/my.cnf`,在`[mysqld]`下面添加一下代码

```apl
#绕过mysql登录所需要输入的密码，直接使用`mysql -uroot -p`登录,输入密码时直接回车
skip-grant-tables
#mysql启动端口，可以默认设置为3306也可以设置为其他的端口
port = 3036
sql-mode=STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION
#设置mysql编码为utf8
character_set_server=utf8
init_connect='SET NAMES utf8'
```

```apl
vim /etc/my.cnf
```

```apl
# For advice on how to change settings please see
# http://dev.mysql.com/doc/refman/5.6/en/server-configuration-defaults.html

[mysqld]
skip-grant-tables
port = 3036
sql-mode=STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION
character_set_server=utf8
init_connect='SET NAMES utf8'
#
# Remove leading # and set to the amount of RAM for the most important data
# cache in MySQL. Start at 70% of total RAM for dedicated server, else 10%.
# innodb_buffer_pool_size = 128M
#
# Remove leading # to turn on a very important data integrity option: logging
# changes to the binary log between backups.
# log_bin
#
# Remove leading # to set options mainly useful for reporting servers.
# The server defaults are faster for transactions and fast SELECTs.
# Adjust sizes as needed, experiment to find the optimal values.
# join_buffer_size = 128M
# sort_buffer_size = 2M
# read_rnd_buffer_size = 2M
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock

# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

# Recommended in standard MySQL setup
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES 

[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

```

重启mysql

```apl
systemctl restart mysqld
```

登录mysql

```apl
[root@VM-4-5-centos local]# mysql -uroot -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.6.51 MySQL Community Server (GPL)

Copyright (c) 2000, 2021, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

输入`flush privileges;`并设置密码

```apl
mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

mysql> set password for root@localhost=password('new_password');
Query OK, 0 rows affected (0.00 sec)

mysql> exit
```

重启mysql

```apl
systemctl restart mysqld
```

登录mysql，在Enter password: 后面输入你设置的新密码

```apl
[root@VM-4-5-centos local]# mysql -uroot -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 5
Server version: 5.6.51 MySQL Community Server (GPL)

Copyright (c) 2000, 2021, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
```

设置可以进行远程连接

```apl
mysql> grant all privileges on *.* to root@'%' identified by 'root' with grant option;
```

重启mysql，并查看编码

```apl
[root@VM-4-5-centos local]# systemctl restart mysqld
[root@VM-4-5-centos local]# mysql -uroot -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 5
Server version: 5.6.51 MySQL Community Server (GPL)

Copyright (c) 2000, 2021, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> SHOW VARIABLES LIKE 'character%';
```

开放mysql端口

1.如果是远程服务器，则直接在对应的服务器上的防火墙开启端口

2.如果是本地虚拟机，则需要关闭防火墙

查看防火墙状态

```apl
systemctl status firewalld.service
```

停止防火墙

```apl
systemctl stop firewalld.service
```

永久关闭防火墙

```apl
systemctl disable firewalld.service
```

关闭SELinux安全机制

```apl
vim /etc/selinux/config
找到并修改: SELINUX=disabled
需要重启系统:reboot
```

## 安装nginx

一、在官网现在你想要的版本的nginx的包，小编使用的是nginx-1.21.6.tar.gz版本的包

二、开始安装nginx

切换到`/usr/local/`目录，并将你下载的jar包上传至当前目录,并解压压缩包

```cassandra
[root@VM-4-5-centos /]# cd /usr/local/
[root@VM-4-5-centos local]# ls -l
总用量 1116
drwxr-xr-x.  2 root root    4096 8月   5 2020 bin
drwxr-xr-x.  2 root root    4096 4月  11 2018 etc
drwxr-xr-x.  2 root root    4096 4月  11 2018 games
drwxr-xr-x.  2 root root    4096 4月  11 2018 include
drwxr-xr-x.  2 root root    4096 4月  11 2018 lib
drwxr-xr-x.  2 root root    4096 4月  11 2018 lib64
drwxr-xr-x.  2 root root    4096 4月  11 2018 libexec
-rw-r--r--   1 root root    6140 11月 12 2015 mysql-community-release-el7-5.noarch.rpm
drwxr-xr-x  11 root root    4096 7月   3 02:28 nginx
drwxr-xr-x   9 1001 1001    4096 7月   3 02:27 nginx-1.21.6
-rw-r--r--   1 root root 1073364 7月   3 02:22 nginx-1.21.6.tar.gz
drwxr-xr-x  14 root root    4096 7月   3 02:18 qcloud
drwxr-xr-x   3 root root    4096 4月  24 14:16 sa
drwxr-xr-x.  2 root root    4096 4月  11 2018 sbin
drwxr-xr-x.  5 root root    4096 3月   7 2019 share
drwxr-xr-x.  2 root root    4096 4月  11 2018 src
[root@VM-4-5-centos local]# tar zxvf nginx-1.21.6.tar.gz
```

前提准备，安装依赖包

```apl
yum install -y zlibyum install gcc-c++
yum install -y openssl openssl-devel zlib-devel
yum install -y pcre pcre-devel
yum install -y zlib zlib-devel
```

或者是

```apl
yum -y install gcc zlib zlib-devel pcre-devel openssl openssl-devel
```

如果在安装依赖包时如果出现error: rpmdb错误，则需要重新构建rpm数据库

```apl
cd /var/lib/rpm
rm -rf __db*
rpm --rebuilddb
```

切换到`nginx-1.21.6`目录，并实现编译

```apl
./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module
```

```apl
 make
 make install
```

查看nginx版本

```apl
/usr/local/nginx/sbin/nginx -V
```

切换到`/usr/local/nginx/sbin`目录

启动nginx

```apl
./nginx
```

刷新nginx

```apl
./nginx -s reload
```

停止nginx

```apl
./nginx stop
```

备份nginx.conf

```apl
cp /usr/local/nginx/conf/nginx.conf /usr/local/nginx/conf/nginx.conf.back
```

查看nginx启动端口

```apl
ps -ef|grep nginx
```

