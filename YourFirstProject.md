# 项目构建手册

> 作者 王立松

通过本文档可以快速开发一个属于你的SERVICE、UI服务，你可以看到服务是如何注册到注册中心，并通过认证服务登录后完成一个UI到SERVICE调用的样例

## 1. 准备

- [已安装发现服务](iplatform-common/DiscoveryService.md)
- [已安装认证服务](iplatform-common/AuthService.md)
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
  -DarchetypeVersion=0.0.8 \
  -DinteractiveMode=false
  ```

- 目录结构说明

  | 目录结构                      | 说明                     |
  | ----------------------------- | ------------------------ |
  | myproject/config/local/run.sh | 启停脚本                 |
  | myproject/service             | SERVICE服务              |
  | myproject/ui                  | UI服务                   |
  | myproject/util                | UI和SEERVICE公用的工具包 |

## 3. 文件修改

### UTIL

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

### SERVICE

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

### UI

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

## 4. 打包

> myproject目录下打包

1. 打包命令

   ```shell
   mvn clean package
   ```

2. 生成目录

   ```text
   myproject/service/target/myproject-service-0.0.1-SNAPSHOT.jar
   myproject/ui/target/myproject-ui-0.0.1-SNAPSHOT.jar
   ```

## 5. 部署

> 复制myproject-service-0.0.1-SNAPSHOT.jar、myproject-ui-0.0.1-SNAPSHOT.jar和run.sh到指定部署目录

## 6. 启停

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

## 2. 验证

1. 验证SERVICE和UI服务注册到注册中心

   > 注册中心地址 http://127.0.0.1:8761

   ![](images/QuickStart/createpro1.png)

2. 验证SERVICE服务接口

   > curl请求返回json串

   ```shell
   [root@localhost opt]# curl -k "https://192.168.55.53:8081/myprojectservice/api/v1/hello?access_token=xx"
   {"success":true,"message":null,"data":"Hi"}[root@localhost opt]# 
   ```

3. 验证UI服务到SERVICE服务接口调用

   > curl请求返回字符串 Hi

   ```shell
   [root@localhost opt]# curl -k "https://192.168.55.53:8080/myprojectui/hello?access_token=xx"
   Hi[root@localhost opt]# 
   ```

## 5. 骨架代码说明

### SERVICE说明

| 文件地址                                                  | 文件说明                             |
| -------------------------------------------------------- | ------------------------------------ |
| org.iplatform.myproject.service.ServiceApplication.java   | 主启动类                             |
| org.iplatform.myproject.service.service.IndexService.java | 接口类，提供对外接口                 |
| src/main/resources/bootstrap.yml                          | 应用定义，包括应用名称、描述、版本等 |
| src/main/resources/application.yml                        | 应用配置，包含IP地址、端口、数据源等 |

### UI说明

| 文件地址                                                     | 文件说明                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| org.iplatform.myproject.ui.UIApplication.java                | 主启动类                                                     |
| org.iplatform.myproject.ui.config.MyprojectUISecurityConfiguration | 拦截器，自定义跳过认证拦截的路径                             |
| org.iplatform.myproject.ui/feign.IndexClient.java            | Feign客户端，通过Feign实现对SERVICE接口的调用                |
| org.iplatform.myproject.ui.controller.IndexController        | 控制器                                                       |
| src/main/resources/bootstrap.yml                             | 应用定义，包括应用名称、描述、版本等                         |
| src/main/resources/application.yml                           | 应用配置，包含IP地址、端口、数据源等                         |
| src/main/resources/templates/index.html                      | 通过认证登录成功以后跳转至该页面，认证地址：https://127.0.0.1:9999/auth |
