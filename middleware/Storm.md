# Storm部署手册

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
  oneitom-storm-zookeeper:
    image: boco/alpine-zookeeper:3.4.9
    hostname: oneitom-storm-zookeeper
    container_name: oneitom-storm-zookeeper
    restart: always
    networks:
      - oneitom-network    

  oneitom-storm-nimbus:
    image: boco/alpine-storm:1.1.3
    hostname: oneitom-storm-nimbus
    container_name: oneitom-storm-nimbus
    command: storm nimbus
    depends_on:
      - oneitom-storm-zookeeper    
    restart: always
    networks:
      - oneitom-network    
    volumes:
      - ${ONEITOM_VOLUME_PATH}/oneitom-storm/conf:/conf
      - ${ONEITOM_VOLUME_PATH}/oneitom-storm/logs:/logs 
      - ${ONEITOM_VOLUME_PATH}/oneitom-storm/data:/data 
    ports:
      - 6627:6627

  oneitom-storm-supervisor:
    image: boco/alpine-storm:1.1.3
    hostname: oneitom-storm-supervisor
    container_name: oneitom-storm-supervisor
    command:
      - /bin/sh
      - -c
      - |
        darkhttpd /logs/workers-artifacts/ --port 8080 --daemon
        storm supervisor
    depends_on:
      - oneitom-storm-nimbus
      - oneitom-storm-zookeeper
    volumes:
      - ${ONEITOM_VOLUME_PATH}/oneitom-storm/conf:/conf      
      - ${ONEITOM_VOLUME_PATH}/oneitom-storm/logs:/logs 
      - ${ONEITOM_VOLUME_PATH}/oneitom-storm/data:/data 
    restart: always 
    ports:
      - 54848:8080      
    networks:
      - oneitom-network    

  oneitom-storm-ui:
    image: boco/alpine-storm:1.1.3
    hostname: oneitom-storm-ui
    container_name: oneitom-storm-ui
    command: storm ui
    depends_on:
      - oneitom-storm-nimbus
      - oneitom-storm-zookeeper
    volumes:
      - ${ONEITOM_VOLUME_PATH}/oneitom-storm/conf:/conf      
      - ${ONEITOM_VOLUME_PATH}/oneitom-storm/logs:/logs 
    restart: always   
    networks:
      - oneitom-network    
    ports:
      - 8765:8765   

  oneitom-storm-logviewer :
    image: boco/alpine-storm:1.1.3
    hostname: oneitom-storm-logviewer
    container_name: oneitom-storm-logviewer 
    command: storm logviewer 
    depends_on:
      - oneitom-storm-nimbus
      - oneitom-storm-zookeeper
    volumes:
      - ${ONEITOM_VOLUME_PATH}/oneitom-storm/conf:/conf      
      - ${ONEITOM_VOLUME_PATH}/oneitom-storm/logs:/logs 
    restart: always  
    networks:
      - oneitom-network     
    ports:
      - 8766:8766         

networks:
  oneitom-network:
    external: true
```

