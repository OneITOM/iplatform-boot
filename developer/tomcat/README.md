# Tomcat开发手册

> 作者 张磊

框架默认使用Embed Tomcat，并对Tomcat做了自定义扩展

## 版本

> tomcat-embed-core-8.0.33

## 参数

## multipart

```properties
# 启动多部分上传功能
multipart.enabled=true
# 超过此阀值后文件写入磁盘，单位可以是MB、KB
multipart.file-size-threshold=0
# 上传文件的临时目录
multipart.location=
# 最大支持上传文件大小，单位可以是MB、KB
multipart.max-file-size=1Mb
# 最大支持请求大小，单位可以是MB、KB
multipart.max-request-size=10Mb 
```

### server

```properties
# 设置服务绑定IP
server.host
# 设置服务绑定端口
server.port
# 设置服务上下文路径
server.contextPath
# 第二绑定非SSL端口开启开关，默认false
server.second.enabled=false
# 第二绑定非SSL端口号
server.second.port=
# 开启响应压缩
server.compression.enabled=false
# 排除响应压缩的代理列表
server.compression.excluded-user-agents=
# 定义MIME的压缩类型，逗号分隔，例如：text/html,text/css,application/json
server.compression.mime-types=
# 触发响应压缩的最小尺寸
server.compression.min-response-size=
```

### session cookie

```properties
# 指定session cookie的comment
server.session.cookie.comment=
# 指定session cookie的domain
server.session.cookie.domain=
# 是否开启HttpOnly
server.session.cookie.http-only=
# 设定session cookie的最大age.
server.session.cookie.max-age=
# 设定session cookie的最大age.
server.session.cookie.name=
# 设定session cookie的路径
server.session.cookie.path=
# 设定session cookie的“Secure” flag
server.session.cookie.secure=
# 重启时是否持久化session，默认false
server.session.persistent=
# session持久化目录
server.session.store-dir=
# session的超时时间
server.session.timeout=
# 设定Session的追踪模式(cookie, url, ssl)
server.session.tracking-modes=
```

### ssl

```properties
# 支持的SSL ciphers.
server.ssl.ciphers= "TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_AES_128_SHA256,TLS_ECDHE_ECDSA_WITH_AES_128_SHA256,TLS_ECDHE_RSA_WITH_AES_128_SHA,TLS_ECDHE_ECDSA_WITH_AES_128_SHA,TLS_ECDHE_RSA_WITH_AES_256_SHA384,TLS_ECDHE_ECDSA_WITH_AES_256_SHA384,TLS_ECDHE_RSA_WITH_AES_256_SHA,TLS_ECDHE_ECDSA_WITH_AES_256_SHA"
# 使用的SSL协议
server.ssl.protocol=TLS
# 使用的SSL协议版本
server.ssl.enabled-protocols=TLSv1.2
```

### tomcat

```properties
# 最大工作线程数（默认2000）
server.tomcat.max-threads=2000
# 消息头最大字节数（默认65536）
server.tomcat.max-http-header-size=65536
# URI编码解码的字符集
server.tomcat.uri-encoding=UTF-8
# 开启tomcat访问日志（默认关闭）
server.tomcat.accesslog.enabled=false
# tomcat日志目录
server.tomcat.accesslog.directory=logs
# 设定access logs的格式，默认: common
server.tomcat.accesslog.pattern=common
# 设定Log 文件的前缀，默认: access_log
server.tomcat.accesslog.prefix=access_log
# 设定Log 文件的后缀，默认: .log
server.tomcat.accesslog.suffix=.log
# 后台线程方法的Delay大小，默认30
server.tomcat.background-processor-delay=30 
# 设定Tomcat的base 目录，如果没有指定则使用临时目录
server.tomcat.basedir=
# 设定信任IP的正则表达式
server.tomcat.internal-proxies=10\\.\\d{1,3}\\.\\d{1,3}\\.\\d{1,3}|\\
        192\\.168\\.\\d{1,3}\\.\\d{1,3}|\\
        169\\.254\\.\\d{1,3}\\.\\d{1,3}|\\
        127\\.\\d{1,3}\\.\\d{1,3}\\.\\d{1,3}|\\
        172\\.1[6-9]{1}\\.\\d{1,3}\\.\\d{1,3}|\\
        172\\.2[0-9]{1}\\.\\d{1,3}\\.\\d{1,3}|\\
        172\\.3[0-1]{1}\\.\\d{1,3}\\.\\d{1,3}
# 设定http header使用的，用来覆盖原来port的value        
server.tomcat.port-header=X-Forwarded-Port
# 设定Header包含的协议，通常是 X-Forwarded-Proto，如果remoteIpHeader有值，则将设置为RemoteIpValve.
server.tomcat.protocol-header=
# 设定使用SSL的header的值，默认https.
server.tomcat.protocol-header-https-value=https
# 设定remote IP的header，如果remoteIpHeader有值，则设置为RemoteIpValve
server.tomcat.remote-ip-header=
```



