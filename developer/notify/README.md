# 通知服务集成手册

> 作者 王立松

通知服务负责接收ActiveMQ消息发送短信和邮件，其底层基于flume，短信和邮件的发送均由相应的sink组件实现

## 消息发送

### 1. 前提

- [已启动ActiveMQ](../../middleware/ActiveMQ.md)
- [已启动通知服务](../../iplatform-common/NotifyService.md)
- [已启动文档服务](../../iplatform-common/DfssService.md)

### 2. 说明

> 队列主题说明

| 队列（queue）/主题（topic） | 名称           | 描述                                   |
| :-------------------------- | :------------- | :------------------------------------- |
| 队列                        | Q_NOTIFY_REQ   | 通知服务监听该队列中的消息进行适配转发 |
| 主题                        | T_NOTIFY_RSP   | 通知服务转发完成后立即发送响应消息     |
| 主题                        | T_NOTIFY_REPLY | 通知服务发送短信网关回复消息           |

> 发送消息说明

| ID                 | 名称             | 必填 | 描述                                     |
| :----------------- | :--------------- | :--- | :--------------------------------------- |
| uuid               | 消息唯一ID       | 是   |                                          |
| req_platform_type  | 平台类型         | 是   | 标识消息来源                             |
| message_type       | 消息类型         | 是   | sms 短信、vioce 语音、email 邮件         |
| message_title      | 消息标题         | 是   |                                          |
| message_content    | 消息正文         | 是   |                                          |
| message_attachment | 邮件附件         | 否   | 文档服务返回的文件ID，多个用英文逗号分隔 |
| create_time        | 消息生成时间     | 是   |                                          |
| validity_days      | 消息有效期       | 是   | 默认3天                                  |
| email_recipients   | 收件人地址       | 否   | 多个用英文逗号分隔，邮件发送时必填       |
| require_reply      | 短信是否需要回复 | 是   | 1 是、0 否                               |
| phone_num          | 手机号           | 否   | 接收短信的手机号，短信发送时必填         |

> 响应消息说明

| ID                | 名称       | 必填 | 描述                  |
| :---------------- | :--------- | :--- | :-------------------- |
| uuid              | 消息唯一ID | 是   |                       |
| req_platform_type | 平台类型   | 是   |                       |
| is_sucessed       | 是否成功   | 是   | true 成功、false 失败 |
| message           | 响应内容   | 否   |                       |
| reply_content     | 回复内容   | 否   | 短信网关回复时必填    |

### 3. 邮件发送

> 发送消息格式

```json
{
  "uuid" : "5ca9e0be-f492-4744-afd3-937bb0230c73",
  "req_platform_type" : "test",
  "message_type" : "email",
  "message_title" : "邮件标题",
  "message_content" : "邮件正文",
  "message_attachment" : "",
  "create_time" : "2018-10-30 10:28:39",
  "validity_days" : 3,
  "email_recipients" : "xxx@126.com"
}

```

> 响应消息格式

```json
{
  "uuid" : "5ca9e0be-f492-4744-afd3-937bb0230c73",
  "req_platform_type" : "test",
  "is_sucessed" : true,
  "message" : ""
}
```

### 4. 短信发送

> 发送消息格式

```json
{
  "uuid" : "a60f79de-13b2-4331-9b7b-50cd3feaee46",
  "req_platform_type" : "test",
  "message_type" : "sms",
  "message_title" : "短信标题",
  "message_content" : "短信正文",
  "create_time" : "2018-10-30 10:28:39",
  "require_reply" : 0,
  "validity_days" : 3,
  "phone_num" : "18610000000"
}

```

> 响应消息格式

```json
{
  "uuid" : "a60f79de-13b2-4331-9b7b-50cd3feaee46",
  "req_platform_type" : "test",
  "is_sucessed" : true,
  "message" : ""
}
```

> 回复消息格式

```json
{
  "uuid" : "a60f79de-13b2-4331-9b7b-50cd3feaee46",
  "reply_content " : "回复内容"
}
```

## 插件集成

> 插件集成只需开发一个自己的sink，通过该sink实现消息的发送

### 1. 依赖

- JDK1.8+
- ActiveMQ
- 发现服务
- 文档服务
- 通知服务主程序notify-service
- 通知服务工具包notify-util
- 邮件插件notify-plugin-email

### 2. 邮件集成

> 邮件插件notify-plugin-email已集成，需要通过配置参数启用邮件发送功能

1. 邮件服务器配置

   ```properties
   notify.email.host=smtp.163.com 
   notify.email.port=25 
   notify.email.from=xxxxxxx@163.com 
   notify.email.password=ENC(SPI49CzmaN1X8C8fuim1wPE9t2DU6iBC) 
   ```

