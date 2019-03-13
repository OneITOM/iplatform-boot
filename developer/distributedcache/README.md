# 分布式缓存开发手册

> 作者 张磊

分布式缓存基于hazelcast提供了一种在系统中嵌入分布式缓存的实现，缓存结构支持Queue、Map、List、Topic

## 增加依赖

```xml
<dependency>
    <groupId>org.iplatform</groupId>
    <artifactId>iplatform-starter-dcache</artifactId>
    <version>1.1.0-SNAPSHOT</version>
</dependency>
```

## 增加配置

配置集群IP地址，多个逗号分隔

```yaml
iplatform.dcache.members=192.168.0.1,192.168.0.2
```

## 代码

注入缓存服务对象

```java
@Autowired
private EmbedDistributedService embedDistributedService;
```

添加数据到缓存

```java
embedDistributedService.map(map_name).put(k,v);
```

从缓存中读取数据

```java
embedDistributedService.map(map_name).get(k)
```

更多实用细节可以看 [iplatform-dcache](https://github.com/OneITOM/iplatform-boot-example/tree/master/example-dcache) 的集成测试代码

## 参数说明

可以通过参数定义缓存属性

1. Queue

   iplatform.dcache.queue.{缓存名称}.{属性名}

   | 属性名           | 默认值            | 说明                                                         |
   | ---------------- | ----------------- | ------------------------------------------------------------ |
   | maxSize          | Integer.MAX_VALUE | 队列最大长度，超过此长度后在此放入对象将根据超时参数抛出超时异常 |
   | backupCount      | 1                 | 定义同步备份的数量                                           |
   | asyncBackupCount | 0                 | 异步备份的数量                                               |

2. List

   iplatform.dcache.list.{缓存名称}.{属性名}

   | 属性名           | 默认值            | 说明                                             |
   | ---------------- | ----------------- | ------------------------------------------------ |
   | maxSize          | Integer.MAX_VALUE | 队列最大长度，超过此长度后再新增将丢弃最早的对象 |
   | backupCount      | 1                 | 定义同步备份的数量                               |
   | asyncBackupCount | 0                 | 异步备份的数量                                   |

3. Map

   iplatform.dcache.map.{缓存名称}.{属性名}

   | 属性名            | 默认值            | 说明                       |
   | ----------------- | ----------------- | -------------------------- |
   | evictionPolicy    | LRU               | 驱逐策略                   |
   | timeToLiveSeconds | Integer.MAX_VALUE | 从创建起的过期时间         |
   | maxIdleSeconds    | Integer.MAX_VALUE | 从最后一次访问起的过期时间 |
   | maxSize           | Integer.MAX_VALUE | 最大存储数量，取决于JVM    |

4. Topic
   iplatform.dcache.topic.{缓存名称}.{属性名}

   | 属性名              | 默认值 | 说明                                                         |
   | ------------------- | ------ | ------------------------------------------------------------ |
   | readBatchSize       | 10     | 尝试读取的最小消息数                                         |
   | topicOverloadPolicy | BLOCK  | 消息过期策略：DISCARD_OLDEST：丢弃最旧的项目。  DISCARD_NEWEST：丢弃最新的项目。  BLOCK：等到Ringbuffer中的项目过期。  ERROR：Ringbuffer没有空间立即抛出TopicOverloadException |
