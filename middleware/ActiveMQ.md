# ActiveMQ部署手册

> 作者 张磊

## 1. 安装

### 1.1 前提

* JDK1.8+
* 磁盘剩余空间大于500M

### 2.2 安装

1. 下载安装包

```bash
wget http://activemq.apache.org/path/tofile/apache-activemq-5.15.2-bin.tar.gz
```

2. 解压缩

```bash
mkdir -p /opt/BOCO
cd /opt/BOCO
tar zxvf activemq-5.15.2-bin.tar.gz
```
3. 参数配置

   web访问端口，默认即可，如需修改，编辑jetty.xml

```xml
<bean id="jettyPort" class="org.apache.activemq.web.WebConsolePort" init-method="start">
             <!-- the default port number for the web console -->
        <property name="host" value="0.0.0.0"/>
        <property name="port" value="8161"/>
    </bean>
```

​        连接端口配置，默认即可，如需修改，编辑activemq.xml

```properties
<transportConnectors>
            <!-- DOS protection, limit concurrent connections to 1000 and frame size to 100MB -->
            <transportConnector name="openwire" uri="tcp://0.0.0.0:61616?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
            <transportConnector name="amqp" uri="amqp://0.0.0.0:5672?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
            <transportConnector name="stomp" uri="stomp://0.0.0.0:61613?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
            <transportConnector name="mqtt" uri="mqtt://0.0.0.0:1883?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
            <transportConnector name="ws" uri="ws://0.0.0.0:61614?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
        </transportConnectors>
```

​      jmx监控开启、端口

```
 <managementContext>
          <managementContext createConnector="true" connectorPort="1093" connectorPath="/jmxrmi" jmxDomainName="org.apache.activemq"/>
        </managementContext>
```

4. 启动

```properties
cd /opt/BOCO/activemq-5.15.2-bin/bin
nohup ./activemq start &
```

5. 验证安装
   - 打开管理控制台
   - - URL: <http://127.0.0.1:8161/admin/>
     - Login: admin
     - Passwort: admin

## 2. Docker

> 基于Docker的单节点配置

```yaml
version: '3.2'
services:
  oneitom-activemq:
    image: boco/alpine-activemq:5.15.2
    container_name: oneitom-activemq
    restart: always
    networks:
      - oneitom-network
    ports:
      - '8161:8161' # Web Console
      - '61616:61616' # JMS
    environment:
      - 'ACTIVEMQ_ADMIN_PASSWORD=admin'
      - 'ACTIVEMQ_OPTS_MEMORY=-Xms256m -Xmx256m'
    labels:
     - oneitom-activemq-cluster           

networks:
  oneitom-network:
    external: true
```



