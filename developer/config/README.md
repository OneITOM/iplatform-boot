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

> 订阅消息总线,实现配置变更后的逻辑

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

## logstash采集kafka应用日志消息

```jruby

input {
  
  kafka {
    zk_connect => "172.18.254.105:2181,172.18.254.106:2181,172.18.254.107:2181/mykafka"
    group_id => "logstash-trident-group"
    topic_id => "trident-service-log"
    reset_beginning => false
    consumer_threads => 1
    codec => "json"
    #smallest or largest
    auto_offset_reset => "smallest"
  }

}

```

## kafka应用日志消息入elasticsearch

json格式kafka日志消息key重命名及指定字段值小写
```jruby

filter {

  mutate {
    rename => {
        "traceid" => "traceId"
    }
    lowercase => [ "busisys", "serviceId" ]
  }

}

```

kafka日志消息入elasticsearch
```jruby

output {

	elasticsearch {
	  hosts => ["172.18.254.105:9200","172.18.254.106:9200","172.18.254.107:9200"]
	  index => "%{busisys}-%{serviceId}-%{+YYYY.MM.dd}"
	  template_name => "oneitom-template"
	  template => "/usr/local/logstash-2.4.1/templates/oneitom-template.json"
	  template_overwrite => true
	  document_type => "logs"
	}

}

```

附：elasticsearch索引模板oneitem-template.json
```json

{
	"template": "oneitom*",
	"settings": {
		"index.refresh_interval": "5s",
		"index.number_of_shards": "5",
		"index.number_of_replicas": "1"
	},
	"mappings": {
		"_default_": {
			"dynamic_templates": [{
				"message_field": {
					"mapping": {
						"fielddata": {
							"format": "disabled"
						},
						"index": "analyzed",
						"type": "string"
					},
					"match_mapping_type": "string",
					"match": "message"
				}
			}],
			"_all": {
				"enabled": true
			},
			"properties": {
				"@timestamp": {
					"type": "date"
				},
				"serviceId": {
					"index": "not_analyzed",
					"type": "string"
				},
				"serviceInstId": {
					"index": "not_analyzed",
					"type": "string"
				},
				"traceId": {
					"index": "not_analyzed",
					"type": "string"
				},
				"tracespanid": {
					"index": "not_analyzed",
					"type": "string"
				},
				"busisys": {
					"index": "not_analyzed",
					"type": "string"
				},
				"level": {
					"index": "not_analyzed",
					"type": "string"
				},
				"@version": {
					"index": "not_analyzed",
					"type": "string"
				}
			}
		}
	}
}

```



