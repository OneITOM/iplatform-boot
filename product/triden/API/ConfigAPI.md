# 服务配置集成

> 作者 于胜强

### 配置参数同步集中配置

`curl -X POST -H 'X-CONFIG-BUSISYS: {busisys}' -i http://localhost:8761/api/v1/syncparams/{operation} --data 'configName={name}&configProfile={profile}&configLabel={label}&jsonParams={jsonStr}'`

说明：`{xxxx}`为具体的业务值,参数解释具体如下：

`{busisys}`: 指定业务系统ID

`{operation}`: 操作类型：add, update, delete

`{name}`: 应用ID

`{profile}`: 应用环境

`{label}`: 应用版本

`{jsonStr}`: 配置项数组json格式如：`[{"key":"spring.key","value":"123","remark":"备注"},{"key":"spring.test.key","value":"123","remark":"备注"}]`

### 集中配置配置项参数变更


### 集中配置查询获取




### 配置变更通知





