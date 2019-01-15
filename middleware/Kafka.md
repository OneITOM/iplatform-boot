# Kafka部署手册

> 作者 xx

## 1. 安装

### 1.1 前提

- JDK1.8+

- 磁盘剩余空间大于20GB

- 空闲内存大于4GB

- 添加主机名、IP对应关系

  修改/etc/hosts，在最后添加

  ```properties
  10.100.1.180 aaa-NLKF-04
  10.100.1.182 aaa-NLKF-02
  10.100.1.181 aaa-NLKF-03
  ```

- 创建kafka用户

```properties
groupadd  kafka
useradd kafka -g kafka
```

- 设置环境变量

```properties
KAFKA_HOME=/opt/BOCO/kafka_2.11-1.1.0
export KAFKA_HOME
STORM_HOME=/opt/BOCO/apache-storm-1.1.0
export STORM_HOME
ZK_HOME=/opt/BOCO/zookeeper-3.4.6
export ZK_HOME
PATH=$KAFKA_HOME/bin:$ZK_HOME/bin:$PATH
export PATH

```

说明：路径需根据实际部署路径修改

### 2.2 安装

####   2.2.1 zookeeper安装

1. 下载安装包

```bash
wget http://archive.apache.org/dist/zookeeper/zookeeper-3.4.6/zookeeper-3.4.6.tar.gz
```

2. 解压缩

```bash
mkdir -p /opt/BOCO
cd /opt/BOCO
tar -zxvf zookeeper-3.4.6.tar.gz
```
3. 参数配置

复制配置文件

```xml
cp zoo_sample.cfg  zoo.cfg
```

​      修改zoo.cfg参数

```
dataDir=/opt/BOCO/zookeeper-3.4.6/data
clientPort=2181
server.1=10.22.1.180:2888:3888
server.2=10.22.1.181:2888:3888
server.3=10.22.1.182:2888:3888
```

   注意：如配置文件中包含参数，需修改，没有包含的在最后添加

每台机器上生成myid：

10.22.1.180：

```
echo "1" > /opt/BOCO/zookeeper-3.4.6/data/myid
```

10.22.1.181：

```
echo "2" > /opt/BOCO/zookeeper-3.4.6/data/myid
```

10.22.1.182：

```
echo "3" > /opt/BOCO/zookeeper-3.4.6/data/myid
```

5. 启动

   ```
   ./ zkServer.sh start
   ```

6. 验证安装

```bash
./zkServer.sh status
```

可以看到有一个是leader，其他的是follower

6. 停止

```
./zkServer.sh stop
```

#### 2.2.2  kafka

1. 下载安装包

```bash
wget http://archive.apache.org/dist/kafka/1.1.0/kafka_2.11-1.1.0.tgz
```

​    2. 解压缩

```bash
mkdir -p /opt/BOCO
cd /opt/BOCO
tar -zxvf kafka_2.11-1.1.0.tgz
```

    3. 参数配置
    
       vi ./conf/server.properties

```xml
broker.id=180 
host.name=10.22.1.180
log.dirs=/opt/BOCO/kafka_2.11-1.1.0/logs
zookeeper.connect=10.22.1.180:2181,10.22.1.181:2181,10.22.1.182:2181
message.max.bytes=10485760
#此处两项配置增加kafka接收消息大小
replica.fetch.max.bytes=10485760
advertised.host.name=10.22.1.180
#默认分区数
num.partitions=20
#数据保留时长，单位小时
log.retention.hours=72
#Broker处理消息的最大线程数,建议为cpu核数+1
num.network.threads=5
#Broker处理磁盘IO的线程数,建议为cpu核数*2
num.io.threads=8
#每写入10000条数据时刷数据到磁盘
log.flush.interval.messages=10000
#每隔1秒刷数据到磁盘
log.flush.interval.ms=1000
```

  说明：broker.id，取主机ip的最后几位，要求唯一

  host.name，advertised.host.name设置本机的ip

其他两台机器类似

​    修改consumer.properties

 

```
fetch.message.max.bytes=10485760
```

  4. 启动

```bash
nohup ./kafka-server-start.sh ../config/server.properties &
```

    6. 停止
    
       ```
       ps -ef|grep kafka
       kill -9
       ```


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


