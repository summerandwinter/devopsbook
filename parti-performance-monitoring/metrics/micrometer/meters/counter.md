# 计数器：Counter

 `Counter`是一种比较简单的Meter，它是一种单值的度量类型，或者说是一个单值计数器。`Counter`接口允许使用者使用一个固定值\(必须为正数\)进行计数。准确来说：`Counter`就是一个增量为正数的单值计数器。这个举个很简单的使用例子：

```java
MeterRegistry meterRegistry = new SimpleMeterRegistry();
Counter counter = meterRegistry.counter("http.request", "createOrder", "/order/create");
counter.increment();
System.out.println(counter.measure()); // [Measurement{statistic='COUNT', value=1.0}]

```

**使用场景：**

`Counter`的作用是记录XXX的总量或者计数值，适用于一些增长类型的统计，例如下单、支付次数、Http请求总量记录等等，通过Tag可以区分不同的场景，对于下单，可以使用不同的Tag标记不同的业务来源或者是按日期划分，对于Http请求总量记录，可以使用Tag区分不同的URL。用下单业务举个例子：

```java
//实体
@Data
public class Order {

	private String orderId;
	private Integer amount;
	private String channel;
	private LocalDateTime createTime;
}


public class CounterMain {

	private static final DateTimeFormatter FORMATTER = DateTimeFormatter.ofPattern("yyyy-MM-dd");

	static {
		Metrics.addRegistry(new SimpleMeterRegistry());
	}

	public static void main(String[] args) throws Exception {
		Order order1 = new Order();
		order1.setOrderId("ORDER_ID_1");
		order1.setAmount(100);
		order1.setChannel("CHANNEL_A");
		order1.setCreateTime(LocalDateTime.now());
		createOrder(order1);
		Order order2 = new Order();
		order2.setOrderId("ORDER_ID_2");
		order2.setAmount(200);
		order2.setChannel("CHANNEL_B");
		order2.setCreateTime(LocalDateTime.now());
		createOrder(order2);
		Search.in(Metrics.globalRegistry).meters().forEach(each -> {
			StringBuilder builder = new StringBuilder();
			builder.append("name:")
					.append(each.getId().getName())
					.append(",tags:")
					.append(each.getId().getTags())
					.append(",type:").append(each.getId().getType())
					.append(",value:").append(each.measure());
			System.out.println(builder.toString());
		});
	}

	private static void createOrder(Order order) {
		//忽略订单入库等操作
		Metrics.counter("order.create",
				"channel", order.getChannel(),
				"createTime", FORMATTER.format(order.getCreateTime())).increment();
	}
}
```

控制台输出：

```text
name:order.create,tags:[tag(channel=CHANNEL_A), tag(createTime=2018-11-10)],type:COUNTER,value:[Measurement{statistic='COUNT', value=1.0}]
name:order.create,tags:[tag(channel=CHANNEL_B), tag(createTime=2018-11-10)],type:COUNTER,value:[Measurement{statistic='COUNT', value=1.0}]
```

上面的例子是使用全局静态方法工厂类`Metrics`去构造Counter实例，实际上，`io.micrometer.core.instrument.Counter`接口提供了一个内部建造器类Counter.Builder去实例化Counter，Counter.Builder的使用方式如下：

```java
public class CounterBuilderMain {
	
	public static void main(String[] args) throws Exception{
		Counter counter = Counter.builder("name")  //名称
				.baseUnit("unit") //基础单位
				.description("desc") //描述
				.tag("tagKey", "tagValue")  //标签
				.register(new SimpleMeterRegistry());//绑定的MeterRegistry
		counter.increment();
	}
}
```

