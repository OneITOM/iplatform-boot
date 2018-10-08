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
mkdir -p /opt/boco
cd /opt/boco
tar zxvf activemq-5.15.2-bin.tar.gz
```
3. 参数配置

```xml
<xml>
</xml>
```

4. 集群配置

```bash
cd /opt/boco/bin
./activemq start
```

5. 验证安装
   - 打开管理控制台
   - - URL: <http://127.0.0.1:8161/admin/>
     - Login: admin
     - Passwort: admin
   - Navigate to "Queues"
   - Add a queue name and click create
   - Send test message by klicking on "Send to"

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



