# 计量器：Meters

计量器用来收集不同类型的性能指标信息。Micrometer 提供了不同类型的计量器实现。计量器对象由计量器注册表创建并管理。Micrometer 本身集成了如下这些计量器的实现： Timer、Counter、Gauge、DistributionSummary、LongTaskTimer、FunctionTimer、FunctionCounter、TimeGuage。不同类型的计量器会产生不同数量的指标（metrics），例如：Gauge 只有一个单独的指标，而 Timer 会收集事件的总数量和总时间。

Micrometer中，Meter的命名约定使用英文逗号\(dot，也就是”.”\)分隔单词。但是对于不同的监控系统，对命名的规约可能并不相同，如果命名规约不一致，在做监控系统迁移或者切换的时候，可能会对新的系统造成破坏。Micrometer 中使用英文逗号分隔单词的命名规则，再通过底层的命名转换接口 NamingConvention 进行转换，最终可以适配不同的监控系统，同时可以消除监控系统不允许的特殊字符的名称和标记等。开发者也可以覆盖 NamingConvention 实现自定义的命名转换规则：`registry.config().namingConvention(myCustomNamingConvention);`。在Micrometer 中，对一些主流的监控系统或者存储系统的命名规则提供了默认的转换方式，例如当我们使用下面的命名时候：

```java
MeterRegistry registry = ...
registry.timer("http.server.requests");
```

对于不同的监控系统或者存储系统，命名会自动转换如下：

* Prometheus - http\_server\_requests\_duration\_seconds
* Atlas - httpServerRequests
* Graphite - http.server.requests
* InfluxDB - http\_server\_requests

其实NamingConvention已经提供了5种默认的转换规则：dot、snakeCase、camelCase、upperCamelCase和slashes。

另外，Tag\(标签\)是Micrometer的一个重要的功能，严格来说，一个度量框架只有实现了标签的功能，才能真正地多维度进行度量数据收集。Tag的命名一般需要是有意义的，所谓有意义就是可以根据Tag的命名可以推断出它指向的数据到底代表什么维度或者什么类型的度量指标。假设我们需要监控数据库的调用和Http请求调用统计，一般推荐的做法是：

```java
MeterRegistry registry = ...
registry.counter("database.calls", "db", "users")
registry.counter("http.requests", "uri", "/api/users")
```

这样，当我们选择命名为”database.calls”的计数器，我们可以进一步选择分组”db”或者”users”分别统计不同分组对总调用数的贡献或者组成。一个反例如下：

```java

MeterRegistry registry = ...
registry.counter("calls",
    "class", "database",
    "db", "users");

registry.counter("calls",
    "class", "http",
    "uri", "/api/users");
```

通过命名”calls”得到的计数器，由于标签混乱，数据是基本无法分组统计分析，这个时候可以认为得到的时间序列的统计数据是没有意义的。可以定义全局的Tag，也就是全局的Tag定义之后，会附加到所有的使用到的Meter上\(只要是使用同一个MeterRegistry\)，全局的Tag可以这样定义：

```java
MeterRegistry registry = ...
registry.config().commonTags("stack", "prod", "region", "us-east-1");
// 和上面的意义是一样的
registry.config().commonTags(Arrays.asList(Tag.of("stack", "prod"), Tag.of("region", "us-east-1")));
```

像上面这样子使用，就能通过主机，实例，区域，堆栈等操作环境进行多维度深入分析。

还有两点点需要注意：

* Tag的值必须不为null。
* Micrometer中，Tag必须成对出现，也就是Tag必须设置为偶数个，实际上它们以Key=Value的形式存在，具体可以看io.micrometer.core.instrument.Tag接口：

```java
public interface Tag extends Comparable<Tag> {
    String getKey();

    String getValue();

    static Tag of(String key, String value) {
        return new ImmutableTag(key, value);
    }

    default int compareTo(Tag o) {
        return this.getKey().compareTo(o.getKey());
    }
}
```

当然，有些时候，我们需要过滤一些必要的标签或者名称进行统计，或者为Meter的名称添加白名单，这个时候可以使用MeterFilter。MeterFilter本身提供一些列的静态方法，多个MeterFilter可以叠加或者组成链实现用户最终的过滤策略。例如：

```java
MeterRegistry registry = ...
registry.config()
    .meterFilter(MeterFilter.ignoreTags("http"))
    .meterFilter(MeterFilter.denyNameStartsWith("jvm"));
```

表示忽略”http”标签，拒绝名称以”jvm”字符串开头的Meter。更多用法可以参详一下MeterFilter这个类。

Meter的命名和Meter的Tag相互结合，以命名为轴心，以Tag为多维度要素，可以使度量数据的维度更加丰富，便于统计和分析。

