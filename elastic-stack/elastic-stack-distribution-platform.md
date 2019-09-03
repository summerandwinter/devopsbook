# Elastic Stack + Kafka 分布式环境搭建

## Elastic Stack + Kafka 工作流

```text
Filebeat --> Kafka --> Logstash --> Elasticsearch --> Kibana
```

Filebeat 读取日志到 Kafka，Logstash 从 Kafka 队列中读取日志信息，对日志信息做分析和切割，并写入到 Elasticsearch 集群， Kibana 读取 Elasticsearch 中的日志信息做可视化展示。

## 部署准备

### 系统信息

```bash
rpm -q centos-release
centos-release-7-5.1804.el7.centos.2.x86_64
```

### 软件信息

| 软件 | 版本 |
| :--- | :--- |
| JDK | 1.8 |
| Filebeat | 7.3.1 |
| Logstash | 7.3.1 |
| Elasticsearch | 7.3.1 |
| Kibana | 7.3.1 |
| kafka | 2.1.0 |

### 服务器信息

| 机器名称 | IP | 安装软件 |
| :--- | :--- | :--- |
| node-1 | 172.17.0.4 | Filebeat + Kafka + Logstash + Elasticsearch + Kibana |
| node-2 | 172.17.0.30 | Filebeat + Kafka + Logstash + Elasticsearch  |
| node-3 | 172.17.0.103 | Filebeat + Kafka + Logstash + Elasticsearch  |

## Kafka 集群部署

假定安装目录为 /opt/devops

### 安装

```bash
cd /opt/devops
curl -L -O https://archive.apache.org/dist/kafka/2.1.0/kafka_2.11-2.1.0.tgz
tar -zxf kafka_2.11-2.1.0.tgz
cd kafka_2.11-2.1.0
```

### 配置 Zookeeper



```bash
vim /opt/devops/kafka_2.11-2.1.0/config/zookeepr.properties
```

三台机器配置都一样

```text
dataDir=/opt/zookeeper-data
clientPort=2181
server.1=172.17.0.4:2888:3888
server.2=172.17.0.30:2888:3888
server.3=172.17.0.103:2888:3888
```

### 配置 Kafka 集群

node-1

```text
broker.id=1
port=9092
host.name=172.17.0.4
log.dirs=/opt/kafka-logs
zookeeper.connect=172.17.0.4:2181,172.17.0.30:2181,172.17.0.103:2181
```

node-2

```text
broker.id=2
port=9092
host.name=172.17.0.30
log.dirs=/opt/kafka-logs
zookeeper.connect=172.17.0.4:2181,172.17.0.30:2181,172.17.0.103:2181
```

node-3

```text
broker.id=3
port=9092
host.name=172.17.0.103
log.dirs=/opt/kafka-logs
zookeeper.connect=172.17.0.4:2181,172.17.0.30:2181,172.17.0.103:2181
```

### 启动 Zookeeper

```bash
/opt/devops/kafka_2.11-2.1.0/bin/kafka-topics.sh --list --zookeeper 172.17.0.4:2181bin/zookeeper-server-start.sh -daemon config/zookeeper.properties
```

### 启动 Kafka

```bash
/opt/devops/kafka_2.11-2.1.0/bin/kafka-server-start.sh -daemon config/server.properties
```

### 创建 Topic

```bash
/opt/devops/kafka_2.11-2.1.0/bin/kafka-topics.sh --create --zookeeper 172.17.0.4:2181 --replication-factor 1 --partitions 1 --topic test
```

### 查询 Topic

```bash
/opt/devops/kafka_2.11-2.1.0/bin/kafka-topics.sh --list --zookeeper 172.17.0.4:2181
```

## Elasticsearch 集群部署

### 安装

```bash
curl -L -O https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.3.1-x86_64.rpm
sudo rpm -i elasticsearch-7.3.1-x86_64.rpm
```

### 配置

```bash
vim /etc/elasticsearch/elasticsearch.yml
```

**node-1**

```text
cluster.name: es-cluster
node.name: es-node-1
node.master: true
node.data: true
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch
network.host: 172.17.0.4
discovery.seed_hosts: ["172.17.0.4:9300", "172.17.0.30:9300", "172.17.0.30:9300"] 
cluster.initial_master_nodes: ["es-node-1", "es-node-2", "es-node-3"]
gateway.recover_after_nodes: 1
http.cors.enabled: true
http.cors.allow-origin: "*"
```

