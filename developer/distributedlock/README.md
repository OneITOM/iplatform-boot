# 分布式锁开发手册

> 作者 张磊

平台提供基于Redis，Mongodb，Database，Zookeeper多种分布式锁的实现方案，并通过注解以及内部方法协助研发人员处理分布式锁操作，以及浮动master节点的判断

## 1. @Scheduled

> 当使用@Scheduled方式实现进程内定式任务的时候，可以通过注解@SchedulerLock实现两种方式的分布式锁判断

* master模式

  > 当希望这个定式任务只在集群中的主节点上执行时使用，此时只有动态主节点会执行这个定时任务，集群中的其他节点不会执行，只有当主节点停止后，获取到主节点的其他节点会继续执行这个定时任务

  ```java
  @Scheduled(fixedDelay = 10000)
  @SchedulerLock(model=LockModel.iammaster)
  public void mySchedule(){
      System.out.println("这个定时任务只能我执行，除非进程退出")
  }
  ```

* 排他锁

  > 节点执行此定时任务时，其他节点此定时任务不执行，直到当前定时任务执行完毕，其他定时任务才可以抢占式执行

  ```java
  @Scheduled(fixedDelay = 10000)
  @SchedulerLock()
  public void mySchedule(){
      System.out.println("我在执行的时候其他进程相同定时任务不能执行，直到我本次执行结束")
  }
  ```

## 2. 代码判断是否主节点

> 我们还可以在代码中判断当前节点是否主节点，从而决定是否执行业务逻辑代码

```java
@Autowired(required=false)
LockService lockService;

public void doSometing(){
    if(lockService!=null && lockService.isMaster()){
        System.out.println("当前节点是主节点");
    }
}
```

## 3. 分布式锁实现方案

> 分布式锁适配多种集中是中间件的实现方案，最大化的简化使用者的应用复杂度，相关服务可以根据自身情况选择以下方案中的任一一种

### 3.1 数据库方案

> 数据库方案需要在数据库中建立一个表

```sql
CREATE TABLE IPLATFORM_LOCK (
    name VARCHAR(64) PRIMARY KEY, 
    lock_until TIMESTAMP(3) NULL, 
    locked_at TIMESTAMP(3) NULL, 
    locked_by  VARCHAR(255)
);
```

参数配置（前提已经配置了数据库参数）

```properties
iplatform.scheduled.lock.enabled=true
iplatform.scheduled.lock.type=jdbc
```

### 3.2 Redis方案

> 基于Redis实现分布式锁

参数配置（前提已经配置了Redis连接参数）

```properties
iplatform.scheduled.lock.enabled=true
iplatform.scheduled.lock.type=redis
```

### 3.3 Mongodb方案

> 基于Mongodb实现分布式锁

参数配置（前提已经配置了Mongodb连接参数）

```properties
iplatform.scheduled.lock.enabled=true
iplatform.scheduled.lock.type=mongo
```

### 3.4 Zookeeper方案

> 基于Zookeeper实现分布式锁，需要在自定义一个@Bean

由于springboot没有为zookeeper配置自动的注入类，所以需要在代码中自己写上这个bean，参数各位根据自身服务中的配置填写

```java
@Bean
public ZookeeperConfig zookeeperConfig(){
    return new ZookeeperConfig("127.0.0.1:2181",10000,10000);
}
```

参数配置

```properties
iplatform.scheduled.lock.enabled=true
iplatform.scheduled.lock.type=zookeeper
```





