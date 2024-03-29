# 计量器注册表：Registry

 Micrometer 中有两个最核心的概念，分别是计量器（Meter）和计量器注册表（MeterRegistry）。计量器表示的是需要收集的性能指标数据，而计量器注册表负责创建和维护计量器。每个监控系统有自己独有的计量器注册表实现。模块 micrometer-core 中提供的类 SimpleMeterRegistry 是一个基于内存的计量器注册表实现。SimpleMeterRegistry 不支持导出数据到监控系统，主要用来进行本地开发和测试。

Micrometer 支持多个不同的监控系统。通过计量器注册表实现类 CompositeMeterRegistry 可以把多个计量器注册表组合起来，从而允许同时发布数据到多个监控系统。对于由这个类创建的计量器，它们所产生的数据会对 CompositeMeterRegistry 中包含的所有计量器注册表都产生影响。

 Micrometer 本身还提供了一个静态的全局计量器注册表对象 Metrics.globalRegistry。该注册表是一个组合注册表。使用 Metrics 类中的静态方法创建的计量器，都会被添加到该全局注册表中。对于大多数应用来说，这个全局注册表对象就可以满足需求，不需要额外创建新的注册表对象。需要注意的是由于该对象是静态的，在某些场合，尤其是进行单元测试时，会产生一些问题。

