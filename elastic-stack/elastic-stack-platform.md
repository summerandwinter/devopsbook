# Elastic Stack 单机环境搭建

## 环境

| 环境 | 版本 |
| :--- | :--- |
| CentOS | 7.0 |
| JDK | 1.8 |
| Elasticsearch | 7.3.1 |
| Beat | 7.3.1 |
| Kinana | 7.3.1 |
| Logstash | 7.3.1 |

## 安装Elasticserach

在控制台中执行下面的命令

```bash
curl -L -O https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.3.1-x86_64.rpm
sudo rpm -i elasticsearch-7.3.1-x86_64.rpm
sudo service elasticsearch start
```

验证Elasticsearch是否启动成功

```bash
curl http://127.0.0.1:9200
```

如果出现类似下面的信息代表启动成功

```text
{
  "name" : "QtI5dUu",
  "cluster_name" : "elasticsearch",
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

## 安装 Kibana

执行下面的命令

```bash
curl -L -O https://artifacts.elastic.co/downloads/kibana/kibana-7.3.1-linux-x86_64.tar.gz
tar xzvf kibana-7.3.1-linux-x86_64.tar.gz
cd kibana-7.3.1-linux-x86_64/
./bin/kibana
```

## 安装 Filebeat

```bash
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.3.1-x86_64.rpm
sudo rpm -vi filebeat-7.3.1-x86_64.rpm
```

## 安装 Logstash

```bash
curl -L -O https://artifacts.elastic.co/downloads/logstash/logstash-7.3.1.rpm
sudo rpm -i logstash-7.3.1.rpm
```

配置

```yaml
input {
  beats {
    port => 5044
  } 
}

output {
  elasticsearch {
    hosts => ["http://172.17.0.4:9200"] 
    index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}" 
  }
}
```

