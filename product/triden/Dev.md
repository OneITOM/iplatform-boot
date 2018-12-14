# 微服务管控平台API

> 作者 张磊

本手册指导第三方系统如何通过API集成微服务管控平台，API接口全部采用RESTful API方式。

## 集中配置

> 参见[集中配置开发手册](../../developer/config/README.md)



## 集中日志

> 参见[集中日志开发手册](../../developer/logger/README.md)



## 远程控制

1. 远程停止服务

   允许通过RESTful API 停止一个服务，调用方式如下

   ```shell
   curl -X POST -H 'end_point_timestamp':'1535007763000' https://{ip}:{port}/{contextpath}/shutdown
   ```

   访问白名单，默认127.0.0.1

   ```properties
   # 允许远程访问的ip白名单
   endpoints.shutdown.allowip=192.168.1.100,192.168.1.101
   ```

   成功返回结果

   ```json
   HTTP: Status 200
   {"message":"Shutting down, bye..."}
   ```

   失败返回结果

   ```
   HTTP: Status 404 # 服务没有开启远程控制
   HTTP: Status 406 # 请求不接受
   HTTP: Status 401 # 没有调用权限
   HTTP: Status 408 # 请求超时
   ```

   

## 服务实例

```properties
# 允许使用参数定义服务实例
server.instanceId=xxxxx
```

## 事件

> Q_MS_EVENT

## 指标

> Q_MS_METRICS

## 调用链

> Q_MS_TRACK

