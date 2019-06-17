# 配置

前面我们讲解了在 Linux 系统中如何离线安装 Prometheus 服务，下面我们详细了解一下 Prometheus 的最常用到的一些配置，更详细的配置文件说明可以参考[官方文档](https://prometheus.io/docs/prometheus/latest/configuration/configuration/)，我们首先看一下它的默认的配置文件。

{% code-tabs %}
{% code-tabs-item title="prometheus.yml" %}
```yaml
# my global config
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

  # Attach these labels to any time series or alerts when communicating with
  # external systems (federation, remote storage, Alertmanager).
  external_labels:
      monitor: 'codelab-monitor'

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first.rules"
  # - "second.rules"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ['localhost:9090']
```
{% endcode-tabs-item %}
{% endcode-tabs %}

* **scrape\_interval** 是指抓取数据的时间间隔，这里设置为 15 秒，默认 1 分钟
* **evaluation\_interval** 指的是计算 Rule 的间隔
* **scrape\_configs** 配置抓取的端点
* **metrics\_path** Metrics服务暴露的端点路径与 targets 配合使用，默认为 '/metrics'
* **scheme** http协议，默认为 'http' 
* **static\_configs** 静态配置，可以配置多个 target
* **targets** Metrics 服务的 IP 地址和端口号
* **job\_name** 设置 job 名称，要注意的是系统会把 job\_name 当做一个 label job='&lt;job\_name&gt;' 标签插入到时间序列中 



