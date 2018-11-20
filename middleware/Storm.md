# Storm部署手册

> 作者 xx

## 1. 安装

### 1.1 前提

* JDK1.8+

* 磁盘剩余空间大于10G

* 空闲内存大于4GB

* 添加主机名、IP对应关系

  修改/etc/hosts，在最后添加

  ```
  192.22.1.180 aaa-NLKF-04
  192.22.1.182 aaa-NLKF-02
  192.22.1.181 aaa-NLKF-03
  ```

- 设置环境变量

```

STORM_HOME=/opt/BOCO/apache-storm-1.1.0
export STORM_HOME
PATH=$STORM_HOME/bin:$PATH
export PATH
```



### 2.2 安装

1. 下载安装包

```bash
wget http://storm.apache.org/downloads.html/apache-storm-1.1.0.tar.gz
```

2. 解压缩

```bash
cd /opt/BOCO
tar -xzvf apache-storm-1.1.0.tar.gz
```
3. 参数配置

   修改conf/storm.yaml,在最后追加

```xml
#zookeeper地址
storm.zookeeper.servers: [ "10.22.1.146","10.22.1.147","10.22.1.148" ]
#集群的主机名（必须是主机名）
nimbus.seeds:  [ "bomc147","guansu-busi","shanxi-oracle" ]
supervisor.slots.ports:
 - 6700
 - 6701
 - 6702
 - 6703
storm.local.dir: "/apache-storm-1.1.0/workdir" 
ui.port: 8765
```

4. 启动

   主机：ui只在一台机器启动即可

```bash
cd /opt/BOCO/
nohup ./storm logviewer & 
nohup ./storm ui &
nohup ./storm supervisor &
nohup ./storm nimbus &
```

5. 停止

   ```
   ps -ef | grep org.apache.storm.daemon.logviewer
   ps -ef | grep org.apache.storm.ui.core
   ps -ef | grep org.apache.storm.daemon.supervisor.Supervisor
   ps -ef | grep org.apache.storm.daemon.nimbus
   kill -9
   ```


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

