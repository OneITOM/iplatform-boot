# 集中日志配置手册

> 作者 张磊

此框架通过内嵌的KafkaAppender实现日志的集中收集，可以通过启动参数开启kafka记录功能并配置kafka参数，后续可以通过Logstash将Kafka中的数据存储到ES中持久化和查询，具体日志标准化参看[日志标准化](../../Logs.md)

## 前提

项目需要引入kafka依赖包

```xml
<dependency>
	<groupId>org.apache.kafka</groupId>
	<artifactId>kafka_2.11</artifactId>
</dependency>
```

## 集中日志配置

> 集中日志配置需要开启logger.kafka.enabled参数

```properties
logger.kafka.enabled=true
logger.kafka.bootstrapServers=localhost:9092
logger.kafka.zookeeper=localhost:2181
logger.kafka.partitions=20
logger.kafka.replication=1
```

## 参数说明

logger.kafka.enabled=true

> 开启日志kafka记录模式（必填）

logger.kafka.bootstrapServers=localhost:9092

> 配置kafka地址（必填）

logger.kafka.zookeeper=localhost:2181

> 配置zookeeper地址（必填）

logger.kafka.partitions=20

> 分区数量（非必填）默认20，如果分区已经事先创建完毕，此处配置请确保和已经存在的topic一致

logger.kafka.replication=1

> 副本数量（非必填）默认1，如果分区已经事先创建完毕，此处配置请确保和已经存在的topic一致

### Topic

服务发送的topic自动创建，iplatform-{spring.application.name}-log

### Key

发送Topic的Key采用固定的命名格式 进程ID+@+主机名

## 启动

> 启动时看到如下日志表示kafka日志记录已经启动

```bash
启动Kafka日志记录,bootstrap.servers=oneitom-kafka:9092,topic=empty-service-log
ProducerConfig values: 
	compression.type = none
	metric.reporters = []
	metadata.max.age.ms = 300000
	metadata.fetch.timeout.ms = 60000
	reconnect.backoff.ms = 50
	sasl.kerberos.ticket.renew.window.factor = 0.8
	bootstrap.servers = [oneitom-kafka:9092]
	retry.backoff.ms = 100
	sasl.kerberos.kinit.cmd = /usr/bin/kinit
	buffer.memory = 33554432
	timeout.ms = 5000
	key.serializer = class org.apache.kafka.common.serialization.ByteArraySerializer
	sasl.kerberos.service.name = null
	sasl.kerberos.ticket.renew.jitter = 0.05
	ssl.keystore.type = JKS
	ssl.trustmanager.algorithm = PKIX
	block.on.buffer.full = false
	ssl.key.password = null
	max.block.ms = 5000
	sasl.kerberos.min.time.before.relogin = 60000
	connections.max.idle.ms = 540000
	ssl.truststore.password = null
	max.in.flight.requests.per.connection = 5
	metrics.num.samples = 2
	client.id = 
	ssl.endpoint.identification.algorithm = null
	ssl.protocol = TLS
	request.timeout.ms = 5000
	ssl.provider = null
	ssl.enabled.protocols = [TLSv1.2, TLSv1.1, TLSv1]
	acks = 0
	batch.size = 16384
	ssl.keystore.location = null
	receive.buffer.bytes = 32768
	ssl.cipher.suites = null
	ssl.truststore.type = JKS
	security.protocol = PLAINTEXT
	retries = 0
	max.request.size = 4096000
	value.serializer = class org.apache.kafka.common.serialization.ByteArraySerializer
	ssl.truststore.location = null
	ssl.keystore.password = null
	ssl.keymanager.algorithm = SunX509
	metrics.sample.window.ms = 30000
	partitioner.class = class org.apache.kafka.clients.producer.internals.DefaultPartitioner
	send.buffer.bytes = 131072
	linger.ms = 0
```

##  数据格式

> 注意：traceid 和 tracespanid 只有在开启了跟踪服务后才会有值

```json
{
	"ip": "192.168.0.1",
    "busisys": "OneITOM",
	"timestamp": 1538965917237,
	"serviceId": "example-service",
	"serviceInstId": "192.168.0.1::example-service:58080",
	"traceid": "6959dcf98356c725",
	"tracespanid": "9ef659e1deb2126c",
	"level": "INFO",
	"thread": "http-nio-58080-exec-4",
	"logger": " o.i.m.c.s.i.m.SQLInterceptor",
	"message": "sqltrace [insert into empty_test (is_enable,create_time) values (true,'2018-10-8 10:31:49')] cost [49ms]"
}
```

| key           | value                              | 说明                 |
| ------------- | ---------------------------------- | -------------------- |
| ip            | 192.168.0.1                        | 应用所在服务器IP地址 |
| busisys       | OneITOM                            | 应用所属业务系统名称 |
| timestamp     | 1538965917237                      | 日志产生时间         |
| serviceId     | example-service                    | 服务ID               |
| serviceInstId | 192.168.0.1::example-service:58080 | 服务实例ID           |
| traceid       | 6959dcf98356c725                   | 跟踪ID               |
| tracespanid   | 9ef659e1deb2126c                   | 当前环节SPANID       |
| level         | INFO                               | 日志级别             |
| thread        | http-nio-58080-exec-4              | 线程ID               |
| logger        | o.i.m.c.s.i.m.SQLInterceptor       | 日志产生类名         |
| message       |                                    | 日志正文             |



