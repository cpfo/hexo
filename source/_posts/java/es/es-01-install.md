---
title: ES相关介绍和安装
categories: [技术]
tags: [java,elascitsearch]
date: 2025-01-17 10:30:50
---

主要记录下ES的概念，安装和使用。

<!-- more -->

# 概念

## 倒排索引

倒排索引的概念是基于MySQL这样的正向索引而言的。

### 正向索引
> 设置了索引的话挺快的，但要是模糊查询则就很慢！

那么什么是正向索引呢？例如给下表（tb_goods）中的id创建索引：

![images](/images/java/es/2729274-20230205171741388-1990334679.png)

如果是根据id查询，那么直接走索引，查询速度非常快。

但如果是基于title做模糊查询，只能是逐行扫描数据，流程如下：

1）用户搜索数据，条件是title符合"%手机%"

2）逐行获取数据，比如id为1的数据

3）判断数据中的title是否符合用户搜索条件

4）如果符合则放入结果集，不符合则丢弃。回到步骤1

逐行扫描，也就是全表扫描，随着数据量增加，其查询效率也会越来越低。当数据量达到数百万时，就是一场灾难。

### 倒排索引

倒排索引中有两个非常重要的概念：

* 文档（Document）：用来搜索的数据，其中的每一条数据就是一个文档。例如一个网页、一个商品信息
* 词条（Term）：对文档数据或用户搜索数据，利用某种算法分词，得到的具备含义的词语就是词条。例如：我是中国人，就可以分为：我、是、中国人、中国、国人这样的几个词条

创建倒排索引是对正向索引的一种特殊处理，流程如下：

* 将每一个文档的数据利用算法分词，得到一个个词条
* 创建表，每行数据包括词条、词条所在文档id、位置等信息
* 因为词条唯一性，可以给词条创建索引，例如hash表结构索引

如图：

![images](/images/java/es/2729274-20230205171751933-576636800.png)

倒排索引的**搜索流程**如下（以搜索"华为手机"为例）：

1）用户输入条件`"华为手机"`进行搜索。

2）对用户输入内容**分词**，得到词条：`华为`、`手机`。

3）拿着词条在倒排索引中查找，可以得到包含词条的文档id：1、2、3。

4）拿着文档id到正向索引中查找具体文档。

如图：

![image](/images/java/es/2729274-20230205171803916-704919285.png)

虽然要先查询倒排索引，再查询倒排索引，但是无论是词条、还是文档id都建立了索引，查询速度非常快！无需全表扫描。

### 正向和倒排对比

概念区别：

+   **正向索引**是最传统的，根据id索引的方式。但根据词条查询时，必须先逐条获取每个文档，然后判断文档中是否包含所需要的词条，是**根据文档找词条的过程**。

+   而**倒排索引**则相反，是先找到用户要搜索的词条，根据词条得到保护词条的文档的id，然后根据id获取文档。是**根据词条找文档的过程**。

## ES数据库基本概念

elasticsearch中有很多独有的概念，与mysql中略有差别，但也有相似之处。

### 文档和字段

> 一个文档就像数据库里的一条数据，字段就像数据库里的列

elasticsearch是面向**文档（Document）**存储的，可以是**数据库中的一条商品数据**，一个订单信息。文档数据会被序列化为json格式后存储在elasticsearch中：

![image](/images/java/es/2729274-20230205171811564-2114810461.png)

而Json文档中往往包含很多的**字段（Field）**，类似于**mysql数据库中的列**。

### 索引和映射


> 索引就像数据库里的表，映射就像数据库中定义的表结构

**索引（Index）**，就是相同类型的文档的集合【**类似mysql中的表**】

例如：

+   所有用户文档，就可以组织在一起，称为用户的索引；
+   所有商品的文档，可以组织在一起，称为商品的索引；
+   所有订单的文档，可以组织在一起，称为订单的索引；

![image](/images/java/es/2729274-20230205171817377-1198317744.png)

因此，我们可以把索引当做是数据库中的表。

数据库的表会有约束信息，用来定义表的结构、字段的名称、类型等信息。因此，索引库中就有**映射（mapping）**，是索引中文档的字段约束信息，类似表的结构约束。


### mysql与elasticsearch

> 各自长处：
>
> +   Mysql：擅长事务类型操作，可以确保数据的安全和一致性
> +   Elasticsearch：擅长海量数据的搜索、分析、计算
>

我们统一的把**mysql与elasticsearch的概念做一下对比**：

