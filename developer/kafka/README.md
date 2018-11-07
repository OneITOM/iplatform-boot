# Kafka开发手册

> 作者 王立松

框架内部已经集成了Kafka，研发人员可通过配置参数激活Kafka实现消息的发布和订阅

## 激活Kafka

> 通过配置参数激活Kafka

```yml
# kafka开关，必填
spring.kafka.enabled: true

# kafka地址，集群模式时多个地址用逗号分隔，必填
spring.kafka.broker.address: localhost:9092

# 消费者群组ID，默认值 iplatformboot
spring.kafka.groupId: iplatformboot

# 消费监听器容器并发数，默认值 1
spring.kafka.concurrency: 1
```

## 消息发送

> 消息发送基于KafkaTemplate实现，示例如下，其中test为topic

```java
@Autowired
KafkaTemplate<String, String> kafkaTemplate;

private void testKafkaProduce(){
	kafkaTemplate.send("test", "hello kafka");
}
```

## 消息接收

> 消息接收基于KafkaListener实现，示例如下

```java
@KafkaListener(topics = {"test"},containerFactory = "kafkaListenerContainerFactory")
private void testKafkaConsume(ConsumerRecord<String, String> consumer){
	System.out.println(consumer.topic()+":"+consumer.value());
}
```
