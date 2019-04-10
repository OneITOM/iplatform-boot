# 分布式事务中间件部署手册

> 分布式事务中间件基于 [Apache ServiceComb Pack 0.4.0版本](https://github.com/apache/servicecomb-pack) 构建

## 1. 准备

- JDK1.8+
- 已经安装了MySQL
- 已经启动了注册服务

## 2. 介质

| 文件名                              | 说明       |
| ----------------------------------- | ---------- |
| servicecomb-alpha-1.0.0.040.110.jar | 主程序文件 |
| run.sh                              | 启停脚本   |

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

## 4. 参数

> 所有的参数都定义在启动脚本 run.sh 中 

| 参数名                                | 必填 | 默认值                | 说明                                                         |
| ------------------------------------- | ---- | --------------------- | ------------------------------------------------------------ |
| discovery.server.address              | 是   |                       | 定义注册服务的地址，当集群模式时配置多个地址逗号分隔  discovery.server.address=https://192.168.0.1:8761/eureka/,https://192.168.0.2:8761/eureka/ |
| server.host                           | 是   |                       | 服务绑定IP                                                   |
| server.port                           |      | 55000                 | 服务绑定端口                                                 |
| alpha.server.port                     |      | 55001                 | 事务通讯端口                                                 |
| spring.datasource.platform            |      | mysql                 | 数据库平台类型配置，例如：mysql                              |
| spring.datasource.dataSourceClassName |      | com.mysql.jdbc.Driver | 数据库JDBC驱动，例如：com.mysql.jdbc.Driver                  |
| spring.datasource.url                 | 是   |                       | 数据库URL，例如：jdbc:mysql://127.0.0.1:3306/saga?serverTimezone=GMT%2b8&useSSL=false。**注意:serverTimezone=GMT%2b8参数必选** |
| spring.datasource.username            | 是   |                       | 数据库用户名                                                 |
| spring.datasource.password            | 是   |                       | 数据库密码                                                   |

## 5. 验证服务

访问以下地址，返回状态信息说明服务启动成功

```shell
curl http://127.0.0.1:55000/servicecomb-alpha/saga/stats
{"totalTransactions":0,"pendingTransactions":0,"committedTransactions":0,"compensatingTransactions":0,"rollbackTransactions":0,"updatedAt":"2019-03-19 11:14:53","failureRate":0}
```

## 6. Docker

```yaml
version: '3.2'
services:
  oneitom-servicecomb-alpha:
    image: boco/oneitom-servicecomb-alpha:1.0.0.040.110
    hostname: oneitom-servicecomb-alpha
    container_name: oneitom-servicecomb-alpha
    restart: always
    networks:
      - oneitom-network
    ports:
      - 55000:55000
      - 55001:55001
    volumes:
      - /Users/zhanglei/Desktop/mydocker/docker_volume/oneitom-servicecomb-alpha/logs:/servicecomb/logs
    environment:
      - 'JAVA_OPTIONS=-Xmx512m -Xms512m'
      - 'discovery.server.address=http://oneitom-discovery:8761/eureka/'
      - 'server.host=127.0.0.1'
      - 'alpha.server.port=55001'
      - 'spring.datasource.platform=mysql'
      - 'spring.datasource.dataSourceClassName=com.mysql.jdbc.Driver'
      - 'spring.datasource.url=jdbc:mysql://oneitom-mysql:3306/saga?useUnicode=true&characterEncoding=utf-8&autoReconnect=true&serverTimezone=GMT%2b8'
      - 'spring.datasource.username=saga-user'
      - 'spring.datasource.password=saga-password'
    labels:
     - oneitom-servicecomb-alpha-cluster                

networks:
  oneitom-network:
    external: true
```

