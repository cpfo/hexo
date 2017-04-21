---
title: linux相关
date: 2015-12-20 15:30:25
categories: [Linux, Hexo]
tags: [Linux, Hexo]
---

### 修改RedHat的yum源

* 下载163或者阿里的yum配置文件
> wget  http://mirrors.163.com/.help/CentOS6-Base-163.repo
* 替换CentOS-Base-163.repo中 $releaserver 参数为对应的centos版本，如6， 使用vi 的替换功能.  
> :%s/$releaserver/6/g
* yum makecache
* yum install yum-fastestmirror 安装fastestmirror插件可自动选择最快的yum 源

<!-- more -->

### unkown host ，无法ping通外网

* 修改 */etc/resolv.conf* 文件，添加**nameserver**配置

```
nameserver 8.8.8.8
nameserver 114.114.114.114
```
### 启动/关闭防火墙

* service iptables start/stop
* /etc/init.d/iptables start/stop

### 开放端口

* 修改 */etc/sysconfig/iptables* 文件，复制22端口的那一行，修改成对应的端口号
> -A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -j ACCEPT

### tomcat 启动时报错，`java.net.UnknownHostException: dev-01: dev-01`

* 修改 /etc/hosts, 127.0.0.1 后面增加 对应的hostname
> 127.0.0.1   localhost localhost.localdomain dev-01

