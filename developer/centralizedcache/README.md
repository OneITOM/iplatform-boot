# 集中缓存开发手册

> 作者 张磊

集中缓存主要用于在集群部署的时集中管理缓存，解决有状态服务的缓存状态问题，框架提供基于Redis的集中式缓存实现，Redis的部署可以参考中间件部署手册中的[Redis部署手册](../../middleware/Redis.md)

## 前提

* 已经部署Redis
* 项目配置了Redis参数，参考[Redis开发手册](../redis/README.md)

## 基于注解使用

参考[缓存使用规约](../coding/Cache.md)

## 原生Redis

> 可直接获取RedisTemplate对象进行Redis操作，自己实现缓存的存取

```java
@Autowired
protected RedisTemplate<String, Object> redisTemplate;

private long recordRate(String key, long duration, TimeUnit unit) {
    long count = redisTemplate.opsForValue().increment(key, 1);
    LOG.debug(String.format("[Redis] %s = %s", key, count));
    if (count == 1) {
        redisTemplate.expire(key, duration, unit);
    }
    return count;
}
```

