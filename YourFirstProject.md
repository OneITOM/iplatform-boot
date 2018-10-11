# 项目构建手册

> 作者 王立松

通过本文档可以快速开发一个属于你的SERVICE、UI服务，你可以看到服务是如何注册到注册中心，并通过认证服务登录后完成一个UI到SERVICE调用的样例

## 1. 准备

- 已安装发现服务
- 已安装认证服务
- JDK1.8+
- MAVEN3.5+

## 2. 项目构建

- 项目信息

  ```properties
  groupId=org.iplatform.myproject
  artifactId=myproject
  package=org.iplatform.myproject
  version=0.0.1-SNAPSHOT
  ```

- 创建项目

  ```properties
  mvn org.apache.maven.plugins:maven-archetype-plugin:2.4:generate \
  -DgroupId=org.iplatform.myproject \
  -DartifactId=myproject \
  -Dpackage=org.iplatform.myproject \
  -Dversion=0.0.1-SNAPSHOT \
  -DarchetypeCatalog=http://127.0.0.1/nexus/content/repositories/releases \
  -DarchetypeGroupId=org.iplatform.archetypes \
  -DarchetypeArtifactId=iplatform-all-archetype \
  -DarchetypeVersion=0.0.7 \
  -DinteractiveMode=false
  ```

- 目录结构说明

  | 目录结构                      | 说明                     |
  | ----------------------------- | ------------------------ |
  | myproject/config/local/run.sh | 启停脚本                 |
  | myproject/service             | SERVICE服务              |
  | myproject/ui                  | UI服务                   |
  | myproject/util                | UI和SEERVICE公用的工具包 |

## 3. UTIL

> 修改myproject/util/pom.xml，自定义项目名称myproject-util

```xml
<modelVersion>4.0.0</modelVersion>
<groupId>org.iplatform.myproject</groupId>
<artifactId>myproject-util</artifactId>
<version>0.0.1-SNAPSHOT</version>
<parent>
    <groupId>org.iplatform.myproject</groupId>
    <artifactId>myproject</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <relativePath>../</relativePath>
</parent>
```

## 4. SERVICE

### 修改

1. 修改启停脚本run.sh，配置服务发现地址、服务IP、服务端口等

   ```properties
   nohup java -jar ${PRONAMESERVICE} \
       --discovery.server.address="https://127.0.0.1:8761/eureka/" \
       --server.host=127.0.0.1 \
       --server.port=8081 \
       --spring.profiles.active=prod >/dev/null 2>&1 &
   ```

2. 修改myproject/service/pom.xml，自定义项目名称myproject-service，调整引用myproject-util

   ```xml
   <modelVersion>4.0.0</modelVersion>
   <groupId>org.iplatform.myproject</groupId>
   <artifactId>myproject-service</artifactId>
   <version>0.0.1-SNAPSHOT</version>
   <name>myproject-service</name>
   <description>myproject-service</description>
   <packaging>jar</packaging>
   <dependencies>
       <dependency>
           <groupId>org.iplatform.myproject</groupId>
           <artifactId>myproject-util</artifactId>
           <version>0.0.1-SNAPSHOT</version>
       </dependency>
   </dependencies>
   ```

3. 修改myproject/service/src/main/resources/bootstrap.yml，自定义应用名称myproject-service

   ```yaml
   info:
     app:
       name: 后台服务
       description: 后台服务
       version: 0.0.1
   spring:
     application:
       name: myproject-service
   ```

### 打包

> myproject目录下打包

1. 打包命令

   ```shell
   mvn clean package
   ```

2. 生成目录

   ```text
   myproject/service/target/myproject-service-0.0.1-SNAPSHOT.jar
   ```

### 部署

> 获取新生成的myproject-service-0.0.1-SNAPSHOT.jar和run.sh，复制到自定义的部署目录，同UI一起部署

### 启停

1. 启动服务

   ```shell
   sh run.sh start
   ```

2. 停止服务

   ```shell
   sh run.sh stop
   ```

