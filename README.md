# IPLATFORM-BOOT

>  作者 张磊 2018-9-30

IPLATFORM-BOOT 是以 SpringCloud Brixton.RELEASE 为核心的微服务开发框架，简化了微服务开发的复杂度，集成了常用的中间件，修复了SpringCloud的BUG

## 1. 公共服务部署手册

* [注册服务部署手册](iplatform-common/DiscoveryService.md)
* [认证服务部署手册](iplatform-common/AuthService.md)
* [文档服务部署手册](iplatfrom-common/DfssService.md)-于胜强
* [跟踪服务部署手册](iplatform-common/TraceService.md)
* [监控服务部署手册](iplatform-common/AdminService.md)
* [消息服务部署手册](iplatform-common/NotifyService.md)-王立松

## 2. 中间件部署手册

* [ActiveMQ](middleware/ActiveMQ.md)-(未开始)
* [Kafka](middleware/Kafka.md)-(未开始)
* [ELK](middleware/ELK.md)-(未开始)
* [Storm](middleware/Storm.md)-(未开始)
* [MySQL](middleware/MysQL.md)-(未开始)
* [Redis](middleware/Redis.md)-(未开始)
* [Nginx](middleware/Nginx.md)-(未开始)
* [Mongo](middleware/Mongo.md)-(未开始)

## 3. 开发手册

* [项目构建手册](YourFirstProject.md)

* [应用打包部署手册](ProjectBuild.md)-(未开始)

* 编码规约

  * [定时任务规约](developer/coding/Schedule.md)
  * 文件上传规约
  * 并发处理规约
  * [异步处理规约](developer/coding/Async.md)
  * RESTful API规约
  * 缓存使用规约
  * 个性化配置规约
  * 鉴权规约

* 开发手册

  * [数据库开发手册](developer/database/README.md)-(未开始)
  * [ActiveMQ开发手册](developer/activemq/README.md)-(未开始)
  * [Flume开发手册](developer/flume/README.md)
  * [Kafka开发手册](developer/kafka/README.md)-(未开始)
  * [ElasticSearch开发手册](developer/elasticsearch/README.md)-于胜强
  * [Redis开发手册](developer/redis/README.md)-(未开始)
  * [Tomcat开发手册](developer/tomcat/README.md)-(未开始)

* 分布式相关开发手册

  * [文档服务集成手册](/developer/dfss/README.md)-(未开始)
  * [集中日志开发手册](/developer/logger/README.md)
  * [分布式锁开发手册](developer/distributedlock/README.md)
  * [集中式缓存开发手册](developer/distributedcache/README.md)-(未开始)
  * [服务跟踪配置手册](developer/trace/README.md)
  * [负载均衡配置手册](developer/loadbalance/README.md)
  * [消息总线集成手册](developer/messagebus/README.md)
  * [多租户配置手册](developer/multitenant/README.md)

* 其他

  * [日志标准化](Logs.md)
  * [安全开发手册](Security.md)


## 4. 微服务管控

- [微服务管控平台部署手册](product/triden/README.md)
- [微服务管控平台集成手册](product/triden/DEV.md)

## 5. 附件

* [框架参数说明](Properties.md)
* [IPv6 Support](IPv6.md)
* [版本跟踪](ChangeLog.md)
* [Spring Boot 1.3.5 Reference Guid](https://docs.spring.io/spring-boot/docs/1.3.5.RELEASE/reference/html/)

## 6.贡献者

文档：[王立松](https://github.com/coderwls) [杜臻](https://github.com/duzhen020623)