| **MySQL** | **Elasticsearch** | **说明** |
| --- | --- | --- |
| Table | Index | 索引(index)，就是文档的集合，类似数据库的表(table) |
| Row | Document | 文档（Document），就是一条条的数据，类似数据库中的行（Row），文档都是JSON格式 |
| Column | Field | 字段（Field），就是JSON文档中的字段，类似数据库中的列（Column） |
| Schema | Mapping | Mapping（映射）是索引中文档的约束，例如字段类型约束。类似数据库的表结构（Schema） |
| SQL | DSL | DSL是elasticsearch提供的JSON风格的请求语句，用来操作elasticsearch，实现CRUD |

![image](/images/java/es/2729274-20230205171824403-1152991371.png)


# ES和Kibana安装

## ES安装

### JDK依赖问题

ES优先使用系统的jdk，如果版本不一致就会报错

```shell
warning: usage of JAVA_HOME is deprecated, use ES_JAVA_HOME
Future versions of Elasticsearch will require Java 11; your Java version from [/usr/local/java/jdk1.8.0_421/jre] does not meet this requirement. Consider switching to a distribution of Elasticsearch with a bundled JDK. If you are already using a distribution with a bundled JDK, ensure the JAVA_HOME environment variable is not set.

```

解决办法：  使用es自带的jdk, 修改 bin/elasticsearch ， 添加下面内容

```shell
############## 添加配置解决jdk版本问题 ##############
# 将jdk修改为es中自带jdk的配置目录
export ES_JAVA_HOME=/usr/local/elasticsearch/elasticsearch-7.12.1/jdk
export PATH=$ES_JAVA_HOME/bin:$PATH

if [ -x "$ES_JAVA_HOME/bin/java" ]; then
        JAVA="$ES_JAVA_HOME/bin/java"
else
        JAVA=`which java`
fi

```

* 修改内存

```shell
-Xms1024m
-Xmx1024m

```

### 创建专用用户

不能使用root启动，需要创建专用的用户。 `can not run elasticsearch as root` 

* 创建用户
> useradd esuser
* 创建所属组
> chown esuser:esuser -R  elasticsearch-7.12.1
* 切换用户
> su esuser
* 启动
> ./elasticsearch -d

### 修改配置

* 修改ip
> network.host: 0.0.0.0 

* 初始化节点名称
```shell
cluster.name: elasticsearch 
node.name: es-node0
cluster.initial_master_nodes: ["es-node0"]
```

### 常见错误

* max file descriptors [4096] for elasticsearch process is too low

```shell
# 使用root
vi /etc/security/limits.conf

#添加
* soft nofile 65536
* hard nofile 131072
* soft nproc 2048
* hard nproc 4096
```

* max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]

elasticsearch用户拥有的内存权限太小，至少需要262144，解决办法：

vim /etc/sysctl.conf , 添加 

> vm.max_map_count=262144

保存，刷新配置  

sysctl -p


启动后，访问 `curl http://127.0.0.1:9200` 出现如下信息，说明成功

```json
{
  "name" : "es-node0",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "N_nV0k8hTXyGW1G1iZjTuw",
  "version" : {
    "number" : "7.12.1",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "3186837139b9c6b6d23c3200870651f10d3343b7",
    "build_date" : "2021-04-20T20:56:39.040728659Z",
    "build_snapshot" : false,
    "lucene_version" : "8.8.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

如果使用ip无法访问的话，可以看下，是否是机器防火墙的问题。

## Kibana安装

1. 下载
> wget https://artifacts.elastic.co/downloads/kibana/kibana-7.12.1-linux-x86_64.tar.gz

2. 解压后修改权限

> chown esuser:esuser -R kibana-7.12.1-linux-x86_64

3. 修改 `config/kibana.yml` 配置

```shell
#server.port：http访问端口
#server.host：ip地址，0.0.0.0表示可远程访问
#server.name：kibana服务名
#elasticsearch.hosts：elasticsearch地址

server.port: 5601
server.host: "0.0.0.0"
server.name: "kibana-test"
elasticsearch.hosts: ["http://127.0.0.1:9200"]

