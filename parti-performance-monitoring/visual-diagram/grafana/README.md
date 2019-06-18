# Grafana

Grafana是一个跨平台的开源的度量分析和可视化工具，可以通过将采集的数据查询然后可视化的展示，并及时通知。它主要有以下六大特点：

1、展示方式：快速灵活的客户端图表，面板插件有许多不同方式的可视化指标和日志，官方库中具有丰富的仪表盘插件，比如热图、折线图、图表等多种展示方式；

2、数据源：官方支持的数据源有 [Graphite](https://grafana.com/docs/features/datasources/graphite/)，[InfluxDB](https://grafana.com/docs/features/datasources/influxdb/)，[OpenTSDB](https://grafana.com/docs/features/datasources/opentsdb/)，[Prometheus](https://grafana.com/docs/features/datasources/prometheus/)，[Elasticsearch](https://grafana.com/docs/features/datasources/elasticsearch/)，[CloudWatch](https://grafana.com/docs/features/datasources/cloudwatch/) ，同时社区也提供了很多第三方 的数据源；

3、通知提醒：以可视方式定义最重要指标的警报规则，Grafana将不断计算并发送通知，在数据达到阈值时通过Slack、PagerDuty等获得通知；

4、混合展示：在同一图表中混合使用不同的数据源，可以基于每个查询指定数据源，甚至自定义数据源；

5、注释：使用来自不同数据源的丰富事件注释图表，将鼠标悬停在事件上会显示完整的事件元数据和标记；

6、过滤器：Ad-hoc过滤器允许动态创建新的键/值过滤器，这些过滤器会自动应用于使用该数据源的所有查询。

