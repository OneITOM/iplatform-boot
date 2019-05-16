# 自动化测试

## 集成测试

> 集成测试可以提高研发人员测试效率，理解产品处理逻辑，需要对一下工具包的用法有一个基本了解
>
> [Mockito](https://github.com/mockito/mockito)
>
> [Awaitility](https://github.com/awaitility/awaitility)
>
> [Hamcrest](https://github.com/hamcrest/JavaHamcrest)
>
> [Powermock](https://github.com/powermock/powermock)
>
> 以下说明样例来自于文档服务

增加依赖库

```xml
<dependency>
  <groupId>org.iplatform</groupId>
  <artifactId>iplatform-test</artifactId>
  <scope>test</scope>
</dependency>
<dependency>
  <groupId>com.jayway.jsonpath</groupId>
  <artifactId>json-path</artifactId>
  <scope>test</scope>
</dependency>
<dependency>
  <groupId>org.awaitility</groupId>
  <artifactId>awaitility</artifactId>
  <scope>test</scope>
</dependency>
```

新建测试类 `DfssServiceApplicationIntegrationTest.java`

增加注解 `@RunWith(SpringJUnit4ClassRunner.class)`

增加测试框架注解 `@IPlatformApplicationTest` classes 属性设置本应用的主启动类（例如文档服务的 DfssServiceApplication.class）, properties 属性设置服务参数

```java
package org.iplatform.microservices.support.dfssservice.service;

import org.iplatform.microservices.core.test.IPlatformApplicationTest;
import org.iplatform.microservices.support.dfssservice.DfssServiceApplication;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;
import org.springframework.web.context.WebApplicationContext;

@RunWith(SpringJUnit4ClassRunner.class)
@IPlatformApplicationTest(classes={DfssServiceApplication.class},properties= {
    "server.port:0",
    "eureka.client.enabled=false",
    "spring.thymeleaf.mode=LEGACYHTML5",
    "dfss.mongodb.uri=mongodb://dfss-user:dfss-password@127.0.0.1:27017/dfss"
})

public class DfssServiceApplicationIntegrationTest{
  @Autowired
  private WebApplicationContext wac;
  private MockMvc mockMvc;

  @Before
  public void before() throws Exception {
    this.mockMvc = MockMvcBuilders.webAppContextSetup(this.wac).build();
  }
}
```

增加首页页面请求测试

```java
@Test
@SneakyThrows
public void indexTest(){
  this.mockMvc.perform(get("/index")).andDo(print()).andExpect(status().isOk());
}
```

增加创建仓库测试

```java
public final String bucketName = "my";
public final String busiSysHeaderName = "X-Config-Busisys";
public final String busiSysHeaderValue = "OneITOM";

@Test
@SneakyThrows
public void createBucketTest(){
  this.mockMvc.perform(post("/api/v1/createBucket")
      .header(busiSysHeaderName,busiSysHeaderValue)
      .param("bucketName",bucketName))
      .andDo(print())
      .andExpect(status().isOk())
      .andExpect(jsonPath("$.success").value(true))
      .andReturn();
}
```

增加删除不存在的文件异常测试

```java
@Test(expected = DfssException.class)
@SneakyThrows
public void deleteNoFileIdExceptionTest(){
  gridFsService.delete(busiSysHeaderValue,bucketName,null);
}
```

更多用法可以参考文档服务的集成测试用例

### 依赖服务模拟

* 数据库模拟

  建议集成测试环境配置内存数据库H2

  ```java
  @RunWith(SpringJUnit4ClassRunner.class)
  @IPlatformApplicationTest(classes = {NotifyServiceApplication.class}, properties = {
      "eureka.client.enabled=false",
      "spring.datasource.platform=H2",
      "spring.datasource.dataSourceClassName=org.h2.Driver",
      "spring.datasource.url=jdbc:h2:mem:notifydb;DB_CLOSE_ON_EXIT=FALSE",
      "spring.datasource.username=sa",
      "spring.datasource.password=",
      ...
  })
  ```

* ActiveMQ模拟

  基于Mock接管 JmsTemplate

  ```java
  @RunWith(SpringJUnit4ClassRunner.class)
  @IPlatformApplicationTest(classes = {NotifyServiceApplication.class,ActiveMQConfigTest.class}, properties = {
      "eureka.client.enabled=false",
      "spring.activemq.broker-url=tcp://localhost:61616",
      ...
  })
  
  @Slf4j
  @Configuration
  public class NotifyServiceApplicationIntegrationTest {
    @Autowired
    @Qualifier("queueJmsTemplate")
    private JmsTemplate queueJmsTemplate;
  
    @Configuration
    public static class ActiveMQConfigTest {
      @Bean(name = "queueJmsTemplate")
      public JmsTemplate queueJmsTemplate() {
        return Mockito.mock(JmsTemplate.class);
      }
    }
    
    @Before
    public void before() throws Exception {
      this.mockMvc = MockMvcBuilders.webAppContextSetup(this.wac).build();
      // mock queueJmsTemplate
      doNothing().when(queueJmsTemplate).convertAndSend(any(Queue.class), anyString());
      doNothing().when(queueJmsTemplate).convertAndSend(any(Topic.class), anyString());
    }  
  }
  ```

* 方法返回值模拟

  例如MyService类有一个getSomething()方法定义如下

  ```javaq
  public String getSomething(String arg){
    xxx
    return xxx;
  }
  ```

  首先自动装配此对象

  ```java
  @Autowired
  MyService myService;
  ```

  在测试方法前模拟此方法的返回值

  ```java
  when(myService.get(any())).thenReturn("模拟的返回值")
  ```

  这样在调用此方法的时候就会按照上边的方法返回"模拟的返回值"

* 方法异步返回值测试

  有的时候在调用某个方法的时候需要等待一段时间才能有正确的返回值，例如方法B要在5秒内返回期望的值，我们还以MyService举例

  ```java
  Awaitility.await().atMost(5, SECONDS).until(() -> {
  	String val = myService.getSomething(xxx);
    return val.equal("模拟的返回值");
  });
  ```

* 方法异常模拟

  有的时候我们希望模拟某个方法调用异常，可以使用此方法

  ```java
  doThrow(RuntimeException.class).when(myService).getSomething(any());
  ```

  

## 验收测试

验收测试是基于Docker在准生产环境下，使用 [cucumber](https://github.com/cucumber/cucumber) 基于描述的支持BDD的自动化测试方法