3. 重启服务

   ```shell
   sh run.sh restart
   ```

### 验证

1. 验证SERVICE服务注册到注册中心

   > 注册中心地址 http://127.0.0.1:8761

   ![](images/QuickStart/createpro1.png)

2. 验证SERVICE服务接口

   > curl请求返回json串

   ```shell
   [root@localhost opt]# curl -k "https://192.168.55.53:8081/myprojectservice/api/v1/hello?access_token=admin"
   {"success":true,"message":null,"data":"Hi"}[root@localhost opt]# 
   ```

## 4. UI

### 修改

1. 修改启停脚本run.sh，配置服务发现地址、服务IP、服务端口等

   ```properties
   nohup java -jar ${PRONAMEUI} \
       --discovery.server.address="https://127.0.0.1:8761/eureka/" \
       --server.host=127.0.0.1 \
       --server.port=8080 \
       --spring.profiles.active=prod >/dev/null 2>&1 &
   ```

2. 修改myproject/ui/pom.xml，自定义项目名称myproject-ui，调整引用myproject-util

   ```xml
   <modelVersion>4.0.0</modelVersion>
   <groupId>org.iplatform.myproject</groupId>
   <artifactId>myproject-ui</artifactId>
   <version>0.0.1-SNAPSHOT</version>
   <name>myproject-ui</name>
   <description>myproject-ui</description>
   <packaging>jar</packaging>
   <parent>
       <groupId>org.iplatform.myproject</groupId>
       <artifactId>myproject</artifactId>
       <version>0.0.1-SNAPSHOT</version>
       <relativePath>../</relativePath>
   </parent>
   <dependencies>
       <dependency>
           <groupId>org.iplatform.myproject</groupId>
           <artifactId>myproject-util</artifactId>
           <version>0.0.1-SNAPSHOT</version>
       </dependency>
   </dependencies>
   ```

3. 修改myproject/ui/src/main/resources/bootstrap.yml，自定义应用名称myproject-ui

   ```yaml
   info:
     app:
       name: 前台服务
       description: 前台服务
       version: 0.0.1
   spring:
     application:
       name: myproject-ui
   ```

### 打包

> myproject目录下打包

1. 打包命令

   ```shell
   mvn clean package
   ```

2. 生成目录

   ```text
   myproject/ui/target/myproject-ui-0.0.1-SNAPSHOT.jar
   ```

### 部署

> 获取新生成的myproject-ui-0.0.1-SNAPSHOT.jar和run.sh，复制到自定义的部署目录，同SERVICE一起部署

### 启停

1. 启动服务

   ```bash
   sh run.sh start
   ```

2. 停止服务

   ```bash
   sh run.sh stop
   ```

3. 重启服务

   ```shell
   sh run.sh restart
   ```

### 验证

1. 验证UI服务注册到注册中心

   > 注册中心地址 http://127.0.0.1:8761

   ![](images/QuickStart/createpro2.png)

2. 验证UI服务到SERVICE服务接口调用

   > curl请求返回字符串 Hi

   ```shell
   [root@localhost opt]# curl -k "https://192.168.55.53:8080/myprojectui/hello?access_token=admin"
   Hi[root@localhost opt]# 
   ```

## 5. 骨架代码说明

### SERVICE说明

1. 主启动类 

   ```java
   myproject/service/src/main/java/org/iplatform/myproject/service/ServiceApplication.java
   
   @EnableOAuth2Client
   @SpringBootApplication
   @EnableTransactionManagement
   @EnableDiscoveryClient
   @EnableEurekaClient
   @EnableResourceServer
   @EnableJms
   @EnableCaching
   @EnableHystrix
   @Configuration
   @ComponentScan({"org.iplatform.microservices", "org.iplatform.myproject.service"})
   public class ServiceApplication extends IPlatformServiceApplication {
   
       private static final Logger LOG = LoggerFactory.getLogger(ServiceApplication.class);
   
       public static void main(String[] args) {
           try {
               run(ServiceApplication.class, args);
           } catch (Exception e) {
               LOG.error("", e);
           }
       }
   }
   ```

