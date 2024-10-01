---
title: linux常用操作
categories: [技术]
tags: [linux]
date: 2017-07-06 15:12:47
---

主要记录一些日常用到的shell命令操作。

<!-- more -->

1. jenkins执行remote主机上面的sh脚本时， 无法拿到远程主机的环境变量。 

解决方法：在文件开头的注释加上 --login

` #!/bin/bash --login`

[参考](http://feihu.me/blog/2014/env-problem-when-ssh-executing-command-on-remote/)


2. sed 替换文件中的内容

> sed -i 's/java -jar/$JAVA_HOME\/bin\/java -jar/g' `find /opt/xxxxx/ -name "xxx.sh"`

3. jenkins shell重启本地tomcat， 无效， 原因，jenkins 会杀死衍生进程

[参考](https://wiki.jenkins-ci.org/display/JENKINS/ProcessTreeKiller)

```shell
sleep 5
BUILD_ID=dontKillMe
bash /opt/xxxx/business_restart.sh
```

4. CentOS/RHEL 修改主机名

* 查看主机名

> hostnamectl status

* 修改

> hostnamectl --static set-hostname 名字

5. 创建组， 用户权限

```
groupadd dev
useradd -g dev cpf
passwd cpf
用户列表文件：/etc/passwd

用户组列表文件：/etc/group

查看系统中有哪些用户：cut -d : -f 1 /etc/passwd

查看可以登录系统的用户：cat /etc/passwd | grep -v /sbin/nologin | cut -d : -f 1

查看用户操作：w命令(需要root权限)

查看某一用户：w 用户名

查看登录用户：who

查看用户登录历史记录：last
```

6. 修改目录所属用户和组

> chown -R cpf.dev tomcat2/

7. 磁盘

```
查看 fdisk –l
fdisk /dev/vdb
输入n 进行分区, 分区类型选 p， 分区个数默认， 柱面默认， 输入w 写入分区表
格式化分区,如果创建的是主分区；
mkfs -t ext3 /dev/vdb1

```

8. find中 -mtime 中的参数n

```
find . –mtime n中的n指的是24*n, +n、-n、n分别表示：
+n： 大于n
-n:  小于n
n：  等于n

但是man find里这样的解释还是比较令人费解的，不如直接看find . -mtime 中的用法

find . –mtime n:  File waslast modified n*24 hours ago.

最后一次修改发生在距离当前时间n*24小时至(n+1)*24 小时

find . –mtime +n:

最后一次修改发生在n+1天以前，距离当前时间为(n+1)*24小时或者更早

find . –mtime –n:

最后一次修改发生在n天以内，距离当前时间为n*24小时以内
```



9. awk

```
awk默认分割符是空格或制表符。
cut命令不能在分割符是空格的字符串中截取列，只能是制表符或具体的分割符。
cut [选项] 文件
awk '条件1{动作1} 条件2{动作2} ...' 文件名

awk '{print $1}' 文件名
awk '{if ( $1 > 10 ){ print $1} }'
```

10. 网络监控工具

iftop,  iptraf-ng

11. root 删除文件 Operation not permitted

```
lsattr 1.txt
——i—— 1.txt
在lsattr命令下，这个1.txt文件带有一个"i"的属性，所以才不可以删除
chattr -i 1.txt

chattr +i filename 加上保护
```

12. 获取目录， 文件名

> dirname, basename

13. redis批量更新

> for i in $(cat news1.txt) ; do echo 'set' $i $((RANDOM % (200 - 50) + 50)); done | redis-cli -c -p 6300

14. nginx统计

```
nginx按分钟统计访问频率高的
awk '{print $3}' access.log | cut -c 14-18 | uniq -c  | sort -rn  | head -n 100

统计耗时的请求
tail -200000 access.log | awk  '{print $3, $6 ,$10}' | sort -k3 -rn | head -100
```

15. 切割大文件

```shell

$ head -3000000 4.txt > 4.1.txt

$ tail -n +3000001 4.txt > 4.2.txt

split -l 2600000 -d imei-part-3.txt  imei-part-3_

```

16. 循环请求url

> for i in `cat udid.txt` ; do curl  -0 $i; echo '' ; done

