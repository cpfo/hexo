---
title: 雪花算法 百度uid-generator生成的id变成负数
categories: [技术]
tags: [java,snowflake,唯一ID]
date: 2024-11-26 15:14:13
---


项目使用了百度的uid-generator来生成唯一ID，在2024-11-21时候，观察到部分项目生成的id变成了负数，下面分析了具体的原因以及给出了解决思路。

<!-- more -->

# 问题分析

在 `2024-11-21` 当天，开发环境发现了部分项目生成的id变成了负数，产生了业务异常，无法入库，经过排查，发现是由于使用了百度的uid-generator进行id生成，配置不当导致的。

项目中同时使用了 `DefaultUidGenerator` 和 `CachedUidGenerator` 两种生成策略。但是，在初始化配置时，只针对 `DefaultUidGenerator` 进行了调整，`CachedUidGenerator` 使用的默认值。

查看代码，发现 CachedUidGenerator 继承自 DefaultUidGenerator，在sequence上做了调整，但是前面依然是雪花算法的策略。

```java
* +------+----------------------+----------------+-----------+
* | sign |     delta seconds    | worker node id | sequence  |
* +------+----------------------+----------------+-----------+
*   1bit          28bits              22bits         13bits

```

Snowflake算法描述：指定机器 & 同一时刻 & 某一并发序列，是唯一的。据此可生成一个64 bits的唯一ID（long）。默认采用上图字节分配方式：

* sign(1bit)
  固定1bit符号标识，即生成的UID为正数。

* delta seconds (28 bits)
  当前时间，相对于时间基点"2016-05-20"的增量值，单位：秒，最多可支持约8.7年

* worker id (22 bits) 
  机器id，最多可支持约420w次机器启动。内置实现为在启动时由数据库分配，默认分配策略为用后即弃，后续可提供复用策略。

* sequence (13 bits)
  每秒下的并发序列，13 bits可支持每秒8192个并发。

在 `DefaultUidGenerator` 中有说明，默认配置是从`2016-05-20`开始，大概支持8.7年，到`2024-11-20 21:24:16`

> delta seconds: The next 28 bits, represents delta seconds since a customer epoch(2016-05-20 00:00:00.000).
> 
> Supports about 8.7 years until to 2024-11-20 21:24:16

CachedUidGenerator不是每次获取当前时间戳作为时间位上的数字，而是在系统启动时获取系统时间作为基础，后续在该时间上进行递增。

```java
    // 启动时获取当前时间
    this.lastSecond = new PaddedAtomicLong(TimeUnit.MILLISECONDS.toSeconds(System.currentTimeMillis()));

    // fill the rest slots until to catch the cursor
    List<Long> uidList = uidProvider.provide(lastSecond.incrementAndGet());
```

# 解决思路

## 调整时间基点

既然是由于初始时间 epochStr 导致的，可以考虑调整 epochStr 到接近当前时间的某个日期。

此时需要注意，如果想要保证生成的id长度固定为19位的话，新的日期不能距离当前时间太近，否则生成的id长度会发生变化。比如可以调整到1年前的日期，如 `2023-11-26` 。

生成id又会从 1 开头的数字开始，如 `1088488405369774084`

这种方案会有一个风险，生成的id可能会和调整前生成的id一样，导致id重复。原因如下

```java
// Allocate bits for UID
return bitsAllocator.allocate(currentSecond - epochSeconds, workerId, sequence);

public long allocate(long deltaSeconds, long workerId, long sequence) {
        return (deltaSeconds << timestampShift) | (workerId << workerIdShift) | sequence;
}

```

理论上，获取id时的时间戳和基准时间的差值(currentSecond - epochSeconds)如果一致，生成的id就会相同。

所以此方案可以用于临时紧急修复问题，不能长期使用。

## 调整位数的划分

可以调整 `timeBits` 到30位，增大2位，`seqBits` 调整到11位，减少2位。 此时可以支持的时间为2^30 -1 , 大概是34年，而每秒可以生成的序列为 2^11-1=2047个。理论上也能满足大部分系统的并发场景需要。

```java
    /** Bits allocate */
    protected int timeBits = 30;
    protected int workerBits = 22;
    protected int seqBits = 11;
```
调整之后，生成的id又会从一个较小的数字开始，理论上到达某个条件之后，生成的id还是有可能和历史已生成的id重复的可能，需要注意。

或者还可以调整 `workerBits` 的位数，默认是22位，接近420w次重启， 实际上可以按需进行取舍，考虑重复使用workId或者循环利用。



# 总结

在系统初始阶段，最好就要先确定好位数的划分，比如增大 timeBits ，保证系统可以支持到更长的时间。减少workerBits，调整 seqBits 等。

* 为什么线上系统在11-21号的时候没有报错

    由于线上大部分使用的是 `CachedUidGenerator`，所以是从系统启动时刻的时间戳开始计算的，如果系统近期没有重启，`lastSecond`会小于当前时间戳，并不会溢出。

* 为什么使用 CachedUidGenerator 生成的id顺序不是严格一致的。

    1. 由于使用了 RingBuffer，如果流量不均衡的话，不同机器上消耗的速度是不同的，可能导致某些机器上消费的速度快，lastSecond 增长的快。
    2. 如果服务使用了动态的扩容缩容方案，不同节点的启动时间相差也会很大，后启动的机器，lastSecond会大点，生成的id 也会偏大。


# 参考

* [uid-generator readme](https://github.com/baidu/uid-generator/blob/master/README.zh_cn.md)





