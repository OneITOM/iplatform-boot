# 项目构建手册

> 作者 王立松

通过本文档可以快速开发一个属于你的SERVICE、UI服务，你可以看到服务是如何注册到注册中心，并通过认证服务登录后完成一个UI到SERVICE调用的样例

## 1. 准备

- 已安装发现服务
- 已安装认证服务
- JDK1.8+
- MAVEN3.5+

## 2. 代码生成

- 模板信息

  ```sh
  groupId=org.iplatform.myproject
  artifactId=myproject
  package=org.iplatform.myproject
  version=0.0.1-SNAPSHOT
  ```

- 创建脚本

  ```sh
  mvn org.apache.maven.plugins:maven-archetype-plugin:2.4:generate \
  -DgroupId=org.iplatform.myproject \
  -DartifactId=myproject \
  -Dpackage=org.iplatform.myproject \
  -Dversion=0.0.1-SNAPSHOT \
  -DarchetypeCatalog=http://192.168.100.10/nexus/content/repositories/releases \
  -DarchetypeGroupId=org.iplatform.archetypes \
  -DarchetypeArtifactId=iplatform-all-archetype \
  -DarchetypeVersion=0.0.7 \
  -DinteractiveMode=false
  ```

- 代码结构说明

  | 目录结构                      | 说明        |
  | ----------------------------- | ----------- |
  | myproject/config/local/run.sh | 启停脚本    |
  | myproject/service             | SERVICE服务 |
  | myproject/ui                  | UI服务      |
  | myproject/util                | 工具包      |

## 3. SERVICE部署

### 修改

> 修改启停脚本run.sh，配置服务发现地址、服务IP、服务端口等

```sh
nohup java -jar ${PRONAMESERVICE} \
	--discovery.server.address="https://127.0.0.1:8761/eureka/" \
    --server.host=127.0.0.1 \
    --server.port=8081 \
    --spring.profiles.active=prod >/dev/null 2>&1 &
```

### 打包

> myproject目录下打包

1. 打包命令

   ```bash
   mvn clean package
   ```

2. 生成目录

   ```text
   myproject/service/target/service-0.0.1-SNAPSHOT.jar
   ```

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

   ```bash
   sh run.sh restart
   ```

### 验证

1. 验证SERVICE服务注册到注册中心
2. 验证SERVICE服务接口

## 4. UI部署

### 修改

> 修改启停脚本run.sh，配置服务发现地址、服务IP、服务端口等

```sh
nohup java -jar ${PRONAMEUI} \
	--discovery.server.address="https://127.0.0.1:8761/eureka/" \
    --server.host=127.0.0.1 \
    --server.port=8080 \
    --spring.profiles.active=prod >/dev/null 2>&1 &
```

### 打包

> myproject目录下打包

1. 打包命令

   ```bash
   mvn clean package
   ```

2. 生成目录

   ```text
   myproject/ui/target/ui-0.0.1-SNAPSHOT.jar
   ```

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

   ```bash
   sh run.sh restart
   ```

### 验证

1. 验证UI服务注册到注册中心
2. 验证UI服务到SERVICE服务接口调用
