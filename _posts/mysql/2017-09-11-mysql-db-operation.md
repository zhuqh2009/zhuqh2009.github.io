
---
layout:     post
title:      MySql数据库使用中常见操作命令简单记录
category: blog
description: MySQL开发管理中常用mySql命令简单记录
---


### mysql CMD

### mysql 创建新用户数据库权限

```
>mysql -u root -p
>...
>insert into mysql.user(Host, User, Password) values('localhost', "testuser", password('xx'));
>flush privileges;

//给用户授权数据库
>create database testDb;
>grant all privileges on testDb.* to 'testuser'@'localhost' identified by 'xx'; 
//用户不存在自动创建xx为密码的testuser用户
>flush privileges;

//查看用户主机数据库权限
>show grants;  //查看当前连接用户所有权限
>show grants for 'testuser'@'%';  

//撤销权限
>revoke all on *.* from 'testuser'@'%';
```

### MySQL执行sql文件
```
>source mysqlfile.sql
```

###查看表结构命令
```
>desc db.table;
//查看表的创建语句
>show create table table_name;
>show create database testDb;
```

### mysql Dump
```
shell 下执行mysqldump程序
备份数据库 
> mysqldump -h host -u root -p dbname >dbname_backup.sql 
恢复数据库 
> mysqladmin -h myhost -u root -p create dbname 
> mysqldump -h host -u root -p dbname < dbname_backup.sql 
如果只想卸出建表指令，则命令如下： 
> mysqladmin -u root -p -d databasename > a.sql 
如果只想卸出插入数据的sql命令，而不需要建表命令，则命令如下： 
> mysqladmin -u root -p -t databasename > a.sql 
```


### 查看编码信息
```
>show variables like 'char%';
```