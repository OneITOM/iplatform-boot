# 分布式事务集成手册

> 分布式事务中间件基于 [Apache ServiceComb Pack 0.4.0版本](https://github.com/apache/servicecomb-pack) 构建

本手册指导如何在现有微服务中增加基于 [分布式事务中间件](../iplatform-common/ServiceCombAlpha.md) 的 Saga 分布式事务处理能力，以下说明基于样例 [example-dts](../)

样例描述：样例由三个微服务组成，分别如下：

* 订单服务 booking-service
* 酒店服务 car-service
* 租车服务 hotel-service

## 正常调用

以下时序图说明了正常情况下三个服务的调用方式以及分布式事务状态

![image-20190319141447113](/Volumes/MyWallet/nextcloud/iplatform-boot/iplatform-boot/developer/distributedtransactions/assets/image-20190319141447113.png)

## 异常调用

以下时序图说明了调用酒店服务时出现异常后的事务回滚动作



1. 增加依赖包

   ```xml
   <dependency>
       <groupId>org.apache.servicecomb.pack</groupId>
       <artifactId>omega-spring-starter</artifactId>
   </dependency>
   <dependency>
       <groupId>org.apache.servicecomb.pack</groupId>
       <artifactId>omega-spring-cloud-eureka-starter</artifactId>
   </dependency>
   <dependency>
       <groupId>org.apache.servicecomb.pack</groupId>
       <artifactId>omega-transport-resttemplate</artifactId>
   </dependency>
   <dependency>
       <groupId>org.apache.servicecomb.pack</groupId>
       <artifactId>omega-transport-feign</artifactId>
   </dependency>
   ```

   