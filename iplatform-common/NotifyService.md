# 通知服务部署手册

> 作者 王立松

通知服务负责接收ActiveMQ消息发送短信和邮件

## 1. 准备

- JDK1.8+
- 已经启动了注册服务
- 已经启动了文档服务

## 2. 介质

| 文件名                        | 说明         |
| :---------------------------- | :----------- |
| notify-service-0.0.1.jar      | 主程序文件   |
| notify-plugin-email-0.0.1.jar | 邮件发送文件 |
| notify-plugin-sms-0.0.1.jar   | 短信发送文件 |
| run.sh                        | 启停脚本     |

## 3. 部署

1. 创建部署目录

   ```
   mkdir -p notify-server/plugins
   ```

2. 复制目标文件到部署目录

   ```
   cp notify-service-0.0.1.jar notify-server
   cp run.sh notify-server
   cp notify-plugin-email-0.0.1.jar notify-server/plugins
   cp notify-plugin-sms-0.0.1.jar notify-server/plugins
   ```

## 4. 启停

启动服务

```shell
sh run.sh start
```

停止服务

```shell
sh run.sh stop
```

重启服务

```shell
sh run.sh restart
```

## 5. 参数

> 所有参数都定义在启停脚本run.sh中

| 参数名                                                   | 必填 | 说明                                                 |
| :------------------------------------------------------- | :--- | :--------------------------------------------------- |
| discovery.server.address                                 | 是   | 注册服务的地址，当集群模式时配置多个地址逗号分隔     |
| server.host                                              | 是   | 服务绑定IP                                           |
| server.port                                              | 是   | 服务绑定端口                                         |
| spring.activemq.broker-url                               | 是   | 短信邮件消息的ActiveMQ地址                           |
| spring.datasource.platform                               | 是   | 数据库平台类型，例如oracle                           |
| spring.datasource.dataSourceClassName                    | 是   | 数据库JDBC驱动，例如 oracle.jdbc.driver.OracleDriver |
| spring.datasource.url                                    | 是   | 数据库url，例如jdbc:oracle:thin:@127.0.0.1:1521:orcl |
| spring.datasource.username                               | 是   | 数据库用户名                                         |
| spring.datasource.password                               | 是   | 数据库密码                                           |
| spring.dfss.bucketname                                   | 否   | 文档服务桶名称（仅用于邮件附件）                     |
| spring.dfss.secretkey                                    | 否   | 文档服务密码（仅用于邮件附件）                       |
| notify.email.host                                        | 否   | 发件服务器                                           |
| notify.email.port                                        | 否   | 发件服务器端口                                       |
| notify.email.from                                        | 否   | 发件人账号                                           |
| notify.email.password                                    | 否   | 发件人密码                                           |
| flume.agent.sinks.smsk.CMPPConnect.host                  | 是   | 短信网关地址                                         |
| flume.agent.sinks.smsk.CMPPConnect.port                  | 是   | 短信网关端口                                         |
| flume.agent.sinks.smsk.CMPPConnect.source-addr           | 是   | 短信网关登录ID                                       |
| flume.agent.sinks.smsk.CMPPConnect.shared-secret         | 是   | 短信网关登录密码                                     |
| flume.agent.sinks.smsk.CMPPSubmitMessage.msg_Src         | 是   | 短信网关登录ID                                       |
| flume.agent.sinks.smsk.CMPPSubmitMessage.src_Terminal_Id | 是   | 短信网关服务代码或前缀                               |
