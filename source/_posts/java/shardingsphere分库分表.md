---
title: shardingsphere分库分表
categories: [技术]
tags: [java,shardingsphere]
date: 2024-09-13 17:20:02
---

主要记录使用shardingsphere-jdbc 5.2.0进行分库，分表，分库加分表，读写分离，自定义复合分片算法等的操作步骤。

<!-- more -->

# 概念介绍

## 基因法

> 基因法原理 : 对一个数取余2的n次方，那么余数就是这个数的二进制的最后n位数。

* 举例

十进制数对10的n次幂取余，余数是10进制数的最后n位， 比如

11 % 10，余数 1

122 % 10，余数 2

122 % 100 , 余数 22

同理，对一个数取余2的n次方，余数就是这个数的二进制的最后n位数，然后可以再转为10进制。

## 为什么分片数要是2的n次幂

* 可以将取模算法优化成性能更高的位运算算法。

如： 11 % 4 等于 11 & (4-1) 

* 可以将表在多个库中均匀分布。

通常分表会和分库一起进行，比如需要分成2个库8个表，分表和分库的数量都是2的n次幂，可以实现均匀分布，8个表均分到2个库中，每个库4个表。

* 第三个也是最重要的原因，可以减少扩容时迁移的数据量，只需要迁移一半。

比如 原来分2个表，order_0,order_1

1. userId % 2 = 0 ，对应 order_0
2. userId % 2 = 1 , 对应 order_1

此时扩容到4个表

1. userId % 4 = 0 ，对应 order_0
2. userId % 4 = 1 , 对应 order_1
3. userId % 4 = 2 ，对应 order_2
4. userId % 4 = 3 , 对应 order_3

只需要迁移原来在 order_0, order_1 中，且 userId % 4 >=2 数据就行。
1. userId % 4 = 2的数据， 从 order_0 迁移到 order_2 
2. userId % 4 = 3的数据， 从order_1  迁移到 order_3


# 具体配置和操作

以电商系统中用户的订单表为例。

电商系统中，常见的业务场景是通过userId查询用户的订单列表，因此使用userId作为分片键，查询时可以快速定位到数据库和表。

但是也会有通过订单号进行查询的情况，默认情况下，非分片键的查询，需要在所有分片上进行查询，然后对结果进行聚合，这样效率非常低，日志中sql如下

```
Actual SQL: ds0 ::: SELECT  user_id,order_id,address_id,status  FROM t_order_0 
 WHERE order_id = ? UNION ALL SELECT  user_id,order_id,address_id,status  FROM t_order_1 
 WHERE order_id = ? ::: 
```
此时可以考虑新增一个订单号和userId的映射关系表即索引表。

## 索引表法

将订单号和userId的映射关系，单独保存到一个表中，通过订单号进行查询时，先从索引表中查询到对应的userId，然后在查询条件中加上userId，这样只需要2步就可以查询出结果。

```sql
select user_id from order_user_relation where order_id = xxx;

select * from t_order where order_id = xxx and user_id = xxx;
```

为了提升查询效率，可以在加上一层分布式缓存，将映射关系保存到redis中。但是这样会带来一定的问题，比如数据库和缓存的存储量增大。


## 基因法自定义复合分片算法

为了解决聚合查询或者索引表的问题，业界一般采用基因法来将分片键的信息，保存到非分片键中，比如取userId的后4位，拼接到order_id后面。

假设订单号的规则为 :  时间戳 + 随机数 + 用户id后四位。这样可以从订单号的后四位中截取到用户id的基因，可以定位到具体的分片位置，一次查询就可以查到对应的数据。

比如淘宝的订单号，后6位都是一样的，大概率也是用户id的后6位。


* 算法实现

```java
    @Override
    public Collection<String> doSharding(Collection availableTargetNames, ComplexKeysShardingValue complexKeysShardingValue) {

        int count = availableTargetNames.size();
        // 判断分片数必须是2的整数次幂
        if ((count & (count - 1)) != 0) {
            log.warn("分片数不是2的整数次幂，当前分片数 {}", count);
            throw new IllegalArgumentException("分片数量不是2的整数次幂，当前数量:" + count);
        }

        ArrayList list = new ArrayList(availableTargetNames);

        List<String> result = new ArrayList<>();
//        String logicName = complexKeysShardingValue.getLogicTableName();

        log.debug("availableTargetNames的值 {}", JSON.toJSONString(availableTargetNames));
        log.debug("complexKeysShardingValue {}", JSON.toJSONString(complexKeysShardingValue));

        // 先判断userId， 如果有userId， 则直接计算
        Map<String, Collection<Comparable<?>>> nameAndValueMap = complexKeysShardingValue.getColumnNameAndShardingValuesMap();
        Collection<Comparable<?>> userIdValueCollection = nameAndValueMap.get(userIdKey);
        if (!CollectionUtils.isEmpty(userIdValueCollection)) {
            userIdValueCollection.stream().findFirst().ifPresent(o -> {
                Long index = (Long) o % count;
                log.debug("以userid {} 对 分片数{} 取余的结果为 {}", o, count, index);
//                result.add(logicName + "_" + index);
                result.add(String.valueOf(list.get(index.intValue())));
            });
        } else {
            // 从订单号中截取用户id基因
            Collection<Comparable<?>> orderIdCollection = nameAndValueMap.get(orderIdKey);
            orderIdCollection.stream().findFirst().ifPresent(comparable -> {
                String orderId = String.valueOf(comparable);
                String subString = orderId.substring(orderId.length() - userIdSuffixLength);
                int index = Integer.parseInt(subString) % count;
                log.debug("orderId中基因 {} 对 分片数{} 取余的结果为 {}", subString, count, index);
//                result.add(logicName + "_" + index);
                result.add(String.valueOf(list.get(index)));
            });
        }
        return result;
    }

```

