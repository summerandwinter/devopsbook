# Spring Boot 集成

 在 Spring Boot 1.x 中  Spring Boot Actuator 使用的是  [`dropwizard-metrics`](https://metrics.dropwizard.io)  来实现度量指标的收集。

Spring Boot 2.x 在 Spring Boot Actuator 中引入了 [`micrometer`](http://micrometer.io)  对 1.x 的 metrics 进行了重构，另外支持对接的监控系统也更加丰富。而 [`micrometer`](http://micrometer.io)  除了一些基本 metrics 与 [`dropwizard-metrics`](https://metrics.dropwizard.io) 相类似外，重点支持了 tag  。这是一个很重要的信号，标志着老一代的 [`statsd`](https://github.com/statsd/statsd) 、[`graphite`](http://graphiteapp.org) 逐步让步于支持 tag  的 [`influx`](https://www.influxdata.com/)  以及 [`prometheus`](https://prometheus.io)  。

支持 tag 的优势在于，可以进行多维度的统计和查询，以同一微服务但是不同实例的 jvm 指标来说，可以通过 tag 来添加 host 标识，这样监控系统就可以灵活根据tag 查询过滤来查看不同主机粒度的，甚至是不同数据中心的粒度。像 [`statsd`](https://github.com/statsd/statsd)不支持 tag，如果要区分多 host 的同一个 jvm 指标，则通常是通过添加 prefix 来解决，不过这个给查询统计以及后续扩展带了诸多的便。

在Spring Boot 1.5.x 中还可以通过 `micrometer-spring-legacy`  来使用`micrometer`（此时就不需要引入Spring Boot Actuator 包了，因为 legacy 和acuator 都是为了实现监控是功能等同的包），但是之前版本是不支持的。从这个角度讲 Spring Boot 2.x 的 actuator  相当于实现了 micrometer-spring-legacy 



参考文档

{% embed url="https://www.jianshu.com/p/255ffdc258f9" %}

{% embed url="http://www.heartthinkdo.com/?p=2457" %}

{% embed url="https://www.jdon.com/soa/dropwizard-vs-spring-boot.html" %}

  
  


