# Redis部署手册

> 作者 xx

## 1. 安装

### 1.1 前提

* JDK1.8+
* 磁盘剩余空间大于xxxMB
* 空闲内存大于xxxGB

### 1.2 单节点安装

   下载安装包

```bash
wget http://download.redis.io/releases/redis-4.0.11.tar.gz
```

   解压缩

```bash
cd /opt/aaa
tar -xzvf redis-4.0.11.tar.gz
```
安装

```xml
cd ./redis-4.0.11
make
```

配置

> 配置文件路径redis-4.0.11/redis.conf

```bash
# 只能通过本机访问
protected-mode no

# 客户端闲置多少秒后，断开连接，默认为300（秒）
timeout 300

# 绑定本机IP
bind 192.168.1.51

# 绑定端口
port 6379

# 是否后台启动 no or yes（docker内部不能配置成yes）
daemonize yes

# 数据库个数 10
databases 10

# 进程ID文件
pidfile "/var/run/redis_6379.pid"

# 日志级别 debug， verbose， notice， warning
loglevel notice

# 日志文件
logfile "redis.log"

# 当dump .rdb数据库的时候是否压缩数据对象
rdbcompression yes

# dump文件名
dbfilename "dump.rdb"

# 最大客户端链接数
maxclients 1000

# 配置认证密码
requirepass Bomc222

stop-writes-on-bgsave-error no

# 存储策略
save 900 1
save 300 10
save 60 10000

# 每秒一次同步磁盘
appendfsync everysec

# 最大内存4G
maxmemory 4294967296

# 关闭日志记录（严格不丢数据的场合应该配置成yes）
appendonly no
```

操作系统配置

```shell
echo "vm.overcommit_memory=1" > /etc/sysctl.conf 
echo 1 > /proc/sys/vm/overcommit_memory
echo 511 > /proc/sys/net/core/somaxconn
```

启动

```
cd redis-4.0.11/src/
./redis-server ../redis.conf
```

验证安装

```
cd redis-4.0.11/src/
#执行如下命令是否正常ok
./redis-cli -h 192.168.1.51 -p 6379 -a "Bomc222"
192.168.1.51:6379> set foo bar
OK
192.168.1.51:6379> get foo
"bar"
```

### 1.3 哨兵模式安装

哨兵模式规划

```
Redis Sentinel 规划
-------------------
IP 端口号 角色
192.168.1.51 6379 Redis Master
192.168.1.52 6379 Redis Slave
192.168.1.51 26379 Sentinel
192.168.1.52 26379 Sentinel
```

编译安装

```
cd ./redis-4.0.11
make
```

配置

edis主服务器redis.conf配置：

```
#绑定本机IP
bind 192.168.1.51
#绑定端口
port 6379
#开启后台启动
daemonize yes
#修改打印日志级别
loglevel verbose
#日志输出路径
logfile "/opt/BOCO/redis-4.0.11/redis.log"
#若开启密码认证：设置认证密码
requirepass Boco$bomc222
#关闭保护模式
protected-mode no
```

redis主服务器sentinel.conf配置：

```
bind 192.168.1.51
protected-mode no
port 26379
daemonize yes
sentinel monitor mymaster 192.168.1.51 6379 1
#若开启密码认证：设置为master requirepass一致的
```

redis从服务器redis.conf配置：

```
#绑定本机IP
bind 192.168.1.52
#绑定端口
port 6379
#开启后台启动
daemonize yes
loglevel verbose
logfile "/opt/BOCO/redis-4.0.11/redis.log"
#若开启密码认证：设置为master requirepass一致的密码
masterauth Boco$bomc222
#关闭保护模式
protected-mode no
#master ip port
slaveof 192.168.1.51 6379
```

redis从服务器sentinel.conf配置：

```
bind 192.168.1.52
protected-mode no
port 26379
daemonize yes
sentinel monitor mymaster 192.168.1.51 6379 1
#若开启密码认证：设置为master requirepass一致的密码
sentinel auth-pass mymaster Boco$bomc222
```

启动

```
1. 主服务器启动Redis Master
cd /opt/BOCO/redis-4.0.11/src
./redis-server ../redis.conf
2. 从服务器启动Redis Slave
cd /opt/BOCO/redis-4.0.11/src
./redis-server ../redis.conf
3. 主服务器启动Redis sentinel
cd /opt/BOCO/redis-4.0.11/src
./redis-sentinel ../sentinel.conf
4. 从服务器启动Redis sentinel
cd /opt/BOCO/redis-4.0.11/src
./redis-sentinel ../sentinel.conf
```

查看redis实例

```
ps -ef | grep redis
```

2222

```
cd /opt/BOCO/redis-4.0.11/src
./redis-cli -h 192.168.1.51 -p 26379 [-a "Boco$bomc222"]
192.168.1.51:26379>sentinel master mymaster
```

### 1.3 cluster模式

1. 部署规划

   ```
   Redis Cluster 规划
   -------------------
   IP 端口号 角色
   192.168.1.51 6379 Master
   192.168.1.51 26379 Slave
   192.168.1.52 6379 Master
   192.168.1.52 26379 Slave
   192.168.1.53 6379 Master
   192.168.1.53 26379 Slave
   ```

2. 编译安装

   ```
   cd ./redis-4.0.11
   make
   ```

