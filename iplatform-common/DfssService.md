# 文档服务部署手册

> 作者 张磊

## 1. 准备

* JDK1.8+
* 已经部署MongoDB
* 已经部署注册服务
* 已经部署认证服务

## 2. 介质

| 文件名                 | 说明       |
| ---------------------- | ---------- |
| dfss-service-0.0.2.jar | 主程序文件 |
| run.sh                 | 启停脚本   |

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

| 参数名                        | 必填 | 默认值 | 说明                                                         |
| ----------------------------- | ---- | ------ | ------------------------------------------------------------ |
| discovery.server.address      | 是   |        | 定义注册服务的地址，当集群模式时配置多个地址逗号分隔  discovery.server.address=https://192.168.0.1:8761/eureka/,https://192.168.0.2:8761/eureka/ |
| server.host                   | 是   |        | 服务绑定IP                                                   |
| server.port                   |      | 8762   | 服务绑定端口                                                 |
| spring.cloud.config.enable    | 是   | true   | 开启集中配置功能                                             |
| spring.cloud.config.profile   | 是   |        | 集中配置环境名，例如：生产环境                               |
| dfss.mongodb.uri              | 是   |        | mongodb库配置，例如： mongodb://dfss-user:dfss-password@oneitom-mongo/dfss |
| dfss.mongodb.grid-fs-database | 是   | dfss   | mongodb grid fs 名称                                         |

## 5. Docker

```yaml
version: '3.2'
services:   
  oneitom-dfss:
    image: boco/oneitom-dfss:0.0.2
    hostname: oneitom-dfss
    container_name: oneitom-dfss
    restart: always
    networks:
      - oneitom-network
    ports:
      - '50000:50000'
    volumes:
      - ${ONEITOM_VOLUME_PATH}/oneitom-dfss/logs:/dfss-service/logs         
    environment:
      - 'JAVA_OPTIONS=-Xmx512m -Xms512m'
      - 'discovery.server.address=https://oneitom-discovery:8761/eureka/'
      - 'server.host=oneitom-dfss'
      - 'dfss.mongodb.uri=mongodb://dfss-user:dfss-password@oneitom-mongo/dfss'
      - 'dfss.mongodb.grid-fs-database=dfss'
      - 'eureka.instance.metadataMap.iplatformtype=平台服务'
      - 'spring.cloud.config.enabled=true'
      - 'endpoints.shutdown.allowip=172.16.0.1'
      - 'spring.cloud.config.profile=测试'
    labels:
     - oneitom-dfss-cluster           

networks:
  oneitom-network:
    external: true
```
