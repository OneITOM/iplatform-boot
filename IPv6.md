# IPv6 Support

> IPv6支持列表

| 名称                         | 版本 | 支持 |
| ---------------------------- | ---- | ---- |
| ActiveMQ                     |  5.14.1    | Y    |
| [MySQL](#user-content-mysql) | 5.6.40    |   Y   |
| Redis                        |  4.0.8    |    Y  |
| Kafka                        |   2.11-1.1.1   | Y     |
| Zookeeper                    |   3.4.13   | Y     |
| MongoDB                      |   3.4.3   |      |
| storm | 1.1.0 | Y |
|                              |      |      |
|                              |      |      |

## CENTOS支持IPV6

1. cp /etc/modprobe.d/disable_ipv6.conf /etc/modprobe.d/disable_ipv6.conf_backup ##先备份原始配置

   ```properties
   vi /etc/modprobe.d/disable_ipv6.conf
   
   修改前
   alias net-pf-10 off
   options ipv6 disable=1
   
   修改后
   alias net-pf-10 off
   options ipv6 disable=0
   ```

2. vim /etc/sysconfig/network

   ```properties
    修改前
   PEERNTP=no
   NETWORKING_IPV6=no
   GATEWAY=139.255.255.0(一般不用修改，默认即可)
   
   修改后
   PEERNTP=no
   NETWORKING_IPV6=yes
   GATEWAY=139.255.255.0
   ```



3. vim /etc/sysconfig/network-scripts/ifcfg-eth0

   ```properties
   修改前
   DEVICE=eth0
   ONBOOT=yes
   BOOTPROTO=static
   IPADDR=10.10.10.1
   NETMASK=255.255.254.0
   
   修改后
   DEVICE=eth0
   ONBOOT=yes
   BOOTPROTO=static
   IPADDR=10.10.10.1
   NETMASK=255.255.254.0
   IPV6INIT=yes
   IPV6_AUTOCONF=yes
   ```

4. 修改 /etc/sysctl.conf

   ```properties
   vim /etc/sysctl.conf
   
   修改前
   vm.swappiness = 0
   net.ipv4.neigh.default.gc_stale_time=120
   net.ipv4.conf.all.rp_filter=0
   net.ipv4.conf.default.rp_filter=0
   net.ipv4.conf.default.arp_announce = 2
   net.ipv4.conf.all.arp_announce=2
   net.ipv4.tcp_max_tw_buckets = 5000
   net.ipv4.tcp_syncookies = 1
   net.ipv4.tcp_max_syn_backlog = 1024
   net.ipv4.tcp_synack_retries = 2
   net.ipv6.conf.all.disable_ipv6 = 1
   net.ipv6.conf.default.disable_ipv6 = 1
   net.ipv6.conf.lo.disable_ipv6 = 1
   net.ipv4.conf.lo.arp_announce=2
   
   修改后
   vm.swappiness = 0
   net.ipv4.neigh.default.gc_stale_time=120
   net.ipv4.conf.all.rp_filter=0
   net.ipv4.conf.default.rp_filter=0
   net.ipv4.conf.default.arp_announce = 2
   net.ipv4.conf.all.arp_announce=2
   net.ipv4.tcp_max_tw_buckets = 5000
   net.ipv4.tcp_syncookies = 1
   net.ipv4.tcp_max_syn_backlog = 1024
   net.ipv4.tcp_synack_retries = 2
   net.ipv6.conf.all.disable_ipv6 = 0
   net.ipv6.conf.default.disable_ipv6 = 0
   net.ipv6.conf.lo.disable_ipv6 = 0
   net.ipv4.conf.lo.arp_announce=2
   ```

5. 创建系统在启动时自动加载 IPv6 模块的脚本

   ```properties
   vim /etc/sysconfig/modules/ipv6.modules
   脚本内容
   #!/bin/sh
   if [ ! -c /proc/net/if_inet6 ] ; then
   exec /sbin/insmod /lib/modules/uname -r/kernel/net/ipv6/ipv6.ko
   fi
   ```

6. 重启系统，加载 IPv6 模块

   ```properties
    ifconfig | grep -i inet6
   ```



## MySQL

1. JDBC URL

```properties
spring.datasource.url="jdbc:mysql://address=(protocol=tcp)(host=fe80::5429:9e58:a3bd:dbd2)(port=3306)/auth?useUnicode=true&amp;characterEncoding=utf-8&autoReconnect=true"
```
2. 安装

  ```properties
  在/etc/my.cnf文件[mysqld]组内添加
  bind-address=::
  ```

## ActiveMQ
直接安装解压即可

## Redis

1. 连接url

   ```properties
   --spring.redis.host=fe80::5429:9e58:a3bd:dbd2 \
   ```

2. 安装

bind注释掉，代表绑定主机上所有的IP

## zookeeper

1. url

   ```properties
   --spring.zookeeper.address=[fe80::5429:9e58:a3bd:dbd2]:2181,[fe80::19b6:180f:1f00:f666]:2181,[fe80::dfea:5b6:22a5:7505]:2181
   ```

2. 安装

/etc/hosts设置IPV6地址、主机名映射

```properties
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
fe80::5429:9e58:a3bd:dbd2       bomcipv61
fe80::19b6:180f:1f00:f666       bomcipv62
fe80::dfea:5b6:22a5:7505        bomcipv63
```



设置IPv6地址，其他安装方式一样

```properties
server.1=[fe80::5429:9e58:a3bd:dbd2]:2888:3888
server.2=[fe80::19b6:180f:1f00:f666]:2888:3888
server.3=[fe80::dfea:5b6:22a5:7505]:2888:3888
```

## kafka

1. url

   ```properties
   --spring.kafka.broker.address=[fe80::5429:9e58:a3bd:dbd2]:9092,[fe80::19b6:180f:1f00:f666]:9092,[fe80::dfea:5b6:22a5:7505]:9092
   ```

2. 安装

这两处需要设置IPv6，其他与IPV4配置放一样

```properties
host.name=fe80::5429:9e58:a3bd:dbd2
zookeeper.connect=[fe80::5429:9e58:a3bd:dbd2]:2181,[fe80::19b6:180f:1f00:f666]:2181,[fe80::dfea:5b6:22a5:7505]:2181
```



## storm

安装

```properties
storm.zookeeper.servers: ["fe80::5429:9e58:a3bd:dbd2","fe80::19b6:180f:1f00:f666","fe80::dfea:5b6:22a5:7505"]
nimbus.seeds: ["bomcipv61","bomcipv62","bomcipv63"]
```

注意：nimbus.seeds这里必须设置主机名，否则ui界面会分别显示ipv6地址、主机名的信息，且ipv6的状态不正常；

