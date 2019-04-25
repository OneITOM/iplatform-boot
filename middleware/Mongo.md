# Mongo部署手册

> 作者 张磊

## 1. 安装

> 本文档指导如何安装一个MongoDB集群，集群模式采用Replica Set方式

### 1.1 前提

* 关闭SElinux或者防火墙允许27017端口访问
* 集群最小规模需要3台服务器

### 1.2 单实例安装

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
   # 创建数据存放目录和日志目录
   mkdir -p /home/oneitom/mongodb/data
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
     logRotate: rename 
     timeStampFormat: iso8601-local
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
     authorization: disabled
   #  keyFile: /home/oneitom/mongodb/key.file 
   #  clusterAuthMode: keyFile  
   #replication:
   #  oplogSizeMB: 2048
   #  replSetName: oneitomReplSet
   #  secondaryIndexPrefetch: all
   #  enableMajorityReadConcern: false  
   ```

5. 设置开机启动脚本

   vi /etc/rc.d/init.d/mongod

   ```bash
   #!/bin/sh
   #chkconfig: - 64 36
   #description:mongod
   start() {
       echo never > /sys/kernel/mm/transparent_hugepage/enabled
       echo never > /sys/kernel/mm/transparent_hugepage/defrag
       ulimit -SHn 655350
       ulimit -SHu 327675
       /home/oneitom/mongodb/bin/mongod --config /home/oneitom/mongodb/mongodb.conf    
   }
   
   stop() {
     /home/oneitom/mongodb/bin/mongo 127.0.0.1:27017/admin --eval "db.shutdownServer()"  
   }
   case $1 in
   start)
   	start
   ;;
   stop)
   	stop
   ;;
   restart)
       stop
       start
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

### 1.3 集群安装

> 集群采用 Replica Set 模式（至少3个节点）
>
> * n个不同类型节点组成 
> * 每个节点数据相同 
> * 建议至少有一个primary和两个secondary节点 
> * 集群中只能有一个primary节点 
> * 写请求都通过primary节点 
> * 支持自动故障恢复 
> * primary节点不可用时，通过投票选举从secondary节点列表中选出primary节点，因此最好节点数量是奇数 
> * secondary节点从primary节点通过oplog异步方式同步数据

1. 集群规划

| 服务器      | 端口  | 服务类型  |
| ----------- | ----- | --------- |
| 10.22.1.235 | 27017 | Primary   |
| 10.22.1.236 | 27017 | Secondary |
| 10.22.1.237 | 27017 | Secondary |

2. 参照 [1.2 单实例安装]()章节在三个服务器上分别安装MongoDB
3. 分别修改三个服务的 /home/oneitom/mongodb/mongodb.conf 配置文件，取消一下内容的注释

```yaml
replication:
  oplogSizeMB: 2048
  replSetName: oneitomReplSet
  secondaryIndexPrefetch: all 
  enableMajorityReadConcern: false
```

4. 重启3个服务器上的服务

```shell
service mongod restart
```

5. 登录任意一台mongo添加集群配置

> config = {_id:&quot;oneitomReplSet&quot;,members:[{_id:0,host:&quot;10.22.1.235:27017&quot;}, {_id:1,host:&quot;10.22.1.236:27017&quot;},{_id:2,host:&quot;10.22.1.237:27017&quot;}]};
>
> **注意：members中的第一个节点默认是Primary节点**

```shell
[root@swarm-docker-237 mongodb]# mongo --host 127.0.0.1:27017
MongoDB shell version v3.4.17
connecting to: mongodb://127.0.0.1:27017/
MongoDB server version: 3.4.17
> use admin
switched to db admin
> config = {_id:"oneitomReplSet",members:[{_id:0,host:"10.22.1.235:27017"}, {_id:1,host:"10.22.1.236:27017"},{_id:2,host:"10.22.1.237:27017"}]};
> rs.initiate(config);
{ "ok" : 1 }
oneitomReplSet:SECONDARY> 
```

6. 初始化副本集

> rs.initiate(config); 返回 { "ok" : 1 } 表示成功

```shell
> rs.initiate(config);
{ "ok" : 1 }
oneitomReplSet:SECONDARY> 
```

7. 查看集群状态

> rs.status();
>
> 可以看到三个节点信息，每个节点的stateStr描述的是节点类型，每个节点的optime值如果一样，说明数据一致

