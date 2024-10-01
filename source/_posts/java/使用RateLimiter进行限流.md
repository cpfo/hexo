---
title: 使用RateLimiter进行限流
categories: [技术]
tags: [java,RateLimiter]
date: 2024-10-01 17:00:02
---

主要介绍使用RateLimiter进行请求的限流。

<!-- more -->


场景: 和第三方对接的项目中，为对接的第三方分配一个appid，需要针对appid进行限流，限制每个appid的请求不能超过指定的qps。

# Guava RateLimiter

`RateLimiter` 是针对单机进行限流的，但是在实际的生产环境中，服务一般是通过集群的方式进行部署。假设限制的QPS是300，集群一共5台机器，如果流量分布均衡的话，每台机器的QPS是 `300/5 = 60` 。

但实际上，通过轮询的方式，流量不可能做到绝对的均匀，所以可以预留一定的空间，比如每台机器的qps限制在80，这样即使某台机器的流量稍微偏高，也不太可能超过限制被拦截掉。

此种方式适合对限流不是很严格的情况，即使部分请求被拦截，也不影响业务。

`RateLimiter.create(rate)` 创建指定QPS的RateLimiter。

`tryAcquire()`  以非阻塞的方式获取令牌。

# RedissonRateLimiter

`RedissonRateLimiter` 是redisson中提供的分布式限流器。

`getRateLimiter(String name)` 获取对应name的 rate limiter。


# 代码

```java

@Slf4j
@RestController
public class TestRateController {

    @Autowired
    private RedissonClient redissonClient;

    private Map<String, RateLimiter> rateLimiterMap = new ConcurrentHashMap<>();
    /**
     * 针对appid配置请求频率
     */
    @ApolloJsonValue("${qps.rate:{}}")
    private Map<String, Integer> appIdRateMap = new HashMap<>();

    @RequestMapping(value = "/rate/test")
    public String testRateLimiter(@RequestParam String appId) {
        boolean result = tryAcquire(appId);
        log.info("获取限流的结果为 {}", result);
        if (result) {
            return "success";
        } else {
            return "rate limiter";
        }
    }


    @RequestMapping(value = "/redisson/rate")
    public String redissonRateLimiter(@RequestParam String appId) {

        RRateLimiter clientRateLimiter = redissonClient.getRateLimiter("limiter:".concat(appId));
        if (!clientRateLimiter.isExists()) {
            clientRateLimiter.setRate(RateType.OVERALL, 1, 1, RateIntervalUnit.SECONDS);
        }
        boolean result = clientRateLimiter.tryAcquire();
        log.info("获取限流的结果为 {}", result);
        if (result) {
            return "success";
        } else {
            return "rate limiter";
        }
    }

    public boolean tryAcquire(String appId) {
        Assert.notNull(appId, "appId不能为空");
        RateLimiter rateLimiter = rateLimiterMap.computeIfAbsent(appId, s -> {
            log.info("初始化appid {} 的限流器", appId);
            int rate = appIdRateMap.getOrDefault(appId, 70);
            return RateLimiter.create(rate);
        });
        boolean result = rateLimiter.tryAcquire();
        return result;
    }
}

```

