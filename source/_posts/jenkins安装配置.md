---
title: jenkins安装配置
date: 2016-06-13 17:00:00
tags: [jenkins]
---
主要包括jenkins的安装以及相关配置和使用。
<!-- more -->

## 安装

- 从[官网](https://jenkins.io/)下载LTS Release版本的war包(当前版本1.651.2)，直接部署到tomcat容器(7.0.69)即可。启动后如图所示：
![首次启动](http://7xsj41.com1.z0.glb.clouddn.com/jenkins_start.jpg)

## 配置

包括tomcat配置和Jenkins配置

### tomcat配置

- Jenkins建议使用utf-8编码，所以配置tomcat的connector，添加URIEncoding="UTF-8"
- tomcat内存调整，防止内存溢出。参考[tomcat优化](http://cpf929.github.io/2016/03/31/%E5%B7%A5%E5%85%B7/tomcat%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/).
- 要把tomcat参数**JAVA_OPTS中关于gc的部分删掉**，否则会报错。例如：[CLONE -Builds fail with FATAL: null](https://issues.jenkins-ci.org/browse/JENKINS-6602)

### jenkins配置

#### 用户权限(Configure Global Security)

 - 点击左侧的系统管理，然后点击Configure Global Security。保存后，系统管理中就会出现管理用户菜单，并且右上角有登录注册链接.配置为如图所示：
 ![Configure Global Security](http://7xsj41.com1.z0.glb.clouddn.com/Security.jpg)

- 点击右上角注册，输入相关信息。然后系统管理，重新进入Configure Global Security，取消勾选允许用户注册，授权策略改为项目矩阵授权策略，并在添加用户/组中，添加刚才注册的用户，保存。防止其他用户注册。如图：
![权限](http://7xsj41.com1.z0.glb.clouddn.com/jenkins_user.jpg)

#### 系统设置

- Maven Configuration
两个settings  provider选择settings file in system。file path 指定到maven的settings.xml文件,如：/maven/conf/settings.xml
- JDK
点击JDK安装，分别指定JDK别名和JAVA_HOME，指向系统的JDK路径
- Maven
点击maven安装，指定maven别名，MAVEN_HOME指向系统maven的根目录
- Maven项目配置
全局MAVEN_OPTS ： maven的参数，如启动内存大小等，可忽略.
Local Maven Repository:  下载的jar的存放路径，默认为/root.m2/repository，可以在maven的settings.xml中指定到其他目录
- Jenkins Location
Jenkins URL : jenkins的管理地址，即tomcat的访问地址+jenkins
系统管理员邮件地址 :  发送邮件的账户，**必须配置，否则下面配置邮件服务器时，发送邮件会认证失败**.
- 邮件服务器配置.

#### 插件安装

系统管理，插件管理
- Deploy to container Plugin
部署war包到容器.
- Email Extension Plugin
邮件扩展，来代替自带的邮件服务配置，安装之后，再进行上面的邮件服务器配置。如图：
![jenkins邮件](http://7xsj41.com1.z0.glb.clouddn.com/jenkins_email.jpg)

## 项目构建

点击左侧新建，选择构建一个maven项目，输入item名称
- 源码管理.
输入项目地址Repository URL，用户名密码。
- 构建触发器
选择Poll SCM，日程表为空即可， 每次构建时手动触发。
- Build
Root POM: pom.xml，保持默认即可，如果找不到，会报错
Goals and options: mvn打包命令,如：clean package -Pproduction
- 增加构建后操作步骤
部署， Deploy war/ear to a container
WAR/EAR files ： war包的目录，target/name.war，target前没有/
Context path : 项目名称，为空，即为war包的名称
如图
![deploy](http://7xsj41.com1.z0.glb.clouddn.com/jenkins_deploy.jpg)  
- 增加构建后操作步骤, Editable Email Notification
保持默认即可， 可以添加trigger， 以及通知developers的邮件地址，也可以在系统设置里面配置相关的trigger，和收件人地址。

## 构建部署

相关构建部署的操作可以在页面上查看，以及相关的输出， 和日志，构建结果
- remote 部署，jenkins执行remote主机上面的sh脚本时， 无法拿到远程主机的环境变量。 解决方法：在文件开头的注释加上 --login
```bash
#!/bin/bash --login
```
参考[executing-command-on-remote](http://feihu.me/blog/2014/env-problem-when-ssh-executing-command-on-remote/)
- jenkins shell重启本地tomcat， 无效， 原因，jenkins 会杀死衍生进程
```bash
sleep 5
BUILD_ID=dontKillMe
bash /opt/gh2/scripts/business_restart.sh
```
[参考](https://wiki.jenkins-ci.org/display/JENKINS/ProcessTreeKiller)

