# Spring Boot 2.x

Spring Boot 中通过 Spring Boot Actuator 帮助我们监控和管理Spring Boot应用，比如健康检查、审计、统计和HTTP追踪等。所有的这些特性可以通过JMX 者HTTP endpoints来获得，这里我们主要介绍 HTTP endpoints 方式。

Spring Boot 2.x 中 Actuator 集成了 Micrometer。借助这一强大的框架我们只需要通过非常小的配置就可以集成大部分的应用监控系统。

通过端点的方式 Actuator 暴露不同类型的应用信息，可以在[官方文档](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-endpoints.html>) 看到完整的列表。这里我们需要用到的主要是 metrics

下面我们介绍如何在 Spring Boot 2.x 中集成 Actuator 

## 引入依赖

```markup
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<dependency>
  <groupId>io.micrometer</groupId>
  <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

spring-boot-starter-actuator 包实现了 metrics 度量信息的采集

micrometer-registry-prometheus 包把 actuator 采集到的度量信息转化为 prometheus 可用的格式

## 配置

```yaml
server:
  port: 18004
spring:
  application:
    name: actuator1
management:
  server:
    port: 19004 # 配置 management 独立的端口避免与业务路径冲突
  metrics:
    tags: # 定义 metrics 公共标签
      application: ${spring.application.name}
      region: region1
  endpoints:
    web:
      exposure:
        include: metrics # 配置需要暴露的端点
      base-path: /actuator # management 访问的跟路径，默认 actuator
  endpoint:
    metrics:
      enabled: true # 开启/关闭某个端点
```

## 验证

启动项目访问 [http://localhost:19004/actuator/metrics ](http://localhost:19004/actuator/prometheus)

看到类似下面的信息说明已经配置成功

```text
# HELP jvm_threads_daemon_threads The current number of live daemon threads
# TYPE jvm_threads_daemon_threads gauge
jvm_threads_daemon_threads{application="actuator1",region="region1",} 53.0
# HELP tomcat_global_error_total  
# TYPE tomcat_global_error_total counter
tomcat_global_error_total{application="actuator1",name="http-nio-18004",region="region1",} 0.0
# HELP tomcat_threads_config_max_threads  
# TYPE tomcat_threads_config_max_threads gauge
tomcat_threads_config_max_threads{application="actuator1",name="http-nio-18004",region="region1",} 200.0
# HELP process_start_time_seconds Start time of the process since unix epoch.
# TYPE process_start_time_seconds gauge
process_start_time_seconds{application="actuator1",region="region1",} 1.560331434337E9
```

Actuator 已经帮我们实现了一系列的 metrics ，这些 metrics 包含了对 system, tomcat, okhttp, log, kafaka, jvm, hystrix, db, cache 的一些常用度量信息的收集，这些度量信息对一般的系统监控来说已经够用了。当然我们也可是借助 Actuator 使用的 micrometer 很方便的实现自己的 metrics 来应对自己个性化的需求。

后面我们会详细介绍在 Spring Boot 2.x 中基于 micrometer 实现我们自定义的 metrics

## 参考资料

{% embed url="https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-endpoints.html" %}

