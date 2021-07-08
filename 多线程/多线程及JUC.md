# 多线程及JUC

### 多线程

##### 状态

![image-20201112114507254](E:%5CJava%5C%E8%B5%84%E6%96%99%5C%E8%B5%84%E6%96%99%5CJUC.assets%5Cimage-20201112114507254.png)

**New**：新建状态，Thread t = new MyThread()，此时为新建状态

**Runable**：就绪状态，t.start()后进入此状态，只是说明此线程已经做好了准备，==随时等待CPU调度执行==，并不是说执行了t.start()此线程立即就会执行；

**Running**：运行状态，当CPU开始调度处于就绪状态的线程时，==此时线程才得以真正执行==，即进入到运行状态。

>   注：就绪状态是进入到运行状态的唯一入口，也就是说，线程要想进入运行状态执行，首先必须处于就绪状态中；

**Blocked**：阻塞状态，处于运行状态中的线程由于某种原因，==暂时放弃对CPU的使用权==，停止执行，此时进入阻塞状态，直到其进入到就绪状态，才有机会再次被CPU调用以进入到运行状态。根据阻塞产生的原因不同，阻塞状态又可以分为三种：

1.  等待阻塞：运行状态中的线程执行==wait()方法==，使本线程进入到等待阻塞状态，==需要notify()==；

2.  同步阻塞：线程在获取synchronized同步锁失败(因为锁被其它线程所占用)，它会进入同步阻塞状态，==如果想从阻塞状态进入就绪状态必须得获取到其他线程所持有的锁。==

3.  其他阻塞：通过调用线程的sleep()或join()或发出了I/O请求时，线程会进入到阻塞状态，==**不会释放锁**==。==当sleep()状态超时、join()等待线程终止或者超时、或者I/O处理完毕时，线程重新转入就绪状态。==

    ==休眠时由运行态到阻塞态；休眠结束后由阻塞态到就绪态==

**Terminated**：死亡状态，线程执行完了或者因异常退出了run()方法，该线程结束生命周期。

##### 方法

```java
// 休眠指定时间，进入阻塞状态，时间过后恢复就绪状态
public static void sleep(long millis);

// 该线程先执行，之后其他的在执行
public final void join();

// 让出当前cpu执行权，重回就绪状态
public static void yield();
    
// 中断线程
public void interrupt();

//Object中的方法
wait();
notify();
notifyAll();
```

为什么wait()，notify()，notifyAll()方法要定义在Object中？

因为任意对象都可以作为锁

##### 方式1

1.  继承Thread类
2.  重写其run()方法
3.  调用start()方法，开启线程

```java
class Demo1 extends Thread {

    private String name;

    public Demo1() {
    }

    public Demo1(String name) {
        this.name = name;
    }

    @Override
    public void run() {
        for (int i = 0; i < 1000; i++) {
            System.out.println(name + "从1数到100:" + i);
        }
    }
}


public class JucApplicationTests {

    public static void main(string[] args) {

        Demo1 demo1 = new Demo1("张三");
        Demo1 demo2 = new Demo1("李四");
        demo1.start();
        demo2.start();

    }

}
```

>   调用start与run的方法的区别
>
>   直接调用run方法，相当于主线程直接调用thread类中的run方法，就是普通的方法调用，就像是调用一个普通类中一个叫run的方法一样，而start则会先开启一个线程，然后调用run方法，此时才是开启了多线程。

##### 方式2

实现**Runnable**接口

1.  自定义类实现**Runnable**接口
2.  覆盖重写run方法
3.  把runable类传递给thread的构造方法
4.  调用线程的start方法

```java
public class Demo3 {
    public static void main(String[] args) {


        Thread thread = new Thread(new MyRunnable());
        
        thread.start();

        // 拉姆达表达式方式
        new Thread(()->{
            for (int i = 0; i < 10000; i++) {
                System.out.println("111从1数到10000:" + i);
            }
        }).start();

    }
}
// 实现接口
class MyRunnable implements Runnable {
    @Override
    public void run() {
        for (int i = 0; i < 10000; i++) {
            System.out.println("从1数到10000:" + i);
        }
    }
}
```

>   与继承thread方式的好处
>
>   1.  没有了java单继承的局限性
>
>   2.  更符合面向对象，将线程对象进行了 封装
>
>   3.  Runnable接口出现，降低了线程对象和线程任务的耦合性。

##### synchronized

Synchronized一句话来解释其作用就是：==能够保证同一时刻最多只有一个线程执行该段代码，以达到并发安全的效果。==

同步代码块

```java
synchronized(obj) {
    ...
}
```

同步函数

```java
public synchronized void add(int n) {
    ...
}
```

>   ==**同步函数使用的锁就是this。**==调用函数的对象本身。
>
>   ==**静态同步函数使用的锁是类名.class。**==