```
4. 启动
> nohup ./kibana &

5. 访问

浏览器访问 http://ip:5601 ， 能访问，说明成功。

6. 可以在页面上添加 Sample data 进行测试

7. 可以在discover 页面进行查询

http://localhost:5601/app/discover

## 安装分词器

1. 下载地址  https://release.infinilabs.com/analysis-ik/stable/

可以在线安装，或者离线下载后解压到plugins目录下面

* 在线安装
> ./elasticsearch-plugin  install https://release.infinilabs.com/analysis-ik/stable/elasticsearch-analysis-ik-7.12.1.zip

### 验证

IK分词器包含两种模式：

+ `ik_smart`：最少切分

+ `ik_max_word`：最细切分

在kibana的Dev tools中输入以下代码：

> ”analyzer“ 就是选择分词器模式

```json
GET /_analyze
{
  "analyzer": "ik_max_word",
  "text": "黑马程序员学习java太棒了"
}
```

结果

```json
{
  "tokens" : [
    {
      "token" : "黑马",
      "start_offset" : 0,
      "end_offset" : 2,
      "type" : "CN_WORD",
      "position" : 0
    },
    {
      "token" : "程序员",
      "start_offset" : 2,
      "end_offset" : 5,
      "type" : "CN_WORD",
      "position" : 1
    },
    {
      "token" : "程序",
      "start_offset" : 2,
      "end_offset" : 4,
      "type" : "CN_WORD",
      "position" : 2
    },
    {
      "token" : "员",
      "start_offset" : 4,
      "end_offset" : 5,
      "type" : "CN_CHAR",
      "position" : 3
    },
    {
      "token" : "学习",
      "start_offset" : 5,
      "end_offset" : 7,
      "type" : "CN_WORD",
      "position" : 4
    },
    {
      "token" : "java",
      "start_offset" : 7,
      "end_offset" : 11,
      "type" : "ENGLISH",
      "position" : 5
    },
    {
      "token" : "太棒了",
      "start_offset" : 11,
      "end_offset" : 14,
      "type" : "CN_WORD",
      "position" : 6
    },
    {
      "token" : "太棒",
      "start_offset" : 11,
      "end_offset" : 13,
      "type" : "CN_WORD",
      "position" : 7
    },
    {
      "token" : "了",
      "start_offset" : 13,
      "end_offset" : 14,
      "type" : "CN_CHAR",
      "position" : 8
    }
  ]
}

```

### 扩展词词典


随着互联网的发展，“造词运动”也越发的频繁。出现了很多新的词语，在原有的词汇列表中并不存在。比如：“奥力给”，“白嫖” 等。

所以我们的词汇也需要不断的更新，IK分词器提供了扩展词汇的功能。

1）打开IK分词器目录  config/analysis-ik：

![image](/images/java/es/2729274-20230205172005673-1129389907.png)

2）在IKAnalyzer.cfg.xml配置文件内容添加：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
<properties>
        <comment>IK Analyzer 扩展配置</comment>
        <!--用户可以在这里配置自己的扩展字典 *** 添加扩展词典-->
        <entry key="ext_dict">ext.dic</entry>
</properties>
```

3）新建一个 `ext.dic`，可以参考config目录下复制一个配置文件进行修改

```properties
白嫖
奥力给
```

4）重启elasticsearch， 查看日志中的加载情况

```shell
[2025-01-17T14:38:30,348][INFO ][o.w.a.d.Dictionary       ] [es-node0] try load config from /usr/local/elasticsearch/elasticsearch-7.12.1/config/analysis-ik/IKAnalyzer.cfg.xml
[2025-01-17T14:38:30,500][INFO ][o.w.a.d.Dictionary       ] [es-node0] [Dict Loading] /usr/local/elasticsearch/elasticsearch-7.12.1/config/analysis-ik/ext.dic

```

> 注意当前文件的编码必须是 UTF-8 格式，严禁使用Windows记事本编辑

### 停用词词典

在互联网项目中，在网络间传输的速度很快，所以很多语言是不允许在网络上传递的，如：关于宗教、政治等敏感词语，那么我们在搜索时也应该忽略当前词汇。

IK分词器也提供了强大的停用词功能，让我们在索引时就直接忽略当前的停用词汇表中的内容。

1）IKAnalyzer.cfg.xml配置文件内容添加：

```xml
<entry key="ext_stopwords">stopword.dic</entry>
```

2）在 stopword.dic 添加停用词

```properties
傻逼
艹
```

3）重启elasticsearch

4）测试效果


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

## 文档操作


> 文档操作有哪些？
>
> +   创建文档：POST /{索引库名}/\_doc/文档id
> +   查询文档：GET /{索引库名}/\_doc/文档id
> +   删除文档：DELETE /{索引库名}/\_doc/文档id
> +   修改文档：
>     +   全量修改：PUT /{索引库名}/\_doc/文档id
>     +   增量修改：POST /{索引库名}/\_update/文档id { "doc": {字段}}

### 文档的CRUD


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

#### 新增文档

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

#### 查询文档

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

#### 删除文档


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

#### 修改文档

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