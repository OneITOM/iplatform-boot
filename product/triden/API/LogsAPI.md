# 服务日志集成

> 作者 张磊

服务日志通过Kafka上报到微服务管控平台

## Kafka配置

TOPIC：服务ID-LOG，例如：example-service-log

KEY：进程ID+@+主机名，例如：6534@192.168.0.1

## 数据格式

```json
{
	"ip": "192.168.0.1",
	"busisys": "OneITOM",
	"timestamp": 1538965917237,
	"serviceId": "example-service",
	"serviceInstId": "192.168.0.1::example-service:58080",
	"traceid": "6959dcf98356c725",
	"tracespanid": "9ef659e1deb2126c",
	"level": "INFO",
	"thread": "http-nio-58080-exec-4",
	"logger": " o.i.m.c.s.i.m.SQLInterceptor",
	"message": "sqltrace [insert into empty_test (is_enable,create_time) values (true,'2018-10-8 10:31:49')] cost [49ms]"
}
```

| key           | value                              | 说明                 |
| ------------- | ---------------------------------- | -------------------- |
| ip            | 192.168.0.1                        | 应用所在服务器IP地址 |
| busisys       | OneITOM                            | 应用所属业务系统名称 |
| timestamp     | 1538965917237                      | 日志产生时间         |
| serviceId     | example-service                    | 服务ID               |
| serviceInstId | 192.168.0.1::example-service:58080 | 服务实例ID           |
| traceid       | 6959dcf98356c725                   | 跟踪ID               |
| tracespanid   | 9ef659e1deb2126c                   | 当前环节SPANID       |
| level         | INFO                               | 日志级别             |
| thread        | http-nio-58080-exec-4              | 线程ID               |
| logger        | o.i.m.c.s.i.m.SQLInterceptor       | 日志产生类名         |
| message       |                                    | 日志正文             |

