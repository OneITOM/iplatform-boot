# 项目打包部署手册

> 作者 王立松

通过本文档你可以了解一个微服务项目是如何自动化打包部署

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

> 压缩启动脚本run.sh以及ui和service对应的jar包

```shell
cp run.sh ${tempdir}
cp ui-0.0.1.jar ${tempdir}
cp service-0.0.1.jar ${tempdir}
tar -zcvf iplatform-myproject-0.0.1.tar.gz ${tempdir}
```

## 4. 部署

1. 创建部署目录

   ```shell
   mkdir -p 
   ```

2. 复制tar包到部署目录

   ```shell
   cp iplatform-myproject-0.0.1.tar.gz
   ```

3. 解压

   ```shell
   tar -zxvf iplatform-myproject-0.0.1.tar.gz
   ```

4. 启动

   ```shell
   sh run.sh start
   ```
