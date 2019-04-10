# ELK部署手册

> 作者 于胜强

## 1. 安装

### 1.1 前提

* JDK1.8+
* 磁盘剩余空间大于500GB
* 空闲内存大于64GB
* 修改系统部署用户参数
  1. `ulimit -n 65536`
  2. `ulimit -l unlimited`

### 2.2 安装

1. 下载安装包

* elasticsearch-2.4.6.tar.gz
* logstash-all-plugins-2.4.1.tar.gz
* kibana-4.6.6-linux-x86_64.tar.gz

```bash
#下载logstash
wget https://download.elastic.co/logstash/logstash/logstash-all-plugins-2.4.1.tar.gz

#下载elasticsearch
wget https://download.elastic.co/elasticsearch/release/org/elasticsearch/distribution/tar/elasticsearch/2.4.6/elasticsearch-2.4.6.tar.gz

#下载kibana
https://download.elastic.co/kibana/kibana/kibana-4.6.6-linux-x86_64.tar.gz
```

2. 解压缩

如：安装目录为/opt/BOCO/elk

```bash
tar -zxvf elasticsearch-2.4.6.tar.gz -C /opt/BOCO/elk
tar -zxvf logstash-all-plugins-2.4.1.tar.gz -C /opt/BOCO/elk
tar -zxvf kibana-4.6.6-linux-x86_64.tar.gz -C /opt/BOCO/elk
```

3. 参数配置

> 修改/opt/BOCO/elk/elasticsearch-2.4.6/config/elasticsearch.yml

```yml
#集群名称
cluster.name: test111
node.name: node1
path.data: /opt/BOCO/elk/elasticsearch-2.4.6/data
path.logs: /opt/BOCO/elk/elasticsearch-2.4.6/logs
# 绑定IP
network.host: 192.168.0.1
http.port: 9200
# 设置分片数量和副本数，分片数量建议为节点数*3，副本数建议为2～3
index.number_of_shards: 9
index.number_of_replicas: 2
# 集群发现种子IP
discovery.zen.ping.unicast.hosts: ["192.168.0.1", "192.168.0.2", "192.168.0.3"]

```

> 修改/opt/BOCO/elk/kibana-4.6.6/config/kibana.yml

```yml
server.port: 5601
server.host: "0.0.0.0"
elasticsearch.url: "http://192.168.0.1:9200"
```

4. 启动

> elasticsearch启动
```bash
cd /opt/BOCO/elk/elasticsearch-2.4.6/bin
./elasticsearch -d
```

> kibana 启动
```shell
cd /opt/BOCO/elk/kibana-4.6.6/bin
nohup ./kibana >> kibana.log 2>&1 &
```
> logstash 启动

```shell
cd /opt/BOCO/elk/logstash-2.4.1/bin
nohup ./logstash -f ../config/logstash.conf -r --verbose >> logstash.log 2>&1 &
```
5. 验证安装

> elasticsearch

```shell
curl http://10.10.2.51:9200/_cat/nodes
```
> kibana

```shell
#浏览器访问
http://192.168.0.1:5601
```

6. 停止

> elasticsearch

```shell
ps -ef | grep elasticsearch
kill -9 pid
```

> kibana

```shell
ps -ef | grep node
kill -9 pid
```

> logstash

```shell
ps -ef | grep logstash
kill -9 pid
```

## 2. Docker

> 基于Docker的单节点配置

```yaml
version: '3.2'
services:
  oneitom-elk:
    image: boco/alpine-elk:2.4.6
    hostname: oneitom-elk
    container_name: oneitom-elk
    restart: always
    ports:
      - 9200:9200
      - 5044:5044
      - 5601:5601  
      - 9300:9300  
    volumes:
      - ${ONEITOM_VOLUME_PATH}/oneitom-elk:/var/lib/elasticsearch     
      - ${ONEITOM_VOLUME_PATH}/oneitom-elk/logs:/opt/elasticsearch/logs            
    networks:
      - oneitom-network         

networks:
  oneitom-network:
    external: true
```


