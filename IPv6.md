# IPv6 Support

> IPv6支持列表

| 名称                         | 版本 | 支持 |
| ---------------------------- | ---- | ---- |
| ActiveMQ                     |  5.14.1    | Y    |
| [MySQL](#user-content-mysql) | 5.6.40    |   Y   |
| Redis                        |  4.0.8    |    Y  |
| Kafka                        |   2.11-1.1.1   | Y     |
| Zookeeper                    |   3.4.13   | Y     |
| MongoDB                      |   3.4.3   |   N   |
|                              |      |      |
|                              |      |      |
|                              |      |      |

## MySQL

1. JDBC URL

```properties
spring.datasource.url="jdbc:mysql://address=(protocol=tcp)(host=fe80::5429:9e58:a3bd:dbd2)(port=3306)/auth?useUnicode=true&amp;characterEncoding=utf-8&autoReconnect=true"
```
2. 安装
在[mysqld]组内添加
bind-address=::

## ActiveMQ
直接安装解压即可


## Redis
bind注释掉，代表绑定主机上所有的IP






