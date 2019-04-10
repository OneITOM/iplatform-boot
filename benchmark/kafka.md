# Kafka 基准测试

> 作者 张磊

## 数据估算

* 单条数据512B

* 按每服务每天16小时（早7点至晚11点）产生2GB数据估算

  * 每秒数据量

    1GB/57600秒=36KB

  * 每秒数据条数

    36KB/512B=72条

* 按管理100个服务估算

  * 每日数据量 200GB
  * 每日数据条数 414W
  * 每秒数据量 3.4MB
  * 每秒数据条数 7200


## 配置

### 物理服务器配置

| 名称     | 描述                                                        |
| -------- | ----------------------------------------------------------- |
| 服务器   | Dell PowerEdge R720xd                                       |
| CPU      | Intel(R) Xeon(R) CPU E5-2620 v2 @ 2.10GHz                   |
| 核数     | 24                                                          |
| 内存     | 128G                                                        |
| 硬盘     | 机械                                                        |
| 网卡     | Broadcom Corporation NetXtreme BCM5720 Gigabit Ethernet * 4 |
| 管理程序 | VMware ESXi, 6.0.0, 3073146                                 |

### 虚拟机

| 服务器 | CPU  | 内存 | 磁盘 | 网络     |      |
| ------ | ---- | ---- | ---- | -------- | ---- |
| VM     | 4    | 16   | 200G | VMXNET 3 |      |
| VM     | 4    | 16   | 200G | VMXNET 3 |      |
| VM     | 4    | 16   | 200G | VMXNET 3 |      |

### 网络带宽

> 集群服务器之间共享网络带宽（6.94 Gbits/sec）

```shell
[ ID] Interval           Transfer     Bandwidth       Retr  Cwnd
[  4]   0.00-1.00   sec   695 MBytes  5.83 Gbits/sec  136    711 KBytes
[  4]   1.00-2.00   sec   776 MBytes  6.51 Gbits/sec    0    960 KBytes
[  4]   2.00-3.00   sec   882 MBytes  7.41 Gbits/sec    0    960 KBytes
[  4]   3.00-4.00   sec  1.04 GBytes  8.89 Gbits/sec    0   1.08 MBytes
[  4]   4.00-5.00   sec   946 MBytes  7.93 Gbits/sec    0   1.08 MBytes
[  4]   5.00-6.00   sec   736 MBytes  6.18 Gbits/sec    0   1.11 MBytes
[  4]   6.00-7.00   sec   726 MBytes  6.09 Gbits/sec    0   1.16 MBytes
[  4]   7.00-8.00   sec   829 MBytes  6.95 Gbits/sec    0   1.20 MBytes
[  4]   8.00-9.00   sec  1009 MBytes  8.46 Gbits/sec  813    440 KBytes
[  4]   9.00-10.00  sec   615 MBytes  5.16 Gbits/sec    0    641 KBytes
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bandwidth       Retr
[  4]   0.00-10.00  sec  8.08 GBytes  6.94 Gbits/sec  949             sender
[  4]   0.00-10.00  sec  8.08 GBytes  6.94 Gbits/sec                  receiver
```

### 操作系统配置

* ulimit -n 65535

### Kafka 配置

