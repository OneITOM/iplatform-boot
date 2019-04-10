# 服务控制集成

> 作者 张磊

被管服务实现以下接口提供被管能力

## 远程停止服务

> 远程停止服务由服务方按照RESTfulAPI接口提供，被微服务管控平台调用实现进程的停止操作

服务端实现示例

```java
@RequestMapping(value = "/shutdown", method = { RequestMethod.POST})
public ResponseBody shutdown(HttpServletRequest request) {
	// timestamp可以用来和当前时间进行比较，超过一定期限认为请求过期，过期返回 http status 408
	String timestamp = request.getHeader("end_point_timestamp");
    
   	// clientip 可以实现白名单过滤，避免未授权的客户端调用，未授权返回 http status 401
	String clientip = request.getRemoteAddr();
    
    // 未开启此功能返回 http status 404
    
    // 其他错误返回 http status 406
    
	Map<String,String> responseBody = new HashMap();
    try{
        //返回 http status 200 
        responseBody.put("message","Shutting down, bye...")
        return responseBody;
    }finally{
		new Thread(new Runnable() {
			@Override
			public void run() {
				try {
					Thread.sleep(500L);
				}catch (InterruptedException ex) {
                }
				System.exit(0);
			}
		}).start();
    }  
}
```

模拟微服务管控平台调用

```shell
curl -X POST -H 'end_point_timestamp':'1535007763000' https://{ip}:{port}/{contextpath}/shutdown
```