##### ReentrantLock

AQS（AbstractQueuedSynchronizer）：实现 ReentrantLock 的基础。

AQS 有一个 state 标记位，值为1  时表示有线程占用，其他线程需要进入到同步队列等待，同步队列是一个双向链表。

当获得锁的线程需要等待某个条件时，会进入 condition 的等待队列，等待队列可以有多个。

当 condition 条件满足时，线程会从等待队列重新进入同步队列进行获取锁的竞争。

ReentrantLock 就是基于 AQS 实现的

![image-20201112170608396](E:%5CJava%5C%E8%B5%84%E6%96%99%5C%E8%B5%84%E6%96%99%5CJUC.assets%5Cimage-20201112170608396.png)

Lock：锁的总接口，用于代替synchronzied，实现类ReentrantLock,ReentrantReadWritelock.ReadLock(读锁), ReentrantReadWriteLock.WriteLock(写锁)

Condition：用来代替notify和notifyAll

```java
final Lock lock = new ReentrantLock();	//获取锁
lock.lock();	//获取锁
lock.tryLock();	//尝试获取锁，返回是否成功获取锁
tryLock(long time, TimeUnit unit);	//尝试获取锁，指定时间内未获取，则返回false
lockInterruptibly();	//可中断锁，若锁已被获取，可使用threadB.interrupt()方法中断
lock.unlock();	//释放锁

final Condition condition1 = lock.newCondition();	//获取该锁的监视器状态对象
final Condition condition2 = lock.newCondition();	//可以有多个对象，实现指定唤醒功能

condition1.await(); 	//代替wait  
condition1.signal(); //代替notify
condition1.signalAll();	//代替notifyAll
```

>   **==注意==**
>
>   **多线程间的条件判断必须用while循环来判断，防止虚假唤醒**
>
>   **使用Lock必须在try…catch…块中进行，并且将释放锁的操作放在finally块中进行，以保证锁一定被被释放，防止死锁的发生。**

```java
Lock lock = ...;
lock.lock();
try{
    //处理任务
}catch(Exception ex){

}finally{
    lock.unlock();   //释放锁
}
```



### JUC

JUC即是java.util.concurrent，在并发编程中使用的工具类。

*   java.util.concurrent
*   java.util.concurrent.atomic
*   java.util.concurrent.locks

之前获取多线程，继承Thread类，或实现Runable接口

##### 集合

```java
// 线程安全的arraylist，也可以转换安全的set和map，但是一般不用,更不会用Vector
List list = Collections.synchronizedList(new ArrayList<>());

//线程安全的list
CopyOnWriteArrayList<E>();
//set
CopyOnWriteArrayset();
//map
ConcurrentHashMap();
```

##### 方式3

使用Callable接口

1.  实现Callable接口
2.  创建FutureTask(Runnable的实现类)，并将Callable的实现类传递给其构造方法
3.  使用Thread的Thread(Runnable r)构造方法

```java
public class Demo4 {
    public static void main(String[] args) throws ExecutionException, InterruptedException {

        FutureTask<String> futureTask = new FutureTask<String>(new MyCallable());

        new Thread(futureTask).start();

        // FutureTask的get方法可以获取未来任务的返回值，即是下面的"hello"
        System.out.println("futureTask.get() = " + futureTask.get());


    }
}

class MyCallable implements Callable<String> {
    @Override
    public String call() throws Exception {
        System.out.println("callable接口");
        return "hello";
    }
}
```

>   面试题：callable接口与runnable接口的区别？
>
>   1.  是否有返回值
>   2.  泛型
>   3.  是否抛异常
>   4.  落地方法不一样，一个是run，一个是call

##### BlockQueue

阻塞队列，当队列满时，可以将队列阻塞，移除一个时将另一个添加入队列。

add(),remove()：当队列满或空时，抛出异常

offer(),poll()：当队列满或空时，阻塞等待

offer(time),poll(time)：超时等待，等待超时，返回

##### ==**线程池**==

Executor	线程池接口

ExecutorService	线程池子接口

ThreadPoolExecutor	线程池的实际操作类

![image-20201013091421621](E:%5CJava%5C%E8%B5%84%E6%96%99%5C%E8%B5%84%E6%96%99%5CJUC.assets%5Cimage-20201013091421621.png)

```java
// Executors是线程池的工具类，可以创建线程池，但阿里巴巴手册不推荐，其内部也是用了ThreadPoolExecutor的构造方法
ExecutorService threadPool= Executors.newFixedThreadPool(nThreads:5);
Executors.newSingleThreadExecutor();// 单线程线程池
Executors.newCachedThreadPool();// 可缓存线程池，可根据请求数量进行扩容
```

