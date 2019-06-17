# 介绍

## 概览

运行良好的应用离不开对性能指标的收集。这些性能指标可以有效地对生产系统的各方面行为进行监控，帮助运维人员掌握系统运行状态和查找问题原因。性能指标监控通常由两个部分组成：第一个部分是性能指标数据的收集，需要在应用程序代码中添加相应的代码来完成；另一个部分是后台监控系统，负责对数据进行聚合计算和提供 API 接口。在应用中使用计数器、计量仪和计时器来记录关键的性能指标。在专用的监控系统中对性能指标进行汇总，并生成相应的图表来进行可视化分析。

目前已经有非常多的监控系统，常用的如 [Prometheus](https://prometheus.io)、[New Relic](https://newrelic.com/)、[Influx](https://www.influxdata.com/)、[Graphite ](https://graphite.readthedocs.io/en/latest/)和 [Datadog](https://www.datadoghq.com/)，每个系统都有自己独特的数据收集方式。这些监控系统有的是需要自主安装的软件，有的则是云服务。它们的后台实现千差万别，数据接口也是各有不同。在指标数据收集方面，大多数时候都是使用与后台监控系统对应的客户端程序。此外，这些监控系统一般都会提供不同语言和平台使用的第三方库，这不可避免的会带来供应商锁定的问题。一旦针对某监控系统的数据收集代码添加到应用程序中，当需要切换监控系统时，也要对应用程序进行大量的修改。Micrometer 的出现恰好解决了这个问题，其作用可以类比于 SLF4J 在 Java 日志记录中的作用。

##  简介

Micrometer 为 Java 平台上的性能数据收集提供了一个通用的 API，应用程序只需要使用 Micrometer 的通用 API 来收集性能指标即可。Micrometer 会负责完成与不同监控系统的适配工作。这就使得切换监控系统变得很容易。Micrometer 还支持推送数据到多个不同的监控系统。

在 Java 应用中使用 Micrometer 非常的简单。只需要在 Maven 或 Gradle 项目中添加相应的依赖即可。Micrometer 包含如下三种模块，分组名称都是 io.micrometer：

* 包含数据收集 SPI 和基于内存的实现的核心模块 `micrometer-core`。
* 针对不同监控系统的实现模块，如针对 Prometheus 的 `micrometer-registry-prometheus`。
* 与测试相关的模块 `micrometer-test`。

在 Java 应用中，只需要根据所使用的监控系统，添加所对应的模块即可。比如，使用 Prometheus 的应用只需要添加 `micrometer-registry-prometheus` 模块即可。模块 `micrometer-core` 会作为传递依赖自动添加。这里使用的 Micrometer 版本是 1.1.4。下面给出了使用 Micrometer 的 Maven 项目的示例：

{% code-tabs %}
{% code-tabs-item title="pom.xml" %}
```markup
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
    <version>1.1.4</version>
</dependency>

```
{% endcode-tabs-item %}
{% endcode-tabs %}

