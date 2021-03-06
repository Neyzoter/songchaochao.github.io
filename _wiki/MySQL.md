---
layout: wiki
title: MySQL
categories: SQL
description: MySQL的用法
keywords: MySQL
---



# 1、MySQL基础

## 1.1 术语

**数据库:** 数据库是一些关联表的集合。.

**数据表:** 表是数据的矩阵。在一个数据库中的表看起来像一个简单的电子表格。

**列:** 一列(数据元素) 包含了相同的数据, 例如邮政编码的数据。

**行：**一行（=元组，或记录）是一组相关的数据，例如一条用户订阅的数据。

**冗余**：存储两倍数据，冗余可以使系统速度更快。

**主键**：主键是唯一的。一个数据表中只能包含一个主键。你可以使用主键来查询数据。

**外键：**外键用于关联两个表。

**复合键**：复合键（组合键）将多个列作为一个索引键，一般用于复合索引。

**索引：**使用索引可快速访问数据库表中的特定信息。索引是对数据库表中一列或多列的值进行排序的一种结构。类似于书籍的目录。

**参照完整性:** 参照的完整性要求关系中不允许引用不存在的实体。与实体完整性是关系模型必须满足的完整性约束条件，目的是保证数据的一致性。

# 2、基本语法

（1）登录

```bash
# 启动mysql
$ systemctl start mysql.service
# 登录
$ mysql -u root -p;
# 退出
$ exit
# 关闭mysql
$ systemctl stop mysql.service
```

（2）显示目录
```bash
$ show 你想要显示的东西;

databases：所有数据库
tables：所有的表

```

（3）删/建库

```bash
# 删除
$ drop database 库名;
# 创建
$ create database 库名;
```

（4）使用库

```bash
$ use 库名;
```

（5）创建表

```bash
$ create table 表名(
	字段名 类型(长度) [约束],
	字段名 类型(长度) [约束],
	字段名 类型(长度) [约束]
);
```

eg.

```bash
$ mysql> create table config(
      -> test varchar(64) not null,
      -> isodate date,
      -> config varchar(5000)
      -> );
```

（6）插入记录

```bash
$ insert into 表名(列名1,列名2,列名3...) values(val1,val2,val3...)
```

（7）修改表记录

```bash
$ update 表名 set 字段名=值,字段名=值,字段名=值...
$ update 表名 set 字段名=值,字段名=值,字段名=值... where 条件
```

（8）删除表记录

```bash
drop 表名 where 条件
```

# 3.MySQL注意事项

## 3.1 大小写问题

| MySQL版本 | 操作对象              | 大小写情况                       |
| --------- | --------------------- | -------------------------------- |
| 5.7       | 命令行,Ubuntu16.04LTS | 默认区分大小写，可以设置成不区分 |
| 5.7       | `com.mysql.cj.jdbc`   | 不区分大小写，默认转化为小写     |

# 4.MySQL高级用法

## 4.1 MySQL缓存机制

**（1）查看缓存情况**

```text
mysql> show variables like '%query_cache%';
+------------------------------+---------+
| Variable_name                | Value   |
+------------------------------+---------+
| have_query_cache             | YES     |      --查询缓存是否可用
| query_cache_limit            | 1048576 |      --可缓存具体查询结果的最大值
| query_cache_min_res_unit     | 4096    |      --查询缓存分配的最小块的大小(字节)
| query_cache_size             | 599040  |      --查询缓存的大小
| query_cache_type             | ON      |      --是否支持查询缓存
| query_cache_wlock_invalidate | OFF     |      --控制当有写锁加在表上的时候，是否先让该表相关的 Query Cache失效
+------------------------------+---------+
6 rows in set (0.02 sec)
```

**（2）开启和关闭缓存**

开启缓存

```text
mysql> set global query_cache_size = 600000; --设置缓存内存大小
mysql> set global query_cache_type = ON;     --开启查询缓存
```

关闭缓存

```text
mysql> set global query_cache_size = 0; --设置缓存内存大小为0， 即初始化是不分配缓存内存
mysql> set global query_cache_type = OFF;     --关闭查询缓存
```

## 4.2 binlog

### 4.2.1 介绍

### 4.2.2 使用

（1）启动

```bash
# my.cnf文件配置binlog
# 默认在 /var/lib/mysql目录下以mysqld-bin.00000X等作为名称
# mysqld-bin.index记录了所有的log的文件名称
bin_log=mysql_bin
# 格式：STATEMENT、ROW、MIXED，主要用于主从复制
# 对应三种模式：基于SQL语句的复制(statement-based replication, SBR)，基于行的复制(row-based replication, RBR)，混合模式复制(mixed-based replication, MBR)
binlog_format=Row
# 服务器ID
server_id=1
# 追踪database操作
mysqlbinlog /var/lib/mysql|grep "*****"	
```

