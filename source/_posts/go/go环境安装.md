---
title: go环境安装
categories: [技术]
tags: [go]
date: 2024-01-31 15:45:31
---

go语言环境的安装

<!-- more -->

安装就按照官网的步骤，直接安装即可，下面主要是列举下安装后的一些问题


* VsCode 插件安装失败的问题

proxy.golang.org 访问失败，需要设置代理

```shell
go env -w GO111MODULE=on
go env -w GOPROXY=https://goproxy.io,direct
```

* verifying module: invalid GOSUMDB: malformed verifier id

```shell
go env -w GOSUMDB="sum.golang.org"
#或者关闭包验证
go env -w GOSUMDB=off
```

* VsCode中红线提示

`gopls was not able to find modules in your workspace.`

在 *.go文件同级目录下初始化一个新的Go模块，会自动创建一个go.mod文件
`main` 和go文件中的package一致。
```shell
go mod init main
```

* VsCode代码自动提示

在VsCode的Command Palette中输入 `go:install/Update Tools`，在弹窗中把列出来的几项都选中，然后安装

