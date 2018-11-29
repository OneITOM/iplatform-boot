# Redis开发手册

> 作者 张磊

## 配置数据源

>  Redis数据源配置支持三种方式，本别是单点模式、哨兵模式、集群模式

### 单点模式

```properties
spring.redis.database=0
spring.redis.host=127.0.0.1
spring.redis.port=6379
spring.redis.password=
spring.redis.timeout=2000ms
```

### 哨兵模式

```properties
spring.redis.database=0
spring.redis.port=6379
spring.redis.sentinel.master=mymaster
spring.redis.sentinel.nodes=192.168.0.1:26379,192.168.0.2:26379 
spring.redis.password=
```

### 集群模式

```properties
spring.redis.cluster.enabled=true
spring.redis.cluster.nodes=192.168.0.1:8000,192.168.0.2:8001,192.168.0.3:8002
spring.redis.cluster.timeout=30
spring.redis.cluster.max-redirects=12
spring.redis.password=
```

## 使用示例

```java
@Autowired
protected RedisTemplate<String, String> redisTemplate;

//计数器加1并返回
private long recordRate(String key, long duration, TimeUnit unit) {
    long count = redisTemplate.opsForValue().increment(key, 1);
    LOG.debug(String.format("[Redis] %s = %s", key, count));
    if (count == 1) {
        redisTemplate.expire(key, duration, unit);
    }
    return count;
}
```





