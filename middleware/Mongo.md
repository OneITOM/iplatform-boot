# Mongo部署手册

> 作者 张磊

## 1. 安装

### 1.1 前提

* 关闭SElinux或者防火墙允许27017端口访问

### 2.2 安装

2. 创建安装目录

   ```bash
   # 创建部署根目录
   mkdir -p /home/oneitom/mongodb
   # 创建数据存放目录
   mkdir -p /home/oneitom/mongodb/data
   # 创建日志目录
   mkdir -p /home/oneitom/mongodb/log
   ```

3. 解压缩

   ```bash
   cd /home/oneitom/mongodb
   curl -O https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-3.4.17.tgz
   tar -zxvf mongodb-linux-x86_64-3.4.17.tgz
   mv mongodb-linux-x86_64-3.4.17 mongodb
   rm -rf mongodb-linux-x86_64-3.4.17.tgz
   ```

4. 环境变量

   在你的环境配置文件中增加一下一行（例如：~/.bash_profile）

   ```xml
   export PATH=/home/oneitom/mongodb/bin:$PATH
   ```

5. 修改系统内核

   ```bash
   ulimit -SHn 655350
   ```

6. 配置MongoDB参数

   编辑 /home/oneitom/mongodb/mongodb.conf 文件（没有则创建一个）

   ```yaml
   systemLog:
   	destination: mongodb.log #日志文件名
   	path: /home/oneitom/mongodb/log/ #日志输出文件路径
   	logAppend: true #日志输出方式
   	traceAllExceptions: false #不开启调试模式
   storage:
   	dbPath: /home/oneitom/mongodb/data #数据文件路径
   	journal:
   		enabled: true
   processManagement:
   	fork: true #设置后台运行
   net:
   	bindIp: 10.22.1.100 #绑定访问IP
   	port: 27017 #绑定访问端口
   	maxIncomingConnections: 65536 #最大并发连接数
   	ipv6: false #是否开启ipv6支持
   setParameter:
   	enableLocalhostAuthBypass: true
   ```

7. 设置开机自启动

   vi /etc/rc.d/init.d/mongod

   ```bash
   #!/bin/sh
   #chkconfig: - 64 36
   #description:mongod
   case $1 in
   start)
   /home/oneitom/mongodb/bin/mongod --config /home/oneitom/mongodb/mongodb.conf
   ;;
   stop)
   /home/oneitom/mongodb/bin/mongo 10.22.1.100:27017/admin --eval "db.shutdownServer()"
   ;;
   status)
   /home/oneitom/mongodb/bin/mongo 10.22.1.100:27017/admin --eval "db.stats()"
   ;;
   esac
   ```

   添加脚本执行权限

   ```shell
   chmod +x /etc/rc.d/init.d/mongod
   ```

   设置开机自启动

   ```shell
   chkconfig mongod on
   ```

8. 启动

   ```shell
   service mongod start
   ```

9. 验证

   登录mongodb控制台

   ```shell
   mongo --host 10.22.1.100:27017
   ```

   查看默认数据库

   ```shell
   show dbs
   ```

   切换到admin数据库

   ```
   use admin
   ```

   退出控制台

   ```shell
   exit
   ```

10. 停止

    ```shell
    service mongod stop
    ```

11.  重启

    ```shell
    service mongod restart
    ```

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


