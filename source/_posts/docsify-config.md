---
title: 使用docsify搭建github pages页面
date: 2023-11-16 14:37:27
tags:
---

docsify 可以快速帮你生成文档网站。不同于 GitBook、Hexo 的地方是它不会生成静态的 `.html` 文件，
所有转换工作都是在运行时。如果你想要开始使用它，只需要创建一个 `index.html` 就可以开始编写文档并直接部署在 GitHub Pages。

<!-- more -->

## 初始化

可以直接参考官方文档 [快速开始](https://docsify.js.org/#/zh-cn/quickstart)



效果参考  [docsify页面](https://cpfe.github.io/#/)

## 相关配置

### index.html

```html


<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8"/>
    <title>Document</title>
    <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1"/>
    <meta name="description" content="Description"/>
    <meta name="viewport" content="width=device-width, initial-scale=1.0, minimum-scale=1.0"/>
    <link rel="stylesheet" href="//cdn.jsdelivr.net/npm/docsify@4/lib/themes/vue.css"/>
    <style type="text/css">
        .sidebar-nav > ul > li.file p > a {
            font-size: 15px;
            font-weight: 700;
            color: #364149;
        }

        .sidebar-nav .folder {
            cursor: pointer;
        }
    </style>
</head>
<body>
<div id="app">
    加载中
</div>
<script>
    window.$docsify = {
        name: '文档',
        repo: '',
        loadSidebar: true,
        subMaxLevel: 2,
        coverpage: true,
        onlyCover: true,
        auto2top: true,
        mergeNavbar: true,

        alias: {
            '/.*/_sidebar.md': '/_sidebar.md'
        },
        count: {
            countable: true,
            fontsize: '0.9em',
            color: 'rgb(90,90,90)',
            language: 'chinese'
        },
        // docsify-pagination
        pagination: {
            crossChapter: true,
            crossChapterText: true,
        },
        search: {
            maxAge: 86400000, // 过期时间，单位毫秒，默认一天
            paths: 'auto', // or 'auto'
            placeholder: '搜索',
            noData: '无结果',
            hideOtherSidebarContent: false, // 是否隐藏其他侧边栏内容
            namespace: 'website-1',
            pathNamespaces: ['/', '/config', '/guide']
        }
    }
</script>
<!-- Docsify v4 -->
<script src="//cdn.jsdelivr.net/npm/docsify/lib/docsify.min.js"></script>
<script src="//cdn.jsdelivr.net/npm/docsify/lib/plugins/search.min.js"></script>
<script src="//cdn.jsdelivr.net/npm/docsify-copy-code/dist/docsify-copy-code.min.js"></script>
<script src="//unpkg.com/docsify-count/dist/countable.js"></script>
<script src="//cdn.jsdelivr.net/npm/docsify-pagination/dist/docsify-pagination.min.js"></script>

<script src="//unpkg.com/docsify-sidebar-collapse@1.3.5/dist/docsify-sidebar-collapse.min.js"></script>

<!-- code highlight -->
<script src="//unpkg.com/prismjs@1.27.0/components/prism-bash.min.js"></script>
<script src="//unpkg.com/prismjs@1.27.0/components/prism-csharp.min.js"></script>
<script src="//unpkg.com/prismjs@1.27.0/components/prism-java.min.js"></script>
<script src="//unpkg.com/prismjs@1.27.0/components/prism-json.min.js"></script>
<script src="//unpkg.com/prismjs@1.27.0/components/prism-markdown.min.js"></script>
<script src="//unpkg.com/prismjs@1.27.0/components/prism-nginx.min.js"></script>
<script src="//unpkg.com/prismjs@1.27.0/components/prism-properties.min.js"></script>
<script src="//unpkg.com/prismjs@1.27.0/components/prism-sql.min.js"></script>
<script src="//unpkg.com/prismjs@1.27.0/components/prism-xml-doc.min.js"></script>
<script src="//unpkg.com/prismjs@1.27.0/components/prism-yaml.min.js"></script>


</body>
</html>

```


### readme.md
默认是使用 readme作为首页的

### _sidebar.md

```markdown
<!-- docs/_sidebar.md -->
* 首页
	* [首页](readme "首页")
	* [指南](guide "很厉害的导航页面")

* 配置
	* [配置说明](config/config)
	* [配置说明1](config/config1)
	* [配置说明2](config/config2)

* 命令
	* [命令1](command/command1)
	* [命令2](command/command2)
	* [命令3](command/command3)

```

