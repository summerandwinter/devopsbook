# 分布摘要：Distribution summaries

Summary\(摘要\)主要用于跟踪事件的分布，在Micrometer 中，对应的类是DistributionSummary\(分布摘要\)。它的使用方式和Timer十分相似，但是它的记录值并不依赖于时间单位。常见的使用场景：使用DistributionSummary测量命中服务器的请求的有效负载大小。使用`MeterRegistry`创建DistributionSummary实例如下：

```java
istributionSummary summary = registry.summary("response.size");
```

通过建造器流式创建如下：

```java
DistributionSummary summary = DistributionSummary
    .builder("response.size")
    .description("a description of what this summary does") // 可选
    .baseUnit("bytes") // 可选
    .tags("region", "test") // 可选
    .scale(100) // 可选
    .register(registry);
```

DistributionSummary中有很多构建参数跟缩放和直方图的表示相关，见下一节。

**使用场景：**

根据个人经验和实践，总结如下：

* 1、不依赖于时间单位的记录值的测量，例如服务器有效负载值，缓存的命中率等。

举个相对具体的例子：

```java
public class DistributionSummaryMain {
	
	private static final DistributionSummary DS  = DistributionSummary.builder("cacheHitPercent")
			.register(new SimpleMeterRegistry());

	private static final LoadingCache<String, String> CACHE = CacheBuilder.newBuilder()
			.maximumSize(1000)
			.recordStats()
			.expireAfterWrite(60, TimeUnit.SECONDS)
			.build(new CacheLoader<String, String>() {
				@Override
				public String load(String s) throws Exception {
					return selectFromDatabase();
				}
			});

	public static void main(String[] args) throws Exception{
		String key = "doge";
		String value = CACHE.get(key);
		record();
	}
	
	private static void record()throws Exception{
		CacheStats stats = CACHE.stats();
		BigDecimal hitCount = new BigDecimal(stats.hitCount());
		BigDecimal requestCount = new BigDecimal(stats.requestCount());
		DS.record(hitCount.divide(requestCount,2,BigDecimal.ROUND_HALF_DOWN).doubleValue());
	}
}
```



