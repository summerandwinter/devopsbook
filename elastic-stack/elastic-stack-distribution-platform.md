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

| IP | 安装软件 |
| :--- | :--- |
| 172.17.0.4 | Filebeat + Kafka + Logstash + Elasticsearch + Kibana |
| 172.17.0.30 | Filebeat + Kafka + Logstash + Elasticsearch  |
| 172.17.0.103 | Filebeat + Kafka + Logstash + Elasticsearch  |

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

**172.17.0.4**

```text
broker.id=1
port=9092
host.name=172.17.0.4
log.dirs=/opt/kafka-logs
zookeeper.connect=172.17.0.4:2181,172.17.0.30:2181,172.17.0.103:2181
```

**172.17.0.30**

```text
broker.id=2
port=9092
host.name=172.17.0.30
log.dirs=/opt/kafka-logs
zookeeper.connect=172.17.0.4:2181,172.17.0.30:2181,172.17.0.103:2181
```

**172.17.0.103**

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

