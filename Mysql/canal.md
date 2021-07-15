Spring Cloud Stream是一个基于Spring Boot用于构建消息驱动的微服务的框架。

### **为什么需要SpringCloud Stream消息驱动呢？**

　　比方说我们用到了RabbitMQ和Kafka，由于这两个消息中间件的架构上的不同，像RabbitMQ有exchange，kafka有Topic，partitions分区，这些中间件的差异性导致我们实际项目开发给我们造成了一定的困扰，我们如果用了两个消息队列的其中一种，后面的业务需求，我想往另外一种消息队列进行迁移，这时候无疑就是一个灾难性的，一大堆东西都要重新推倒重新做，因为它跟我们的系统耦合了，这时候springcloud Stream给我们提供了一种解耦合的方式。

​    Spring Cloud Stream 是一个构建消息驱动微服务的框架。应用程序通过 inputs 或者 outputs 来与 Spring Cloud Stream 中binder 交互，通过我们配置来 binding ，而 Spring Cloud Stream 的 binder 负责与中间件交互。所以，我们只需要搞清楚如何与 Spring Cloud Stream 交互就可以方便使用消息驱动的方式。

​    Spring Cloud Stream由一个中间件中立的核组成。应用通过Spring Cloud Stream插入的input(相当于消费者consumer，它是从队列中接收消息的)和output(相当于生产者producer，它是从队列中发送消息的。)通道与外界交流。

通道通过指定中间件的Binder实现与外部代理连接。业务开发者不再关注具体消息中间件，只需关注Binder对应用程序提供的抽象概念来使用消息中间件实现业务即可。

## Binder

destinations:对应于 Kafka 的topic，Rabbit MQ 的 exchanges

## Consumer Groups

​    “Group”，如果使用过 Kafka 的童鞋并不会陌生。Spring Cloud Stream 的这个分组概念的意思基本和 Kafka 一致。

​    微服务中动态的缩放同一个应用的数量以此来达到更高的处理能力是非常必须的。对于这种情况，同一个事件防止被重复消费，只要把这些应用放置于同一个 “group” 中，就能够保证消息只会被其中一个应用消费一次。

### 基础配置

以`spring.cloud.stream.`为前缀

```yaml
# 应用程序部署的实例数量。当使用Kalka的时候需要设置分区
instanceCount:
# 默认绑定器配置，在应用程序中有多个绑定器时使用
default Binder: kafka
```

### 通用配置

对于绑定通道的通用配置，它们既适用于输入通道，也适用于输出通道，以`spring-cloud-stream.bindings.<channelName>`为前缀

```yaml
bindins:
  <channelName>:
    # kafka中topic或rabbit中的Exchange
    destination: test-topic
    # 消费者组
    group: group-1
    # 消费者配置
    consumer:
      # 并发数量
      concurrency: 1
      # 生产者是否使用了分区
      partitioned: false
      # 对输入通道消息处理的最大重试次数，默认为3
      maxAttempts: 3
      # 重试消息处理的初始间隔时间
      backOffInitialInterval: 1000
      # 重试消息处理的最大间隔时间
      backOffMaxInterval: 10000
      # 重试消息处理时间间隔的递增乘数
      backoffMultiplier: 2.0
    # 生产者配置
    procucer:  
```

### kafka配置

kafka绑定器配置，以`spring.cloud.stream.kafka.binder.`为前缀

```yaml
# kafka中间件列表，ip+端口，或者ip，使用默认端口，多个逗号分隔
brokers: 192.168.1.1:9092,192.168.1.2
# 默认端口号，上面未配置端口的默认使用此端口，默认9092
defaultBrokerport: 9092
# zookeeper节点，多个以,分隔
zkNodes:192.168.1.1：2181
# zookeeper默认端口，默认2181
defaultZkPort: 2181
# 自动创建主题，默认true
autoCreateTopics： true
```

#### 消费者配置

下面这些配置仅对Kafka输入通道的绑定有效,以`spring.cloud.stream.kafka.bindings.<channelName>.consumer.`格式作为前缀.

```yaml
# 自动提交offset，默认true
autoCommitOffset: true
```

#### 生产者配置

仅对Kaka输出通道的绑定有效,它们以 `spring. cloud. stream.kafka. bindings.< channelName>.producer.`格式作为前缀

```yaml
# 批量发送前，保存数据上限，字节为单位，默认16384
buffersize: 16384
# 默认false，批量，为true一条一条发送
sync：false
# 批量发送，为积累更多数据而设置的等待时间，默认为0
batchTimeout: 0
```

### StreamBridge

用于发送不是生产者发送的消息给消费者

```java
@SpringBootApplication
@Controller
public class WebSourceApplication {

	public static void main(String[] args) {
		SpringApplication.run(WebSourceApplication.class, "--spring.cloud.stream.source=toStream");
	}

	@Autowired
	private StreamBridge streamBridge;

	@RequestMapping
	@ResponseStatus(HttpStatus.ACCEPTED)
	public void delegateToSupplier(@RequestBody String body) {
		System.out.println("Sending " + body);
		streamBridge.send("toStream-out-0", body);
	}
}
```

上面示例，将rest请求的数据发送到`"toStream-out-0"`源

### 函数式编程

从3.0开始，基本上`@EnableBInding`、`@StreamListener `和所有相关的注解现在都被弃用了，取而代之的是函数式编程模型。

- input - `<functionName> + -in- + <index>`
- output - `<functionName> + -out- + <index>`

The `in` and `out` corresponds to the type of binding (such as *input* or *output*). The `index` is the index of the input or output binding. It is always 0 for typical single input/output function, so it’s only relevant for [Functions with multiple input and output arguments](https://docs.spring.io/spring-cloud-stream/docs/3.1.3/reference/html/spring-cloud-stream.html#_functions_with_multiple_input_and_output_arguments).

生产者默认每秒执行一次，可通过配置更改[定时刷新](https://docs.spring.io/spring-cloud-stream/docs/3.1.3/reference/html/spring-cloud-stream.html#_polling_configuration_properties)

The following properties are exposed by `org.springframework.cloud.stream.config.DefaultPollerProperties` and are prefixed with `spring.cloud.stream.poller`:

- fixedDelay

  刷新延迟，毫秒为单位，默认1000,即1s

- maxMessagesPerPoll

  Maximum messages for each polling event of the default poller.Default: 1L.

- cron

  Cron表达式.Default: none.

- initialDelay

  Initial delay for periodic triggers.Default: 0.

- timeUnit

  The TimeUnit to apply to delay values.Default: MILLISECONDS.

For example `--spring.cloud.stream.poller.fixed-delay=2000` sets the poller interval to poll every two seconds.
