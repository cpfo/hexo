---
title: hexo-next主题配置方式
date: 2023-11-15 14:29:15
tags: hexo
---

## Next主题配置
主要是修改themes/next 文件夹内的`_config.yml` 文件里面的配置
<!-- more -->
### 开启阅读时长
1. npm安装插件

> npm install hexo-symbols-count-time -g

2. 修改hexo的主配置，添加

```
symbols_count_time:
  symbols: true
  time: true
  total_symbols: true
  total_time: true
  exclude_codeblock: false
  awl: 3
  wpm: 200
  suffix: "mins."
```
3. 查看next的配置文件中的内容

```
symbols_count_time:
  separated_meta: true
  item_text_post: true
  item_text_total: false
```
4. 配置完成后，需要执行 hexo clean，否则阅读时长可能会显示 NaN

>  hexo clean && hexo g && hexo s

### 搜索功能

1. 安装插件 

> npm install hexo-generator-searchdb

2. 开启next的搜索配置

```properties
local_search:
  enable: true

```

