# 异步处理规约

> 作者 张磊

## 1. @Service上的异步方法

> 使用@Async注解声明一个方法为异步方法

### 1.1 开启异步调用(框架默认以开启)

```java
@EnableAsync
```

### 1.2 无返回值

```java
@Async
public void asyncMethod() {
    log.info("我是异步执行的");
}
```

### 1.3 带返回值

```java
@Async
public Future<String> asyncMethod() {
    log.info("异步调用开始");
    Future<String> future;
    try {
        //业务逻辑
        future = new AsyncResult<String>("success");
    } catch (InterruptedException e) {
        future = new AsyncResult<String>("error");
    }
    return future;
}
```

 