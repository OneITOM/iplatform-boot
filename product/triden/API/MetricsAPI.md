# 服务度量集成

> 作者 张磊

服务度量通过ActiveMQ上报到微服务管控平台

## ActiveMQ配置

QUEUE：Q_MS_METRICS

上报周期：建议每5分钟上报一次

度量值格式：JSON

## 数据格式

> 服务度量值采用json格式，示例如下：

```json
{
	"message_id": "dff512c7-b344-4095-a24a-4e7ac984abbc",
	"version": "0.1",
	"time": "2018-09-13 09:10:05",
	"type": "metrics",
	"ms_id": "example-server",
	"ms_ins_id": "[10.50.7.10]:example-server:8761",
	"ms_from": "example-server",
	"host": "192.168.0.1",
	"data": {
		"mem": 887538,
		"mem.free": 287955,
		"processors": 4,
		"instance.uptime": 53246,
		"uptime": 88939,
		"systemload.average": 6.71533203125,
		"heap.committed": 797696,
		"heap.init": 131072,
		"heap.used": 509740,
		"heap": 1864192,
		"threads.peak": 54,
		"threads.daemon": 43,
		"threads.totalStarted": 184,
		"threads": 54,
		"classes": 13601,
		"classes.loaded": 13602,
		"classes.unloaded": 1,
		"gc.ps_scavenge.count": 30,
		"gc.ps_scavenge.time": 1121,
		"gc.ps_marksweep.count": 3,
		"gc.ps_marksweep.time": 980
	}
}
```

## 度量值说明

### 消息头

| key        | value                                | 必填 | 说明                                                         |
| ---------- | ------------------------------------ | ---- | ------------------------------------------------------------ |
| message_id | dff512c7-b344-4095-a24a-4e7ac984abbc | 是 | UUID格式，例如                                               |
| version    | 0.1                                  | 是 | 消息格式版本，默认0.1                                        |
| time       | 2018-09-13 09:10:05                  | 是 | 消息发送时间                                                 |
| type       | metrics                              | 是 | 消息类型，固定填写metrics                                    |
| ms_id      | example-server                       | 是 | 服务ID                                                       |
| ms_ins_id  | [10.50.7.10]:discovery-server:8761]  | 是 | 服务实例ID,格式不固定，要能全网表示一个唯一服务，建议格式[IP]:ms_id:port |
| ms_form    | example-server                       | 是 | 事件发送者服务ID，如果事件发送者就是服务本身，那么与ms_id相同 |
| host | 192.168.0.1 | 是 | 事件发送服务器IP |

### 指标数据data

> 指标数据data部分可集成 spring boot 的 [/metircs](https://docs.spring.io/spring-boot/docs/1.3.5.RELEASE/reference/html/production-ready-metrics.html) 获取，例如：

| key        | value                                | 必填 | 说明                                                         |
| ---------- | ------------------------------------ | ---- | ------------------------------------------------------------ |
| data.mem | 887538 | 是 |分配给应用的总内存，kb|
| data.mem.free | 287955 | 是 |空闲内存，kb|
| data.processors | 4 | 是 |处理器数量|
| data.instance.uptime | 53246 |  |实例正常运行时间，毫秒|
| data.uptime | 88939 | 是 |系统正常运行时间，毫秒|
| data.systemload.average | 6.71533203125 |  |系统平均负载|
| data.heap.committed | 797696 |  |堆信息，kb|
| data.heap.init | 131072 | |初始化堆信息，kb|
| data.heap.used | 509740 | |已使用堆信息，kb|
| data.heap | 1864192 | |堆信息，kb|
| data.threads.peak | 54 | |线程峰值数|
| data.threads.daemon | 43 | |守护线程数|
| data.threads.totalStarted | 184 | |启动线程总数|
| data.threads | 54 | |线程数|
| data.classes | 13601 | |类加载信息|
| data.classes.loaded | 13602 | |类加载信息|
| data.classes.unloaded | 1 | |类加载信息|
| data.gc.ps_scavenge.count | 30 | |垃圾回收次数|
| data.gc.ps_scavenge.time | 1121 | |垃圾回收时间|
| data.gc.ps_marksweep.count | 3 | |标记清除算法次数|
| data.gc.ps_marksweep.time | 980 | |标记清除算法消耗时间|