```shell
"members": [{
		"_id": 0,
		"name": "10.22.1.235:27017",
		"health": 1,
		"state": 1,
		"stateStr": "PRIMARY",
		"uptime": 245,
		"optime": {
			"ts": Timestamp(1539679755, 1),
			"t": NumberLong(1)
		},
		"optimeDate": ISODate("2018-10-16T08:49:15Z"),
		"syncingTo": "",
		"syncSourceHost": "",
		"syncSourceId": -1,
		"infoMessage": "",
		"electionTime": Timestamp(1539679573, 1),
		"electionDate": ISODate("2018-10-16T08:46:13Z"),
		"configVersion": 1,
		"self": true,
		"lastHeartbeatMessage": ""
	},
	{
		"_id": 1,
		"name": "10.22.1.236:27017",
		"health": 1,
		"state": 2,
		"stateStr": "SECONDARY",
		"uptime": 197,
		"optime": {
			"ts": Timestamp(1539679755, 1),
			"t": NumberLong(1)
		},
		"optimeDurable": {
			"ts": Timestamp(1539679755, 1),
			"t": NumberLong(1)
		},
		"optimeDate": ISODate("2018-10-16T08:49:15Z"),
		"optimeDurableDate": ISODate("2018-10-16T08:49:15Z"),
		"lastHeartbeat": ISODate("2018-10-16T08:49:19.954Z"),
		"lastHeartbeatRecv": ISODate("2018-10-16T08:49:20.716Z"),
		"pingMs": NumberLong(0),
		"lastHeartbeatMessage": "",
		"syncingTo": "10.22.1.237:27017",
		"syncSourceHost": "10.22.1.237:27017",
		"syncSourceId": 2,
		"infoMessage": "",
		"configVersion": 1
	},
	{
		"_id": 2,
		"name": "10.22.1.237:27017",
		"health": 1,
		"state": 2,
		"stateStr": "SECONDARY",
		"uptime": 197,
		"optime": {
			"ts": Timestamp(1539679755, 1),
			"t": NumberLong(1)
		},
		"optimeDurable": {
			"ts": Timestamp(1539679755, 1),
			"t": NumberLong(1)
		},
		"optimeDate": ISODate("2018-10-16T08:49:15Z"),
		"optimeDurableDate": ISODate("2018-10-16T08:49:15Z"),
		"lastHeartbeat": ISODate("2018-10-16T08:49:19.953Z"),
		"lastHeartbeatRecv": ISODate("2018-10-16T08:49:20.697Z"),
		"pingMs": NumberLong(0),
		"lastHeartbeatMessage": "",
		"syncingTo": "10.22.1.235:27017",
		"syncSourceHost": "10.22.1.235:27017",
		"syncSourceId": 0,
		"infoMessage": "",
		"configVersion": 1
	}
]
```

8. 验证集群

> 登录到Primary节点上写入一条数据

```shell
[root@swarm-docker-235 mongodb]# mongo --host 10.22.1.235:27017
MongoDB shell version v3.4.17
connecting to: mongodb://10.22.1.235:27017/
MongoDB server version: 3.4.17
oneitomReplSet:PRIMARY> use testdb
switched to db testdb
oneitomReplSet:PRIMARY> db.test.insert({"name":"hello"})
WriteResult({ "nInserted" : 1 })
oneitomReplSet:PRIMARY> 
```

> 登录到两个SECONDARY节点查询记录，同样可以看到在Primary节点上插入的数据，表示集群工作正常

```shell
[root@swarm-docker-237 mongodb]# mongo --host 10.22.1.237:27017
MongoDB shell version v3.4.17
connecting to: mongodb://10.22.1.237:27017/
MongoDB server version: 3.4.17
oneitomReplSet:SECONDARY> use testdb
switched to db testdb
oneitomReplSet:SECONDARY> db.getMongo().setSlaveOk();
oneitomReplSet:SECONDARY> db.test.find()
{ "_id" : ObjectId("5bc5a866cad432e1121afd91"), "name" : "hello" }
oneitomReplSet:SECONDARY> 
```

### 1.4 增加安全认证

> 以下操作全部在Primary节点进行

1. 创建管理员账户

   > 注意：以下脚本中的useradmin，adminpassword为管理员的用户名密码，请自行修改

   执行命令

   db.createUser({user: "useradmin", pwd: "adminpassword", roles: [{ role: "userAdminAnyDatabase", db: "admin" },{ role: "hostManager", db: "admin" }]})

   ```shell
   [root@swarm-docker-235 mongodb]# mongo --host 10.22.1.235:27017
   MongoDB shell version v3.4.17
   connecting to: mongodb://10.22.1.235:27017/
   MongoDB server version: 3.4.17
   oneitomReplSet:PRIMARY> use admin
   switched to db admin
   oneitomReplSet:PRIMARY> db.createUser({user: "useradmin", pwd: "adminpassword", roles: [{ role: "userAdminAnyDatabase", db: "admin" },{ role: "hostManager", db: "admin" }]})
   Successfully added user: {
           "user" : "useradmin",
           "roles" : [
                   {
                           "role" : "userAdminAnyDatabase",
                           "db" : "admin"
                   },
                   {
                           "role" : "hostManager",
                           "db" : "admin"
                   }
           ]
   }
   oneitomReplSet:PRIMARY> 
   ```

