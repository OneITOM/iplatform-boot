# IPLATFORM-BOOT

>  作者 张磊

IPLATFORM-BOOT 是以 SpringCloud Brixton.RELEASE 为核心的微服务开发框架

## 1. 公共服务部署手册

* [注册服务部署手册](iplatform-common/DiscoveryService.md)
* [认证服务部署手册](iplatform-common/AuthService.md)
* [文档服务部署手册](iplatfrom-common/DfssService.md)
* [跟踪服务部署手册](iplatfrom-common/TraceService.md)
* [监控服务部署手册](iplatfrom-common/AdminService.md)
* [消息服务部署手册](iplatfrom-common/NotifyService.md)

## 2. 中间件部署手册

* [ActiveMQ](middleware/ActiveMQ.md)
* [Kafka](middleware/Kafka.md)
* [ELK](middleware/ELK.md)
* [Storm](middleware/Storm.md)
* [MySQL](middleware/MysQL.md)
* [Redis](middleware/Redis.md)
* [Nginx](Nginx.md)

## 3. 开发手册

* [快速开始](QuickStart.md) 

  > 注册服务、认证服务搭建

* [项目构建手册](YourFirstProject.md)

  > 快速构建一个SERVICE、UI的服务框架

* [应用打包部署手册](ProjectBuild.md)

* [编码规范](CodeStandards.md)

* 中间件相关开发手册

  * [数据库开发手册](developer/database/README.md)
  * [ActiveMQ开发手册](developer/activemq/README.md)
  * [Flume开发手册](developer/flume/README.md)
  * [Kafka开发手册](developer/kafka/README.md)
  * [ElasticSearch开发手册](developer/elasticsearch/README.md)
  * [Redis开发手册](developer/redis/README.md)

* 分布式相关开发手册

  * 文档服务集成手册
  * 集中日志集成手册
  * [分布式锁开发手册](developer/distributedlock/README.md)
  * [集中式缓存开发手册](developer/distributedcache/README.md)
  * [服务跟踪配置手册](developer/trace/README.md)
  * 负载均衡、流控、租户、路由标签配置手册
  * 消息总线集成手册

* 其他

  * 日志
  * [安全开发手册](Security.md)

  > 项目安全有关的配置说明，配置文件加密、单点认证、服务鉴权、漏洞应对等

## 产品手册

- [采集产品手册](product/octopus/README.md)
- 云管平台手册
- [CMDB产品手册](product/cmdb/README.md)
- [自动化部署手册](product/autodeploy/README.md)
- [自动化运维手册](product/automatic/README.md)

## 附件

* [框架参数说明](Properties.md)
* [版本跟踪](ChangeLog.md)

