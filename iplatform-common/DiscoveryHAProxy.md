# 访问代理服务部署手册

> 作者 张磊
>
> 访问代理服务主要用于通过统一的IP地址进行访问，同时提供页面URL地址重写功能

## 1. 准备

* JDK1.8+
* 服务器已经安装了HAProxy

## 2. 介质

| 文件名                          | 说明                        |
| ------------------------------- | --------------------------- |
| discovery-haproxy-1.0.0.100.jar | 主程序文件                  |
| run.sh                          | 启停脚本                    |
| reloadhaproxy.sh                | 重新加载HAProxy配置文件脚本 |

## 3. 启停

启动服务

```bash
sh run.sh start
```

停止服务

```bash
sh run.sh stop
```

 重启服务

```bash
sh run.sh restart
```

##  4. 参数

> 所有的参数都定义在启动脚本 run.sh 中 

| 参数名                           | 必填 | 默认值 | 说明                                                         |
| -------------------------------- | ---- | ------ | ------------------------------------------------------------ |
| discovery.server.address         | 是   |        | 定义注册服务的地址，当集群模式时配置多个地址逗号分隔  discovery.server.address=https://192.168.0.1:8761/eureka/,https://192.168.0.2:8761/eureka/ |
| server.host                      | 是   |        | 服务绑定IP                                                   |
| server.port                      |      | 8764   | 服务绑定端口                                                 |
| spring.cloud.config.enable       | 是   | true   | 开启集中配置功能                                             |
| spring.cloud.config.profile      | 是   |        | 集中配置环境名，例如：生产环境                               |
| haproxy.file                     | 是   |        | haproxy.cfg 的文件全路径                                     |
| haproxy.reload                   | 是   |        | reloadhaproxy.sh 文件全路径                                  |
| haproxy.urlrewrite.clientIpMatch | 是   |        | 被代理的访问客户端IP地址段，多个地址逗号分隔，例如：10.50.,10.127. |
| haproxy.urlrewrite.destIp        | 是   |        | 外网访问的代理IP地址                                         |
| haproxy.binds.status_http.port   | 是   | 8080   | HAProxy状态页面访问端口                                      |
| haproxy.binds.frontend_http.port | 是   | 80     | 访问微服务的代理端口                                         |
| haproxy.bind.tcp.enabled         |      | false  | 是否开始四层访问代理（主要用于代理没有contextpath的服务）    |

## 5. 界面

访问代理服务器状态页面

```shell
http://${haproxy.urlrewrite.destIp}:${aproxy.binds.status_http.port}/stats
```

访问被管认证服务

```shell
http://${haproxy.urlrewrite.destIp}:${haproxy.binds.frontend_http.port}/auth
```

## 6. 页面URL地址重写

> 通过代理访问页面时，可以使用Thymeleaf的ProxyURL.resolverURL标签实现URL地址重写为代理地址

原始页面写法

```html
<form th:action="${serviceInfo.homePageUrl}">
	<input type="hidden" name="access_token" th:value="${access_token}"></input>
	<button class="btn btn-xs btn-primary" type="submit">进入</button>
</form>
```

URL地址重写

```html
<form th:action="${#ProxyURL.resolverURL('__${serviceInfo.homePageUrl}__')}">
	<input type="hidden" name="access_token" th:value="${access_token}"></input>
	<button class="btn btn-xs btn-primary" type="submit">进入</button>
</form>
```

效果说明：

* serviceInfo.homePageUrl 在后台配置的地址是 http://10.22.1.111:9999/auth
* haproxy的代理访问地址为 http://111.204.35.100/auth，并且服务器端获取的客户端IP段为10.127.x.x
* 如果通过10.22.1.111地址访问页面，那么页面中serviceInfo.homePageUrl地址依然为http://10.22.1.111:9999/auth
* 如果通过111.204.35.100地址访问页面，那么页面中serviceInfo.homePageUrl地址变为为http://10.22.1.100/auth

## 7.Docker

```yaml
version: '3.2'
services:
  oneitom-discovery-haproxy:
    image: boco/oneitom-discovery-haproxy:1.0.0.100
    container_name: oneitom-discovery-haproxy
    restart: always
    network_mode: host
    environment:
      - 'JAVA_OPTIONS=-Xmx512m -Xms512m'
      - 'discovery.server.address=http://10.22.1.236:8761/eureka/'
      - 'eureka.instance.metadataMap.tenant=trident'
      - 'spring.cloud.config.busisys=trident'
      - 'spring.cloud.config.enabled=true'
      - 'spring.cloud.config.profile=生产'
      - 'server.host.prefix=10.22.'
      - 'haproxy.urlrewrite.destIp=111.204.35.100'
      - 'haproxy.urlrewrite.clientIpMatch=10.127,10.50'
      - 'haproxy.binds.frontend_http.port=80'
      - 'haproxy.binds.stats_http.port=8080'
    volumes:
      - /opt/BOCO/oneitom/docker_volume/oneitom-discovery-haproxy:/usr/local/etc/haproxy:rw
      - /opt/BOCO/oneitom/docker_volume/oneitom-discovery-haproxy/logs:/discovery-haproxy/logs
    labels:
     - oneitom-discovery-haproxy-cluster
```