**node-2**

```text
cluster.name: es-cluster
node.name: es-node-2
node.master: true
node.data: true
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch
network.host: 172.17.0.30
discovery.seed_hosts: ["172.17.0.4:9300", "172.17.0.30:9300", "172.17.0.30:9300"] 
cluster.initial_master_nodes: ["es-node-1", "es-node-2", "es-node-3"]
gateway.recover_after_nodes: 1
http.cors.enabled: true
http.cors.allow-origin: "*"
```

**node-3**

```text
cluster.name: es-cluster
node.name: es-node-1
node.master: true
node.data: true
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch
network.host: 172.17.0.103
discovery.seed_hosts: ["172.17.0.4:9300", "172.17.0.30:9300", "172.17.0.30:9300"] 
cluster.initial_master_nodes: ["es-node-1", "es-node-2", "es-node-3"]
gateway.recover_after_nodes: 1
http.cors.enabled: true
http.cors.allow-origin: "*"
```

### 启动

```bash
service elasticsearch start
```

查看集群是否启动成功

```bash
curl http://172.17.0.4:9200
```

如果出现类似下面的信息说明集群启动成功

```text
{
  "name" : "es-node-1",
  "cluster_name" : "es-cluster",
  "cluster_uuid" : "Vj5OJkf5SxKsfHmjeAYScw",
  "version" : {
    "number" : "7.3.1",
    "build_flavor" : "default",
    "build_type" : "rpm",
    "build_hash" : "4749ba6",
    "build_date" : "2019-08-19T20:19:25.651794Z",
    "build_snapshot" : false,
    "lucene_version" : "8.1.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

## Kibana 部署

假定 Kibana 部署在 node-1

### 安装

```bash
curl -L -O https://artifacts.elastic.co/downloads/kibana/kibana-7.3.1-linux-x86_64.tar.gz
tar xzvf kibana-7.3.1-linux-x86_64.tar.gz
```

### 配置

```bash
vim /opt/devops/kibana-7.3.1-linux-x86_64/config/kibana.yml
```

```text
server.host: "0.0.0.0"
elasticsearch.hosts: ["http://172.17.0.4:9200", "http://172.17.0.30:9200", "http://172.17.0.30:9200"]
i18n.locale: "zh-CN"
```

### 启动

```bash
/opt/devops/kibana-7.3.1-linux-x86_64/bin/kibana -d
```

访问 [http://172.17.0.4:560](http://127.0.0.1:5601) 验证kibana是否启动成功

## Logstash 部署

安装

```bash
curl -L -O https://artifacts.elastic.co/downloads/logstash/logstash-7.3.1.rpm
sudo rpm -i logstash-7.3.1.rpm
```

配置

```bash
vim /etc/logstash/conf.d/logstash.conf 
input {
  beats {
    port => 5044
  }
}

filter {
  grok {
    match => {
       "message" => "\s*\[%{TIMESTAMP_ISO8601:time}\]\s* \s*%{LOGLEVEL:level}\s* \s*(?<prefix>\w+)\s* \s*%{JAVACLASS:class}\((?<function>[A-Za-z0-9_. -:]+)\)\s* \s*%{JAVALOGMESSAGE:logmessage}\s*"
    }
  }
}

output {
  elasticsearch {
    hosts => ["http://172.17.0.4:9200", "http://172.17.0.30:9200", "http://172.17.0.103:9200"]
   index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
  }
}
```

启动

```bash
sudo service logstash start
```

## Filebeat 部署

### 安装

```bash
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.3.1-x86_64.rpm
sudo rpm -vi filebeat-7.3.1-x86_64.rpm
```

```text
- type: log
  paths:
    - /Users/summer/data/log/spring-boot/spring-boot-activemq-consumer-action.log
  multiline:  
    pattern: ^\[
    negate: true                      
    match: after                           
    max_lines: 1000     
    timeout: 30s
    tags: ["spring-boot"]
setup.kibana:
  host: "http://172.17.0.4:5601"
output.logstash:
  hosts: ["172.17.0.4:5044"]
```

### 启动

```bash
sudo filebeat setup # 加载 Kibana 仪表板
sudo service filebeat start
```





