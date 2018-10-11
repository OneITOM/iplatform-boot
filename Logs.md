# 日志

> 作者 张磊
>
> 框架内置logback框架，并提供默认的控制台、文件、kafka日志输出配置，每个产品可以自己扩展这个配置，定义日志格式，输出周期等

## 日志标准化

> 日志的输出可以通过spring.profiles.active参数进行控制

* 只输出到控制台 

  > spring.profiles.active=dev

* 只输出到文件 

  > spring.profiles.active=prod

* 输出到文件并且输出到kafka，详情可以查看[集中日志配置手册](developer/logger/README.md)

  > spring.profiles.active=prod
  >
  > logger.kafka.enabled=true

* 日志格式

  > 日志格式定义在框架的logback-spring.xml中，不建议修改，如果要扩展则需要在项目的resources目录下自己创建logback-spring.xml文件

  默认logback-spring.xml

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <configuration>
  	<springProperty name="spring.application.name" source="spring.application.name"/>
  	<springProperty name="server.host" source="server.host"/>
  	<springProperty name="eureka.instance.instance-id" source="eureka.instance.instance-id"/>
  	<springProperty name="spring.cloud.config.busisys" source="spring.cloud.config.busisys"/>
  	<include resource="org/springframework/boot/logging/logback/defaults.xml" />	
  	<property name="LOG_FILE" value="./logs/${spring.application.name}.log"/>	
  	<property name="LOG_PATTERN" value="${server.host} ${spring.cloud.config.busisys} ${spring.application.name} ${eureka.instance.instance-id} %d{yyyy-MM-dd HH:mm:ss.SSS}${LOG_LEVEL_PATTERN:-%5p} ${PID:- } [%t] %logger{39}.%M : %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}"/>
  	
  	<springProfile name="dev">
  		<include resource="logback-dev.xml"/>
  	</springProfile>
  	<springProfile name="prod">
  		<include resource="logback-prod.xml"/>
  	</springProfile>	
  	
  	<jmxConfigurator/>
  </configuration>
  ```

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

**注意:调整日志pattern输出格式将可能导致集中日志与跟踪服务解析冲突😬，因为集中日志要解析日志中的一些内容，例如主机或者所属业务系统，服务跟踪要解析日志中的traceId**

## 修改日志级别

> 日志级别支持两种修改方式

* 查看日志级别

  > 可以通过RESTful API 查看日志界别

  ```shell
  curl https://10.50.7.6:58080/emptyservice/jolokia/exec/ch.qos.logback.classic:Name=default,Type=ch.qos.logback.classic.jmx.JMXConfigurator/getLoggerLevel/ROOT
  ```

  返回结果如下

  ```json
  {  
     "request":{  
        "arguments":[  
           "ROOT"
        ],
        "mbean":"ch.qos.logback.classic:Name=default,Type=ch.qos.logback.classic.jmx.JMXConfigurator",
        "operation":"getLoggerLevel",
        "type":"exec"
     },
     "status":200,
     "timestamp":1539254990,
     "value":"INFO"
  }
  ```

* 通过参数控制日志级别

  > 重启服务生效

  ```properties
  logging.level.root=INFO
  logging.level.包名=INFO
  ```

* 通过RESTful修改日志级别

  > 动态立即生效

  ```shell
  curl https://10.50.7.6:58080/emptyservice/jolokia/exec/ch.qos.logback.classic:Name=default,Type=ch.qos.logback.classic.jmx.JMXConfigurator/setLoggerLevel/ROOT/debug
  ```

  返回结果如下

  ```json
  {  
     "request":{  
        "arguments":[  
           "ROOT",
           "debug"
        ],
        "mbean":"ch.qos.logback.classic:Name=default,Type=ch.qos.logback.classic.jmx.JMXConfigurator",
        "operation":"setLoggerLevel",
        "type":"exec"
     },
     "status":200,
     "timestamp":1539255322,
     "value":null
  }
  ```

## 日志集中化

参考[集中日志开发手册](developer/logger/README.md)

