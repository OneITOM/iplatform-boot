# 服务跟踪配置手册

> 作者 张磊
>
> 框架基于Zipkin内置调用链跟踪功能，实现HTTP Request、Http Response，定时@Scheduled，Mybatis的调用链跟踪记录功能，每个服务调用链消息将通过集中配置的消息总线（ActiveMQ）发送到默认对列Q_MS_TRACK中，再由[跟踪服务](../../iplatform-common/TraceService.md)统一收集并记录到ElasticSearch中

## 前提

* 已经启动了跟踪服务 trace-service
* 已经启动了注册中心服务 discovery-service（并开启了集中配置功能）

## 服务参数配置

```properties
# 跟踪服务开关（默认true）
spring.sleuth.enabled=true
# JDBC跟踪服务开关（默认true）
spring.sleuth.mybatis.enabled=true
# 数据上报方式（可选项是 activemq、http），默认activemq
spring.zipkin.type=activemq
# 集中配置开关
spring.cloud.config.enabled=true
# 集中配置环境
spring.cloud.config.profile=测试
# 跟踪服务采样率设置
eureka.instance.metadataMap.trackSampling=1
```

## 数据结构

> 框架对Zipkin数据结构进行了扩展

### Zipkin标准结构

```json
{
	"traceId": "aa334f946a66d248",
	"id": "743942b4672a01cd",
	"name": "https:/auth/user",
	"parentId": "aa334f946a66d248",
	"timestamp": 1537866304523000,
	"duration": 60000,
	"annotations": [{
		"timestamp": 1537866304524000,
		"value": "cs",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"timestamp": 1537866304583000,
		"value": "cr",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}],
	"binaryAnnotations": [{
		"key": "sa",
		"value": true,
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}]
}
```

- traceId 跟踪ID（根ID）
- id 过程ID
- name 过程名称
- parentId 父过程ID
- timestamp 时间戳（毫秒）
- duration 执行耗时（纳秒)
- annotations  关键事件，只有四种，cs（Client Send）、sr（Server Receive）、ss（Server Send）、cr（Client Receive）
- annotations.timestamp 关键事件的发生时间
- annotations.value 只可能是cs,sr,ss,cr四种
- annotations.endpoint.serviceName 记录发生的服务名
- annotations.endpoint.port 记录发生的服务端口
- annotations.endpoint.ipv4 记录发生的服务器IP
- binaryAnnotations 自定义信息，结构与annotations一样

### iplatform扩展

> 基于Zipkin的binaryAnnotations扩展组件类型、耗时平均值、成功失败状态、所属业务系统、服务实例ID等

* 组建类型扩展
  * scheduled
  * jdbc
  * httpclient
  * httprservice
* 成功失败状态
  * succeed
* 业务系统
  * busisys
* 服务实例ID
  * serviceInstId
* 性能警告百分比阀值
  * optimize_warn_pct
* 性能警告类型
  - optimize_warn_type
* 异常类型
  * exception_class

#### SPAN头格式
> 扩展key
>
> * lc 组件类型
> * succeed 是否成功
> * busisys 所属业务系统
> * serviceInstId 服务实例ID
> * durationMoveAvg 移动平均耗时
> * optimize_warn_pct 性能警告百分比阀值
> * optimize_warn_type 性能告警类型
> * exception_class 异常类型，Exception的类名或者Http Status Code
```json
{
	"traceId": "b22d96fb827f754f",
	"id": "9402043fdbaa81c2",
	"name": "https:/auth/user",
	"parentId": "b22d96fb827f754f",
	"timestamp": 1537967701308000,
	"duration": 53000,
	"lc": "scheduled",
	"optimize_warn_type": "none",
    "exception_class": "none",
	"serviceInstId": "10.50.7.14::empty-service:58080",
	"succeed": "true",    
	"binaryAnnotations": [{
		"key": "lc",
		"value": "组件类型",
		"endpoint": {
			"serviceName": "服务ID",
			"ipv4": "服务IP地址",
			"port": "服务端口"
		}	  
	},{
	  "key": "succeed",
	  "value": "true",
	  "endpoint": {}
	},{
	  "key": "busisys",
	  "value": "OneITOM",
	  "endpoint": {}
	},{
	  "key": "durationMoveAvg",
	  "value": 60000,
	  "endpoint": {}
	},{
		"key": "serviceInstId",
		"value": "10.50.7.14::empty-service:58080",
		"endpoint": {}
	},{
		"key": "optimize_warn_pct",
		"value": "200.0%",
		"endpoint": {}
	},{
		"key": "optimize_warn_type",
		"value": "warn_http_optimize_time",
		"endpoint": {}
	},{
		"key": "exception_class",
		"value": "java.net.SocketTimeoutException",
		"endpoint": {}
	}
]
```

#### SPAN HttpClient

> 扩展 key
>
> * httpclient.method HTTP Method方法类型
> * httpclient.path HTTP客户端调用路径
> * httpclient.status_code HTTP客户端调用返回码
> * httpclient.url HTTP客户端调用全路径
> * sa HTTP客户端调用服务端地址

