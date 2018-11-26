# Docker

> 作者 张磊

本文档指导如何将微服务程序制作成docker镜像，本文档以下示例基于[项目构建手册](YourFirstProject.md)中的myporject项目

## 1. 前提

- 制作镜像的服务器已经安装了docker环境
- docker私服已经搭建

## 2. 定义镜像制作脚本

### 2.1 创建Docker目录

在myproject项目根目录下创建docker目录

```
mkdir docker
```

### 2.2 创建Dockerfile文件

> vi dokcer/Dockerfile

```
FROM boco/alpine-jre8:3.7
MAINTAINER Lei Zhang <zhang_lei@boco.com.cn>
ENV HOME_DIR=/myproject
ENV JAR_SERVICE_NAME=myproject-service-0.0.1.jar
ENV JAR_UI_NAME=myproject-ui-0.0.1.jar

RUN \
    mkdir -p ${HOME_DIR} \
    && curl -O http://127.0.0.1/myproject/${JAR_SERVICE_NAME} \
    && curl -O http://127.0.0.1/myproject/${JAR_UI_NAME} \
    && curl -O http://127.0.0.1/myproject/run-ui.sh \
    && curl -O http://127.0.0.1/myproject/run-service.sh \
    && mv ${JAR_SERVICE_NAME} ${HOME_DIR} \
    && mv ${JAR_UI_NAME} ${HOME_DIR} \
    && mv *.sh ${HOME_DIR} \
    && chmod +x ${HOME_DIR}/*.sh

WORKDIR ${HOME_DIR}

EXPOSE 50010 50011
```

* HOME_DIR  是docker内部的部署路径
* JAR_SERVICE_NAME 是service的jar名
* JAR_UI_NAME 是ui的jar名
* 部署文件上传采用curl的方式，这样可以通过避免使用COPY，从而减少镜像尺寸，文件地址可通过自动化编译后放到apahce服务上
* EXPOSE 定义的是 ui和service的对外访问端口

### 2.3 创建run-ui.sh

> vi dokcer/run-ui.sh

```bash
#!/bin/bash

VERSION=0.0.1
UI_JAR=cmdb-ui-${VERSION}.jar

trap "shut_down" SIGKILL SIGTERM SIGINT
function shut_down() {
    kill $PID
}

java $JAVA_OPTIONS -XX:OnOutOfMemoryError="kill -9 %p" -jar ${UI_JAR} \
    --spring.profiles.active=prod & PID=$!
    
wait $PID
```

- VERSION  定义的是版本号
- UI_JAR 定义的是ui jar名称

### 2.4 创建run-service.sh

> vi dokcer/run-service.sh

```bash
#!/bin/bash

VERSION=0.0.1
SERVICE_JAR=cmdb-service-${VERSION}.jar

trap "shut_down" SIGKILL SIGTERM SIGINT
function shut_down() {
    kill $PID
}

java $JAVA_OPTIONS -XX:OnOutOfMemoryError="kill -9 %p" -jar ${SERVICE_JAR} \
    --spring.profiles.active=prod & PID=$!
    
wait $PID
```

* VERSION  定义的是版本号
* SERVICE_JAR 定义的是ui jar名称

### 2.5 创建build.sh

vi dokcer/build.sh

```bash
#! /usr/bin/env bash
cd `dirname $0`

docker build -t 127.0.0.1:8089/demo/myproject:0.0.1 .
docker push 127.0.0.1:8089/demo/myproject:0.0.1
docker tag 127.0.0.1:8089/demo/myproject:0.0.1 demo/myproject:0.0.1
docker save demo/myproject:0.0.1 | gzip > docker_demo_myproject_0.0.1.tar.gz
docker rmi demo/myproject:0.0.1
docker rmi 127.0.0.1:8089/demo/myproject:0.0.1
```

- 以上文件中的127.0.0.1:8090请替换成真实Docker仓库私服地址端口
- /demo/myproject 镜像名称
- 0.0.1 镜像版本
- docker_demo_myproject_0.0.1.tar.gz 导出的镜像文件

## 3. 制作镜像

```
sh build.sh
```

命令执行成功后将得到如下结果

* 发布demo/myproject:0.0.1镜像到私服127.0.0.1:8089上
* 在docker目录下生成了离线镜像文件docker_demo_myproject_0.0.1.tar.gz

## 4. 启动

### 4.1 导入离线镜像文件到运行服务器

> 如果启动环境可以连接Docker私服请跳过此步骤，并自行配置docker仓库地址

```shell
docker load -i docker_demo_myproject_0.0.1.tar.gz 
```

### 4.2 创建网络

> 此步骤不是必须，建议创建

```
  docker network create \
    --driver=bridge \
    --subnet=172.16.0.0/24 \
    oneitom-network
```

### 4.3 启动容器

在目标服务器上创建docker-compos文件

vi docker-compose-myproject.yml

```yaml
version: '3.2'
services:
  myproject-service:
    image: demo/myproject:0.0.1
    hostname: myproject-service
    container_name: myproject-service
    restart: always
    command: /myproject/run-service.sh
    networks:
      - oneitom-network
    ports:
      - '50010:50010'
    volumes:
      - ${ONEITOM_VOLUME_PATH}/myproject/logs:/myproject/logs
    environment:
      - 'JAVA_OPTIONS=-Xmx512m -Xms512m'
      - 'discovery.server.address=https://oneitom-discovery:8761/eureka/'

  myproject-ui:
    image: demo/myproject:0.0.1
    hostname: myproject-ui
    container_name: myproject-ui
    restart: always
    command: /myproject/run-ui.sh
    networks:
      - oneitom-network
    ports:
      - '50011:50011'
    volumes:
      - ${ONEITOM_VOLUME_PATH}/myproject/logs:/myproject/logs
    environment:
      - 'JAVA_OPTIONS=-Xmx512m -Xms512m'
      - 'discovery.server.address=https://oneitom-discovery:8761/eureka/'       

networks:
  oneitom-network:
    external: true
```

* ONEITOM_VOLUME_PATH 通过环境变量设置的外部卷目录
* oneitom-network 是创建的docker网络

启动

```
docker-compose -f docker-compose-myproject.yml up -d
```

停止

```
docker-compose -f docker-compose-myproject.yml stop
```

删除

```
docker-compose -f docker-compose-myproject.yml rm
```

## 5. K8S & Mesos Support

