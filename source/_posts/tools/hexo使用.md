---
title: hexo使用
date: 2015-01-01 00:00:00
categories: [工具]
tags: hexo
---
Welcome to [Hexo](https://hexo.io/)! This is your very first post. Check [documentation](https://hexo.io/docs/) for more info. If you get any problems when using Hexo, you can find the answer in [troubleshooting](https://hexo.io/docs/troubleshooting.html) or you can ask me on [GitHub](https://github.com/hexojs/hexo/issues).

<!-- more -->

## Quick Start

### Create a new post

``` bash
$ hexo new "My New Post"

 hexo new post -p folder/fileanme "titlename"
 
 比如 >hexo new post -p tools/testfile "测试文章"
  // 就是在posts 下的 tools目录中创建了文件 testfile.md，文件中对应的title是 测试文章

```

More info: [Writing](https://hexo.io/docs/writing.html)


## 网址

按照官网文档上 [配置](https://hexo.io/zh-cn/docs/configuration#%E7%BD%91%E5%9D%80) 的说明，配置后，部署到username.github.io 之后，发现访问时部分资源加载失败。

感觉这种方式仅适合项目页面，[参考](https://hexo.io/zh-cn/docs/github-pages#%E9%A1%B9%E7%9B%AE%E9%A1%B5%E9%9D%A2)

因为username.github.io里面的内容部署完成之后，地址是 https://username.github.io 

> ### 网站存放在子目录
> 如果您的网站存放在子目录中，例如 http://example.com/blog，则请将您的 url 设为 http://example.com/blog 并把 root 设为 /blog/。

上面的 `blog` 应该是github上对应的repo的名称


### Run server

``` bash
$ hexo server
```

More info: [Server](https://hexo.io/docs/server.html)

### Generate static files

``` bash
$ hexo generate
```

More info: [Generating](https://hexo.io/docs/generating.html)

### Deploy to remote sites

``` bash
$ hexo deploy
```

More info: [Deployment](https://hexo.io/docs/deployment.html)

### 使用图片

可以将图片放到 source/images 目录下面即可，然后在文章中使用markdown语法访问他们, 参考 [资源文件夹](https://fuhailin.github.io/Hexo-images/)

>  `![](/images/image.jpg)`

效果如下:

![](/images/assets/40703300-d32f-11ea-938e-f5ee97dc461f.jpg)
