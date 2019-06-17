# 长任务计时：Long task timers

LongTaskTimer也是一种Timer的特化类型，主要用于记录长时间执行的任务的持续时间，在任务完成之前，被监测的事件或者任务仍然处于运行状态，任务完成的时候，任务执行的总耗时才会被记录下来。LongTaskTimer适合用于长时间持续运行的事件耗时的记录，例如相对耗时的定时任务。在Spring应用中，可以简单地使用@Scheduled和@Timed注解，基于spring-aop完成定时调度任务的总耗时记录：

```java
@Timed(value = "aws.scrape", longTask = true)
@Scheduled(fixedDelay = 360000)
void scrapeResources() {
    //这里做相对耗时的业务逻辑
}
```

当然，在非spring体系中也能方便地使用LongTaskTimer：

```java
public class LongTaskTimerMain {

	public static void main(String[] args) throws Exception{
		MeterRegistry meterRegistry = new SimpleMeterRegistry();
		LongTaskTimer longTaskTimer = meterRegistry.more().longTaskTimer("longTaskTimer");
		longTaskTimer.record(() -> {
			
			//这里编写Task的逻辑
		});
		//或者这样
		Metrics.more().longTaskTimer("longTaskTimer").record(()-> {
			//这里编写Task的逻辑
		});
	}
}
```