2. 验证用户是否添加成功，返回1表示成功

   ```shell
   oneitomReplSet:PRIMARY> db.auth("useradmin", "adminpassword") 
   1
   oneitomReplSet:PRIMARY> 
   ```

3. 退出控制台

   ```shell
   exit
   ```

4. 生成集群间认证key

   ```shell
   openssl rand -base64 745 > /home/oneitom/mongodb/key.file 
   ```

   将/home/oneitom/mongodb/key.file文件分别复制到另外两台服务器的相同目录下

   ```shell
   scp k/home/oneitom/mongodb/key.file root@10.22.1.236:/home/oneitom/mongodb/
   scp k/home/oneitom/mongodb/key.file root@10.22.1.237:/home/oneitom/mongodb/
   ```

   为每个服务器下的/home/oneitom/mongodb/key.file文件赋权

   ```shell
   chmod 600 /home/oneitom/mongodb/key.file
   ```

5. 开启安全验证配置

   vi /home/oneitom/mongodb/mongodb.conf 文件，修改security.authorization为enabled，并取消keyFile和clusterAuthMode的注释

   ```yaml
   security:
     authorization: enabled
     keyFile: /home/oneitom/mongodb/key.file 
     clusterAuthMode: keyFile
   ```

6. 重启三个服务器的mongodb

   > 注意启动顺序，先启动的会自动变成PRIMARY节点

   ```shell
   service mongod restart
   ```

7. 验证权限是否配置成功

   > 由于我先启动的是10.22.1.237，所以这个变成主节点

   未认证查询时提示没有权限 Unauthorized

   ```shell
   [root@swarm-docker-237 mongodb]# mongo --host 10.22.1.237:27017
   MongoDB shell version v3.4.17
   connecting to: mongodb://10.22.1.237:27017/
   MongoDB server version: 3.4.17
   oneitomReplSet:PRIMARY> use admin
   switched to db admin
   oneitomReplSet:PRIMARY> show dbs
   2018-10-16T17:27:22.778+0800 E QUERY    [thread1] Error: listDatabases failed:{
           "ok" : 0,
           "errmsg" : "not authorized on admin to execute command { listDatabases: 1.0 }",
           "code" : 13,
           "codeName" : "Unauthorized"
   } :
   _getErrorWithCode@src/mongo/shell/utils.js:25:13
   Mongo.prototype.getDBs@src/mongo/shell/mongo.js:62:1
   shellHelper.show@src/mongo/shell/utils.js:814:19
   shellHelper@src/mongo/shell/utils.js:704:15
   @(shellhelp2):1:1
   oneitomReplSet:PRIMARY> 
   ```

8. 认证后在查询正常

   ```shell
   oneitomReplSet:PRIMARY> db.auth("useradmin", "adminpassword") 
   1
   oneitomReplSet:PRIMARY> show dbs
   admin   0.000GB
   local   0.000GB
   testdb  0.000GB
   oneitomReplSet:PRIMARY>
   ```



### 1.5 创建一个新库

> 下边的演示了使用管理员账户登录后创建了一个cmdb库，并且配置cmdb的管理员账号为cmdb-user，密码为cmdb-password

```shell
[root@swarm-docker-235 mongodb]# mongo --host 10.22.1.237:27017
MongoDB shell version v3.4.17
connecting to: mongodb://10.22.1.237:27017/
MongoDB server version: 3.4.17
oneitomReplSet:PRIMARY> use admin
switched to db admin
oneitomReplSet:PRIMARY> db.auth("useradmin", "adminpassword")
1
oneitomReplSet:PRIMARY> db.createUser({user: "cmdb-user", pwd: "cmdb-password", roles: [{ role: "dbOwner", db: "cmdb" }]})
Successfully added user: {
        "user" : "cmdb-user",
        "roles" : [
                {
                        "role" : "dbOwner",
                        "db" : "cmdb"
                }
        ]
}
oneitomReplSet:PRIMARY> 
```

创建完毕后就可以使用以下地址连接了

```shell
mongodb://cmdb-user:cmdb-password@10.22.1.235:27017,10.22.1.236:27017,10.22.1.237:27017/cmdb? replicaSet=oneitomReplSet;slaveOk=true;safe=true;w=2;wtimeoutMS=2000;connectTimeoutMS=300000;socketTimeoutMS=60000
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