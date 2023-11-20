---
title: nginx拦截非法host的请求
date: 2023-11-20 16:58:44
tags: nginx
---


在nginx的访问日志中发现了一些奇怪的host域名，并非是实际用到的，经过排查发现，可以在本地的hosts文件中指定ip和域名的映射关系，
这样就可以通过访问一个任意的域名，来请求到对应的ip地址上面，因为nginx会使用默认server来处理未匹配到server_name的请求，可以通过这种方式来绕过waf等，或者访问到nginx中其他的servername下，
会有一定的风险，需要将这种方式拦截掉
比如在hosts文件中指定ip对应 www.abc.com 
> xx.xx.xx.xx www.abc.com

## 配置

因为nginx默认按照ASCII码的顺序来加载conf配置文件，所以可以通过创建一个00开头的conf文件，来保证是最先加载的，在该文件中指定默认的server_name,
来拦截所有的非法host的请求

可以使用 `ls -nl` 来查看nginx conf文件的ASCII顺序。


00block.conf 文件的配置

```shell

server {
	listen 80 default_server;
        server_name __;

        location / {
		deny all;
        }
}


server {
        listen 443 ssl default_server;
        server_name _;
        ssl_certificate cert/xxx.com.pem;
        ssl_certificate_key cert/xxx.com.key;
        ssl_session_timeout 5m;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE; 
        ssl_prefer_server_ciphers on;

        location / {
                deny all;
        }
}


```


这样就可以禁止掉直接通过IP或者本地绑定域名的方式的非法请求。


