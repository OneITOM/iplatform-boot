# 版本跟踪

> 作者 张磊

## iplatform-boot 0.0.1

> 孵化阶段：基于Spring Cloud Brixton.RELEASE版本封装基础框架

## iplatform-boot 0.0.2

> 孵化阶段：增加OAuth2支持，增加注册中心支持，增加HTTPS支持

## iplatform-boot 0.0.3

> 孵化阶段：增加监控服务支持，增加服务跟踪支持

## iplatform-boot 0.0.4

> 重大改进，简化了微服务的配置文件，平台拆分成三个开发包

| iplatform-service | 后台服务包 |
| ----------------- | ---------- |
| iplatform-ui      | UI服务包   |
| iplatform-util    | 公共服务包 |

### 新功能

集成Redis框架

集成Kafka框架

集成ActiveMQ框架

### 改进

简化application.yml配置

增加配置文件加密功能

## iplatform-boot 0.0.5

### 新功能

增加服务降级断路器监控支持

XSS漏洞可配置框架

集成Flume引擎框架

集成SQL注入漏洞注解支持@SQLInjection

MyBatis多数据源类型支持

多数据源支持

框架支持License功能

发布自监控指标自定义说明

增加服务标签路由功能

增加租户标签功能

发布方法级鉴权和页面鉴权方案

### 改进

支持HTTP/HTTPS开关参数，-Dssl参数设置

ActiveMQ支持同步异步模式

UI支持统一错误页面

默认禁用h2 web console

Spring版本升级到4.2.9

框架支持多服务间消息头透传 x-ipf-xxxxx

增加禁用注册中心开关

增加Redis集群连接方式

支持自动化编译部署

### Bug修复

修复生成WAR包后SSL错误（土法）

修复多数据源配置不支持配置文件加密



## iplatform-boot 0.0.6

### 新功能

提供统一日志下载页面

增加流量控制功能

分布式锁框架支持，支持基于db，mongodb，redis，zookeeper，redis等方式的锁

增加集中日志框架

增加CSRF跨站请求伪造解决方案

增加点击劫持X-Frame-Options解决方案

增加不安全HTTP方法解决方案

增加敏感信息泄漏解决方案

增加Slow Http Denial of Service Attack漏洞解决方案

增加自动生成Docker镜像方案

增加MVC方法级鉴权的方案

增加@SkipSleuth，用于在@Scheduled前跳过跟踪链

### Bug修复

解决开启Flume后无法启动问题

修复SSL/TLS 受诫礼(BAR-MITZVAH)攻击漏洞

### 改进

提供框架瘦身说明

支持流控、路由标签、租户标签动态修改

修改admin的默认密码为admin@boco

## iplatform-boot 1.0.0

### 新功能

集中配置支持

增加消息框架，支持发送指标，服务起停事件，SQL错误或耗时事件，HTTP错误事件

支持开启第二端口（非SSL）

增加远程停止服务RESTful接口

增加断路器指标支持

增加框架级消息总线支持MessageBusService，@MessageBusConsumer

页面支持URL地址重写标签#ProxyURL.resolverURL（需要启动[访问代理服务](iplatform-common/DiscoveryHAProxy.md)）

集成数据库版本管理Flyway

增加@RateLimit支持RESTful API访问限速

增加SQL敏感词定义，默认不允许drop,truncate

### Bug修复

修复无租户服务可以调用租户服务的BUG

修复了@SQLInjection 表达式中默认敏感词括号没有转义的问题

修复了@SQLInjection检测到侵入后SQLInjectionPolicy.BREAK策略时sql依然继续执行的问题

修复跟踪服务cs-cr时binaryAnnotations的sa不是目标服务名、ip、端口的BUG

修复发现服务EurekaInstanceRenewedEvent中InstanceInfo属性为空的BUG

### 改进

流量控制增加乒乓功能，如果没有流控大于1的服务，那么也可以调用流控等于0的服务

改进路由标签选择算法

日志标准化改进，增加了租户，服务实例ID

跟踪服务增加SQL个跟踪功能

增加跟踪链通过ActiveMQ方式发送spring.zipkin.type=messagebus 

跟踪服务增加lc类型httpclient，httpservice，jdbc

默认关闭服务跟踪功能，可通过spring.sleuth.enabled=true开启

默认不开启集中配置功能，可通过spring.cloud.config.enabled=true开启

默认不开启HTTPS，可通过 -Dssl=true开启

跟踪服务Span存储到ES的时候在JSON根节点增加succeed，exception_class，optimize_warn_type，lc，serviceInstId属性

支持了server.tomcat.accept-count参数

数据库版本管理支持动态数据源

集成sharding-jdbc支持客户端数据库分片组件

基于数据库的分布式锁支持通过参数 `iplatform.scheduled.lock.jdbc.tablename` 自定义表名，默认表名为 `IPLATFORM_LOCK`



## iplatform-boot 1.1.0

### 新功能

集成hazelcast的嵌入分布式缓存

支持为 @FeginClient 方法单独配置超时[参数](Timeout.md)

支持自定义 @RateLimit 实现

微服务管控：增加服务数据库版本配置上报功能

### BUG修复

mongo自动装配增加 `spring.data.mongodb.uri` 判断

### 改进

重构模块，增加单独的依赖管理模块

# Roadmap

分布式UUID生成器

调用链跟踪-ActiveMQ、Redis、Kafka

Fegin异步调用支持

分布式事务框架集成

页面提交增加csrf防御

支持HTTP2.0






