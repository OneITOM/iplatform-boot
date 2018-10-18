# 项目打包部署手册

> 作者 王立松
>
> 通过本文档你可以了解一个微服务项目是如何自动化打包部署
>
> 以下示例为[项目构建手册](YourFirstProject.md)中的myproject项目

## 1. 前提

- [已安装发现服务](iplatform-common/DiscoveryService.md)
- [已安装认证服务](iplatform-common/AuthService.md)
- JDK1.8+
- MAVEN3.5+

## 2. 编译

```shell
mvn clean package
```

## 3. 打包

1. 创建临时目录

   ```shell
   mkdir temp
   ```

2. 复制目标文件到临时目录

   ```shell
   cp config/local/run.sh temp
   cp service/target/service-0.0.1-SNAPSHOT.jar temp
   cp ui/target/ui-0.0.1-SNAPSHOT.jar temp
   ```

3. 压缩并删除临时目录

   ```
   tar -zcvf iplatform-myproject-0.0.1.tar.gz -C temp .
   rm -rf temp
   ```

## 4. 部署

1. 创建部署目录

   ```shell
   mkdir -p /home/oneitom/myproject
   ```

2. 移动tar包到部署目录

   ```shell
   mv iplatform-myproject-0.0.1.tar.gz /home/oneitom/myproject
   ```

3. 解压缩

   ```shell
   cd /home/oneitom/myproject
   tar -zxvf iplatform-myproject-0.0.1.tar.gz
   ```

4. 启动

   ```shell
   sh run.sh start
   ```
