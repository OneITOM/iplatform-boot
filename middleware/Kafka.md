# Kafka部署手册

> 作者 xx

## 1. 安装

### 1.1 前提

* JDK1.8+
* 磁盘剩余空间大于xxxMB
* 空闲内存大于xxxGB

### 2.2 安装

1. 下载安装包

```bash

```

2. 解压缩

```bash

```
3. 参数配置

```xml

```

4. 启动

```bash

```

5. 验证安装
6. 停止

## 2. Docker

> 基于Docker的单节点配置

```yaml
version: '3.2'
services:
  oneitom-zookeeper:
    image: boco/alpine-zookeeper:3.4.9
    hostname: oneitom-zookeeper
    container_name: oneitom-zookeeper
    restart: always  
    networks:
      - oneitom-network    
    ports:
      - 2181:2181

  oneitom-kafka:
    image: boco/alpine-kafka:1.0.1
    hostname: oneitom-kafka
    container_name: oneitom-kafka
    networks:
      - oneitom-network    
    environment:
      KAFKA_ADVERTISED_HOST_NAME: oneitom-kafka
      KAFKA_ZOOKEEPER_CONNECT: oneitom-zookeeper:2181
      KAFKA_CREATE_TOPICS: 'topic_test:1:1,topic_dataflow_plugin_pretreat:1:1'
    ports:
      - 9092:9092

networks:
  oneitom-network:
    external: true
```