>   【强制】==**线程池不允许使用 Executors 去创建，而是通过 ThreadPoolExecutor 的方式**==，这 样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险。 
>
>   说明：Executors 返回的线程池对象的弊端如下： 
>
>   ​	1） FixedThreadPool 和 SingleThreadPool： 允许的请求队列长度为 Integer.MAX_VALUE，可能会堆积大量的请求，从而导致 OOM。
>
>    	2） CachedThreadPool： 允许的创建线程数量为 Integer.MAX_VALUE，可能会创建大量的线程，从而导致 OOM。

```java
// 线程池的重要参数
public ThreadPoolExecutor(int corePoolSize,	// 线程池中的 常驻核心线程数
                          int maximumPoolsize,	// 线程池中的最大允许线程数
                          long keepAliveTime,	// 空闲线程的存活时间
                          TimeUnit unit,	// 存活时间的单位
                          BlockingQueue<Runnable> workQueue,	//任务队列，存储被提交但尚未被执行的任务
                          //SynchronousQueue、LinkedBlockingQueue(此时最大线程数无效)、ArrayBlockingQueue
                          ThreadFactory threadFactory,	// 用于生成线程池中工作线程的线程工厂，用于创建线程，一般默认的即可
                          RejectedExecutionHandler handler) {	// 拒绝策略，表示当队列满了，并且工作线程大于等于线程池的最大线程数（maximumPoolSize)时如何来拒绝请求执行的runnable的策略
    if(corePoolSize < 0 ||
       maximumPoolSize <= 0 ||
       maximumPoolsize< corepoolsize ||
       keepAliveTime < 0)
        throw new IllegalArgumentException();
        if(workoueue== null || threadFactory == null || handler == null)
            throw new NullPointerException();
    this.corePoolsize=corePoolSize;
    this.maximumPoolsize=maximumPoolSize;
    this.workQueue=workQueue;
    this.keepAliveTime=unit.toNanos(keepAliveTime);
    this.threadFactory=threadFactory;
    this.handler=handler；
}

ExecutorService threadPool= new ThreadPoolExecutor(...);
//线程池调用线程执行
//execute只能接受Runnable，submit还可以接收Callable
threaPool.execute(xxx);
threaPool.submit(xxx);
```

###### 原理

*   在创建了线程池后，开始等待请求。
*   当调用execute()方法添加一个请求任务时，线程池会做出如下判断：
    *   如果正在运行的线程数量小于corePoolSize，那么马上创建线程运行这个任务；
    *   如果正在运行的线程数量大于或等于corePoolSize，那么将这个任务放入队列
    *   如果这个时候队列满了且正在运行的线程数量还小于maximumPoolSize，那么扩容线程池并立刻运行这个任务；
    *   如果队列满了且正在运行的线程数量大于或等于maximumPoolSize，那么线程池会启动饱和拒绝策略来执行。

*   当一个线程完成任务时，它会从队列中取下一个任务来执行。
*   当一个线程无事可做超过一定的时间（keepAliveTime)时，线程会判断：
    *   如果当前运行的线程数大于corePoolSize，那么这个线程就被停掉。所以线程池的所有任务完成后，它最终会收缩到corePoolSize的大小。

###### 拒绝策略

即是当所有线程全部在工作，且等待队列也已经满时，新任务执行的策略：

*   **AbortPolicy(默认)**：直接抛出RejectedExecutionException异常阻止系统正常运行。
*   **CallerRunsPolicy**：调用者执行机制，将任务回退到调用者的线程来执行。
*   **DiscardOldestPolicy**：抛弃队列中等待最久的任务，执行当前任务。
*   **DiscardPolicy**：该策略默默地丢弃无法处理的任务，不予任何处理也不抛出异常。
    如果允许任务丢失，这是最好的一种策略。

```java
public class Demo6 {
    public static void main(String[] args) {
        ExecutorService threadPool = new ThreadPoolExecutor(
            5,
            10,
            60,
            TimeUnit.SECONDS,
            new ArrayBlockingQueue<>(10),
            Executors.defaultThreadFactory(),
            new ThreadPoolExecutor.AbortPolicy());

        /**
         * 线程数小于5，启动5个线程
         * 线程数大于五，小于核心线程+阻塞队列15，启动5个线程
         * 线程数大于15，小于20，多出15的部分，启动线程
         * 线程数大于20，根据拒绝策略，抛出异常
         */
        for (int i = 0; i < 21; i++) {
            threadPool.execute( () -> {
                try {
                    TimeUnit.MILLISECONDS.sleep(200);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + "------233333");
            });
        }

        threadPool.shutdown();

    }
}
```



##### 四大函数式接口

