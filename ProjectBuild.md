# 项目打包部署手册

> 作者 王立松

通过本文档你可以了解一个微服务项目是如何自动化打包部署

## 准备

- [已安装发现服务](iplatform-common/DiscoveryService.md)
- [已安装认证服务](iplatform-common/AuthService.md)
- JDK1.8+
- MAVEN3.5+

## 打包

> 基于mvn命令打包

```shell
mvn clean package
```

## 部署

> 通过shell脚本整合压缩目标文件，复制到部署目录并解压

```shell
deploydir=/opt/BOCO/deploy/myproject \
&& workspacedir=/data/workspace/myproject \
&& tmptardir=/tmp/iplatform-myproject \
&& tempdir=${tmptardir}/$(date +%Y%m%d%H%M%S) \
&& tarname=iplatform-myproject-0.0.1.tar.gz \
&& mkdir -p ${tempdir} \
&& cp ${workspacedir}/service/target/service-0.0.1-SNAPSHOT.jar ${tempdir} \
&& cp ${workspacedir}/ui/target/ui-0.0.1-SNAPSHOT.jar ${tempdir} \
&& cp ${workspacedir}/config/local/run.sh ${tempdir} \
&& tar -zcvf ${tmptardir}/${tarname} -C ${tempdir} . \
&& cp -f ${tmptardir}/${tarname} ${deploydir} \
&& rm -rf ${tmptardir} \
&& tar -xzvf ${deploydir}/${tarname} -C ${deploydir}
```

## 重启

```shell
sh run.sh restart
```

## 参数

> 部署参数，部署脚本中涉及的变量

| 名称         | 说明              |
| :----------- | :---------------- |
| deploydir    | 部署目录          |
| workspacedir | 源码存放地址      |
| tmptardir    | 临时tar包存放目录 |
| tempdir      | 临时文件存放目录  |
| tarname      | tar包名称         |

> 启动参数，启动脚本run.sh中的主要参数定义

| 名称                                  | 说明                                                 |
| :------------------------------------ | :--------------------------------------------------- |
| discovery.server.address              | 定义注册服务的地址，当集群模式时配置多个地址逗号分隔 |
| server.host                           | 服务绑定IP                                           |
| server.port                           | 服务绑定端口                                         |
| spring.datasource.platform            | 数据库平台类型配置，例如：mysql                      |
| spring.datasource.dataSourceClassName | 数据库JDBC驱动，例如：com.mysql.jdbc.Driver          |
| spring.datasource.url                 | 数据库URL                                            |
| spring.datasource.username            | 数据库用户名                                         |
| spring.datasource.password            | 数据库密码                                           |
