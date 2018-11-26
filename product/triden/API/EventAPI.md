# 服务事件集成

> 作者 张磊

服务事件通过ActiveMQ上报到微服务管控平台

## ActiveMQ配置

QUEUE：Q_MS_EVENT

事件数据格式：JSON

## 事件类型

| 类型                        | 说明         | 描述                              |
| --------------------------- | ------------ | --------------------------------- |
| service_up                  | 服务启动     | 服务启动完成后发送                |
| service_down                | 服务停止     | 服务正常停止后发送                |
| service_heart（默认不开启） | 服务心跳     | 定时发送心跳事件                  |
| sql_error                   | SQL错误      | SQL错误事件                       |
| sql_cost                    | SQL耗时      | SQL执行耗时（超过动态基线时发送） |
| http_error                  | HTTP错误     | HTTP调用错误                      |
| track_sampling_change       | 采样率变更   | 采样率发生变更时发送              |
| traffic_weight_change       | 流控权重变更 | 流控权重发生变更时发送            |
| route_label_change          | 路由标签变更 | 路由标签变更时发送                |
| teant_change                | 租户变更     | 租户标签变更时发送                |
| circuit_breaker             | 服务熔断     | 发生服务熔断后发送                |

## 事件数据格式

> 事件数据格式由消息头和消息体组成

```json
{
    消息头,
    "data": 消息体
}
```

#### 消息头

| key        | value                                | 必填 | 说明                                                         |
| ---------- | ------------------------------------ | ---- | ------------------------------------------------------------ |
| message_id | dff512c7-b344-4095-a24a-4e7ac984abbc | 是   | UUID格式，例如                                               |
| version    | 0.1                                  | 是   | 消息格式版本，默认0.1                                        |
| time       | 2018-09-13 09:10:05                  | 是   | 消息发送时间                                                 |
| type       | event                                | 是   | 消息类型，固定填写event                                      |
| ms_id      | example-server                       | 是   | 服务ID                                                       |
| ms_ins_id  | [10.50.7.10]:discovery-server:8761]  | 是   | 服务实例ID,格式不固定，要能全网表示一个唯一服务，建议格式[IP]:ms_id:port |
| ms_form    | example-server                       | 是   | 事件发送者服务ID，如果事件发送者就是服务本身，那么与ms_id相同 |
| host       | 192.168.0.1                          | 是   | 发送服务器IP                                                 |

## 事件数据示例

1. 服务启动

   ```json
   {
   	"message_id": "939f1b4f-860c-42e7-8142-bf8bb32f0a72",
   	"version": "0.1",
   	"time": "2018-09-13 09:15:20",
   	"type": "event",
	"ms_id": "example-server",
   	"ms_ins_id": "[10.50.7.10]:example-server:8761",
   	"ms_from": "example-server",
       "host": "192.168.0.1",
   	"data": {
   		"event_code": "service_up",
   		"event_title": "服务启动"
   	}
   }
   ```

   

2. 服务停止

   ```json
   {
   	"message_id": "939f1b4f-860c-42e7-8142-bf8bb32f0a72",
   	"version": "0.1",
   	"time": "2018-09-13 09:15:20",
   	"type": "event",
   	"ms_id": "discovery-server",
   	"ms_ins_id": "[10.50.7.10]:discovery-server:8761",
   	"ms_from": "discovery-server",
	"host": "192.168.0.1",    
   	"data": {
   		"event_code": "service_down",
   		"event_title": "服务下线"
   	}
   }
   ```

   

3. 服务心跳

   ```json
   {
   	"message_id": "939f1b4f-860c-42e7-8142-bf8bb32f0a72",
   	"version": "0.1",
   	"time": "2018-09-13 09:15:20",
   	"type": "event",
   	"ms_id": "discovery-server",
   	"ms_ins_id": "[10.50.7.10]:discovery-server:8761",
   	"ms_from": "discovery-server",
	"host": "192.168.0.1",    
   	"data": {
   		"event_code": "service_heart",
   		"event_title": "服务心跳"
   	}
   }
   ```

   

4. SQL错误

   ```json
   {
   	"message_id": "9a10c919-b5fa-463c-a24e-6e5e5eae09fa",
   	"version": "0.1",
   	"time": "2018-09-22 18:38:44",
   	"type": "event",
   	"ms_id": null,
   	"ms_ins_id": null,
   	"ms_from": null,
	"host": "192.168.0.1",    
   	"data": {
   		"sqlid": "org.iplatform.microservices.emptyservice.service.dao.TestMapper.create",
   		"event_code": "sql_error",
   		"event_title": "SQL错误",
   		"event_result": true,
   		"error": "NULL not allowed for column \"CREATE_TIME\"; SQL statement:\ninsert into empty_test (is_enable,create_time) values (?,?) [23502-191]",
   		"sql": "insert into empty_test (is_enable,create_time) values (?,?)"
   	}
   }
   ```

   

5. SQL耗时

   ```json
   {
   	"message_id": "81a5ea05-0a36-4ed0-8271-bc10a8f04cc4",
   	"version": "0.1",
   	"time": "2018-09-22 18:24:53",
   	"type": "event",
   	"ms_id": null,
   	"ms_ins_id": null,
   	"ms_from": null,
	"host": "192.168.0.1",    
   	"data": {
   		"sqlid": "org.iplatform.microservices.emptyservice.service.dao.TestMapper.getAll",
   		"cost": "1",
   		"event_code": "sql_cost",
   		"event_title": "SQL耗时",
   		"event_result": true,
   		"sql": "select * from empty_test"
   	}
   }
   ```

   