```json
{
	"traceId": "b1ce90a546e67ba5",
	"id": "c0e7a6f66745b037",
	"name": "https:/auth/user",
	"parentId": "b1ce90a546e67ba5",
	"timestamp": 1538033737523000,
	"duration": 63000,
	"annotations": [{
		"timestamp": 1538033737524000,
		"value": "cs",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"timestamp": 1538033737586000,
		"value": "cr",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}],
	"binaryAnnotations": [{
		"key": "busisys",
		"value": "OneITOM",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "durationMoveAvg",
		"value": "39000.0",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "httpclient.method",
		"value": "GET",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "httpclient.path",
		"value": "/auth/user",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "httpclient.status_code",
		"value": "200",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "httpclient.url",
		"value": "https://oneitom-auth:9999/auth/user",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "lc",
		"value": "httpclient",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "sa",
		"value": "true",
		"endpoint": {
			"serviceName": "auth-service",
			"ipv4": "10.22.1.234",
			"port": 9999
		}
	}, {
		"key": "serviceInstId",
		"value": "10.50.7.14::empty-service:58080",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "succeed",
		"value": "true",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}]
}
```

#### SPAN JDBC

> 扩展 key
>
> * jdbc.class Mapper接口类
> * jdbc.method Mapper接口类方法
> * jdbc.databasevendor 数据库厂家标示
> * jdbc.sql SQL语句
> * jdbc.sql_cost SQL耗时（单位纳秒）
> * jdbc.url 数据库JDBC连接串
> * jdbc.username 数据库JDBC用户名
> * jdbc.version JDBC驱动版本

```json
{
	"traceId": "b1ce90a546e67ba5",
	"id": "d3b277175669b1f6",
	"name": "o.i.m.e.s.d.testmapper.create",
	"parentId": "b1ce90a546e67ba5",
	"timestamp": 1538033737686000,
	"duration": 164000,
	"binaryAnnotations": [{
		"key": "busisys",
		"value": "OneITOM",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "durationMoveAvg",
		"value": "138000.0",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "jdbc.class",
		"value": "org.iplatform.microservices.emptyservice.service.dao.TestMapper",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "jdbc.databasevendor",
		"value": "H2",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "jdbc.method",
		"value": "create",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "jdbc.sql",
		"value": "insert into empty_test (is_enable,create_time) values (true,'2018-9-27 15:35:37')",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "jdbc.sql_cost",
		"value": "138000",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "jdbc.url",
		"value": "jdbc:h2:mem:memdb",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "jdbc.username",
		"value": "SA",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "jdbc.version",
		"value": "1.4.191 (2016-01-21)",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "lc",
		"value": "jdbc",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "serviceInstId",
		"value": "10.50.7.14::empty-service:58080",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "succeed",
		"value": "true",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}]
}
```

#### SPAN HttpService

> 扩展 key
>
> * httpservice.host 服务主机名
> * httpservice.method 服务方法名
> * httpservice.path 服务路径
> * httpservice.remoteaddr 远程请求地址
> * httpservice.url 服务全路径

```json
{
	"traceId": "b1ce90a546e67ba5",
	"id": "b1ce90a546e67ba5",
	"name": "http:/api/v1/test/testmethodwithdb",
	"timestamp": 1538033736991000,
	"duration": 901000,
	"annotations": [{
		"timestamp": 1538033736991000,
		"value": "sr",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"timestamp": 1538033737892000,
		"value": "ss",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}],
	"binaryAnnotations": [{
		"key": "busisys",
		"value": "OneITOM",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "durationMoveAvg",
		"value": "901000.0",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "httpservice.host",
		"value": "0.0.0.0",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "httpservice.method",
		"value": "GET",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "httpservice.path",
		"value": "/api/v1/test/testmethodwithdb",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "httpservice.remoteaddr",
		"value": "127.0.0.1",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "httpservice.url",
		"value": "https://0.0.0.0:58080/emptyservice//api/v1/test/testmethodwithdb?access_token=8af3680b14a233b58b9bdfa463d7172064e6181d2b4cffbc4f391920eb1f8ae34362a16fbc81377dc4790e2c6053d1097b0acb26a28fa4a70086fe3199199657be312b11d2e2f7a0a086c4c65ff88dbd204e0ebd2ba94e73e4b40a0c9bd65dd5",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "lc",
		"value": "httpservice",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "serviceInstId",
		"value": "10.50.7.14::empty-service:58080",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "succeed",
		"value": "true",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}]
}
```

#### SPAN Scheduled

> 扩展 key
>
> * scheduled.class 类名
> * scheduled.method 方法名
> * scheduled.config 配置

