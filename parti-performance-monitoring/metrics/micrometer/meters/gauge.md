# 仪表：Gauge

 Gauge\(仪表\)是获取当前度量记录值的句柄，也就是它表示一个可以任意上下浮动的单数值度量Meter。Gauge通常用于变动的测量值，测量值用`ToDoubleFunction`参数的返回值设置，如当前的内存使用情况，同时也可以测量上下移动的”计数”，比如队列中的消息数量。官网文档中提到Gauge的典型使用场景是用于测量集合或映射的大小或运行状态中的线程数。Gauge一般用于监测有自然上界的事件或者任务，而Counter一般使用于无自然上界的事件或者任务的监测，所以像Http请求总量计数应该使用Counter而非Gauge。`MeterRegistry`中提供了一些便于构建用于观察数值、函数、集合和映射的Gauge相关的方法：

```java
List<String> list = registry.gauge("listGauge", Collections.emptyList(), new ArrayList<>(), List::size); 
List<String> list2 = registry.gaugeCollectionSize("listSize2", Tags.empty(), new ArrayList<>()); 
Map<String, Integer> map = registry.gaugeMapSize("mapGauge", Tags.empty(), new HashMap<>());
```

 上面的三个方法通过`MeterRegistry`构建Gauge并且返回了集合或者映射实例，使用这些集合或者映射实例就能在其size变化过程中记录这个变更值。更重要的优点是，我们不需要感知Gauge接口的存在，只需要像平时一样使用集合或者映射实例就可以了。此外，Gauge还支持`java.lang.Number`的子类，`java.util.concurrent.atomic`包中的`AtomicInteger`和`AtomicLong`，还有Guava提供的`AtomicDouble`：

```java
AtomicInteger n = registry.gauge("numberGauge", new AtomicInteger(0));
n.set(1);
n.set(2);
```

 除了使用`MeterRegistry`创建Gauge之外，还可以使用建造器流式创建：

```java
//一般我们不需要操作Gauge实例
Gauge gauge = Gauge
    .builder("gauge", myObj, myObj::gaugeValue)
    .description("a description of what this gauge does") // 可选
    .tags("region", "test") // 可选
    .register(registry);
```

**使用场景：**

根据个人经验和实践，总结如下：

* 1、有自然\(物理\)上界的浮动值的监测，例如物理内存、集合、映射、数值等。
* 2、有逻辑上界的浮动值的监测，例如积压的消息、\(线程池中\)积压的任务等，其实本质也是集合或者映射的监测。

举个相对实际的例子，假设我们需要对登录后的用户发送一条短信或者推送，做法是消息先投放到一个阻塞队列，再由一个线程消费消息进行其他操作：

```java
public class GaugeMain {

	private static final MeterRegistry MR = new SimpleMeterRegistry();
	private static final BlockingQueue<Message> QUEUE = new ArrayBlockingQueue<>(500);
	private static BlockingQueue<Message> REAL_QUEUE;

	static {
		REAL_QUEUE = MR.gauge("messageGauge", QUEUE, Collection::size);
	}

	public static void main(String[] args) throws Exception {
		consume();
		Message message = new Message();
		message.setUserId(1L);
		message.setContent("content");
		REAL_QUEUE.put(message);
	}

	private static void consume() throws Exception {
		new Thread(() -> {
			while (true) {
				try {
					Message message = REAL_QUEUE.take();
					//handle message
					System.out.println(message);
				} catch (InterruptedException e) {
					//no-op
				}
			}
		}).start();
	}
}
```

