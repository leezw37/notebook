# 目录

[TOC]


# 常用命令

以下列出了使用Mysql数据库过程中常用的命令：

-   **USE _数据库名_** ;
    选择要操作的Mysql数据库，使用该命令后所有Mysql命令都只针对该数据库。

-   **SHOW DATABASES;**
    列出 MySQL 数据库管理系统的数据库列表。

-   **SHOW TABLES;**
    显示指定数据库的所有表，使用该命令前需要使用 use 命令来选择要操作的数据库。

-   **SHOW COLUMNS FROM _数据表_;**
    显示数据表的属性，属性类型，主键信息 ，是否为 NULL，默认值等其他信息。

-   **SHOW INDEX FROM _数据表_;**
    显示数据表的详细索引信息，包括PRIMARY KEY（主键）。

-   **SHOW TABLE STATUS \[FROM db\_name\] \[LIKE 'pattern'\] \\G;**
    该命令将输出Mysql数据库管理系统的性能及统计信息。

```
    mysql> SHOW TABLE STATUS  FROM RUNOOB;   # 显示数据库 RUNOOB 中所有表的信息
    mysql> SHOW TABLE STATUS from RUNOOB LIKE 'runoob%';     # 表名以runoob开头的表的信息
    mysql> SHOW TABLE STATUS from RUNOOB LIKE 'runoob%'\G;   # 加上 \G，查询结果按列打印

```
- create DATABASE <数据库名>;
    mysqladmin -u root -p create RUNOOB
- drop database <数据库名>;
    mysqladmin -u root -p drop RUNOOB
- use RUNOOB;
- mysql -uroot -p test < test.sql

#
## 安装


```bash
# 下载，重启服务
sudo apt install mysql-server
sudo service mysql restart

# 运行安全脚本设置密码等
sudo mysql_secure_installation

# 登录（第一次）
sudo mysql

```

```SQL
-- 允许使用上述密码简化登录
SELECT user,authentication_string,plugin,host FROM mysql.user;
ALTER USER 'root'@'localhost' IDENTIFIED WITH caching_sha2_password BY '上述密码';
FLUSH PRIVILEGES;

```

```bash
# 测试服务是否成功运行
sudo mysqladmin -p -u root version
# 退出
exit;

```

```bash
# 登录（以后）（输入密码）
mysql -u root -p

```
[参考](https://kalacloud.com/blog/ubuntu-install-mysql/#%E7%AC%AC%E4%B8%80%E6%AD%A5---%E5%AE%89%E8%A3%85-mysql)

## 修改密码

```SQL
# MySQL内
# 查看密码策略，修改
# （0：仅要求长度。）
show VARIABLES like "%password%";
set global validate_password.policy=0;
set global validate_password.length=4;

```

```bash
# MySQL外
mysqladmin -uroot -p12345678(原密码) password 1234(新)

```

参考  :
[修改策略](https://blog.csdn.net/ManagementAndJava/article/details/80173695)
[三种方式修改密码](http://c.biancheng.net/view/7152.html)
