

异步  解耦  削峰

异步：例如注册成功后，发送信息

解耦：订阅/发布，自行订阅需要的信息，解耦

削峰：如秒杀阶段，削峰

Producer生产者

Customer消费者

Kafka Cluster集群

Broker集群中的一个

Topic主题

Partition分区

Leader主

Follewer从

架构

[集成kafka](https://blog.csdn.net/yuanlong122716/article/details/105160545/)

```properties
# kafka集群，多个用逗号分隔
spring.kafka.bootstrap-servers=112.126.74.249:9092,112.126.74.249:9093
###########【初始化生产者配置】###########
# 重试次数
spring.kafka.producer.retries=0
# 应答级别:多少个分区副本备份完成时向生产者发送ack确认(可选0、1、all/-1)
spring.kafka.producer.acks=1
# 批量大小
spring.kafka.producer.batch-size=16384
# 提交延时
spring.kafka.producer.properties.linger.ms=0
# 当生产端积累的消息达到batch-size或接收到消息linger.ms后,生产者就会将消息提交给kafka
# linger.ms为0表示每接收到一条消息就提交给kafka,这时候batch-size其实就没用了
​
# 生产端缓冲区大小
spring.kafka.producer.buffer-memory = 33554432
# Kafka提供的序列化和反序列化类
spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.StringSerializer
spring.kafka.producer.value-serializer=org.apache.kafka.common.serialization.StringSerializer
# 自定义分区器
# spring.kafka.producer.properties.partitioner.class=com.felix.kafka.producer.CustomizePartitioner
​
###########【初始化消费者配置】###########
# 默认的消费组ID
spring.kafka.consumer.properties.group.id=defaultConsumerGroup
# 是否自动提交offset
spring.kafka.consumer.enable-auto-commit=true
# 提交offset延时(接收到消息后多久提交offset)
spring.kafka.consumer.auto.commit.interval.ms=1000
# 当kafka中没有初始offset或offset超出范围时将自动重置offset
# earliest:重置为分区中最小的offset;
# latest:重置为分区中最新的offset(消费分区中新产生的数据);
# none:只要有一个分区不存在已提交的offset,就抛出异常;
spring.kafka.consumer.auto-offset-reset=latest
# 消费会话超时时间(超过这个时间consumer没有发送心跳,就会触发rebalance操作)
spring.kafka.consumer.properties.session.timeout.ms=120000
# 消费请求超时时间
spring.kafka.consumer.properties.request.timeout.ms=180000
# Kafka提供的序列化和反序列化类
spring.kafka.consumer.key-deserializer=org.apache.kafka.common.serialization.StringDeserializer
spring.kafka.consumer.value-deserializer=org.apache.kafka.common.serialization.StringDeserializer
# 消费端监听的topic不存在时，项目启动会报错(关掉)
spring.kafka.listener.missing-topics-fatal=false
# 设置批量消费
# spring.kafka.listener.type=batch
# 批量消费每次最多消费多少条消息
# spring.kafka.consumer.max-poll-records=50
```

```yaml
spring:
  kafka:
    # kafka集群，多个用逗号分隔
    bootstrap-servers: 192.168.2.129:9092
    
    # 生产者配置
    producer:
      # 0,1,-1或all
      acks: 0
      # 重试次数
      retries: 3
      # producer可以用来缓存数据的内存大小
      buffer-memory: 33554432
      # key和value的序列化方式
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.apache.kafka.common.serialization.StringSerializer
      # 批量处理大小
      batch-size: 16384
      # 提交延时
      # 当生产端积累的消息达到batch-size或接收到消息linger.ms后,生产者就会将消息提交给kafka
# linger.ms为0表示每接收到一条消息就提交给kafka,这时候batch-size其实就没用了
      properties.linger.ms=0
	  # 当向server发出请求时，这个字符串会发送给server。目的是能够追踪请求源头，以此来允许ip/port许可列表之外的一些应用可以发送信息。这项应用可以设置任意字符串，因为没有任何功能性的目的，除了记录和跟踪
      client-id: client-gf-data
      bootstrap-servers: 192.168.2.129:9092
      
      
    consumer:
      enable-auto-commit: true
      auto-commit-interval: 100
      auto-offset-reset: earliest
      max-poll-records: 100
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      bootstrap-servers: 192.168.2.129:9092
      group-id: groupid-gf-dev
```

| acks      | producer吞吐量 | 消息持久性 | 使用场景                                                     |
| --------- | -------------- | ---------- | ------------------------------------------------------------ |
| 0         | 最高           | 最差       | ①完全不关心消息是否发送成功<br/>②允许消息丢失（比如计服务器日志等) |
| 1         | 适中           | 适中       | 一般场景即可                                                 |
| all 或一1 | 最差           | 最高       | 不能容忍消息丢失                                             |

