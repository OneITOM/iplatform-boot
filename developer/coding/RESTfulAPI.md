# RESTful API规约

> 作者 张磊

本文档作为RESTful API 的指导规约为开发人员提供指导性建议，并不作为严格规则要求

基于简单原则如下

* URL有意义并且唯一
* GET调用不进行写操作，不能改变服务状态
* PUT和DELETE调用需要遵循幂等原则
* POST调用谨慎使用
* 幂等性原则：可以重复执行，并且结果总是一样的（注意重复调用总是存在的，硬编码、自动重试等）

## 新增资源

* 新增数据使用PUT方法 **<font color=red>注意这里与常规推荐POST用法不同</font>**
* 参数适用JSON格式通过data传递，不要跟在URL末尾
* 幂等性

幂等性正确用法，增加一条告警数据

```shell
HTTP -H "Content-Type:application/json" -X PUT --data '{"alarminfo":"xxx","alarmlevel":1}' http://127.0.0.1/alarm
```

新增用法注意需要在程序内部对新增的数据进行唯一性校验，确保多次调用同一方法不会出现重复数据

## 修改资源

* 修改数据使用PUT方法
* 参数适用JSON格式通过data传递，不要跟在URL末尾
* 幂等性

幂等性正确用法，根据告警ID修改告警级别

```shell
HTTP -H "Content-Type:application/json" -X PUT --data '{"alarmlevel:2}' http://127.0.0.1/alarm/{alarmid}
```

幂等性返例，将告警表中最老的100调记录移动到历史表中，并且参数跟在了URL后边

```shell
HTTP PUT http://127.0.0.1/alarm/movehistory?oldrecords=100
```

## 删除资源

* 删除数据使用DELETE方法
* 幂等性

幂等性正确用法，根据告警ID删除一条告警记录

```shell
HTTP DELETE http://127.0.0.1/alarm/{alarmid}
```

幂等性返例，删除告警表中最老的5条记录，并且参数跟在了URL后边

```shell
HTTP DELETE http://127.0.0.1/alarm?oldrecords=5
```

## 查询资源

* 查询数据使用GET方法
* GET方法不能改变服务状态

```shell
HTTP GET http://127.0.0.1/user/{userid}
```

## 违反原则的处理

> 违反以上原则的场景统一使用POST方法，注意谨慎使用，避免过度使用

 常见的RESTful API设计会基于以下两种方式

* 基于服务：会频繁使用POST，或者POST=新增，PUT=修改，DELETE=删除
* 基于资源：本文的设计原则（建议使用，虽然目前本框架服务还不完全遵从此规则😅）

基于服务与基于资源的区别可以简单理解为SOAP与REST的区别，基于SOAP的模型下，我们在代理层只能看到POST，如果要知道当前HTTP具体要做什么（增加，修改，删除）就需要分析SOAP消息体。而在基于资源的方式时可以更加有利于服务器的识别，更可实现安全控制。

## 附件

[Architectural Styles and the Design of Network-based Software Architectures](https://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm)