* 配置信息

```properties
# 模式配置
spring.shardingsphere.mode.type=Standalone
spring.shardingsphere.mode.repository.type=JDBC
# 数据源配置 配置真实数据源
spring.shardingsphere.datasource.names=ds0

spring.shardingsphere.datasource.ds0.jdbc-url=jdbc:mysql://localhost:3306/shop_ds_0?serverTimezone=Asia/Shanghai&useSSL=false&useUnicode=true&characterEncoding=UTF-8
spring.shardingsphere.datasource.ds0.type=com.zaxxer.hikari.HikariDataSource
spring.shardingsphere.datasource.ds0.driver-class-name=com.mysql.jdbc.Driver
spring.shardingsphere.datasource.ds0.username=root
spring.shardingsphere.datasource.ds0.password=root

# 广播表规则列表
spring.shardingsphere.rules.sharding.broadcast-tables=t_address

# 标准分片表配置
# 由数据源名 + 表名组成，以小数点分隔。多个表以逗号分隔，支持 inline 表达式。
# 缺省表示使用已知数据源与逻辑表名称生成数据节点，用于广播表（即每个库中都需要一个同样的表用于关联查询，多为字典表）或只分库不分表且所有库的表结构完全一致的情况
spring.shardingsphere.rules.sharding.tables.t_order.actual-data-nodes=ds0.t_order_$->{0..1}
# 复合分片，自定义策略
spring.shardingsphere.rules.sharding.tables.t_order.table-strategy.complex.sharding-columns=user_id,order_id
# 分片算法名称
spring.shardingsphere.rules.sharding.tables.t_order.table-strategy.complex.sharding-algorithm-name=t_order_complex_custom_algorithm

# 自定义复合分片算法
spring.shardingsphere.rules.sharding.sharding-algorithms.t_order_complex_custom_algorithm.type=CLASS_BASED
spring.shardingsphere.rules.sharding.sharding-algorithms.t_order_complex_custom_algorithm.props.strategy=complex
spring.shardingsphere.rules.sharding.sharding-algorithms.t_order_complex_custom_algorithm.props.algorithmClassName=com.bsd.xxyp.sharding.config.GeneComplexKeysShardingAlgorithm

```

# 参考文章

1. [大众点评订单系统分库分表实践](https://tech.meituan.com/2016/11/18/dianping-order-db-sharding.html)
2. [基于Apache ShardingSphere的核心业务分库分表实践](https://community.sphere-ex.com/t/topic/674)
3. [DIY 3 种分库分表分片算法](https://www.cnblogs.com/chengxy-nds/p/18108596)
4. [实战：Springboot3+ShardingSphere5.2.1生产级分库分表实现](https://mp.weixin.qq.com/s/PqXJKXCzQlMVwR4EjPpXew)
5. [聊聊分库分表后非Sharding Key查询的三种方案](https://mp.weixin.qq.com/s/TdCqxsBeSnugWXP_MuosdQ)
6. [一些补充的知识点-MySQL大表分库分表基因法](https://mp.weixin.qq.com/s/6sI92hEH6Sxt7nzXWDW-kA)
7. [看完这一篇，ShardingSphere-jdbc 实战再也不怕了](https://mp.weixin.qq.com/s/3cP_emfG15qJapeQwV5gtQ)
8. [SpringBoot 2 种方式快速实现分库分表，轻松拿捏](https://www.cnblogs.com/chengxy-nds/p/17513945.html)
9. [分布式数据库：如何正确选择分片键？](https://mp.weixin.qq.com/s/1syeeqzusxt4Z-yJff0mng)

# FAQ整理

1. 点评的文章中，userid后4位，为什么说最大是8192个表

按照上面的规范，分片数必须是2的n次幂，4位数的最大值是9999，假设分片数是 2的14次幂 16384，4位数字对16384取余的结果范围是[0, 9999], 实际计算出来需要的分片数是10000个，不是2的n次幂，不满足分片数规则。
所以最大只能取 8192 个

2. 点评的方式和基因法的区别。

点评文章中userid和orderid的取模都是用的userid的后四位，所以最大是8192个。基因法里面，通过userid计算分片是按照userid % 2^n，按照订单id计算分片 = orderId.substring(length-4) % 2^n，
是使用订单号的后四位进行计算，所以userid后四位的最大分片数是16个。




