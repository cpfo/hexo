---
title: mysql相关
date: 2016-03-12 15:00:00
tags: [mysql]

---

##  安装
网上的方法互相抄袭，可以参考官网的方法
- 使用yum
[mysql-yum-quick-guide](http://dev.mysql.com/doc/mysql-yum-repo-quick-guide/en/)
- 使用源文件安装
[参考官网5.6安装指导](http://dev.mysql.com/doc/refman/5.6/en/binary-installation.html)

<!-- more -->

## 用户授权
- 创建用户
```sql
create user username@'%' identified by 'password';
#创建了可以在任意地址登录的用户
grant all privileges on databasename.* to username@'%' with grant option;
#给用户授权某个数据库的权限，with grant options，带有授权的权限即也可以该用户也可以给其他用户授权
flush privileges;
#上面的执行之后可能在localhost不能登录，需要重新创建一个在localhost登录的。
create user username@'localhost' identified by 'password';
```
## 数据data目录迁移
- 停用mysql服务
service mysqld stop
或者mysqladmin -u root -p shutdown 
- 修改/etc/my.cnf，[mysqld]下面
```bash
datadir=/data/mysql
socket=/data/mysql/mysql.sock
log-error=/data/mysql/log/mysqld.log
```
- 同时也要修改[mysql]下面的，否则虽然正常启动，但是登录会报错。
```
[mysql]
socket=/data/mysql/mysql.sock
```
- 将/var/lib/mysql目录移动到新的数据目录