> [版本 kafka-2.11-1.1.0](http://kafka.apache.org/11/documentation.html#configuration)
>

[Broker Configs](http://kafka.apache.org/11/documentation.html#brokerconfigs)

| 参数名                    | 值          |                                     |
| ---------------------------- | ---------------- | ----------------------------------- |
| num.network.threads          | 4                | 处理网络请求的最大线程数，CPU核数+1 |
| num.io.threads               | 8                | 处理磁盘IO线程数，CPU核数*2         |
| num.partitions               | 5                | 默认分区5                           |
| num.replica.fetchers         | 1                | Leader中进行复制的线程数            |
| default.replication.factor   | 2                |                                     |
| log.flush.interval.messages  | 40000            | 每写入4W条刷数据到磁盘              |
| log.flush.interval.ms        | 10000            | 每间隔10秒钟刷数据到磁盘            |
| log.retention.hours          | 12               | 清理策略，每12小时定期清理过期数据  |
| log.segment.bytes            | 1073741824       | 段文件大小1G                        |
| replica.fetch.max.bytes      | 1000000 | Broker可复制的消息的最大字节数 |
| replica.lag.time.max.ms      | 20000            | 复制延迟等待时间                    |
| replica.lag.max.messages     | 10000            | 复制延迟消息数                      |
| replica.fetch.min.bytes      | 1                | 拉取最小字节数                      |
| replica.fetch.max.bytes      | 10 * 1024 * 1024 | 拉取最大字节数                      |
| queued.max.requests          | 500              | 等待IO线程处理的请求队列最大数      |
| zookeeper.session.timeout.ms | 6000    | ZK会话超时时间 |
| message.max.bytes            | 1000000 | Broker能接收消息的最大字节数 |
| request.timeout.ms  | 10000   | 发送请求超时10s |

[Producer Configs](http://kafka.apache.org/11/documentation.html#producerconfigs)

> 此发送者参数目的是为了尽可能避免高压发送时产生的超时错误

| 参数名             | 值         | 说明                                              |
| ------------------ | ---------- | ------------------------------------------------- |
| buffer.memory      | 1073741824 | 每发送线程的缓冲大小，50MB                        |
| compression.type   | none       |                                                   |
| linger.ms          | 0          | 聚合消息时间间隔                                  |
| batch.size         | 10485760   | 批次发送大小，10MB                                |
| asks               | 0 或 1     | 不需要Broker确认返回（存在丢数据风险，可以设置1） |
| request.timeout.ms | 60000      |                                                   |
| retries            | 500        |                                                   |
| max.block.ms       | 60000      |                                                   |
| max.request.size   | 10485760   |                                                   |
| send.buffer.bytes  | 104857600  |                                                   |

[Consumer Configs](http://kafka.apache.org/11/documentation.html#newconsumerconfigs)

| 参数名                    | 值               | 说明 |
| ------------------------- | ---------------- | ---- |
| num.consumer.fetchers     | 1                |      |
| fetch.min.bytes           | 1                |      |
| fetch.max.bytes           | 52428800         |      |
| fetch.wait.max.ms         | 500              |      |
| isolation.level           | read_uncommitted |      |
| heartbeat.interval.ms     | 3000             |      |
| max.partition.fetch.bytes | 1048576          |      |
| session.timeout.ms        | 10000            |      |
| auto.offset.reset         | earliest         |      |
| auto.commit.interval.ms   | 5000             |      |
| connections.max.idle.ms   | 540000           |      |
| enable.auto.commit        | true             |      |
| exclude.internal.topics   | true             |      |
| max.poll.interval.ms      | 300000           |      |
| max.poll.records          | 500              |      |
| eceive.buffer.bytes       | 65536            |      |
| request.timeout.ms        | 305000           |      |

### Zookeepr 配置

> [版本 zookeeper-3.4.6](http://zookeeper.apache.org/doc/r3.4.6/zookeeperAdmin.html)

zoo.cfg

| 参数名                    | 值   |                                    |
| ------------------------- | ---- | ---------------------------------- |
| autopurge.purgeInterval   | 12   | 事务日志定期清理频率，1小时        |
| autopurge.snapRetainCount | 7    | 十五日志定期清理后，保留的文件数目 |

## 基准测试
>  总计发送6G数据，按照每秒发送10M（日均843GB），20M（日均1.68T），50M（日均4.21T），100M（日均84T）进行测试

### 写入测试

> 客户端千兆网接入

| 副本 | 分区 | 每秒发送字节 | 每秒写字节 | 每秒写入条数 | 总延迟 |
| ---- | ---- | ------------ | ---------- | ------------ | ------ |
| 1    | 1    | 2MB          | 3.8MB      | 6298         | 0ms    |
| 1    | 1    | 10MB         | 16.5MB     | 27680        | 73ms   |
| 1    | 1    | 20MB         | 27.9MB     | 46783        | 96ms   |
| 1    | 1    | 50MB         | 48.3MB     | 80886        | 614ms  |
| 1    | 1    | 100MB        | 65.6.9MB   | 109890       | 11.8s  |
| 2    | 5    | 2MB          | 3.8MB      | 6400         | 2ms    |
| 2    | 5    | 10MB         | 16.7MB     | 27973        | 84ms   |
| 2    | 5    | 20MB         | 28MB       | 47350        | 474ms  |
| 2    | 5    | 50MB         | 40.4MB     | 67652        | 1.6s   |
| 2    | 5    | 100MB        | 59.8MB     | 100088       | 8s     |

### 消费测试

> 连续消费100W条记录（总计消费597MB），FetchSize分别按照 2MB、10MB、20MB、50MB、100MB测试

| Fetch Size | 消息总数 | 消息总大小 | 每秒消费大小 | 每秒消费条数 |
| ---------- | -------- | ---------- | ------------ | ------------ |
| 2MB        | 100W     | 597MB      | 76.4MB       | 12.79W       |
| 10MB       | 100W     | 597MB      | 64.7MB       | 10.83W       |
| 20MB       | 100W     | 597MB      | 108.4MB      | 18.16W       |
| 50MB       | 100W     | 597MB      | 125.7MB      | 21.05W       |
| 100MB      | 100W     | 597MB      | 127.6MB      | 21.36W       |

### 写入&消费测试

> 取单独写、消费测试中相对均衡的数据每秒发送50MB，同时按照FetchSize=50MB消费测试

写入压测

| 副本 | 分区 | 每秒发送字节 | 每秒写字节 | 每秒写入条数 | 总延迟 |
| ---- | ---- | ------------ | ---------- | ------------ | ------ |
| 2    | 5    | 50MB         | 42MB       | 7.1W         | 261ms  |

消费压测

| Fetch Size | 消息总数 | 每秒消费大小 | 每秒消费条数 |
| ---------- | -------- | ------------ | ------------ |
| 50MB       | 100W     | 105.6MB      | 17.68W       |

## 总结

* 3节点Kafka集群，Topic 5分区，2副本
* 日志量日均1T，平均延迟可以控制在1秒以内
* 日志量日均5T，平均延迟可控制在10秒以内
* 以上延迟只是消息中间件测试，不包含日志落地存储，所以本基准只试用于评估虚拟机3节点Kafka集群自身性能。
* 吞吐量受带宽影响较大，虚拟机网卡吞吐量取决于服务器CPU，更高基准测试建议还是采用物理服务器，或增加CPU核数
* Topic分区数越多单线程写入效率越低
