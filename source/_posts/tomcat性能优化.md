---
title: tomcat性能优化
date: 2016-03-31 15:00:00
tags: [tomcat]
---
tomcat相关的内容
<!-- more -->

## 安装
- tomcat的安装，一般从官网下载绿色版，直接解压就可以。
## 启动参数优化
- 常见的内存溢出有以下两种:
java.lang.OutOfMemoryError: PermGen space
java.lang.OutOfMemoryError: Java heap space
### PermGen space  
- java.lang.OutOfMemoryError: PermGen space
PermGen space的全称是Permanent Generation space,是指内存的永久保存区域,这块内存主要是被JVM存放Class和Meta信息的,Class在被Loader时就会被放到PermGen space中,它和存放类实例(Instance)的Heap区域不同,GC(Garbage Collection)不会在主程序运行期对PermGen space进行清理，所以如果你的应用中有很多CLASS的话,就很可能出现PermGen space错误,这种错误常见在web服务器对JSP进行pre compile的时候。如果你的WEB APP下都用了大量的第三方jar, 其大小超过了jvm默认的大小(4M)那么就会产生此错误信息了。
- 建议：将相同的第三方jar文件移置到tomcat/shared/lib目录下，这样可以达到减少jar 文档重复占用内存的目的。
### heap space
- java.lang.OutOfMemoryError: Java heap space
JVM堆的设置是指java程序运行过程中JVM可以调配使用的内存空间的设置.JVM在启动的时候会自动设置Heap size的值，其初始空间(即-Xms)是物理内存的1/64，最大空间(-Xmx)是物理内存的1/4。可以利用JVM提供的-Xmn -Xms -Xmx等选项可进行设置。Heap size 的大小是Young Generation 和Tenured Generaion 之和。
- 提示：在JVM中如果98％的时间是用于GC且可用的Heap size 不足2％的时候将抛出此异常信息。
- 提示：Heap Size 最大不要超过可用物理内存的80％，一般的要将-Xms和-Xmx选项设置为相同，而-Xmn为1/4的-Xmx值。 
- 修改tomcat目录下面，bin目录下面的catalina.sh，在cygwin=false之前添加下面内容:
```bash
JAVA_OPTS="-server -Dfile.encoding=UTF-8 -Xms1024m -Xmx1024m -XX:PermSize=256m -XX:MaxPermSize=512m -Xverify:none -Djava.net.preferIPv4Stack=true -XX:+AggressiveOpts -XX:+DisableExplicitGC -XX:+UseConcMarkSweepGC -XX:+UseParNewGC -Djava.awt.headless=true"
```
- -server, 因为tomcat默认是以一种叫java –client的模式来运行的，server即意味着你的tomcat是以真实的production环境运行，可以有更高的性能
-  -Xms–Xmx, 一定要设置成相等的，否则内存大小在两个值中间变动时，会造成系统卡顿。
-  -Djava.awt.headless=true
这个参数一般我们都是放在最后使用的，这全参数的作用是这样的，有时我们会在我们的J2EE工程中使用一些图表工具如：jfreechart，用于在web网页输出GIF/JPG等流，在winodws环境下，一般我们的app server在输出图形时不会碰到什么问题，但是在linux/unix环境下经常会碰到一个exception导致你在winodws开发环境下图片显示的好好可是在linux/unix下却显示不出来，因此加上这个参数以免避这样的情况出现。
## 配置优化
- 修改server.xml中Connector标签
```xml
<Connector port="8080" executor="tomcatThreadPool" protocol="HTTP/1.1"
   connectionTimeout="20000" redirectPort="8443" 
   enableLookups="false" disableUploadTimeout="true"
   URIEncoding="UTF-8" compression="on" compressionMinSize="2048"
 compressableMimeType="text/html,text/xml,text/javascript,text/css,text/plain" />
```
- URIEncoding=”UTF-8”
避免url中包含中文，导致后面拿到的结果乱码
- executor="tomcatThreadPool"
使用tomcat线程池，需要将上面的Executor标签的注释去掉
- enableLookups
关闭dns查询
- compression
对响应进行gzip压缩，在客户端请求网页后，从服务器端将网页文件压缩，再下载到客户端，由客户端的浏览器负责解压缩并浏览。
## 使用Apr插件
- APR(Apache Portable Runtime)是一个高可移植库，它是Apache HTTP Server 2.x的核心。使用tomcat Native原生库。
- 从apache官网下载apr和apr-util的压缩包。
- 解压apr-1.5.2.tar.gz，切换到该目录下，依次执行 
 ./configure;make;make install

- 解压apr-util，切换到该目录

```bash 
./configure --with-apr=/usr/local/apr ; make; make install;
```

- 切换到tomcat的bin目录下面，解压tomcat-native.tar.gz文件
然后切换到tomcat-native-1.1.33-src/jni/native，依次执行

```bash
./configure --with-apr=/usr/local/apr;make;make install
```

- 修改catalina.sh, 在cygwin=false之前加上：
```bash
CATALINA_OPTS="$CATALINA_OPTS -Djava.library.path=/usr/local/apr/lib"
```
- 如果启动报错，org.apache.tomcat.jni.Error: 70023: This function has not been implemented on this platform，是因为没有配置ssl， 可以在server.xml中将SSLEngine改为off
```xml
 <!--APR library loader. Documentation at /docs/apr.html -->
  <Listener className="org.apache.catalina.core.AprLifecycleListener" SSLEngine="off" />
```
- 查看catalina.out输出，如果看到如下信息，且无异常，则说明配置成功
```
信息: Starting ProtocolHandler ["http-apr-8080"]
三月 31, 2016 5:52:23 下午 org.apache.coyote.AbstractProtocol start
信息: Starting ProtocolHandler ["ajp-apr-8009"]
三月 31, 2016 5:52:23 下午 org.apache.catalina.startup.Catalina start
信息: Server startup in 1312 ms
```
 