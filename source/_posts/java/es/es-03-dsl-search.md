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


> 精准查询类型：
>
> +   term查询：根据词条精确匹配，一般搜索keyword类型、数值类型、布尔类型、日期类型字段
> +   range查询：根据数值范围查询，可以是数值、日期的范围

精确查询一般是查找keyword、数值、日期、boolean等类型字段。所以**不会**对搜索条件分词。常见的有：

+ **term：根据词条精确值查询**
+ **range：根据值的范围查询**

### term查询

因为精确查询的字段是不分词的字段，因此查询的条件也必须是**不分词**的词条。查询时，用户输入的内容跟字段值完全匹配时才认为符合条件。如果用户输入的内容过多，反而搜索不到数据。


语法说明：

```shell
// term查询
GET /indexName/_search
{
  "query": {
    "term": {
      "FIELD": {
        "value": "VALUE"
      }
    }
  }
}
```

### range查询

> 范围查询，一般应用在对数值类型做范围过滤的时候。比如做价格范围过滤。

基本语法：

```shell
// range查询
GET /indexName/_search
{
  "query": {
    "range": {
      "FIELD": {
        "gte": 10, // 这里的gte代表大于等于，gt则代表大于
        "lte": 20 // lte代表小于等于，lt则代表小于
      }
    }
  }
}
```

示例

```shell
GET /testcrud/_search
{
  "query": {
    "range": {
      "age" : {
        "gte" : 10,
        "lte" : 12
      }
    }
  }
}

```

### 地理坐标查询


所谓的地理坐标查询，其实就是根据经纬度查询，官方文档：[geo-queries.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/geo-queries.html)

常见的使用场景包括：

+   携程：搜索我附近的酒店
+   滴滴：搜索我附近的出租车
+   微信：搜索我附近的人

#### 矩形范围查询

> 很少有业务有这种需求

矩形范围查询，也就是geo_bounding_box查询，查询坐标落在某个矩形范围的所有文档：  
![image](/images/java/es/2729274-20230205172556695-448469947.gif)

查询时，需要指定矩形的**左上**、**右下**两个点的坐标，然后画出一个矩形，落在该矩形内的都是符合条件的点。

语法如下：

```shell
// geo_bounding_box查询
GET /indexName/_search
{
  "query": {
    "geo_bounding_box": {
      "FIELD": {
        "top_left": { // 左上点
          "lat": 31.1,
          "lon": 121.5
        },
        "bottom_right": { // 右下点
          "lat": 30.9,
          "lon": 121.7
        }
      }
    }
  }
}
```

附近(圆形)查询

附近查询，也叫做距离查询（geo_distance）：查询到指定中心点小于某个距离值的所有文档。

换句话来说，在地图上找一个点作为圆心，以指定距离为半径，画一个圆，落在圆内的坐标都算符合条件：

![image](/images/java/es/2729274-20230205172657385-1268854128.gif)

语法说明：

```shell
// geo_distance 查询
GET /indexName/_search
{
  "query": {
    "geo_distance": {
      "distance": "15km", // 半径
      "FIELD": "31.21,121.5" // 圆心
    }
  }
}
```

示例：

我们先搜索陆家嘴附近15km的酒店：

![image](/images/java/es/2729274-20230205172724869-1773754207.png)

发现共有47家酒店。

## 复合查询


复合（compound）查询：复合查询可以将其它简单查询组合起来，实现更复杂的搜索逻辑。常见的有两种：

+ fuction score：算分函数查询，可以控制文档相关性算分，控制文档排名
+ bool query：布尔查询，利用逻辑关系组合多个其它的查询，实现复杂搜索

### 复合查询归纳

```shell
GET /hotel/_search
{
  "query": {
    "function_score": {           
      "query": { // 原始查询，可以是任意条件
          "bool": {
              "must": [
                  {"term": {"city": "上海" }}
              ],
              "should": [
                  {"term": {"brand": "皇冠假日" }},
                  {"term": {"brand": "华美达" }}
              ],
              "must_not": [
                  { "range": { "price": { "lte": 500 } }}
              ],
              "filter": [
                  { "range": {"score": { "gte": 45 } }}
              ]
          }
      },
      "functions": [ // 算分函数
        {
          "filter": { // 满足的条件，品牌必须是如家【品牌是如家的才加分，这里是加分条件】
            "term": {
              "brand": "如家"
            }
          },
          "weight": 2 // 算分权重为2
        }
      ],
      "boost_mode": "sum" // 加权模式，求和
    }
  }  
}
```

### 相关性算分


> elasticsearch会根据词条和文档的相关度做打分，算法由两种：
>
> +   TF-IDF算法
> +   BM25算法，elasticsearch5.1版本后采用的算法

当我们利用match查询时，文档结果会根据与搜索词条的关联度打分（_score），返回结果时按照分值降序排列。

例如，我们搜索 "虹桥如家"，结果如下：

```json
[
  {
    "_score" : 17.850193,
    "_source" : {
      "name" : "虹桥如家酒店真不错",
    }
  },
  {
    "_score" : 12.259849,
    "_source" : {
      "name" : "外滩如家酒店真不错",
    }
  },
  {
    "_score" : 11.91091,
    "_source" : {
      "name" : "迪士尼如家酒店真不错",
    }
  }
]

```
在elasticsearch中，早期使用的打分算法是TF-IDF算法，公式如下：


![image](/images/java/es/2729274-20230205173433826-368331600.png)

