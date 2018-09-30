# Flume开发手册

> 作者 张磊

此框架已经内嵌了[flume 1.6.0](http://flume.apache.org/releases/content/1.6.0/FlumeUserGuide.html) 框架，研发人员可以通过配置参数激活内嵌Flume实现数据流处理

## 开启Flume

通过参数开启Flume

```properties
flume.enabled=true
```

## Flume 参数配置

> 配置方式兼容1.6.0官方配置，只需要在官方配置前增加flume.作为前缀即可，可以在项目的application.yml或者启动参数中指定flume的配置

```properties
flume.a1.sources = r1
flume.a1.sinks = k1
flume.a1.channels = c1

# Describe/configure the source
flume.a1.sources.r1.type = netcat
flume.a1.sources.r1.bind = localhost
flume.a1.sources.r1.port = 44444

# Describe the sink
flume.a1.sinks.k1.type = logger

# Use a channel which buffers events in memory
flume.a1.channels.c1.type = memory
flume.a1.channels.c1.capacity = 1000
flume.a1.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
flume.a1.sources.r1.channels = c1
flume.a1.sinks.k1.channel = c1
```

