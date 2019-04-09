# 关于超时设置

> iplatform boot 1.1.0+ 版本支持

UI 调用 SERVICE 相关的超时参数有如下几个，默认值如下

```properties
# Ribbon 超时
ribbon.ConnectTimeout=10000
ribbon.ReadTimeout=60000
ribbon.MaxAutoRetriesNextServer=1
ribbon.MaxAutoRetries=0

# 断路器超时
hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds=180000
```

Ribbon 是 RestTemplate 调用时的超时设置

Hystrix 是断路器控制的超时设置，这个设置还和断路器重试次数有关，这个配置一定要大与 Ribbon 的配置，推荐公式 `(1 + ribbon.MaxAutoRetries + ribbon.MaxAutoRetriesNextServer) * ReadTimeout`

## 定制某个方法的超时

有的时候部分 SERVICE 接口的时常比较长，那么就需要针对某个接口单独定义超时时常，假设以下方法耗时较长，平均大概在2分钟才能返回

IndexClient.class

```java
@FeignClient("example-service")
public interface IndexClient {

    @RequestMapping(value = "exampleservice/api/v1/hello", method = RequestMethod.GET)
    public ResponseEntity<RestResponse<String>> hello();
}

```

单独配置这个方法的超时参数如下：

```yaml
example-service#/exampleservice/api/v1/hello:
	ribbon:
		ConnectTimeout: 10000
		ReadTimeout: 120000

hystrix:
  command:
    IndexClient#hello:
      execution:
        isolation:
          thread:
            timeoutInMilliseconds: 360000
```

**注意ribbon配置：** example-service#/exampleservice/api/v1/hello 由 `@FeignClient` 中的 name 和 `@RequestMapping` 中的 value 组成，中间用 `#` 号分割

**注意hystrix配置：** hystrix.command.IndexClient#hello 由 `@FeignClient` 所在类的类名 和 方法名组成，中间用 `#` 分割