---
title: ES-RestAPI的使用
categories: [技术]
tags: [java,elascitsearch]
date: 2025-01-17 18:13:40
---

RestAPI的使用 [Java High Level REST Client 7.12](https://www.elastic.co/guide/en/elasticsearch/client/java-rest/7.12/java-rest-high.html)

需要注意的是， 此api对应的es版本为7.12.1，目前此api已被官方标记为deprecated，如果使用新的es版本的话，需要使用新的api [java-api-client](https://www.elastic.co/guide/en/elasticsearch/client/index.html) 。

<!-- more -->


# API操作索引库


> JavaRestClient操作elasticsearch的流程基本类似。核心是client.indices()方法来获取索引库的操作对象。
>
> 索引库操作的基本步骤：【可以根据发送请求那步的第一个参数，来判断需要创建什么XXXXRequest】
>
> +   初始化RestHighLevelClient
> +   创建XxxIndexRequest。XXX是Create、Get、Delete
> +   准备DSL（ Create时需要，其它是无参）
> +   发送请求。调用RestHighLevelClient#indices().xxx()方法，xxx是create、exists、delete

## mapping映射分析

> **根据MySQL数据库表结构（建表语句），去写索引库结构JSON。表和索引库一一对应**
>
> **注意**：地理坐标、组合字段。索引库里的地理坐标是一个字段：`坐标：维度,精度` 。copy_to组合字段作用是供用户查询（输入关键字可以查询多个字段）

创建索引库，最关键的是mapping映射，而mapping映射要考虑的信息包括：

+ 字段名
+ 字段数据类型
+ 是否参与搜索
+ 是否需要分词
+ 如果分词，分词器是什么？

其中：

+   字段名、字段数据类型，可以参考数据表结构的名称和类型
+   是否参与搜索要分析业务来判断，例如图片地址，就无需参与搜索
+   是否分词呢要看内容，内容如果是一个整体就无需分词，反之则要分词
+   分词器，我们可以统一使用ik_max_word

来看下酒店数据的索引库结构:



```json
PUT /hotel
{
  "mappings": {
    "properties": {
      "id": {
        "type": "keyword"
      },
      "name":{
        "type": "text",
        "analyzer": "ik_max_word",
        "copy_to": "all"
      },
      "address":{
        "type": "keyword",
        "index": false
      },
      "price":{
        "type": "integer"
      },
      "score":{
        "type": "integer"
      },
      "brand":{
        "type": "keyword",
        "copy_to": "all"
      },
      "city":{
        "type": "keyword",
        "copy_to": "all"
      },
      "starName":{
        "type": "keyword"
      },
      "business":{
        "type": "keyword"
      },
      "location":{
        "type": "geo_point"
      },
      "pic":{
        "type": "keyword",
        "index": false
      },
      "all":{
        "type": "text",
        "analyzer": "ik_max_word"
      }
    }
  }
}
```
几个特殊字段说明：

+ location：地理坐标，里面包含精度、纬度
+ all：一个组合字段，其目的是将多字段的值 利用copy\_to合并，提供给用户搜索

地理坐标说明：

![image](/images/java/es/2729274-20230205172124085-626335563.png)

copy_to说明：

![image](/images/java/es/2729274-20230205172128097-289087887.png)

## 初始化RestClient

在elasticsearch提供的API中，与elasticsearch一切交互都封装在一个名为RestHighLevelClient的类中，必须先完成这个对象的初始化，建立与elasticsearch的连接。

分为三步：

1）引入es的RestHighLevelClient依赖：

```xml
    <dependency>
        <groupId>org.elasticsearch.client</groupId>
        <artifactId>elasticsearch-rest-high-level-client</artifactId>
        <version>7.12.1</version>
    </dependency>

```

2）初始化RestHighLevelClient：这里一般在启动类或者配置类里注入该Bean，用于告诉Java 访问ES的ip地址

```java

    RestHighLevelClient client;
    
    @BeforeEach
    public void init(){
        client = new RestHighLevelClient(
        RestClient.builder(
        new HttpHost("10.1.40.136", 9200, "http")));
    }

```

## 索引库CRUD

### 创建索引库


> 代码分为三步：
>
> +   1）创建Request对象。因为是创建索引库的操作，因此Request是CreateIndexRequest。
> +   2）添加请求参数，其实就是DSL的JSON参数部分。因为json字符串很长，这里是定义了静态字符串常量MAPPING\_TEMPLATE，让代码看起来更加优雅。
> +   3）发送请求，client.indices()方法的返回值是IndicesClient类型，封装了所有与索引库操作有关的方法。

创建索引库的API如下：

![image](/images/java/es/2729274-20230205172138463-1252488196.png)

单元测试的代码

```java
    @Test
    void createHotelIndex() throws IOException {
        // 1.创建Request对象
        CreateIndexRequest request = new CreateIndexRequest("hotel");
        // 2.准备请求的参数：DSL语句 , json 内容为上面 PUT /hotel 中的json内容
        request.source(json, XContentType.JSON);
        // 3.发送请求
        CreateIndexResponse createIndexResponse = client.indices().create(request, RequestOptions.DEFAULT);
        // 重复创建会报错
        log.info("创建索引的结果为 {}", createIndexResponse.isAcknowledged());
    }

```


### 删除索引库

> 三步走：
>
> +   1）创建Request对象。这次是DeleteIndexRequest对象
> +   2）准备参数。这里是无参
> +   3）发送请求。改用delete方法

删除索引库的DSL语句非常简单：

```json
DELETE /hotel
```

```java
    @Test
    public void testDeleteHotelIndex() throws IOException {
        // 1.创建Request对象
        DeleteIndexRequest request = new DeleteIndexRequest("hotel");
        // 2.发送请求
        AcknowledgedResponse acknowledgedResponse = client.indices().delete(request, RequestOptions.DEFAULT);
        log.info("删除索引的结果为 {}", acknowledgedResponse.isAcknowledged());
    }

```

### 查询索引库

> 三步走：
>
> +   1）创建Request对象。这次是GetIndexRequest对象
> +   2）准备参数。这里是无参
> +   3）发送请求。改用exists方法


判断索引库是否存在，本质就是查询，对应的DSL是：

```shell
GET /hotel
```

```java

    @Test
    void testExistsHotelIndex() throws IOException {
        // 1.创建Request对象
        GetIndexRequest request = new GetIndexRequest("hotel");
        // 2.发送请求
        boolean exists = client.indices().exists(request, RequestOptions.DEFAULT);
        // 3.输出
        log.info("索引是否存在 {}", exists);
    }
```

# API操作文档

> 这里更多的是先读取Mysql中的数据，然后再存进ES中。
>
> 文档操作的基本步骤：【可以根据发送请求那步的第一个参数，反过来判断需要创建什么XXXXRequest】
>
> +   初始化RestHighLevelClient
> +   创建XxxRequest。XXX是Index、Get、Update、Delete、Bulk
> +   准备参数（Index、Update、Bulk时需要）
> +   发送请求。调用RestHighLevelClient#.xxx()方法，xxx是index、get、update、delete、bulk
> +   解析结果（Get时需要）

初始化`RestHighLevelClient` 的操作和索引库的一样。


##  批量导入文档

> 三步走：
>
> +   1）创建Request对象。这里是BulkRequest
> +   2）准备参数。批处理的参数，就是其它Request对象，这里就是多个IndexRequest
> +   3）发起请求。这里是批处理，调用的方法为client.bulk()方法

案例需求：利用BulkRequest批量将数据库数据导入到索引库中。

步骤如下：

+   利用mybatis-plus查询酒店数据
+   将查询到的酒店数据（Hotel）转换为文档类型数据（HotelDoc）
+   利用JavaRestClient中的BulkRequest批处理，实现批量新增文档


**语法说明：**

批量处理BulkRequest，其本质就是将多个普通的CRUD请求组合在一起发送。

其中提供了一个add方法，用来添加其他请求：

![image](/images/java/es/2729274-20230205172220573-1990074801.png)

可以看到，能添加的请求包括：

+   IndexRequest，也就是新增
+   UpdateRequest，也就是修改
+   DeleteRequest，也就是删除

因此Bulk中添加了多个IndexRequest，就是批量新增功能了。示例：

![image](/images/java/es/2729274-20230205172227358-759338253.png)


```java
    @Test
    public void testBulk() throws IOException {
        BulkRequest bulkRequest = new BulkRequest();
        bulkRequest.add(new IndexRequest("testcrud").id("2").source("{\"age\":10,\"info\":\"我是中国人\"}", XContentType.JSON));
        bulkRequest.add(new IndexRequest("testcrud").id("3").source("{\"age\":15,\"info\":\"这是一段测试的文本内容\"}", XContentType.JSON));
        BulkResponse response = client.bulk(bulkRequest, RequestOptions.DEFAULT);
        log.info("批量添加文档的结果 {}", Arrays.stream(response.getItems()).collect(Collectors.toMap(BulkItemResponse::getId, BulkItemResponse::status)));
    }
```

## 批量新增文档


> 四步走：
>
> +   0）创建索引库实体类
> +   1）创建Request对象
> +   2）准备请求参数，也就是DSL中的JSON文档
> +   3）发送请求 （注意：这里直接使用client.xxx()的API，不再需要client.indices()了）

我们要将数据库的酒店数据查询出来，写入elasticsearch中。


**1）创建索引库实体类**

> 一般实体类里包含经纬度都需要创建一个新的实体类，将经纬度拼成一个字段

数据库查询后的结果是一个Hotel类型的对象。结构如下：

```java
@Data
@TableName("tb_hotel")
public class Hotel {
    @TableId(type = IdType.INPUT)
    private Long id;
    private String name;
    private String address;
    private Integer price;
    private Integer score;
    private String brand;
    private String city;
    private String starName;
    private String business;
    private String longitude;
    private String latitude;
    private String pic;
}
```


与我们的索引库结构存在差异：

+   longitude和latitude需要合并为location

因此，我们需要定义一个新的类型，与索引库结构吻合：

```java
package cn.itcast.hotel.pojo;

import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@NoArgsConstructor
public class HotelDoc {
    private Long id;
    private String name;
    private String address;
    private Integer price;
    private Integer score;
    private String brand;
    private String city;
    private String starName;
    private String business;
    private String location;
    private String pic;

    public HotelDoc(Hotel hotel) {
        this.id = hotel.getId();
        this.name = hotel.getName();
        this.address = hotel.getAddress();
        this.price = hotel.getPrice();
        this.score = hotel.getScore();
        this.brand = hotel.getBrand();
        this.city = hotel.getCity();
        this.starName = hotel.getStarName();
        this.business = hotel.getBusiness();
        this.location = hotel.getLatitude() + ", " + hotel.getLongitude();
        this.pic = hotel.getPic();
    }
}
```


**2）新增代码**

新增文档的DSL语句如下：

```json
POST /{索引库名}/_doc/1
{
    "name": "Jack",
    "age": 21
}
```

对应的java代码如图：

![image](/images/java/es/2729274-20230205172259030-1988634807.png)


我们导入酒店数据，基本流程一致，但是需要考虑几点变化：

+ 酒店数据来自于数据库，我们需要先查询出来，得到hotel对象
+ hotel对象需要转为HotelDoc对象
+ HotelDoc需要序列化为json格式

## 查询文档


> 查询文档是根据id查询的，所以没有批量查询
>
> 三步走：
>
> +   1）准备Request对象。这次是查询，所以是GetRequest
> +   2）发送请求，得到结果。因为是查询，这里调用client.get()方法
> +   3）解析结果，就是对JSON做反序列化

查询的DSL语句如下：

```json
GET /hotel/_doc/{id}
```

非常简单，因此代码大概分两步：

+ 准备Request对象
+ 发送请求

不过查询的目的是得到结果，解析为HotelDoc，因此难点是结果的解析。完整代码如下：

![image](/images/java/es/2729274-20230205172309780-381577770.png)

可以看到，结果是一个JSON，其中文档放在一个`_source`属性中，因此解析就是拿到`_source`，使用工具反序列化为Java对象即可。

```java
@Test
public void testGetDocumentById() throws IOException {
    // 查询文档
    GetRequest getRequest = new GetRequest("testcrud", "2");
    GetResponse getResponse = client.get(getRequest, RequestOptions.DEFAULT);
    log.info("查询文档的结果为 {}", getResponse.getSourceAsString());
}

```


## 批量删除文档


> 三步走：
>
> +   1）准备Request对象，因为是删除，这次是DeleteRequest对象。要指定索引库名和id
> +   2）准备参数，无参
> +   3）发送请求。因为是删除，所以是client.delete()方法


删除的DSL为是这样的：

```json
DELETE /hotel/_doc/{id}
```

```java
@Test
public void testDeleteDocumentById() throws IOException {
    // 删除文档
    DeleteRequest deleteRequest = new DeleteRequest("testcrud", "2");
    DeleteResponse deleteResponse = client.delete(deleteRequest, RequestOptions.DEFAULT);
    log.info("删除文档的结果为 {}", deleteResponse.status());
}
```

## 批量修改文档

> 三步走：
>
> +   1）准备Request对象。这次是修改，所以是UpdateRequest
> +   2）准备参数。也就是JSON文档，里面包含要修改的字段
> +   3）更新文档。这里调用client.update()方法

修改有两种方式：

+   全量修改：本质是先根据id删除，再新增
+   增量修改：修改文档中的指定字段值

在RestClient的API中，全量修改与新增的API完全一致，判断依据是ID：

+   如果新增时，ID已经存在，则修改
+   如果新增时，ID不存在，则新增

**只演示增量修改：**


代码示例如图：

![image](/images/java/es/2729274-20230205172320621-611636092.png)

```java
@Test
public void testUpdateDocumentById() throws IOException {
    // 修改文档
    UpdateRequest updateRequest = new UpdateRequest("testcrud", "2");

    Map<String, String> nameMap = new HashMap<>();
    nameMap.put("firstName", "三");
    nameMap.put("lastName", "张");
    updateRequest.doc(
            "info", "测试下name",
            "name", nameMap
    );
    UpdateResponse updateResponse = client.update(updateRequest, RequestOptions.DEFAULT);
    log.info("更新文档的结果为 {}", updateResponse.status().getStatus());
}

```





**参考文章**

1. [ElasticSearch (ES从入门到精通一篇就够了)](https://www.cnblogs.com/buchizicai/p/17093719.html)
2. 