# Quarzt

SpringBoot内置了简单的定时任务，可使用`@EnableScheduling`和`@Scheduled`

### 核心概念

- **Job**：接口，只定义一个方法 execute（JobExecutionContext context），在实现类的 execute 方法中编写所需要定时执行的 Job（任务），JobExecutionContext 类提供了调度应用的一些信息；Job 运行时的信息保存在 JobDataMap 实例中。
- **JobDetail**：Quartz 每次调度 Job 时，都会创建一个 Job 实例，它接收一个 Job 实现类（JobDetail，描述 Job 的实现类及其他相关的静态信息，如 Job 名字、描述、关联监听器等信息），以便运行时通过 newInstance() 的反射机制实例化 Job。

**==为什么设计成JobDetail + Job，不直接使用Job?==**

> JobDetail定义的是任务数据，真正的执行逻辑是在Job中。
> 这是因为任务是有可能并发执行，如果Scheduler直接使用Job，就会存在对同一个Job实例并发访问的问题。而JobDetail & Job 方式，Sheduler每次执行，都会根据JobDetail创建一个新的Job实例，这样就可以规避并发访问的问题。

- **Trigger**：触发器，描述触发 Job 执行的时间触发规则，主要有 SimpleTrigger 和 CronTrigger 这两个子类。一定时间内调度一次或者以固定时间间隔周期执行调度，SimpleTrigger 是适合的选择；而 CronTrigger 则可以通过 Cron 表达式定义出各种复杂时间规则的调度方案：如工作日周一到周五的 15：00 ~ 16：00 执行调度等。
- **Scheduler**：调度器，装载着任务和触发器，该类是一个接口，代表一个 Quartz 的独立运行容器，Trigger 和 JobDetail 可以注册到 Scheduler 中，两者在 Scheduler 中拥有各自的组及名称，组及名称是 Scheduler 查找定位容器中某一对象的依据，Trigger 的组及名称必须唯一，JobDetail 的组和名称也必须唯一（但可以和 Trigger 的组和名称相同，因为它们是不同类型的）。Scheduler 定义了多个接口方法，允许外部通过组及名称访问和控制容器中 Trigger 和 JobDetail。

### 依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-quartz</artifactId>
</dependency>
```

