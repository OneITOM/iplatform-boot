# Redis部署手册

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
  oneitom-redis:
    image: boco/alpine-redis:4.0.6
    hostname: oneitom-redis
    container_name: oneitom-redis
    networks:
      - oneitom-network
    ports:
      - '6379:6379'
    volumes:
      - ${ONEITOM_VOLUME_PATH}/oneitom-redis:/app
    labels:
      - oneitom-redis-cluster   
networks:
  oneitom-network:
    external: true
```


