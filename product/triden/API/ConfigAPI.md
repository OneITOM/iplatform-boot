# 服务配置集成

> 作者 于胜强

## 集中配置参数

| key | value | 备注
|:-----:|:------:|:-----:|
|spring.cloud.config.busisys|OneITOM|业务系统,默认为OneITOM
|spring.cloud.config.enabled|false|是否启用集中配置,默认为false
|spring.cloud.config.profile||应用环境，启用集中配置时必填


## 配置参数同步集中配置

`curl -X POST -H 'X-CONFIG-BUSISYS: {busisys}' -i http://localhost:8761/api/v1/syncparams/{operation} --data 'configName={name}&configProfile={profile}&configLabel={label}&jsonParams={jsonStr}'`

> 说明：`{xxxx}`为具体的业务值,参数解释具体如下：

`{busisys}`: 指定业务系统ID

`{operation}`: 操作类型：add, update, delete

`{name}`: 应用ID

`{profile}`: 应用环境

`{label}`: 应用版本

`{jsonStr}`: 配置项数组json格式如：`[{"key":"spring.key","value":"123","remark":"备注"},{"key":"spring.test.key","value":"123","remark":"备注"}]`

> 返回结果：

成功 -> {"success":true, "message": null, "data": null}

失败 -> {"success":false, "message", "XXX错误信息", "data": null}

## 集中配置查询获取配置参数

> 返回properties文件格式 restful api：

`curl -X GET -H 'X-CONFIG-BUSISYS: {busisys}' -H 'Content-Type: application/x-www-form-urlencoded' -i 'http://localhost:8761/config/{label}/{name}-{profile}.properties'`

> 返回yml文件格式 restful api：

`curl -X GET -H 'X-CONFIG-BUSISYS: {busisys}' -H 'Content-Type: application/x-www-form-urlencoded' -i 'http://localhost:8761/config/{label}/{name}-{profile}.yml'`

参数说明：

`{busisys}`: 指定业务系统ID

`{name}`: 应用ID

`{profile}`: 应用环境

`{label}`: 应用版本

## 配置变更通知

> 采用ActiveMQ进行通知

发布TOPIC: T_CCMP_CONFIG

订阅消息：json字符串

```
{
	"uuid": "23e9b957c4e245c7a0c6f7a93b031b5e",
	"timestamp": "1540823027123",
	"busisys": "OneITOM",
	"application": "dfss-service",
	"profile": "prod",
	"label": "0.0.2",
	"propItems": [
    {
		"key": "spring.test.key1",
		"value": "123",
		"type": "add"
	},
	{
		"key": "spring.test.key2",
		"value": "123",
		"type": "update"
	},
	{
		"key": "spring.test.key3",
		"value": "123",
		"type": "delete"
	}]
}
```