| 函数式接口                   | 参数类型 | 返回类型 | 用途                                                         |
| ---------------------------- | :------: | :------: | ------------------------------------------------------------ |
| Consumer<T><br>消费型接口    |    T     |   void   | 对类型为T的对象应用操作，包含方法：void accept（T t)         |
| Supplier<T><br/>供给型接口   |    无    |    T     | 返回类型为T的对象，包含方法：T get()：                       |
| Function<T,R><br/>函数型接口 |    T     |    R     | 对类型为T的对象应用操作，并返回结果。结果是R类型的对象。包含方法：R apply（T t)； |
| Predicate<T><br/>断定型接口  |    T     | boolean  | 确定类型T的对象是否满足某约束，并返回boolean值。包含方法     |

##### stream流

```java
List<Integer> list = Arrays.asList()
List<Integer> transactionsIds = widgets.stream()
             .filter(b -> b.getColor() == RED)
             .sorted((x,y) -> x.getWeight() - y.getWeight())
             .mapToInt(Widget::getWeight)
             .sum();
```

```properties
filter() : 过滤
map() : 进行转换，如值+1，大小写转换等
flatMap
max() : 最大值
min() : 最小值
count() : 计算数量
findFirst() : 第一个
findAny() : 任一个
collect() : 集合操作
	toList/toSet/toMap : 转为list/map/set
	.collect(Collectors.toSet())
	groupBy()
sorted() : 排序
distinct() : 去重
limit() : 前n个
skip() : 跳过前n个
concat() : 合并 
```

##### ForkJoinPool

ForkJoinTask：抽象类，用于分支处理，类似于递归的方式实现分治算法

RecursiveTask：ForkJoinTask的实现类，递归，实现compute方法

```java
class MyTask extends RecursiveTask<Integer> {
    private static final Integer ADJUST_VALUE=10;
    private int begin;
    private int end;
    private int result;

    public MyTask(int Hegin,int end) {
        this.begin=begin;
        this.end=end;
    }

    @Override
    protected Integer compute() {
        if((end-begin)<=ADJUST_VALUE) {
            for(inti=begin;i<=end；i++){
                result=result+i;
            }
        }else{
            int middle = (end+begin)/2;
            //调用
            MyTask taskel = new MyTask(begin,middle);
            MyTask taske2 = new MyTask(middle+1,end);
            // 递归调用
            taske1.fork();
            taske2.fork();
            // 将结果合并
            result = taskel.join() + taske2.join();
        }
        return result;
    }
}
```

```java
public class ForkJoinDemo {
    public static void main(string[] args) throws Exception {
        MyTask myTask=new MyTask(0,100);
        ForkJoinPool threadPool = new ForkJoinpool();
        ForkJoinTask<Integer> forkJoinTask= threadpool.submit(myTask);
        system.out.println(forkJoinrask.get());
        threadPool.shutdown();
    }
}
```

##### 异步回调

CompletableFuture类

```java
// 无需返回值的回调
CompletableFuture<Void> cf = CompletableFuture.runAsync(Runnable r);
cf.get();

//带返回值的回调
CompletableFuture<Integer> completableFuture1 = CompletableFuture.supplyAsync(()->{
            System.out.println("执行异步操作");
    		//得到返回值
            return 2333;
    	//得到返回值以后的操作，可以不用，直接使用get方法获取结果即可
        }).whenComplete((i,e)->{
            System.out.println("i = " + i);
            System.out.println("e = " + e);
    	//发生异常，为获取返回值的操作
        }).exceptionally(e->{
            System.out.println("fashengyichang" + e.getMessage());
            return null;
        });

        System.out.println("completableFuture1.get() = " + completableFuture1.get());
```

##### CountDownLatch

一个倒计数器

```java
CountDownLatch countDownLatch = new CountDownLatch(6);
countDownLatch.countDown();	//执行-1操作

//直到减到0，才开始执行await后面的任务
countDownLatch.await();
```

##### CyclicBarrier

一个计数器，数到指定数字，才开始执行指定任务

```java
// 要计数的个数，也可以指定Runnable表示到达指定个数以后的操作
CyclicBarrier c = new CyclicBarrier(int count);
CyclicBarrier c = new CyclicBarrier(int count, Runnable r);

//直到此方法执行到指定次数，才开始执行Runnable中的任务
c.await();
```

##### Semaphore

抢车位，较多的任务抢夺指定数目的执行资源，先到先得，只有空出了新的资源，才可以接着抢夺空出来的位置。

==**可用于线程数量的控制或多个资源的互斥使用**==

当资源数设置为1时，与synchronized类似

```java
// 需要指定资源的个数
Semaphore semaphore = new Semaphore(5);

//获取到资源后，资源剩余个数-1
semaphore.acquire();

//完成后，释放资源，资源个数+1
semaphore.release();
```

### jvm

类加载器

启动类加载器Bootstrap：加载/lib/rt.jar(runtime)

拓展类加载器Extension：加载/ext下的*.jar

应用类加载器AppClassLoader：加载类路径下的class