```json
{
	"traceId": "a82ca1044e28cbfa",
	"id": "a82ca1044e28cbfa",
	"name": "o.i.m.e.s.testservice.fixedrate",
	"timestamp": 1538033651605000,
	"duration": 2000,
	"binaryAnnotations": [{
		"key": "busisys",
		"value": "OneITOM",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "scheduled.class",
		"value": "org.iplatform.microservices.emptyservice.service.TestService",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "durationMoveAvg",
		"value": "1750.0",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "lc",
		"value": "scheduled",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "scheduled.method",
		"value": "fixedRate",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "scheduled.config",
		"value": "@org.springframework.scheduling.annotation.Scheduled(cron=, fixedRateString=, zone=, fixedDelay=30000, fixedRate=-1, initialDelayString=, initialDelay=-1, fixedDelayString=)",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "serviceInstId",
		"value": "10.50.7.14::empty-service:58080",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "succeed",
		"value": "true",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}]
}
```



## 数据样例

> 这个数据样例描述了一个调用链，这个调用链包含了3个调用过程

样例说明

| ID | 组件类型 | 名称 | 说明 |
| ---| ----- | ----- | -----|
| b1ce90a546e67ba5 | httpservice     | http:/api/v1/test/testmethodwithdb | 服务端收到请求 |
| c0e7a6f66745b037 | httpclient      | https:/auth/user                   | 服务端向认证服务鉴权 |
| d3b277175669b1f6 | jdbc            | o.i.m.e.s.d.testmapper.create      | 服务端调用SQL |

样例数据

```json
[{
	"traceId": "b1ce90a546e67ba5",
	"id": "c0e7a6f66745b037",
	"name": "https:/auth/user",
	"parentId": "b1ce90a546e67ba5",
	"timestamp": 1538033737523000,
	"duration": 63000,
	"annotations": [{
		"timestamp": 1538033737524000,
		"value": "cs",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"timestamp": 1538033737586000,
		"value": "cr",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}],
	"binaryAnnotations": [{
		"key": "busisys",
		"value": "OneITOM",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "durationMoveAvg",
		"value": "39000.0",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "httpclient.method",
		"value": "GET",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "httpclient.path",
		"value": "/auth/user",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "httpclient.status_code",
		"value": "200",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "httpclient.url",
		"value": "https://oneitom-auth:9999/auth/user",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "lc",
		"value": "httpclient",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "sa",
		"value": "true",
		"endpoint": {
			"serviceName": "auth-service",
			"ipv4": "10.22.1.234",
			"port": 9999
		}
	}, {
		"key": "serviceInstId",
		"value": "10.50.7.14::empty-service:58080",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "succeed",
		"value": "true",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}]
}, {
	"traceId": "b1ce90a546e67ba5",
	"id": "d3b277175669b1f6",
	"name": "o.i.m.e.s.d.testmapper.create",
	"parentId": "b1ce90a546e67ba5",
	"timestamp": 1538033737686000,
	"duration": 164000,
	"binaryAnnotations": [{
		"key": "busisys",
		"value": "OneITOM",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "durationMoveAvg",
		"value": "138000.0",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "jdbc.class",
		"value": "org.iplatform.microservices.emptyservice.service.dao.TestMapper",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "jdbc.databasevendor",
		"value": "H2",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "jdbc.method",
		"value": "create",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "jdbc.sql",
		"value": "insert into empty_test (is_enable,create_time) values (true,'2018-9-27 15:35:37')",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "jdbc.sql_cost",
		"value": "138000",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "jdbc.url",
		"value": "jdbc:h2:mem:memdb",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "jdbc.username",
		"value": "SA",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "jdbc.version",
		"value": "1.4.191 (2016-01-21)",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "lc",
		"value": "jdbc",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "serviceInstId",
		"value": "10.50.7.14::empty-service:58080",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "succeed",
		"value": "true",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}]
}, {
	"traceId": "b1ce90a546e67ba5",
	"id": "b1ce90a546e67ba5",
	"name": "http:/api/v1/test/testmethodwithdb",
	"timestamp": 1538033736991000,
	"duration": 901000,
	"annotations": [{
		"timestamp": 1538033736991000,
		"value": "sr",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"timestamp": 1538033737892000,
		"value": "ss",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}],
	"binaryAnnotations": [{
		"key": "busisys",
		"value": "OneITOM",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "durationMoveAvg",
		"value": "901000.0",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "httpservice.host",
		"value": "0.0.0.0",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "httpservice.method",
		"value": "GET",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "httpservice.path",
		"value": "/api/v1/test/testmethodwithdb",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "httpservice.remoteaddr",
		"value": "127.0.0.1",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "httpservice.url",
		"value": "https://0.0.0.0:58080/emptyservice//api/v1/test/testmethodwithdb?access_token=8af3680b14a233b58b9bdfa463d7172064e6181d2b4cffbc4f391920eb1f8ae34362a16fbc81377dc4790e2c6053d1097b0acb26a28fa4a70086fe3199199657be312b11d2e2f7a0a086c4c65ff88dbd204e0ebd2ba94e73e4b40a0c9bd65dd5",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "lc",
		"value": "httpservice",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "serviceInstId",
		"value": "10.50.7.14::empty-service:58080",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}, {
		"key": "succeed",
		"value": "true",
		"endpoint": {
			"serviceName": "empty-service",
			"ipv4": "10.50.7.13",
			"port": 58080
		}
	}]
}]
```

## 调用链存储

> 调用链的存储由[跟踪服务](../../iplatform-common/TraceService.md)负责