在后来的5.1版本升级中，elasticsearch将算法改进为BM25算法，公式如下：

![image](/images/java/es/2729274-20230205173438720-1124832591.png)

TF-IDF算法有一各缺陷，就是词条频率越高，文档得分也会越高，单个词条对文档影响较大。而BM25则会让单个词条的算分有一个上限，曲线更加平滑：

![image](/images/java/es/2729274-20230205173443307-150506578.png)

### 算分函数查询


> 在搜索出来的结果的分数基础上，再手动与指定的数字进行一定运算来改变算分，从而改变结果的排序。

> function score query定义的三要素是什么？
>
> +   过滤条件：哪些文档要加分
> +   算分函数：如何计算function score
> +   加权方式：function score 与 query score如何运算

根据相关度打分是比较合理的需求，但**合理的不一定是产品经理需要**的。

以百度为例，你搜索的结果中，并不是相关度越高排名越靠前，而是谁掏的钱多排名就越靠前。如图：

要想人为控制相关性算分，就需要利用elasticsearch中的function score 查询了。

#### function score 查询


**1）语法说明**

![image](/images/java/es/2729274-20230205173456315-243388005.png)

function score 查询中包含四部分内容：

+   **原始查询**条件：query部分，基于这个条件搜索文档，并且基于BM25算法给文档打分，**原始算分**（query score)
+   **过滤条件**：filter部分，符合该条件的文档才会重新算分
+   **算分函数**：符合filter条件的文档要根据这个函数做运算，得到的**函数算分**（function score），有四种函数
    +   weight：函数结果是常量
    +   field_value_factor：以文档中的某个字段值作为函数结果
    +   random_score：以随机数作为函数结果
    +   script_score：自定义算分函数算法
+   **运算模式**：算分函数的结果、原始查询的相关性算分，两者之间的运算方式，包括：
    +   multiply：相乘
    +   replace：用function score替换query score
    +   其它，例如：sum、avg、max、min

function score的运行流程如下：

+   1）根据**原始条件**查询搜索文档，并且计算相关性算分，称为**原始算分**（query score）
+   2）根据**过滤条件**，过滤文档
+   3）符合**过滤条件**的文档，基于**算分函数**运算，得到**函数算分**（function score）
+   4）将**原始算分**（query score）和**函数算分**（function score）基于**运算模式**做运算，得到最终结果，作为相关性算分。


**2）举例**

需求：给“如家”这个品牌的酒店排名靠前一些

翻译一下这个需求，转换为之前说的四个要点：

+   原始条件：不确定，可以任意变化
+   过滤条件：brand = "如家"
+   算分函数：可以简单粗暴，直接给固定的算分结果，weight
+   运算模式：比如求和

因此最终的DSL语句如下：

```shell
GET /hotel/_search
{
  "query": {
    "function_score": {
      "query": {  .... }, // 原始查询，可以是任意条件
      "functions": [ // 算分函数
        {
          "filter": { // 满足的条件，品牌必须是如家【品牌是如家的才加分，这里是加分条件】
            "term": {
              "brand": "如家"
            }
          },
          "weight": 2 // 算分权重为2
        }
      ],
      "boost_mode": "sum" // 加权模式，求和
    }
  }
}
```
测试，在未添加算分函数时，如家得分如下：


![image](/images/java/es/2729274-20230205173504670-1488473639.png)

添加了算分函数后，如家得分就提升了：

![image](/images/java/es/2729274-20230205173509427-1841421871.png)

### 布尔查询


> 布尔查询是一个或多个查询子句的组合，每一个子句就是一个**子查询**。子查询的组合方式有：
>
> +   must：必须匹配每个子查询，类似“与”
> +   should：选择性匹配子查询，类似“或”
> +   must_not：必须不匹配，**不参与算分**，类似“非”
> +   filter：必须匹配，**不参与算分**
>
> 注意：尽量在筛选的时候多使用不参与算分的must_not和filter，以保证性能良好

比如在搜索酒店时，除了关键字搜索外，我们还可能根据品牌、价格、城市等字段做过滤：

![image](/images/java/es/2729274-20230205173518420-1928367735.png)


每一个不同的字段，其查询的条件、方式都不一样，必须是多个不同的查询，而要组合这些查询，就必须用bool查询了。

需要注意的是，搜索时，参与**打分的字段越多，查询的性能也越差**。因此这种多条件查询时，建议这样做：

+   搜索框的关键字搜索，是全文检索查询，使用must查询，参与算分
+   其它过滤条件，采用filter查询。不参与算分

#### bool查询

1）语法

```shell
GET /hotel/_search
{
  "query": {
    "bool": {
      "must": [
        {"term": {"city": "上海" }}
      ],
      "should": [
        {"term": {"brand": "皇冠假日" }},
        {"term": {"brand": "华美达" }}
      ],
      "must_not": [
        { "range": { "price": { "lte": 500 } }}
      ],
      "filter": [
        { "range": {"score": { "gte": 45 } }}
      ]
    }
  }
}
```

2）示例

需求：搜索名字包含“如家”，价格不高于400，在坐标31.21,121.5周围10km范围内的酒店。

分析：

+   名称搜索，属于全文检索查询，应该参与算分。放到must中
+   价格不高于400，用range查询，属于过滤条件，不参与算分。放到must_not中
+   周围10km范围内，用geo_distance查询，属于过滤条件，不参与算分。放到filter中

![image](/images/java/es/2729274-20230205173526171-218786081.png)

# 设置搜索结果

