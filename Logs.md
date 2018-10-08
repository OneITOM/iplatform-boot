# 日志

> 作者 张磊
>
> 框架内置logback框架，并提供默认的控制台、文件、kafka日志输出配置，每个产品可以自己扩展这个配置，定义日志格式，输出周期等

## 日志控制

> 日志的输出可以通过spring.profiles.active参数进行控制

* 只输出到控制台 

  > spring.profiles.active=dev

* 只输出到文件 

  > spring.profiles.active=prod

* 输出到文件并且输出到kafka，详情可以查看[集中日志配置手册](developer/logger/README.md)

  > spring.profiles.active=prod
  >
  > logger.kafka.enabled=true

## 控制台日志配置

console-appender.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<included>
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>${LOG_PATTERN}</pattern>
            <charset>utf8</charset>
        </encoder>
    </appender>
</included>
```

## 文件日志配置

file-appender.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<included>
	<appender name="FILE"
		class="ch.qos.logback.core.rolling.RollingFileAppender">
		<encoder>
			<pattern>${LOG_PATTERN}</pattern>
		</encoder>
		<file>${LOG_FILE}</file>
		<rollingPolicy class="ch.qos.logback.core.rolling.FixedWindowRollingPolicy">
			<fileNamePattern>${LOG_FILE}.%i</fileNamePattern>
			<minIndex>1</minIndex>
			<maxIndex>20</maxIndex>
		</rollingPolicy>
		<triggeringPolicy
			class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
			<MaxFileSize>50MB</MaxFileSize>
		</triggeringPolicy>
	</appender>
</included>
```

## Kafka日志配置

kafka-appender.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<included>
	<springProperty name="enabled" source="logger.kafka.enabled"
		defaultValue="false" />
	<springProperty name="bootstrapServers" source="logger.kafka.bootstrapServers" />
	<springProperty name="zookeeper" source="logger.kafka.zookeeper" />
	<springProperty name="partitions" source="logger.kafka.partitions"
		defaultValue="20" />
	<springProperty name="replication" source="logger.kafka.replication"
		defaultValue="1" />
	<springProperty name="topic" source="spring.application.name" />
	<appender name="kafkaAppender"
		class="org.iplatform.microservices.core.logback.kafka.KafkaAppender">
		<encoder
			class="org.iplatform.microservices.core.logback.kafka.encoding.LayoutKafkaMessageEncoder">
			<layout class="ch.qos.logback.classic.PatternLayout">
				<pattern>${LOG_PATTERN}</pattern>
			</layout>
		</encoder>
		<enabled>${enabled}</enabled>
		<topic>${topic}-log</topic>
		<zookeeper>${zookeeper}</zookeeper>
		<partitions>${partitions}</partitions>
		<replication>${replication}</replication>
		<deliveryStrategy
			class="org.iplatform.microservices.core.logback.kafka.delivery.AsynchronousDeliveryStrategy" />
		<producerConfig>bootstrap.servers=${bootstrapServers}</producerConfig>
		<producerConfig>acks=0</producerConfig>
		<producerConfig>max.block.ms=5000</producerConfig>
		<producerConfig>request.timeout.ms=5000</producerConfig>
		<producerConfig>timeout.ms=5000</producerConfig>
		<producerConfig>max.request.size=4096000</producerConfig>
	</appender>
	<appender name="KAFKA" class="ch.qos.logback.classic.AsyncAppender">
		<appender-ref ref="kafkaAppender" />
	</appender>
</included>

```

## 自定日志格式

> 以上三个日志配置是系统框架默认的配置，每个服务可以自定义自己的日志格式，只需要在本项目的resources目录下创建console-appender.xml、file-appender.xml、kafka-appender.xml文件，并自定义xml内容即可

**注意:调整日志pattern输出格式将可能导致集中日志与跟踪服务解析冲突**😬