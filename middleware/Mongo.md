# Mongo部署手册

> 作者 张磊

## 1. 安装

### 1.1 前提

* 关闭SElinux或者防火墙允许27017端口访问

### 1.2 安装

1. 下载并解压缩

   ```shell
   cd /home/oneitom/
   curl -O https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-3.4.17.tgz
   tar -zxvf mongodb-linux-x86_64-3.4.17.tgz
   mv mongodb-linux-x86_64-3.4.17 mongodb
   rm -rf mongodb-linux-x86_64-3.4.17.tgz
   ```

2. 创建数据目录

   ```bash
   # 创建数据存放目录
   mkdir -p /home/oneitom/mongodb/data
   # 创建日志目录
   mkdir -p /home/oneitom/mongodb/log
   ```

3. 设置环境变量

   在你的环境配置文件中增加一下一行（例如：~/.bash_profile）

   ```xml
   export PATH=/home/oneitom/mongodb/bin:$PATH
   ```

4. 配置MongoDB参数

   vi /home/oneitom/mongodb/mongodb.conf 文件（没有则创建一个）

   **注意bindIp中的10.22.1.237要改成服务器的实际IP**

   ```yaml
   systemLog:
     destination: file #日志文件名
     path: /home/oneitom/mongodb/log/mongodb.log #日志输出文件路径
     logAppend: true #日志输出方式
     traceAllExceptions: false #不开启调试模式
   storage:
     dbPath: /home/oneitom/mongodb/data #数据文件路径
     journal:
       enabled: true
   processManagement:
     fork: true #设置后台运行
   net:
     bindIp: 127.0.0.1,10.22.1.237 #绑定访问IP
     port: 27017 #绑定访问端口
     maxIncomingConnections: 65536 #最大并发连接数
     ipv6: false #是否开启ipv6支持
   setParameter:
     enableLocalhostAuthBypass: true
   security:
     authorization: disabled #不需要权限，后续增加用户后会再修改
   ```

5. 设置开机启动脚本

   vi /etc/rc.d/init.d/mongod

   ```bash
   #!/bin/sh
   #chkconfig: - 64 36
   #description:mongod
   case $1 in
   start)
   echo never > /sys/kernel/mm/transparent_hugepage/enabled
   echo never > /sys/kernel/mm/transparent_hugepage/defrag
   ulimit -SHn 655350
   ulimit -SHu 327675
   /home/oneitom/mongodb/bin/mongod --config /home/oneitom/mongodb/mongodb.conf
   ;;
   stop)
   /home/oneitom/mongodb/bin/mongo 127.0.0.1:27017/admin --eval "db.shutdownServer()"
   ;;
   status)
   /home/oneitom/mongodb/bin/mongo 127.0.0.1:27017/admin --eval "db.stats()"
   ;;
   esac
   ```

6. 添加脚本执行权限

   ```shell
   chmod +x /etc/rc.d/init.d/mongod
   ```

7. 设置开机自启动

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
   mongo --host 127.0.0.1:27017
   ```

   查看默认数据库

   ```shell
   show dbs
   ```

10. 创建管理员账户（注意：生产环境不要使用本文档中的默认用户名和密码）

    ```shell
    > use admin
    switched to db admin
    > db.createUser({user: "useradmin", pwd: "adminpassword", roles: [{ role: "userAdminAnyDatabase", db: "admin" }]})
    Successfully added user: {
            "user" : "useradmin",
            "roles" : [
                    {
                            "role" : "userAdminAnyDatabase",
                            "db" : "admin"
                    }
            ]
    }
    ```

11. 验证用户是否添加成功，返回1表示成功

    ```shell
    > db.auth("useradmin", "adminpassword") 
    1
    ```

12. 退出控制台

    ```shell
    exit
    ```

13. 开启安全验证配置

    vi /home/oneitom/mongodb/mongodb.conf 文件，修改security.authorization为enabled

    ```yaml
    security:
      authorization: enabled
    ```

14. 重启mongodb

    ```shell
    service mongod restart
    ```

15. 验证权限是否配置成功

    未认证查询时提示没有权限 Unauthorized

    ```shell
    mongo --host 127.0.0.1:27017
    MongoDB shell version v3.4.17
    connecting to: mongodb://10.22.1.237:27017/
    MongoDB server version: 3.4.17
    > use admin
    switched to db admin
    > db.system.users.find()
    Error: error: {
        "ok" : 0,
        "errmsg" : "not authorized on admin to execute command { find: \"system.users\", filter: {} }",
        "code" : 13,
        "codeName" : "Unauthorized"
    }
    ```

16. 认证后在查询正常

    ```shell
    > use admin
    switched to db admin
    > db.auth("useradmin", "adminpassword") 
    1
    > db.system.users.find()
    { "_id" : "admin.useradmin", "user" : "useradmin", "db" : "admin", "credentials" : { "SCRAM-SHA-1" : { "iterationCount" : 10000, "salt" : "iFK8yoqn/Lv5UlLYz4xrwg==", "storedKey" : "ihnTsytWNBcCHweLz87pBTWYCZQ=", "serverKey" : "Fzaruna8hZumb3zfpThjONawOMs=" } }, "roles" : [ { "role" : "userAdminAnyDatabase", "db" : "admin" } ] }
    > 
    ```

### 1.3 创建一个新库

> 下边的演示了使用管理员账户登录后创建了一个cmdb库，并且配置cmdb的管理员账号为cmdb-user，密码为cmdb-password

```shell
mongo --host 10.22.1.237:27017
MongoDB shell version v3.4.17
connecting to: mongodb://10.22.1.237:27017/
MongoDB server version: 3.4.17
> use admin
switched to db admin
> db.auth("useradmin", "adminpassword")
1
> use cmdb
switched to db cmdb
> db.createUser({user: "cmdb-user", pwd: "cmdb-password", roles: [{ role: "dbOwner", db: "cmdb" }]})
Successfully added user: {
        "user" : "cmdb-user",
        "roles" : [
                {
                        "role" : "dbOwner",
                        "db" : "cmdb"
                }
        ]
}
```

创建完毕后就可以使用以下地址连接了

```shell
mongodb://cmdb-user:cmdb-password@10.22.1.237/cmdb
```



```shell
mongodb://cmdb-user:cmdb-password@10.22.1.237/cmdb
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