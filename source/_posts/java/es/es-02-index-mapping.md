---
title: es索引和文档操作
categories: [技术]
tags: [java,elascitsearch]
date: 2025-01-17 13:39:45
---

主要介绍索引和文档的CRUD操作。

<!-- more -->


# 常用API

常用api列表 [rest-apis.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.12/rest-apis.html) 

|  操作   | 方式  | 接口 | 参数 |
|  ----  | ----  |---- | ----|
| 集群健康| GET | /_cat/health?v | - |
| 节点列表| GET | /_cat/nodes?v  | - |
|索引列表 | GET | /_cat/indices?v | - |
|创建索引 | PUT | /indexName | - |
|查看设置 | GET | /indexname/_settings | - |
/查看映射| GET  | /indexname/_mapping | - |
|关闭索引| POST | /name/_close | - | 
|打开索引| POST | /name/_open | - | 
|删除索引| DELETE | /name | - | 



# 索引库操作

索引库就类似数据库表，mapping映射就类似表的结构。

我们要向es中存储数据，必须先创建“库”和“表”。


## Mapping映射属性

mapping是对索引库中文档的约束，常见的mapping属性包括：

+ type：字段数据类型，常见的简单类型有：

    +   字符串：text（可分词的文本）、keyword（精确值，例如：品牌、国家、ip地址）

        > **keyword类型只能整体搜索，不支持搜索部分内容**

    +   数值：long、integer、short、byte、double、float、

    +   布尔：boolean

    +   日期：date

    +   对象：object

+ index：是否创建索引，默认为true

+ analyzer：使用哪种分词器

