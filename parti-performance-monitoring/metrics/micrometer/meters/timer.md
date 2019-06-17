# 计时器：Timer



Timer\(计时器\)适用于记录耗时比较短的事件的执行时间，通过时间分布展示事件的序列和发生频率。所有的Timer的实现至少记录了发生的事件的数量和这些事件的总耗时，从而生成一个时间序列。Timer的基本单位基于服务端的指标而定，但是实际上我们不需要过于关注Timer的基本单位，因为Micrometer在存储生成的时间序列的时候会自动选择适当的基本单位。Timer接口提供的常用方法如下：

```java
public interface Timer extends Meter {
    ...
    void record(long var1, TimeUnit var3);

    default void record(Duration duration) {
        this.record(duration.toNanos(), TimeUnit.NANOSECONDS);
    }

    <T> T record(Supplier<T> var1);

    <T> T recordCallable(Callable<T> var1) throws Exception;

    void record(Runnable var1);

    default Runnable wrap(Runnable f) {
        return () -> {
            this.record(f);
        };
    }

    default <T> Callable<T> wrap(Callable<T> f) {
        return () -> {
            return this.recordCallable(f);
        };
    }

    long count();

    double totalTime(TimeUnit var1);

    default double mean(TimeUnit unit) {
        return this.count() == 0L ? 0.0D : this.totalTime(unit) / (double)this.count();
    }

    double max(TimeUnit var1);
	...
}
```

实际上，比较常用和方便的方法是几个函数式接口入参的方法：

```java
Timer timer = ...
timer.record(() -> dontCareAboutReturnValue());
timer.recordCallable(() -> returnValue());

Runnable r = timer.wrap(() -> dontCareAboutReturnValue());
Callable c = timer.wrap(() -> returnValue());
```

**使用场景：**

根据个人经验和实践，总结如下：

* 1、记录指定方法的执行时间用于展示。
* 2、记录一些任务的执行时间，从而确定某些数据来源的速率，例如消息队列消息的消费速率等。

这里举个实际的例子，要对系统做一个功能，记录指定方法的执行时间，还是用下单方法做例子：

```java
public class TimerMain {

	private static final Random R = new Random();

	static {
		Metrics.addRegistry(new SimpleMeterRegistry());
	}

	public static void main(String[] args) throws Exception {
		Order order1 = new Order();
		order1.setOrderId("ORDER_ID_1");
		order1.setAmount(100);
		order1.setChannel("CHANNEL_A");
		order1.setCreateTime(LocalDateTime.now());
		Timer timer = Metrics.timer("timer", "createOrder", "cost");
		timer.record(() -> createOrder(order1));
	}

	private static void createOrder(Order order) {
		try {
			TimeUnit.SECONDS.sleep(R.nextInt(5)); //模拟方法耗时
		} catch (InterruptedException e) {
			//no-op
		}
	}
}
```

 在实际生产环境中，可以通过`spring-aop`把记录方法耗时的逻辑抽象到一个切面中，这样就能减少不必要的冗余的模板代码。上面的例子是通过Mertics构造Timer实例，实际上也可以使用Builder构造：

```java
MeterRegistry registry = ...
Timer timer = Timer
    .builder("my.timer")
    .description("a description of what this timer does") // 可选
    .tags("region", "test") // 可选
    .register(registry);
```

 另外，Timer的使用还可以基于它的内部类`Timer.Sample`，通过start和stop两个方法记录两者之间的逻辑的执行耗时。例如：

```java
Timer.Sample sample = Timer.start(registry);

// 这里做业务逻辑
Response response = ...

sample.stop(registry.timer("my.timer", "response", response.status()));

```