3. 配置

   在51上进行配置redis的配置文件redis.conf：

   redis-6379.conf配置实例：

   ```
   #后台启动
   daemonize yes
   #后台启动进程ID
   pidfile /opt/BOCO/redis-4.0.2/redis-6379.pid
   port 6379
   bind 192.168.1.51
   cluster-enabled yes
   cluster-config-file nodes-6379.conf
   cluster-node-timeout 15000
   cluster-require-full-coverage no
   maxclients 15000
   appendonly yes
   appendfilename "appendonly-6379.aof"
   aof-rewrite-incremental-fsync yes
   save 900 1
   save 300 10
   save 60 10000
   dbfilename dump-6379.rdb
   dir /opt/BOCO/redis-4.0.2/data
   loglevel notice
   logfile "/opt/BOCO/redis-4.0.2/redis-6379.log"
   tcp-backlog 128
   #5分钟空闲断开连接，0：禁用
   timeout 300
   tcp-keepalive 300
   databases 16
   stop-writes-on-bgsave-error no
   rdbcompression yes
   rdbchecksum yes
   slave-serve-stale-data yes
   slave-read-only yes
   repl-diskless-sync no
   repl-diskless-sync-delay 5
   repl-disable-tcp-nodelay no
   slave-priority 100
   appendfsync everysec
   no-appendfsync-on-rewrite no
   auto-aof-rewrite-percentage 100
   auto-aof-rewrite-min-size 64mb
   aof-load-truncated yes
   lua-time-limit 5000
   slowlog-log-slower-than 10000
   slowlog-max-len 1000
   latency-monitor-threshold 0
   notify-keyspace-events ""
   hash-max-ziplist-entries 512
   hash-max-ziplist-value 64
   list-max-ziplist-entries 512
   list-max-ziplist-value 64
   set-max-intset-entries 512
   zset-max-ziplist-entries 128
   zset-max-ziplist-value 64
   hll-sparse-max-bytes 3000
   activerehashing yes
   client-output-buffer-limit normal 0 0 0
   client-output-buffer-limit slave 256mb 64mb 60
   client-output-buffer-limit pubsub 32mb 8mb 60
   hz 10
   ```

   edis-26379.conf配置实例：

   ```
   #后台启动
   daemonize yes
   #后台启动进程ID
   pidfile /opt/BOCO/redis-4.0.2/redis-26379.pid
   port 26379
   bind 192.168.1.51
   cluster-enabled yes
   cluster-config-file nodes-26379.conf
   cluster-node-timeout 15000
   cluster-require-full-coverage no
   maxclients 15000
   appendonly yes
   appendfilename "appendonly-26379.aof"
   aof-rewrite-incremental-fsync yes
   save 900 1
   save 300 10
   save 60 10000
   dbfilename dump-26379.rdb
   dir /opt/BOCO/redis-4.0.2/data
   loglevel notice
   logfile "/opt/BOCO/redis-4.0.2/redis-26379.log"
   tcp-backlog 128
   #5分钟空闲断开连接，0：禁用
   timeout 300
   tcp-keepalive 300
   databases 16
   stop-writes-on-bgsave-error no
   rdbcompression yes
   rdbchecksum yes
   slave-serve-stale-data yes
   slave-read-only yes
   repl-diskless-sync no
   repl-diskless-sync-delay 5
   repl-disable-tcp-nodelay no
   slave-priority 100
   appendfsync everysec
   no-appendfsync-on-rewrite no
   auto-aof-rewrite-percentage 100
   auto-aof-rewrite-min-size 64mb
   aof-load-truncated yes
   lua-time-limit 5000
   slowlog-log-slower-than 10000
   slowlog-max-len 1000
   latency-monitor-threshold 0
   notify-keyspace-events ""
   hash-max-ziplist-entries 512
   hash-max-ziplist-value 64
   list-max-ziplist-entries 512
   list-max-ziplist-value 64
   set-max-intset-entries 512
   zset-max-ziplist-entries 128
   zset-max-ziplist-value 64
   hll-sparse-max-bytes 3000
   activerehashing yes
   client-output-buffer-limit normal 0 0 0
   client-output-buffer-limit slave 256mb 64mb 60
   client-output-buffer-limit pubsub 32mb 8mb 60
   hz 10
   ```

   另外两台52，53机器进行类似配置即可。
   分别到51-53机器上启动6379，26379进程实例：

4. 启动

   分别到51-53机器上启动6379，26379进程实例：

   ```
   cd /opt/BOCO/redis-4.0.11/src
   ./redis-server ../redis-6379.conf
   ./redis-server ../redis-26379.conf
   #查看进程实例是否存在
   ps -ef | grep redis
   ```

5. 安装依赖

   由于 Redis 集群需要使用 ruby 命令，所以我们需要安装 ruby 和相关接口。

   ```
   yum install ruby
   yum install rubygems
   gem install redis
   ```

6. 创建集群

   ```
   cd /opt/BOCO/redis-4.0.11/src
   ./redis-trib.rb create --replicas 1 192.168.1.51:6379 192.168.1.51:26379 192.168.1.52:6379 192.168.1.52:26379 192.168.1.53:6379 192.168.1.53:#日志最后输出如下内容表示成功
   [OK] All 16384 slots covered.
   ```

7. 查看状态

```
cd /opt/BOCO/redis-4.0.11/src
./redis-cli -h 192.168.1.51 -p 6379
192.168.1.51:6379> cluster info
192.168.1.51:6379> cluster nodes
```



## 1.4 Docker

> 基于Docker的单节点配置

```yaml
version: '3.2'
services:
  oneitom-redis:
    image: boco/alpine-redis:4.0.6
    hostname: oneitom-redis
    container_name: oneitom-redis
    networks:
      - oneitom-network
    ports:
      - '6379:6379'
    volumes:
      - ${ONEITOM_VOLUME_PATH}/oneitom-redis:/app
    labels:
      - oneitom-redis-cluster   
networks:
  oneitom-network:
    external: true
```


