# 通知服务集成手册

> 作者 王立松

通知服务负责接收ActiveMQ消息发送到短信网关和邮件服务器，其底层基于flume，发送短信和邮件均由相应的sink组件实现，所以进行通知服务集成时，只需要开发相应的sink即可

## 1. 依赖

- JDK1.8+
- ActiveMQ
- 发现服务
- 文档服务
- 通知服务主程序notify-service
- 通知服务工具包notify-util
- 邮件插件notify-plugin-email

## 2. 邮件集成

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

## 3. 短信集成

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



## 4. flume配置

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

