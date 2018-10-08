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

* [ActiveMQ](middleware/ActiveMQ.md)-李光辉
* [Kafka](middleware/Kafka.md)-李光辉
* [ELK](middleware/ELK.md)-于胜强
* [Storm](middleware/Storm.md)-于胜强
* [MySQL](middleware/MysQL.md)-李晓琴
* [Redis](middleware/Redis.md)-于胜强
* [Nginx](middleware/Nginx.md)-杜臻
* [Mongo](middleware/Mongo.md)-于胜强

## 3. 开发手册

* [快速开始](QuickStart.md)-王立松

  > 注册服务、认证服务搭建

* [项目构建手册](YourFirstProject.md)-王立送

  > 快速构建一个SERVICE、UI的服务框架

* [应用打包部署手册](ProjectBuild.md)-王立松

* 编码规约

  * 定时任务规约
  * 并发处理规约
  * 异步处理规约
  * RESTfulAPI规约
  * 缓存使用规约
  * 无状态规约
  * 服务实例化规约
  * 基于配置的服务装载规约
  * 鉴权规约

* 中间件相关开发手册

  * [数据库开发手册](developer/database/README.md)-王立松
  * [ActiveMQ开发手册](developer/activemq/README.md)-王立松
  * [Flume开发手册](developer/flume/README.md)
  * [Kafka开发手册](developer/kafka/README.md)-王立松
  * [ElasticSearch开发手册](developer/elasticsearch/README.md)-于胜强
  * [Redis开发手册](developer/redis/README.md)-王立松
  * [Tomcat开发手册](developer/tomcat/README.md)-张磊

* 分布式相关开发手册

  * [文档服务集成手册](/developer/dfss/README.md)-于胜强
  * [集中日志开发手册](/developer/logger/README.md)
  * [分布式锁开发手册](developer/distributedlock/README.md)
  * [集中式缓存开发手册](developer/distributedcache/README.md)-未开始
  * [服务跟踪配置手册](developer/trace/README.md)
  * 负载均衡、流控、租户、路由标签配置手册-张磊
  * [消息总线集成手册](developer/messagebus/README.md)

* 其他

  * [日志](Logs.md)-未开始
  * [安全开发手册](Security.md)

  > 项目安全有关的配置说明，配置文件加密、单点认证、服务鉴权、漏洞应对等

## 4. 产品手册

- [采集产品手册](product/octopus/README.md)-未开始
- 云管平台手册
- [CMDB产品手册](product/cmdb/README.md)-未开始
- [自动化部署手册](product/autodeploy/README.md)-未开始
- [自动化运维手册](product/automatic/README.md)-未开始

## 5. 附件

* [框架参数说明](Properties.md)
* [版本跟踪](ChangeLog.md)
* [Spring Boot 1.3.5 Reference Guid](https://docs.spring.io/spring-boot/docs/1.3.5.RELEASE/reference/html/)

