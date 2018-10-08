# 消息总线集成手册

> 作者 张磊
>
> 框架提供跨进程消息总线集成能力，基于集中配置的ActiveMQ，并提供开发脚手架，便于研发人员开发出跨进程的消息通信功能

## 1. 前提

* 已经启动了注册中心（并开启集中配置功能）
* 本服务开启了集中配置参数
  * spring.cloud.config.enabled=true
  * spring.cloud.config.profile=XXX

## 2. 验证绑定

> 本服务启动时可以通过日志查看是否已经绑定了消息总线服务

```shell
o.i.m.core.messagebus.MessageBusService.initActiveQM : 框架消息总线初始化完成
```

## 3. 发送消息

1. 自动注入消息总线服务

    ```java
    @Autowired(required = false)
    MessageBusService messageBusService;
    ```


2. 使用消息总线服务发送消息

    ```java
    if(messageBusService!=null && messageBusService.isConnected){
      messageBusService.getQueueJmsTemplate().send("topic或者queue名称", new MessageCreator() {
          @Override
          public Message createMessage(Session session) throws JMSException {
              return session.createTextMessage(json);
          }
      });      
    }
    ```

## 4. 接收消息

1. 服务类继承抽象类AbstractMessageBusListener

   ```java
   @RestController
   public class TestService extends AbstractMessageBusListener {
   	
   }
   ```

2. 实现抽象类的监听方法

   ```java
   @MessageBusConsumer(destination = "topic或者queue名称")
   public void onMessageBus(Message message){
   	if(message instanceof TextMessage){
   		try {
   			System.out.println(((TextMessage)message).getText());
   		} catch (JMSException e) {
   			LOG.error("消息接收错误",e)
   		}
   	}
   } 
   ```

   