2. 文档服务配置

   ```properties
   spring.dfss.bucketname=test
   spring.dfss.secretkey=test
   ```

### 3. 短信集成

1. 创建maven项目

   ```java
   notify-plugin-sms
       src/main/java
       -- org.iplatform.microservices.support.notifyservice.plugin.sms
       ---- MySmsSink.java
       src/main/resources
       pom.xml
   ```

2. 调整maven依赖

   ```xml
   	<parent>
   		<groupId>org.iplatform</groupId>
   		<artifactId>notify-parent</artifactId>
   		<version>0.0.1</version>
   		<relativePath>../</relativePath>
   	</parent>
   	
   	<dependencies>	
           <dependency>
               <groupId>org.iplatform</groupId>
               <artifactId>notify-util</artifactId>
               <version>0.0.1</version>
           </dependency>
   	</dependencies>
   ```

3. MySmsSink开发

   > MySmsSink继承NotifySink，负责与短信网关交互

   ```java
   package org.iplatform.microservices.support.notifyservice.plugin.sms;
   
   import org.apache.flume.Context;
   import org.apache.flume.Event;
   import org.codehaus.jackson.map.ObjectMapper;
   import org.slf4j.Logger;
   import org.slf4j.LoggerFactory;
   
   public class MySmsSink extends NotifySink {
   	
   	private static final Logger log = LoggerFactory.getLogger(MySmsSink.class);
   	private static CmppCfg cfg = null;
   	private ObjectMapper mapper;
   
   	@Override
   	public void doPluginProcess(Event event) throws Exception {
   		try{
   			NotifySmsEvent smsEvent = mapper.readValue(event.getBody(), NotifySmsEvent.class);
   			String smsNum = smsEvent.getPhone_num();
   	        String smsText = smsEvent.getMessage_content();
   	        if(SmsProxy.connect(cfg)){
   	        	SmsProxy.sendLongMessage(smsNum, smsText, smsEvent.getUuid());
   	        }
   		}catch(Exception e){
   			log.error("发送短信异常", e);
   			throw new NotifyException(e.getMessage());
   		}
   	}
   
   	@Override
   	public void doPluginConfigure(Context context) {
   		mapper = new ObjectMapper();
   		if(cfg == null){
   			cfg = new CmppCfg();
   			cfg.setHost(context.getString("CMPPConnect.host"));
   			cfg.setPort(context.getString("CMPPConnect.port"));
   			cfg.setUser(context.getString("CMPPConnect.user"));
   			cfg.setPass(context.getString("CMPPConnect.pass"));
   			cfg.setIcpid(context.getString("CMPPConnect.icpid"));
   			cfg.setIcpcode(context.getString("CMPPConnect.icpcode"));
   		}
   	}
   
   	@Override
   	public void doPluginStart() {
   		
   	}
   
   	@Override
   	public void doPluginStop() {
   		
   	}
   
   }
   
   ```



### 4. flume配置

```properties
# 开启flume
flume.enabled=true

flume.agent.sources=s
flume.agent.sinks="smsk emailk"
flume.agent.channels="smsc emailc"

flume.agent.sources.s.type=org.iplatform.microservices.support.notifyservice.flume.source.NotifySource
flume.agent.sources.s.channels="smsc emailc"
flume.agent.sources.s.selector.type=multiplexing
flume.agent.sources.s.selector.mapping.email=emailc
flume.agent.sources.s.selector.mapping.sms=smsc
flume.agent.sources.s.selector.header=notify_type

flume.agent.channels.emailc.type=memory
flume.agent.channels.emailc.capacity=500000
flume.agent.channels.emailc.transactionCapacity=20000
flume.agent.channels.smsc.type=memory
flume.agent.channels.smsc.capacity=500000
flume.agent.channels.smsc.transactionCapacity=20000

flume.agent.sinks.emailk.type=org.iplatform.microservices.support.notifyservice.plugin.email.EmailSink
flume.agent.sinks.emailk.channel=emailc
flume.agent.sinks.smsk.type=org.iplatform.microservices.support.notifyservice.plugin.sms.MySmsSink
flume.agent.sinks.smsk.channel=smsc

# 短信网关配置
flume.agent.sinks.smsk.CMPPConnect.host=127.0.0.1
flume.agent.sinks.smsk.CMPPConnect.port=6789
flume.agent.sinks.smsk.CMPPConnect.user=917111
flume.agent.sinks.smsk.CMPPConnect.pass=test1234
flume.agent.sinks.smsk.CMPPConnect.icpid=917111
flume.agent.sinks.smsk.CMPPConnect.icpcode=106584
```