2. 接口类

   ```java
   myproject/service/src/main/java/org/iplatform/myproject/service/service/IndexService.java
   
   @Configuration
   @Service
   @RestController
   @RequestMapping("/api/v1")
   public class IndexService {
       private static final Logger LOG = LoggerFactory.getLogger(IndexService.class);
   
       @PostConstruct
       public void init() {
           LOG.info("类实例化");
       }
   
       /**
        * 仅演示用
        */
       @RequestMapping(value = "/hello", method = RequestMethod.GET)
       public ResponseEntity<RestResponse<Map>> hello() {
           RestResponse<String> response = new RestResponse<>();
           try {
               response.setData("Hi");
               response.setSuccess(Boolean.TRUE);
               return new ResponseEntity(response, HttpStatus.OK);
           } catch (Exception ex) {
               LOG.error("内部错误", ex);
               response.setSuccess(Boolean.FALSE);
               return new ResponseEntity(response, HttpStatus.INTERNAL_SERVER_ERROR);
           }
       }
   }
   ```

### UI说明

1. 主启动类

   ```java
   myproject/ui/src/main/java/org/iplatform/myproject/ui/UIApplication.java
   
   @Configuration
   @SpringBootApplication
   @EnableDiscoveryClient
   @EnableEurekaClient
   @EnableResourceServer
   @EnableFeignClients
   @RestController
   @EnableHystrix
   @EnableOAuth2Client
   @EnableCaching
   @EnableJms
   @EnableAspectJAutoProxy
   @ComponentScan({"org.iplatform.microservices", "org.iplatform.myproject.ui"})
   public class UIApplication extends IPlatformUIApplication {
   
       private static final Logger LOG = LoggerFactory.getLogger(UIApplication.class);
   
       public static void main(String[] args) throws Exception {
           try {
               run(UIApplication.class, args);
           } catch (Exception e) {
               LOG.error("", e);
           }
       }
   }
   ```

2. Feign客户端

   > 通过Feign实现对SERVICE接口的调用

   ```java
   myproject/ui/src/main/java/org/iplatform/myproject/ui/feign/IndexClient.java
   
   @FeignClient("myproject-service")
   public interface IndexClient {
   
       @RequestMapping(value = "myprojectservice/api/v1/hello", method = RequestMethod.GET)
       public ResponseEntity<RestResponse<String>> hello();
   }
   ```

3. controller类

   ```java
   myproject/ui/src/main/java/org/iplatform/myproject/ui/controller/IndexController.java
   
   @Controller
   public class IndexController {
   	private static final Logger LOG = LoggerFactory.getLogger(IndexController.class);
   	
   	@Autowired
   	private UserDetailsUtil userDetailsUtil;
   
   	@Autowired
   	private IndexClient indexClient;
   	
   	@RequestMapping("/")
   	public String index(ModelMap map,Principal principal) throws Exception {	
   		//每次进入首页都清除用户缓存信息，以便于后续操作会重新从认证服务器中获取后再次缓存
   		userDetailsUtil.removeUserDetails(principal.getName());		
   		return "index";	//首页index.html	
   	}
   
   	/**
   	 * 仅演示用
   	 */
   	@RequestMapping("/hello")
   	@ResponseBody
   	public String hello() throws Exception{
   		return indexClient.hello().getBody().getData();
   	}
   }
   ```

4. 拦截器

   > 自定义跳过认证拦截的路径

   ```
   @Configuration
   @EnableWebSecurity
   @Order(102)
   public class MyprojectUISecurityConfiguration extends WebSecurityConfigurerAdapter {
   
   	@Override
   	public void configure(WebSecurity web) throws Exception {
           //web.ignoring().antMatchers("/xx").antMatchers("/xxx");		
   	}
   }
   ```

5. 首页

   > 登录地址：https://127.0.0.1:9999/auth/?redirect_uri=https://127.0.0.1:8080/myprojectui/

   ```html
   myproject/ui/src/main/resources/templates/index.html
   ```

   
