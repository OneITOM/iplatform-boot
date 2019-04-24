# 注册服务部署手册

> 作者 张磊

## 1. 准备

* JDK1.8+

## 2. 介质

| 文件名                     | 说明       |
| -------------------------- | ---------- |
| discovery-server-0.0.2.jar | 主程序文件 |
| run.sh                     | 启停脚本   |

## 3. 启停

启动服务

```bash
sh run.sh start
```

停止服务

```bash
sh run.sh stop
```

 重启服务

```bash
sh run.sh restart
```

##  4. 参数

> 所有的参数都定义在启动脚本 run.sh 中 

| 参数名                      | 必填 | 默认值 | 说明                                                         |
| --------------------------- | ---- | ------ | ------------------------------------------------------------ |
| discovery.server.address    | 是   |        | 定义注册服务的地址，当集群模式时配置多个地址逗号分隔  discovery.server.address=http://192.168.0.1:8761/eureka/,http://192.168.0.2:8761/eureka/ |
| server.host                 | 是   |        | 服务绑定IP                                                   |
| server.port                 |      | 8761   | 服务绑定端口                                                 |
| spring.data.mongodb.uri     | 是   |        | 集中配置存储mongodb地址，mongodb://localhost/config-db        |
| spring.activemq.broker-url  | 是   |        | 集中配置推送ActiveMQ地址，tcp://localhost:61616              |

## 5. 界面

```
https://127.0.0.1:8761/
```

![discovery.png](images/DiscoveryService/discovery.png)

## 6. Docker

```yaml
version: '3.2'
services:
  oneitom-discovery:
    image: boco/oneitom-discovery:0.0.2
    hostname: oneitom-discovery
    container_name: oneitom-discovery
    restart: always
    networks:
      - oneitom-network
    ports:
      - '8761:8761'
    volumes:
      - ${ONEITOM_VOLUME_PATH}/oneitom-discovery/logs:/discovery-server/logs      
    environment:
      - 'JAVA_OPTIONS=-Xmx512m -Xms512m'
      - 'discovery.server.address=https://oneitom-discovery:8761/eureka/'
      - 'server.host=oneitom-discovery'
      - 'eureka.client.register-with-eureka=true'
      - 'eureka.client.fetch-registry=true'
      - 'spring.activemq.broker-url=tcp://oneitom-activemq:61616'
      - 'eureka.instance.metadataMap.iplatformtype=平台服务'
      - 'spring.cloud.config.enabled=true'
      - 'spring.data.mongodb.uri=mongodb://configdb-user:configdb-password@oneitom-mongo/configdb'
    labels:
     - oneitom-discovery-cluster             

networks:
  oneitom-network:
    external: true
```

