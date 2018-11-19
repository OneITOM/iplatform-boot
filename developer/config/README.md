# 集中配置开发手册

> 作者 于胜强

## 修改接口

> 通过RESTful API提供配置的新增、修改、删除

```shell
POST https://localhost:8761/api/v1/configparams/{operation}?configName={XXX}&configProfile={XXX}&jsonParams={XXX}
```

> 参数说明：

- operation: `add`, `update`, `delete`
- configName：应用微服务ID 如：`dfss-service`
- configProfile：应用环境标签 如：`development`
- jsonParams：配置项json数组 如：`[{"key":"testkey1","value":"testStr"},{"key":"testkey2","value":123456}]`

> 接口返回

- 正确返回：`{"succcess":true, "message":null, "data":null}`
- 错误返回：`{"succcess":false, "message":"您的参数不合法XXX.", "data":null}`

## 监听接口

> 订阅消息总线,实现配置变更后的逻辑，每个微服务可以通过开启集中配置参数，实现配置参数的集中管理，和配置变更监听

开启集中配置开关

```properties
spring.cloud.config.enabled=true
```



```java
  /** 装配变更通知消息总线 */
  @Autowired
  private AsyncEventBus asyncEventBus;
```

```java
  /** 注册订阅方法至消息总线 */
  @PostConstruct
  public void register() {
    asyncEventBus.register(this);
  }
```

```java
  /** 注册订阅变更通知事件 */
  @Subscribe
  public void listener(ChangeMsgEvent event) {
    //获取变更的配置列表PropItem->{type, key, value}
    List<PropItem> propList = event.getPropItems();
		for(PropItem item : propList) {
			type = item.getType();
			if(PropItemTypeEnum.add == type) {
				//TODO: 新增
			} else if(PropItemTypeEnum.update == type) {
				//TODO: 修改
			} else if(PropItemTypeEnum.delete == type) {
				//TODO: 删除
			}
		}
  }
}
```