6. HTTP错误

   ```json
   {
   	"message_id": "4f227c62-1e92-43b0-8df4-f5f3d2758d00",
       "version": "0.1",
       "time": "2018-09-25 12:09:21",
       "type": "event",
   	"ms_id": null,
       "ms_ins_id": null,
   	"ms_from": null,
	"host": "192.168.0.1",    
   	"data": {
   		"http_method": "GET",
   		"event_code": "http_error",
   		"event_title": "HTTP错误",
   		"event_result": true,
   		"http_error": "Request processing failed; nested exception is org.springframework.dao.DataIntegrityViolationException: \n### Error updating database.  Cause: org.h2.jdbc.JdbcSQLException: NULL not allowed for column \"CREATE_TIME\"; SQL statement:\ninsert into empty_test (is_enable,create_time) values (?,?) [23502-191]\n### The error may involve org.iplatform.microservices.emptyservice.service.dao.TestMapper.create-Inline\n### The error occurred while setting parameters\n### SQL: insert into empty_test (is_enable,create_time) values (?,?)\n### Cause: org.h2.jdbc.JdbcSQLException: NULL not allowed for column \"CREATE_TIME\"; SQL statement:\ninsert into empty_test (is_enable,create_time) values (?,?) [23502-191]\n; SQL []; NULL not allowed for column \"CREATE_TIME\"; SQL statement:\ninsert into empty_test (is_enable,create_time) values (?,?) [23502-191]; nested exception is org.h2.jdbc.JdbcSQLException: NULL not allowed for column \"CREATE_TIME\"; SQL statement:\ninsert into empty_test (is_enable,create_time) values (?,?) [23502-191]",
   		"http_path": "/api/v1/test/testmethodwithdb",
   		"http_remote_addr": "127.0.0.1",
   		"http_host": "0.0.0.0",
   		"http_url": "https://0.0.0.0:58080/emptyservice/api/v1/test/testmethodwithdb?access_token=809cd6021c5b98c872324609bdc5352d5c476085723993684c8a536de421b30c16fa9902042594d676ab6950e5f7320e87527d6600200c874a3261857b41f0007379af2d64d385d13bab3155326b767c3d103f979afe033eff6f1ae370d3864b",
   		"event_trace_id": "1a2f1eaadc672c83"
   	}
   }
   ```

   

7. 采样率变更

   ```json
   {
   	"message_id": "0d55174a-2df2-4209-8d9c-ca17c1b3d8bd",
   	"version": "0.1",
   	"time": "2018-09-25 13:25:33",
   	"type": "event",
   	"ms_id": "empty-service",
   	"ms_ins_id": "10.50.7.13::empty-service:58080",
   	"ms_from": "empty-service",
	"host": "192.168.0.1",    
   	"data": {
   		"event_code": "track_sampling_change",
   		"old_sampling": "0.5",
   		"event_title": "采样率变更",
   		"event_result": true,
   		"new_sampling": "1"
   	}
   }
   ```

   

8. 流控权重变更

   ```json
   {
   	"message_id": "086f7448-0778-475a-a5cd-3a63463caaf3",
   	"version": "0.1",
   	"time": "2018-09-25 13:26:48",
   	"type": "event",
   	"ms_id": "empty-service",
   	"ms_ins_id": "10.50.7.13::empty-service:58080",
   	"ms_from": "empty-service",
	"host": "192.168.0.1",    
   	"data": {
   		"event_code": "traffic_weight_change",
   		"event_title": "流控权重变更",
   		"event_result": true,
   		"new_weight": "1",
   		"old_weight": "0.5"
   	}
   }
   ```

   

9. 路由标签变更

   ```json
   {
   	"message_id": "46dc8f4f-5c6c-4358-aabb-0faaccb94b96",
   	"version": "0.1",
   	"time": "2018-09-25 14:57:51",
   	"type": "event",
   	"ms_id": "empty-service",
   	"ms_ins_id": "10.50.7.13::empty-service:58080",
   	"ms_from": "empty-service",
	"host": "192.168.0.1",    
   	"data": {
   		"event_code": "route_label_change",
   		"event_title": "路由标签变更",
   		"event_result": true,
   		"old_label": "",
   		"new_label": "shanghai", 
   		"label_type": "AND"
   	}
   }
   ```

   

10. 租户变更

    ```json
    {
    	"message_id": "82531fa2-b65e-4ed7-af57-61512ef49e5d",
    	"version": "0.1",
    	"time": "2018-09-25 14:54:44",
    	"type": "event",
    	"ms_id": "empty-service",
    	"ms_ins_id": "10.50.7.13::empty-service:58080",
    	"ms_from": "empty-service",
    	"host": "192.168.0.1",    
    	"data": {
    		"old_tenant": "",
    		"event_code": "teant_change",
    		"new_tenant": "zhanglei",
    		"event_title": "租户变更",
    		"event_result": true
    	}
    }
    ```

    

11. 服务熔断

    ```json
    {
    	"message_id": "d6d40055-a549-4195-b10a-68f47789dde7",
    	"version": "0.1",
    	"time": "2018-10-12 20:22:53",
    	"type": "event",
    	"ms_id": "empty-service",
    	"ms_ins_id": "10.50.7.9::empty-service:58080",
    	"ms_from": "empty-service",
    	"host": "192.168.0.1",    
    	"data": {
    		"event_code": "circuit_breaker",
    		"circuit_type": "SHORT_CIRCUITED",
    		"event_title": "服务熔断",
    		"circuit_path": "testmethod"
    	}
    }
    ```

    