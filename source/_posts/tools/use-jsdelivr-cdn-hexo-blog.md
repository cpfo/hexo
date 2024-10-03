---
title: 使用jsdelivr加速hexo github pages博客的静态资源
categories: [工具]
tags: [hexo, jsdelivr]
date: 2024-10-03 10:46:02
---

使用hexo将博客部署在github pages上面，有时候访问速度会比较慢，可以将图片等静态资源，使用jsdelivr进行cdn加速。

目前由于一些非法的滥用，jsdelivr在国内被墙，无法访问了。

<!-- more -->

### 使用介绍

[jsdelivr加速github介绍](https://github.com/jsdelivr/jsdelivr?tab=readme-ov-file#github)

根路径是 :  https://cdn.jsdelivr.net

#### Load any GitHub release, commit, or branch:
> /gh/user/repo@version/file

#### Load exact version:

> /gh/jquery/jquery@3.1.0/dist/jquery.min.js
> /gh/jquery/jquery@32b00373b3f42e5cdcb709df53f3b08b7184a944/dist/jquery.min.js

#### Use a version range instead of an exact version (only works with valid semver versions):

> /gh/jquery/jquery@3/dist/jquery.min.js
> /gh/jquery/jquery@3.1/dist/jquery.min.js

* 访问方式

https://cdn.jsdelivr.net/gh/Github用户名/仓库名/资源路径

默认访问 `master`分支下面的文件。

比如: 

* https://cdn.jsdelivr.net/gh/jquery/jquery/dist/jquery.min.js 访问的是master下的
* https://cdn.jsdelivr.net/gh/jquery/jquery@3.1.0/dist/jquery.min.js 访问的是精确的版本
* https://cdn.jsdelivr.net/gh/jquery/jquery@3.1/dist/jquery.min.js  访问的是范围版本

加速静态资源的方式， 可以在github上创建一个仓库，比如叫cdn，然后将静态资源放到方库中。

> https://cdn.jsdelivr.net/gh/cpfo/cdn/images/2024/40703300-d32f-11ea-938e-f5ee97dc461f.jpg

使用加速后的图片

![](https://cdn.jsdelivr.net/gh/cpfo/cdn/images/2024/40703300-d32f-11ea-938e-f5ee97dc461f.jpg)
