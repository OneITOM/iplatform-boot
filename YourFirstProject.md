# 项目构建手册

> 作者 王立松

通过本文档可以快速部署一个认证服务、注册服务，并开发一个你的SERVICE、UI服务，你可以看到服务是如何注册到注册中心，并通过认证服务登录后完成一个UI到SERVICE调用的样例

## 1. 准备

* 已安装发现服务
* 已安装认证服务
* 开发环境 JDK1.8+
* 开发工具 Spring Tool Suite

## 2. 项目构建

* SERVICE创建

1. 打开Spring Tool Suite，创建Spring Starter Project

![](images/QuickStart/createpro1.png)
   
![](images/QuickStart/createpro2.png)
   
![](images/QuickStart/createpro3.png)
   
2. demo-service/pom.xml调整，调整后内容如下

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>org.iplatform</groupId>
	<artifactId>demo-service</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>

	<name>demo-service</name>
	<description>demo-service</description>
	
	<parent>
		<groupId>org.iplatform</groupId>
		<artifactId>iplatform-parent</artifactId>
		<version>0.0.7-SNAPSHOT</version>
		<relativePath></relativePath>
	</parent>
	
	<dependencies>
		<dependency>
			<groupId>org.iplatform</groupId>
			<artifactId>iplatform-util</artifactId>
			<exclusions>
				<exclusion>
					<artifactId>slf4j-log4j12</artifactId>
					<groupId>org.slf4j</groupId>
				</exclusion>
			</exclusions>
		</dependency>
        <dependency>
            <groupId>org.iplatform</groupId>
            <artifactId>iplatform-service</artifactId>
        </dependency>       
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-surefire-plugin</artifactId>
				<configuration>
					<skipTests>true</skipTests>
				</configuration>
			</plugin>
		</plugins>
	</build>

	<repositories>
		<repository>
			<id>boco-nexus-public</id>
			<name>Team Nexus Repository</name>
			<url>http://111.204.35.232/nexus/content/groups/public</url>
		</repository>
	</repositories>

</project>
```
3. demo-service/src/main/resources/application.properties调整为application.yml，具体配置如下

```
discovery.server.address: https://localhost:8761/eureka/
server:
  port: 50090
  host: localhost
  contextPath: /demoservice
```
| 参数名                       | 说明                                                         |
| --------------------------- | ------------------------------------------------------------ |
| discovery.server.address    | 定义注册服务的地址，当集群模式时配置多个地址逗号分隔              |
| server.host                 | 服务绑定IP                                                    |
| server.port                 | 服务绑定端口                                                  |

4. 测试类DemoServiceApplicationTests中@RunWith(SpringRunner.class)更改为@RunWith(SpringJUnit4ClassRunner.class)

5. 主启动类DemoServiceApplication调整，调整后内容如下

```
package org.iplatform.microservices.demoservice;

import org.iplatform.microservices.service.IPlatformServiceApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.EnableAspectJAutoProxy;
import org.springframework.jms.annotation.EnableJms;
import org.springframework.security.oauth2.config.annotation.web.configuration.EnableOAuth2Client;
import org.springframework.security.oauth2.config.annotation.web.configuration.EnableResourceServer;
import org.springframework.transaction.annotation.EnableTransactionManagement;

@EnableOAuth2Client
@SpringBootApplication
@EnableTransactionManagement
@EnableDiscoveryClient
@EnableEurekaClient
@EnableResourceServer
@EnableJms
@EnableCaching
@EnableAspectJAutoProxy
@ComponentScan(basePackages = {"org.iplatform.microservices"})
public class DemoServiceApplication extends IPlatformServiceApplication {

	public static void main(String[] args) throws Exception {
		run(DemoServiceApplication.class, args);
	}
}
```

* UI创建

## 3. 添加依赖

## 4. 服务配置

## 5. 服务启动

## 6. 服务验证
