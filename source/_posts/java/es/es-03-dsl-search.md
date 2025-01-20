---
title: ES搜索查询
categories: [技术]
tags: [java,elascitsearch]
date: 2025-01-20 16:46:26
---

主要介绍ES DSL的查询

<!-- more -->


elasticsearch的查询依然是基于JSON风格的DSL来实现的。

# DSL设置查询条件

## DSL查询分类


Elasticsearch提供了基于JSON的DSL ( [Domain Specific Language](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html) ) 来定义查询。常见的查询类型包括：

+   **查询所有**：查询出所有数据，*一般测试用*。例如：match_all

+   **全文检索（full text）查询**：利用分词器对用户输入内容分词，然后去倒排索引库中匹配。例如：

    +   match_query
    +   multi_match_query
+   **精确查询**：根据精确词条值查找数据，一般是查找keyword、数值、日期、boolean等类型字段。例如：

    +   ids
    +   range
    +   term
+   **地理（geo）查询**：根据经纬度查询。例如：

    +   geo_distance
    +   geo_bounding_box
+   **复合（compound）查询**：复合查询可以将上述各种查询条件组合起来，合并查询条件。例如：

    +   bool
    +   function_score

查询的语法基本一致：

```json
GET /indexName/_search
{
  "query": {
    "查询类型": {
      "查询条件": "条件值"
    }
  }
}
```


我们以查询所有为例，其中：

+   查询类型为match_all
+   没有查询条件

```json
// 查询所有
GET /indexName/_search
{
  "query": {
    "match_all": {
    }
  }
}
```

其它查询无非就是**查询类型**、**查询条件**的变化。

## 全文检索查询


> match和multi_match的区别是什么？
>
> +   match：根据一个字段查询【推荐：使用copy_to构造all字段】
> +   multi_match：根据多个字段查询，参与查询字段越多，查询性能越差
>
> **注：搜索字段越多，对查询性能影响越大，因此建议采用copy\_to，然后单字段查询的方式。**

### 使用场景


全文检索查询的**基本流程**如下：

+   对用户搜索的内容做分词，得到词条
+   根据词条去倒排索引库中匹配，得到文档id
+   根据文档id找到文档，返回给用户

比较常用的**场景**包括：

+   商城的输入框搜索
+   百度输入框搜索


例如京东：

因为是拿着词条去匹配，因此参与搜索的字段也必须是可分词的text类型的字段。

常见的全文检索查询包括：

+   match查询：单字段查询
+   multi_match查询：多字段查询，任意一个字段符合条件就算符合查询条件

### match查询

match查询语法如下：

```json
GET /indexName/_search
{
  "query": {
    "match": {
      "FIELD": "TEXT"
    }
  }
}
```


match查询示例：

```shell
GET /testcrud/_search
{
  "query": {
    "match": {
      "info" : "测试"
    }
  }
}
```

### mulit_match查询


mulit_match语法如下：

```json
GET /indexName/_search
{
  "query": {
    "multi_match": {
      "query": "TEXT",
      "fields": ["FIELD1", " FIELD12"]
    }
  }
}

```

## 精准查询