（2）配置格式

```bash
# 查看默认格式
mysql> show variables like 'character%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | latin1                     |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | latin1                     |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
# 更改格式
# 在my.cnf后面追加
collation-server=utf8_unicode_ci
init-connect='SET NAMES utf8'
character-set-server=utf8
sql_mode=NO_ZERO_IN_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
[client]
default-character-set=utf8
[mysql]
default-character-set=utf8
# 重启mysqld

# 查看格式
mysql> SHOW VARIABLES LIKE 'character%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | utf8                       |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | utf8                       |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+

# 设置过期天数
## 查看过期天数
mysql > show variables like '%expire_logs_days%'
## 设置过期天数
mysql > set global expire_logs_days=7;
```

（3）备份实现

周期性运行备份脚本，比如使用crontab

```bash
# 找到mysqldump位置
whereis mysqldump
# 运行mysqldump
# 周期性将test数据库保存到文件
# -l：读锁
# -F：flush logs指令，重新生成日志文件，方便后边恢复（从新的bignlog文件开始就可以）
/usr/bin/mysqldump -uroot -p123456 -l - F test > /tmp/test.sql
# 查看bin日志当前位置
# 可以记录下来，方便下次备份
mysql> show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql_bin.000002 |     5427 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
```

恢复数据

```bash
# 首先恢复sql备份
## -v：查看导入的详细信息
## -f：当中间遇到错误时，可以skip过去，继续执行下面的语句
mysql -uroot -p123456 test -v -f < /tmp/test.sql
# 查看binlog
## 直接查看文件（需要mysqlbinlog打开，cat等无法识别）
/usr/bin/mysqlbinlog --no-defaults  /var/lib/mysql/mysql_bin.000002 --start-position="794" --stop-position="1055" | more
## 通过mysql查看（更加直观）
mysql > show binlog events in 'mysql_bin.000002'
# 从binlog恢复没有备份的内容
## --no-defaults ： mysqlbinlog无法识别default-character-set=utf8这个指令，需要加上--no-defaults
## --start-position：开始位置
##  --stop-position=5911：结束位置
## --database：指定数据库
## -v：表示执行该mysql命令
/usr/bin/mysqlbinlog  --no-defaults --start-position=378 --stop-position=5911 --database=test  /var/lib/mysql/mysql_bin.000002 | /usr/bin/mysql -uroot -pRoot123456@ -v test
```



# X.MySQL安装

服务器端的安装见，`wiki->Linux`说明。

## X.1 安装

```shell
$ sudo apt-get install mysql-server
$ sudo apt install mysql-client
$ sudo apt install libmysqlclient-dev
```

系统将提示您在安装过程中创建 root 密码。选择一个安全的密码，并确保你记住它，因为你以后需要它。
## X.2 配置
```
# 可以默认
$ sudo mysql_secure_installation
```

>$ sudo mysql_secure_installation
NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MySQL
SERVERS IN PRODUCTION USE! PLEASE READ EACH STEP CAREFULLY!
In order to log into MySQL to secure it, we'll need the current
password for the root user. If you've just installed MySQL, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.
Enter current password for root (enter for none):<–初次运行直接回车
OK, successfully used password, moving on…
Setting the root password ensures that nobody can log into the MySQL
root user without the proper authorisation.
Set root password? [Y/n] <– 是否设置root用户密码，输入y并回车或直接回车
New password: <– 设置root用户的密码
Re-enter new password: <– 再输入一次你设置的密码
Password updated successfully!
Reloading privilege tables..
… Success!
By default, a MySQL installation has an anonymous user, allowing anyone
to log into MySQL without having to have a user account created for
them. This is intended only for testing, and to make the installation
go a bit smoother. You should remove them before moving into a
production environment.
Remove anonymous users? [Y/n] <– 是否删除匿名用户,生产环境建议删除，所以直接回车
… Success!
Normally, root should only be allowed to connect from 'localhost'. This
ensures that someone cannot guess at the root password from the network.
Disallow root login remotely? [Y/n] <–是否禁止root远程登录,根据自己的需求选择Y/n并回车,建议禁止
… Success!
By default, MySQL comes with a database named 'test' that anyone can
access. This is also intended only for testing, and should be removed
before moving into a production environment.
Remove test database and access to it? [Y/n] <– 是否删除test数据库,直接回车
- Dropping test database…
… Success!
- Removing privileges on test database…
… Success!
Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.
Reload privilege tables now? [Y/n] <– 是否重新加载权限表，直接回车
… Success!
Cleaning up…
All done! If you've completed all of the above steps, your MySQL
installation should now be secure.
Thanks for using MySQL!
[root@server1 ~]#