+ properties：该字段的子字段
+ fields: 给field 创建多字段，用于不同目的(全文检索或者聚合分析排序)， 参考 [es的mapping参数-fields](https://www.cnblogs.com/hld123/p/16538466.html) , [mapping-params](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-params.html)

例如下面的json文档

```json
{
    "age": 21,
    "weight": 52.1,
    "isMarried": false,
    "info": "真相只有一个！",
    "email": "zy@itcast.cn",
    "score": [99.1, 99.5, 98.9],
    "name": {
        "firstName": "柯",
        "lastName": "南"
    }
}
```


对应的每个字段映射（mapping）：

+   age：类型为 integer；参与搜索，因此需要index为true；无需分词器
+   weight：类型为float；参与搜索，因此需要index为true；无需分词器
+   isMarried：类型为boolean；参与搜索，因此需要index为true；无需分词器
+   info：类型为字符串，需要分词，因此是text；参与搜索，因此需要index为true；分词器可以用ik\_smart
+   email：类型为字符串，但是不需要分词，因此是keyword；不参与搜索，因此需要index为false；无需分词器
+   score：虽然是数组，但是我们只看元素的类型，类型为float；参与搜索，因此需要index为true；无需分词器
+   name：类型为object，需要定义多个子属性
    +   name.firstName；类型为字符串，但是不需要分词，因此是keyword；参与搜索，因此需要index为true；无需分词器
    +   name.lastName；类型为字符串，但是不需要分词，因此是keyword；参与搜索，因此需要index为true；无需分词器

## 索引库的CRUD

> CRUD简单描述：
>
> +   创建索引库：PUT /索引库名
> +   查询索引库：GET /索引库名
> +   删除索引库：DELETE /索引库名
> +   修改索引库（添加字段）：PUT /索引库名/\_mapping

这里统一使用Kibana编写DSL的方式来演示。

### 创建索引库和映射

**基本语法：**

+   请求方式：PUT
+   请求路径：/索引库名，可以自定义
+   请求参数：mapping映射

格式：

```json
PUT /索引库名称
{
  "mappings": {
    "properties": {
      "字段名":{
        "type": "text",
        "analyzer": "ik_smart"
      },
      "字段名2":{
        "type": "keyword",
        "index": "false"
      },
      "字段名3":{
        "properties": {
          "子字段": {
            "type": "keyword"
          }
        }
      },
      // ...略
    }
  }
}
```

**示例：**

```json
PUT /conan
{
  "mappings": {
    "properties": {
      "column1":{
        "type": "text",
        "analyzer": "ik_smart"
      },
      "column2":{
        "type": "keyword",
        "index": "false"
      },
      "column3":{
        "properties": {
          "子字段1": {
            "type": "keyword"
          },
          "子字段2": {
            "type": "keyword"
          }
        }
      }
    }
  }
}
```

### 查询索引库


**基本语法**：

+   请求方式：GET
+   请求路径：/索引库名
+   请求参数：无


**格式**：

```shell
GET /索引库名
```

### 修改索引库


> 这里的修改是只能增加新的字段到mapping中

倒排索引结构虽然不复杂，但是一旦数据结构改变（比如改变了分词器），就需要重新创建倒排索引，这简直是灾难。因此索引库**一旦创建，无法修改mapping**。

虽然无法修改mapping中已有的字段，但是却**允许添加新的字段**到mapping中，因为不会对倒排索引产生影响。

**语法说明**：

```json
PUT /索引库名/_mapping
{
  "properties": {
    "新字段名":{
      "type": "integer"
    }
  }
}
```

**示例**

```json
PUT /conan/_mapping
{
  "properties": {
    "age":{
      "type": "integer"
    }
  }
}

// 结果
{
  "acknowledged" : true
}

```

### 删除索引库

**语法：**

+   请求方式：DELETE

+   请求路径：/索引库名

+   请求参数：无


**格式：**

```sql
DELETE /索引库名
```

# 文档操作


> 文档操作有哪些？
>
> +   创建文档：POST /{索引库名}/\_doc/文档id
> +   查询文档：GET /{索引库名}/\_doc/文档id
> +   删除文档：DELETE /{索引库名}/\_doc/文档id
> +   修改文档：
      >     +   全量修改：PUT /{索引库名}/\_doc/文档id
>     +   增量修改：POST /{索引库名}/\_update/文档id { "doc": {字段}}

## 文档的CRUD


先创建索引库

```json
PUT /testcrud
{
  "mappings": {
    "properties": {
      "info":{
        "type": "text",
        "analyzer": "ik_smart"
      },
      "age":{
        "type": "integer",
        "index": "true"
      },
      "name":{
        "properties": {
          "firstName": {
            "type": "keyword"
          },
          "lastName": {
            "type": "keyword"
          }
        }
      }
    }
  }
}
```

### 新增文档

**语法：**

```json
POST /索引库名/_doc/文档id
{
    "字段1": "值1",
    "字段2": "值2",
    "字段3": {
        "子属性1": "值3",
        "子属性2": "值4"
    },
    // ...
}
```

**示例：**

```json
POST /testcrud/_doc/1
{
    "info": "真相只有一个！",
    "age": 20,
    "name": {
        "firstName": "柯",
        "lastName": "南"
    }
}

// 结果
{
  "_index" : "testcrud",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}

```

### 查询文档

根据rest风格，新增是post，查询应该是get，不过查询一般都需要条件，这里我们把文档id带上。

**语法：**

```json
GET /{索引库名称}/_doc/{id}
//批量查询：查询该索引库下的全部文档
GET /{索引库名称}/_search
```
**通过kibana查看数据**

```json
GET /testcrud/_doc/1

{
  "_index" : "testcrud",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "_seq_no" : 0,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "info" : "真相只有一个！",
    "age" : 20,
    "name" : {
      "firstName" : "柯",
      "lastName" : "南"
    }
  }
}

```

### 删除文档


删除使用DELETE请求，同样，需要根据id进行删除：

**语法：**

```js
DELETE /{索引库名}/_doc/id值
```

**示例：**

```json
# 根据id删除数据
DELETE /testcrud/_doc/1

{
  "_index" : "testcrud",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 2,
  "result" : "deleted",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 1,
  "_primary_term" : 1
}

```

### 修改文档

修改有两种方式：

+ 全量修改：直接覆盖原来的文档
+ 增量修改：修改文档中的部分字段

**全量修改**


全量修改是覆盖原来的文档，其本质是：

+   根据指定的id删除文档
+   新增一个相同id的文档

**注意**：如果根据id删除时，id不存在，第二步的新增也会执行，也就从修改变成了新增操作了。

**语法：**

```json
PUT /{索引库名}/_doc/文档id
{
  "字段1": "值1",
  "字段2": "值2",
  // ... 略
}

```
**示例**

```json
PUT /testcrud/_doc/1
{
    "info": "真相只有一个！, 哈哈哈哈",
    "age": 20,
    "name": {
        "firstName": "柯xxx",
        "lastName": "南"
    }
}

// 结果
{
  "_index" : "testcrud",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 2,
  "_seq_no" : 3,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "info" : "真相只有一个！, 哈哈哈哈",
    "age" : 20,
    "name" : {
      "firstName" : "柯xxx",
      "lastName" : "南"
    }
  }
}

```

**增量修改**


增量修改是只修改指定id匹配的文档中的部分字段。

**语法：**

```json
POST /{索引库名}/_update/文档id
{
  "doc": {
    "字段名": "新的值",
  }
}
```

**示例**

```shell

# 增量修改
POST /testcrud/_update/1
{
    "doc": {
         "age": 10
    }
}

// 结果

{
  "_index" : "testcrud",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 4,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 5,
  "_primary_term" : 1
}

```


**参考文章**

1. [ElasticSearch (ES从入门到精通一篇就够了)](https://www.cnblogs.com/buchizicai/p/17093719.html)
2. 