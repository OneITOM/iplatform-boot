# ActiveMQ开发手册

> 作者 王立松

框架内部已经集成了ActiveMQ，研发人员可通过配置启用MQ从而实现消息的发送和接收

## 1. 启用MQ

> 通过配置参数启用MQ

```yml
# 指定broker的url
spring.activemq.broker-url: tcp://localhost:61616

# 指定broker的用户
spring.activemq.user: admin

# 指定broker的密码
spring.activemq.password: admin
```



## 2. 消息定义

> 定义queue和topic

```java
@Bean
public Queue Q_MS_TEST() {
    return new ActiveMQQueue("Q_MS_TEST");
}

@Bean
public Topic T_MS_TEST() {
    return new ActiveMQTopic("T_MS_TEST");
}
```



## 3. 消息发送

> 消息发送基于JmsTemplate实现，示例如下

```java
@Autowired
JmsTemplate jmsTemplate;

@Autowired
Queue Q_MS_TEST;

@Autowired
Topic T_MS_TEST;

private void testActiveMQ(){
		
    //发送queue消息
    jmsTemplate.convertAndSend(Q_MS_TEST, "hello queue");

    //发送topic消息
    jmsTemplate.convertAndSend(T_MS_TEST, "hello topic");

}
```



## 4. 消息接收

> 消息接收基于JmsListener实现，示例分别展示了queue和topic的接收方式

```java
@ApiOperation("Q_MS_TEST消息接收")
@JmsListener(destination = "Q_MS_TEST", containerFactory = "queueListenerContainerFactory")
private void receiveQueueMessage(String message){

	System.out.println("Q_MS_TEST:"+message);

}

@ApiOperation("T_MS_TEST消息接收")
@JmsListener(destination = "T_MS_TEST", containerFactory = "topicListenerContainerFactory")
private void receiveTopicMessage(String message){

	System.out.println("T_MS_TEST:"+message);

}
```
