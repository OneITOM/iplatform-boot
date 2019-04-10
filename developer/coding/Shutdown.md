# Shutdown规约

> 作者 张磊

当需要在程序正常停止前处理业务逻辑是，可以使用Runtime.getRuntime().addShutdownHook方式增加相关逻辑代码

** 注意此方法只适用于正常退出的程序，或者kill PID方式退出的程序

```java
Runtime.getRuntime().addShutdownHook(new Thread() {
	public void run() {
		LOG.info("服务退出前处理");
	}
});
```

 

 