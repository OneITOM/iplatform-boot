# Mongo部署手册

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
  oneitom-mongo:
    image: boco/alpine-mongo:3.4.10
    hostname: oneitom-mongo
    container_name: oneitom-mongo
    restart: always
    networks:
      - oneitom-network
    ports:
      - '27017:27017'
    volumes:
      - ${ONEITOM_VOLUME_PATH}/oneitom-mongo:/data/db:rw
    environment:
      - 'MONGO_USERNAME=root'
      - 'MONGO_PASSWORD=root'
      - 'MONGO_DATABASE=dfss:dfss-user:dfss-password,cmdb:cmdb-user:cmdb-password,datashare:datashare-user:datashare-password,configdb:configdb-user:configdb-password,'
    labels:
     - oneitom-mongo-cluster       
networks:
  oneitom-network:
    external: true
```


