# 			Mantis安装与备份

## 					Manits安装

### 安装工具

1. 安装apache2

``` 
$ sudo apt-get install apache2 libapache2-mod-php5
```

2. 安装Mysql
```
$ sudo apt-get install -y mysql-server mysql-client libmysqlclient-dev
```

3. 安装php
```
$ sudo apt-get install php5-mysql  php5
```

### 修改配置文件

1. ### 修改apache2配置文件

   在/etc/apache2/httpd.conf或者apache2.conf中添加：
   AddType application/x-httpd-php .php
   AddType application/x-httpd-php .html
   LoadModule php5_module /usr/lib/apache2/modules/libphp5.so

2. 修改php5配置文件

   在/etc/php5/apache2/php.ini中添加
   extension=mysql.so
   extension=gd.so

### 安装Mantis

#### 使用备份Mantis恢复

1. 使用备份的mantis恢复

mantisbt目录

​		a.  把之前的mantisbt目录拷贝过来放到本机的者/var/www/html/

​		b. 修改mantisbt目录的属主：
```
$ cd /var/www/html
$ chown -R www-data:www-data mantisbt
```

2. 数据库恢复

   a. 创建数据库	

```
$ create database bugtracker;
```

​        	b. 导入备份的数据库(创建数据后先不要使用新系统的mantis来初始化，不然再导入会报错)	
```
$ mysql -uroot -ppasswd
mysql> create database bugtracker;		
mysql> use bugtracker;	
mysql>source /root/bugtracker.sql;
```

3. 修改mantis的配置文件：

​	修改配置文件mantisbt/config_inc.php中连接数据库使用的用户名，密码

重启apache2
```
$ service apache2 restart
```
#### 第一次安装

1. 下载

​		地址：地址为http://www.mantisbt.org/download.php， 下载后解压缩到/var/www/或者/var/www/html/，看自己的apache设置的根目录

2. 建立数据库

```
$ mysql -u root -p，进入mysql提示符
mysql> create database bugtracker;
mysql> grant all privileges on bugtracker.* to root@localhost identified by 'password';
mysql> flush privileges;
mysql> \q
```

3. 创建mantis配置文件
```
$ sudo cp /var/www/html/mantisbt/config_inc.php.sample /var/www/html/mantisbt/config_inc.php
```
编辑这个文件添加 $g_default_language = 'chinese_simplified';

​		mantis新版本的config_inc.php.sample可能在mantisbt/config中

4. 创建数据库表

```
$ mysql -uroot -p bugtracker < /var/www/html/mantisbt/library/adodb/session/adodb-sessions.mysql.sql
```

​		新版本文件可能在/var/www/html/mantisbt/vendor/adodb/adodb-php/session/adodb-sessions.mysql.sql

4. 重启apache2

```
$ service apache2 restart
```

访问路径：http://IP/mantisbt/view_all_bug_page.php

Mantis安装后默认用户:administrator 密码:root



## 					Mantis 备份

1. 备份数据库：
```
mysqldump -uroot -p123456 bugtracker > bugtracker.sql
```

2.  备份mantis整个目录