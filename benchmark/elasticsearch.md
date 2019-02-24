# ElasticSearch基准测试

> 作者 张磊

## 1. 数据估算

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

## 2. 配置

### 2.1 物理服务器配置

| 名称     | 描述                                                        |
| -------- | ----------------------------------------------------------- |
| 服务器   | Dell PowerEdge R720xd                                       |
| CPU      | Intel(R) Xeon(R) CPU E5-2620 v2 @ 2.10GHz                   |
| 核数     | 24                                                          |
| 内存     | 128G                                                        |
| 硬盘     | 机械                                                        |
| 网卡     | Broadcom Corporation NetXtreme BCM5720 Gigabit Ethernet * 4 |
| 管理程序 | VMware ESXi, 6.0.0, 3073146                                 |

### 2.2 虚拟机

| 服务器 | CPU  | 内存 | 磁盘 | 网络     |      |
| ------ | ---- | ---- | ---- | -------- | ---- |
| VM     | 4    | 16   | 200G | VMXNET 3 |      |
| VM     | 4    | 16   | 200G | VMXNET 3 |      |
| VM     | 4    | 16   | 200G | VMXNET 3 |      |

### 2.3 网络带宽

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

### 2.4 操作系统配置

* ulimit -n 65535

### 2.5 ElasticSearch 服务配置

> [elasticsearch 2.4](https://www.elastic.co/guide/en/elasticsearch/reference/2.4/index.html)

修改 bin/elasticsearch.in.sh 的 ES_MIN_MEM ES_MAX_MES 为 8g

```shell
if [ "x$ES_MIN_MEM" = "x" ]; then
    ES_MIN_MEM=8g
fi
if [ "x$ES_MAX_MEM" = "x" ]; then
    ES_MAX_MEM=8g
fi
```

conf/elasticsearch.yml

| 参数名                      | 参数值     |      |
| --------------------------- | ---------- | ---- |
| bootstrap.memory_lock       | true        | 锁定物理内存地址 |
| index.number_of_shards | 3    | 分区数 |
| index.number_of_replicas | 1 | 默认副本数0 |
| threadpool.index.queue_size | 1000 |  |
| indices.memory.index_buffer_size | 30% |  |
| http.max_content_length | 200mb |  |

### 2.6 日志索引模版参数配置

| 参数名                      | 参数值     |      |
| --------------------------- | ---------- | ---- |
| index.refresh_interval        | 30s          | 搜索实时性30s |
| index.number_of_replicas       | 1         | 副本数 |
| index.number_of_shards      | 3        | 分区数 |
| index.translog.durability     | async          | 异步写入 |
| index.translog.flush_threshold_ops       | 1000000          |      |
| index.translog.sync_interval      | 120s       | 持久化同步间隔 |
| index.translog.flush_threshold_size      | 1024mb  | 持久化大小阀值 |
| index.translog.flush_threshold_period      | 120m       | 持久化时间周期 |
| index.merge.policy.floor_segment      | 100mb          | 小于100m文件归并 |
| index.merge.scheduler.max_thread_count    | 1          | 归并线程数 |
| index.merge.policy.min_merge_size      | 10mb          |      |

## 写入压测

> 写入100W记录，每条记录676字节，总大小644.7MB

| 并发数 | 副本数 | 分片数 | Flush Size | 每秒写入大小 | 每秒写入条数 | 耗时 |
| ----| ---- | ---- | ---------- | ------------- | ------------ | ------- |
| 3 | 0 | 3 | 5MB   | 21MB    | 3.2W | 30.62s |
| 3 | 1 | 3 | 5MB     | 19.2MB    | 2.9W  | 33.62s |
| 3 | 1 | 3 | 10MB      | 22.5MB    | 3.4W    | 28.71s |
| 3 | 1 | 3 | 20MB     | 21.1MB    | 3.2W   | 30.6s |
| 3 | 1 | 3 | 50MB      | 22.9MB  | 3.5W    | 28.1s |
| 6 | 1 | 3 | 20MB | 23.8MB | 3.6W | 27s |
| 6 | 1 | 6 | 20MB | 14.5MB | 2.2W | 44.4s |

### 数据样例

```json
{
  "traceId": "0001",
  "serviceInstId": "benchmark::admin-server:8762",
  "level": "INFO",
  "ip": "10.22.1.234",
  "logger": "org.springframework.context.support.PostProcessorRegistrationDelegate$BeanPostProcessorChecker",
  "tracespanid": "0001",
  "thread": "main",
  "busisys": "trident",
  "message": "Bean 'org.springframework.security.access.expression.method.DefaultMethodSecurityExpressionHandler@75a1ab87' of type [class org.springframework.security.oauth2.provider.expression.OAuth2MethodSecurityExpressionHandler] is not eligible for getting processed by all BeanPostProcessors (for example)",
  "serviceId": "admin-server",
  "timestamp": "1547041636544"
}
```

### 索引模版

```json
{
  "template": "*",
  "order": 5,
  "settings": {
      "index.number_of_shards": 3,
      "index.number_of_replicas": 0,
      "index.refresh_interval": -1,
      "translog.durability": "async",
      "translog.flush_threshold_ops": 1000000,
      "translog.sync_interval": "600s",
      "translog.flush_threshold_size": "1024mb",
      "translog.flush_threshold_period": "120m",
      "merge.policy.floor_segment": "100mb",
      "merge.scheduler.max_thread_count": 1,
      "merge.policy.min_merge_size": "10mb"
  },
  "mappings": {
    "_default_": {
      "dynamic_templates": [{
        "message_field": {
          "mapping": {
            "fielddata": {
              "format": "disabled"
            },
            "index": "analyzed",
            "type": "string"
          },
          "match_mapping_type": "string",
          "match": "message"
        }
      }],
      "_all": {
        "enabled": false
      },
      "properties": {
        "timestamp": {
          "type": "date"
        },
        "serviceId": {
          "index": "not_analyzed",
          "type": "string"
        },
        "serviceInstId": {
          "index": "not_analyzed",
          "type": "string"
        },
        "traceId": {
          "index": "not_analyzed",
          "type": "string"
        },
        "tracespanid": {
          "index": "not_analyzed",
          "type": "string"
        },
        "busisys": {
          "index": "not_analyzed",
          "type": "string"
        },
        "level": {
          "index": "not_analyzed",
          "type": "string"
        },
        "ip": {
          "index": "not_analyzed",
          "type": "string"
        },
        "logger": {
          "index": "not_analyzed",
          "type": "string"
        },
        "busisys": {
          "index": "not_analyzed",
          "type": "string"
        },
        "thread": {
          "index": "not_analyzed",
          "type": "string"
        }
      }
    }
  }
}
```



