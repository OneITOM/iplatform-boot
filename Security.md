# 安全相关配置手册

> 作者 张磊

本文档主说明项目安全保护相关的解决方案，以及已知漏洞的解决方法

- 安全保护
  - [配置文件加密](#user-content-1.1)
  - [RESTful API 限速](#user-content-1.2)
- [安全漏洞](#user-content-2)
  - [SQL注入漏洞](#user-content-2.1)
  - [CSRF跨站请求伪造](#user-content-2.2)
  - [点击劫持](#user-content-2.2)
  - [不安全的HTTP方法](#user-content-2.4)
  - [SSL/TLS受诫礼(BAR-MITZVAH)攻击漏洞](#user-content-2.5)
  - [Slow HTTP Denial of Service Attack漏洞](#user-content-2.6)



## <a id="1.1"></a>配置文件加密
> 配置文件中或者启动参数的参数值可以使用加密方式配置

例如以下是未加密的数据库密码配置参数

```properties
spring.datasource.password=123456
```

可以使用密码生成工具生成明文对应的密文后配置成密文格式

```shell
java -jar enctool-0.0.1-SNAPSHOT.jar "123456" 
======================================================================  
微服务配置文件密文生成工具  java -jar enctool-0.0.1-SNAPSHOT.jar 要加密的明文 
----------------------------------------------------------------------         
Copyright© BOCO  
====================================================================== 
明文:123456 
密文:ENC(mBaGBXPu1VFgECoBH5NGWeTdFLy79Ic5)
```

使用生成的密文配置数据库密码的加密配置方式

```properties
spring.datasource.password=ENC(mBaGBXPu1VFgECoBH5NGWeTdFLy79Ic5)
```

## <a id="1.2"></a>RESTful API 限速

> 可以通过@RateLimit注解实现限速保护，限速粒度是客户端IP+URL，当超过调用阀值设置后，这个IP对于相同的URL的调用将不再被受理，窗口期结束后自动恢复。
>
> 另外可以通过实现org.iplatform.microservices.core.limit.controller.provide.IRateLimitProvide接口，然后自动注入的方式进行自定义限制。重写方法putLimit实现增加限制，removeLimit删除限制，get方法实现自定义限制处理(如通过request对象获取IP、接口、用户等信息作为限制条件),样例见下方自定义限制实现，通过接口+IP限制访问频率。

limit 调用次数阀值

duration 窗口期时长（也表示冷却期，当超过调用阀值后在duration定义的时间范围内将不能再调用）

unit 窗口期时长单位

onlyLimitThrow 只对异常调用进行限制

onlyLimitThrowClass 异常类定义，不定义则对所有异常进行阀值计数

rateLimitType 限制类型，包括RateLimitType.COUNT和RateLimitType.WINDOW，默认为RateLimitType.WINDOW，COUNT为时间范围计数，WINDOW为滑动时间窗口计数

* 1秒限制调用2次

  ```java
  @RateLimit(limit = 2, duration = 1, unit = TimeUnit.SECONDS)
  ```

* 1分钟限制调用60次

  ```java
  @RateLimit(limit = 60, duration = 1, unit = TimeUnit.MINUTES)
  ```

* 1分钟出现10次AuthException异常后进行访问限制

  ```java
  @RateLimit(limit = 10, duration = 1, unit = TimeUnit.MINUTES, onlyLimitThrow=Boolean.TRUE, onlyLimitThrowClass = {AuthException.class})
  ```

- 自定义限制实现

  ```java
  @Component
  public class RateLimitProvide implements IRateLimitProvide {
  
    private Map<String, RateLimitProvideBean> limitMap = new ConcurrentHashMap<>();
  
    @Override
    public RateLimitProvideBean get(String key) {
      return limitMap.get(key);
    }
  
    @Override
    public RateLimitProvideBean get(HttpServletRequest request) {
      String ip = request.getHeader("X-FORWARDED-FOR");
      if (ip == null || ip.trim().length() == 0) {
        ip = request.getRemoteHost();
      }
      String url = request.getRequestURI();
      String key = String.format("req:limit:%s:%s", url, ip);
      return limitMap.get(key);
    }
  
    @Override
    public void putLimit(String key, int limit, long duration, TimeUnit unit) {
      RateLimitProvideBean bean = new RateLimitProvideBean();
      bean.setKey(key);
      bean.setLimit(limit);
      bean.setUnit(unit);
      bean.setDuration(duration);
      limitMap.put(key, bean);
    }
  
    @Override
    public void removeLimit(String key) {
      limitMap.remove(key);
    }
  }
  ```

## <a id="3"></a>安全漏洞

安全漏洞主要是目前已知的一些漏洞的配置或者开发注意事项，大部分漏洞在框架级都有默认配置

### <a id="3.1"></a>SQL注入漏洞

> 本框架使用的是Mybatis，所以SQL注入漏洞主要是针对此框架

* 在SQL中尽量使用#，避免使用$
* 如果必须使用$，那么可以通过在Mapper方法上使用@SQLInjection实现检查实现检查和策略

@SQLInjection参数说明

1. value 敏感词定义，可以不填写，默认敏感词如下

```java
String[] value() default {";","'","\"","\\(","\\)","and","or","union","where","limit","select","delete","substr","group","by"};
```

2. ignore 敏感词忽略，默认空，用于要排除的默认敏感词定义

例如要忽略select敏感词，那么使用如下配置

```java
@SQLInjection(ignore = {"select"})
```
3. regex 正则敏感词定义，默认空，使用正则表达式定义敏感词，次参数不可和value, ingore同时使用

例如：要检查参数中是否包含 select 和 >

```java
@SQLInjection(regex = "(select)|(>)")
```

4. policy 检查到疑似注入后的处理策略，默认策略是打印警告日志，可以通过配置终止策略SQLInjectionPolicy.BREAK达到紧致此SQL中断执行的目的

例如：以下策略将在发现疑似注入漏洞发生时中断SQL执行

```java
@SQLInjection(policy = SQLInjectionPolicy.BREAK)
```
### <a id="3.2"></a>CSRF跨站请求伪造

> 漏洞描述：发送请求的时候认为修改header中的Host或者referer信息，并在返回的消息头中看到这个修改后的Host，Referer信息

* 在启动参数中打开防篡改开关

```properties
iplatform.tamperproofing.headerhost.enabled=true
```

* 设置允许篡改白名单

  当系统通过域名或者ip跳转访问时，系统会自动设置浏览器中的ip和端口到header的host中，所以如果要通过防篡改验证，则需要手工设置白名单

```properties
iplatform.tamperproofing.headerhost.whitelist=www.mysite.com:8761,www.mysite.org:8761
```

### <a id="3.3"></a>点击劫持 X-Frame-Options DENY

> 目前框架中默认设置允许页面被第三方iframe嵌入，如果要禁止页面被第三方页面嵌入需要扩展HttpSecurity的configure方法，自定义策略

```java
//设置只允许相同域名的iframe嵌入
http.headers().frameOptions().sameOrigin();
//设置禁止被iframe嵌入    
http.headers().frameOptions().deny();
//不增加设置(默认)
http.headers().frameOptions().disable()  
```

### <a id="3.4"></a>不安全的HTTP方法

> 检查原始测试响应的“Allow”头，并验证是否包含下列一个或多个不需要的选项：DELTE，SEARCE，COPY，MOVE，PROPFIND，PROPPATCH，MKCOL，LOCK，UNLOCK，PUT

* 检验方法如下，使用curl命令查看返回的Allow内容

```bash
curl -v -X OPTIONS http://127.0.0.1:5000
Allow: HEAD, GET, OPTIONS
```

* 通过参数配置要禁用哪些方法

```properties
server.tomcat.disabled.methods=HEAD,OPTIONS,TRACE
```

### <a id="3.5"></a>SSL/TLS受诫礼(BAR-MITZVAH)攻击漏洞

> 这种攻击利用了一个RC4加密算法中的漏洞窃取通过SSL和TLS协议传输的机密数据。

解决办法就是禁用RC4算法，我们可以通过参数配置SSL/TLS使用的版本和协议（以下是默认启动的TLS版本和算法）

```properties
server.ssl.ciphers=TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_AES_128_SHA256,TLS_ECDHE_ECDSA_WITH_AES_128_SHA256,TLS_ECDHE_RSA_WITH_AES_128_SHA,TLS_ECDHE_ECDSA_WITH_AES_128_SHA,TLS_ECDHE_RSA_WITH_AES_256_SHA384,TLS_ECDHE_ECDSA_WITH_AES_256_SHA384,TLS_ECDHE_RSA_WITH_AES_256_SHA,TLS_ECDHE_ECDSA_WITH_AES_256_SHA
server.ssl.protocol=TLS
server.ssl.enabled-protocols=TLSv1.2
```

注意：以上算法需要JDK版本支持

### <a id="3.7"></a>Slow HTTP Denial of Service Attack漏洞

> 利用的HTTP POST：POST的时候，指定一个非常大的content-length，然后以很低的速度发包，比如10-100s发一个字节，hold住这个连接不断开。这样当客户端连接多了后，占用住了webserver的所有可用连接，从而导致DDOS。

* 漏洞自检

  使用漏洞检测工具[slowhttptest](<https://github.com/shekyan/slowhttptest>)检测

  ```bash
  slowhttptest -H -c 500 -l 200 -k 5 -p 30 -g -u https://127.0.0.1:8080/
  ```

  如果执行报告中没有closed，都是connected说明链接都有效，并没有出现自动关闭的链接，说明慢速攻击已经hold住了所有的链接，此时这个服务无法再对外提供服务

* 缓解办法

  这种攻击作为程序提供放只能通过减小连接超时时间来达到自动关闭超时连接的方法进行连接释放，可以通过以下参数设置超时时间，例如设置成10秒，但是这种方式无法从根本上避免这种DDOS

  ```properties
  server.connection-timeout=10000
  ```

  