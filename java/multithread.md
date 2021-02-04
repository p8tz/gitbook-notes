> 环境：JDK11

## 一、创建线程

### 1、继承Thread

### 2、实现Runnable

### 3、实现Callable

依靠`FutureTask`获取返回值

```java
public static void main(String[] args) throws ExecutionException, InterruptedException, TimeoutException {
    Callable<String> c = new Callable<>() {
        @Override
        public String call() {
            try {
                TimeUnit.SECONDS.sleep(3);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "result";
        }
    };
    FutureTask<String> ft = new FutureTask<>(c);
    // FutureTask实现了Runnable
    Thread t = new Thread(ft);
    t.start();

    // 阻塞直到获取到返回值, 可响应中断
    System.out.println(ft.get());

    // 阻塞1秒, 如果没有获取到返回值, 抛TimeoutException
    System.out.println(ft.get(1, TimeUnit.SECONDS));
}
```

## 二、线程状态

一个线程只能处于一种状态，并且这里的线程状态特指 Java 虚拟机的线程状态，不能反映线程在特定操作系统下的状态。

![image-20201126225758381](https://gitee.com/p8t/picbed/raw/master/imgs/20201126225800.png)

### 新建（NEW）

创建后尚未启动。

### 可运行（RUNABLE）

正在 Java 虚拟机中运行。但是在操作系统层面，它可能处于运行状态，也可能等待资源调度（例如处理器资源），资源调度完成就进入运行状态。所以该状态的可运行是指可以被运行，具体有没有运行要看底层操作系统的资源调度。

### 阻塞（BLOCKED）

请求获取`monitor lock`从而进入 `synchronized`函数或者代码块，但是其它线程已经占用了该`monitor lock`，所以处于阻塞状态。要结束该状态进入从而`RUNABLE`需要其他线程释放`monitor lock`。

### 无限期等待（WAITING）

等待其它线程显式地唤醒。

阻塞和等待的区别在于，**阻塞是被动的**，它是在等待获取`monitor lock`。而等待是主动的，通过调用`Object.wait()`等方法进入。

| 进入方法                                     | 退出方法                             |
| -------------------------------------------- | ------------------------------------ |
| 没有设置` Timeout` 参数的`Object.wait()`方法 | `Object.notify()/Object.notifyAll()` |
| 没有设置 `Timeout`参数的`Thread.join()`方法  | 被调用的线程执行完毕                 |
| `LockSupport.park()` 方法                    | `LockSupport.unpark(Thread)`         |

### 限期等待（TIMED_WAITING）

无需等待其它线程显式地唤醒，在一定时间之后会被系统自动唤醒。

| 进入方法                                     | 退出方法                                            |
| -------------------------------------------- | --------------------------------------------------- |
| Thread.sleep() 方法                          | 时间结束                                            |
| 设置了 `Timeout `参数的 `Object.wait()` 方法 | 时间结束 / `Object.notify()` / `Object.notifyAll()` |
| 设置了 `Timeout` 参数的 `Thread.join()` 方法 | 时间结束 / 被调用的线程执行完毕                     |
| `LockSupport.parkNanos()` 方法               | `LockSupport.unpark(Thread)`                        |
| `LockSupport.parkUntil()` 方法               | `LockSupport.unpark(Thread)`                        |

调用 Thread.sleep() 方法使线程进入限期等待状态时，常常用“使一个线程睡眠”进行描述。调用 Object.wait() 方法使线程进入限期等待或者无限期等待时，常常用“挂起一个线程”进行描述。睡眠和挂起是用来描述行为，而阻塞和等待用来描述状态。

### 终结（TERMINATED）

可以是线程结束任务之后自己结束，或者产生了异常而结束。

## 三、线程池

### 1、生命周期

![image-20201128174300987](https://gitee.com/p8t/picbed/raw/master/imgs/20201128174302.png)

- `RUNNING`：线程池创建后的正常工作状态。可以接受新的任务, 可以处理已添加的任务
- `SHUTDOWN`：不接受新的任务，但可以处理已添加的任务
- `STOP`：不接受新的任务，不处理已添加的任务，并且**尝试中断**正在执行的任务
- `TIDYING`：当所有任务已被终止，任务数量为0，则会变为`TIDYING`状态，这时会调用`terminated()`。`terminated()`方法体为空，可以重写。
- `TERMINATED`：`terminated()`执行完，就会变为`TERMINATED`状态

**`shutdown()`**

拒绝接受新任务并把当前任务处理完，然后进入`TIDYING`状态

注：该方法非阻塞，即不会等到任务执行完才结束`shutdown()`方法。可以使用`awaitTermination()`阻塞等待

**`shutdownNow()`**

拒绝接受新任务，**尝试中断（给每个正在执行任务的线程调用`interrupt()`）**正在执行的任务，返回任务队列里还未开始的任务。非阻塞，使用`awaitTermination()`阻塞等待

### 2、线程池参数

线程池创建需要用到`ThreadPoolExecutor`类，全参构造函数如下

```java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler) {
    // ...
}
```

**`corePoolSize`**

核心线程数量。通过`prestartAllCoreThreads()`或者`prestartCoreThread()`在线程池启动时就开启全部或一个核心线程

**`maximumPoolSize`**

最大线程数量。任务超出核心线程数量时，先把新来的任务加到任务队列，如果任务队列也满了才创建非核心线程；如果非核心线程也满了，则把新来的任务执行任务拒绝策略（`RejectedExecutionHandler`）

**`keepAliveTime`**

非核心线程最大存活时间。如果允许核心线程自动销毁，使用`allowCoreThreadTimeOut()`设置

**`unit`**

`keepAliveTime`的时间单位

**`workQueue`**

任务队列组织形式，常见有以下几种

```java
// 数组形式组织, 必须赋初值
ArrayBlockingQueue;

// 链表形式组织, 默认容量: 0x7fffffff
LinkedBlockingQueue;

// 没有容量的任务队列, 就是用来转交任务的
SynchronousQueue;

// 优先级队列
PriorityBlockingQueue;
```

**`threadFactory`**

线程工厂，用于创建线程

**`handler`**

任务拒绝处理方式，默认提供了4种

```java
// 在调用者线程中(也就是说谁把这个任务甩来的), 运行当前被丢弃的任务。
// 只会用调用者所在线程来运行任务, 也就是说任务不会进入线程池.
// 如果线程池已经被关闭, 则直接丢弃该任务.
public static class CallerRunsPolicy implements RejectedExecutionHandler {
    public CallerRunsPolicy() { }
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
        if (!e.isShutdown()) {
            r.run();
        }
    }
}
// 丢弃任务, 抛出异常 (默认)
public static class AbortPolicy implements RejectedExecutionHandler {
    public AbortPolicy() { }
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
        throw new RejectedExecutionException("Task " + r.toString() +
                                             " rejected from " +
                                             e.toString());
    }
}
// 丢弃任务
public static class DiscardPolicy implements RejectedExecutionHandler {
    public DiscardPolicy() { }
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
    }
}
// 丢弃阻塞队列最前面的任务(也就是最老的), 然后再次尝试提交新任务
public static class DiscardOldestPolicy implements RejectedExecutionHandler {
    public DiscardOldestPolicy() { }
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
        if (!e.isShutdown()) {
            e.getQueue().poll();
            e.execute(r);
        }
    }
}
```

### 3、线程池工作流程

![image-20201128194817592](https://gitee.com/p8t/picbed/raw/master/imgs/20201128194818.png)

### 4、内置线程池

通过`Executors`工具类创建

**`Executors.newFixedThreadPool()`**

```java
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>(),
                                  threadFactory);
}
```

- 核心线程数等于最大线程数
- 使用链表组织任务队列

**`Executors.newSingleThreadExecutor()`**

```java
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>(),
                                threadFactory));
}
```

- 核心线程数与最大线程数都等于1
- 使用链表组织任务队列

**`Executors.newCachedThreadPool()`**

```java
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>(),
                                  threadFactory);
}
```

- 核心线程数为0，不限制最大线程数（目前环境没可能超过`Integer.MAX_VALUE`），使用`SynchronousQueue`来让每个任务都得到执行，也就是不存在有任务停留在任务队列中的情况
- 线程空闲时间为60秒
- 慎用，线程创建过多反而会影响效率

**`Executors.newScheduledThreadPool()`**

```java
public static ScheduledExecutorService newScheduledThreadPool(
    int corePoolSize) {
    return new ScheduledThreadPoolExecutor(corePoolSize);
}
// ScheduledThreadPoolExecutor extends ThreadPoolExecutor
public ScheduledThreadPoolExecutor(int corePoolSize) {
    super(corePoolSize, Integer.MAX_VALUE,
          // DEFAULT_KEEPALIVE_MILLIS = 10
          // JDK8里面这块是0, 这个改动算是一点小优化, 详见JavaDoc
          DEFAULT_KEEPALIVE_MILLIS, MILLISECONDS,
          new DelayedWorkQueue());
}
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue) {
    this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
         Executors.defaultThreadFactory(), defaultHandler);
}
```

- 不限制最大线程数，空闲线程立即被销毁

- 两种定时策略，图中`task`长度表示时间跨度

  ![image-20201128203330123](https://gitee.com/p8t/picbed/raw/master/imgs/20201128203331.png)

### 5、使用示例

```java
public static void main(String[] args) throws ExecutionException, InterruptedException {
    ExecutorService es = Executors.newFixedThreadPool(5);
    Callable<String> c = () -> "result";
    // submit提交任务返回Future
    Future<?> f = es.submit(c);
    System.out.println(f.get());
    // execute提交任务无返回值
    es.execute(() -> System.out.println("task"));
    // 但仍然可以使用execute()获取方法返回值
    FutureTask<String> ft = new FutureTask<>(c);
    es.execute(new Thread(ft));
    System.out.println(ft.get());
    es.shutdown();
}
```

## 四、Thread方法

```java
// 等待别的线程执行完, 可以设置等待时间
void join();
// 当前线程失去CPU, 重新竞争
static void yield();
// 休眠一段时间, 不会释放synchronized锁
static void sleep();
// 中断线程, 只能单纯的把中断标志位置为true
void interrupt();
// 判断是否被中断
boolean isInterrupted()
// 判断是否被中断, 并清除中断状态
static boolean interrupted();

// 已废弃方法
void stop();    // 停止当前线程
void suspend(); // 挂起线程
void resume();  // 恢复线程
```

## 五、中断

在Thread中提供了与中断相关的方法，除了手动判断中断标志位外，以下方法也会响应中断

- `Thread.sleep()`
- `Thread.join()`
- `Object.wait()`
- `Condition.await()`

## 六、线程同步与互斥

### synchronized

#### 1、使用

`synchronized`实现了原子性，可见性，有序性。可以加在以下地方

- 加载静态方法上，锁当前类对象
- 加载普通方法上，锁当前对象
- 加载代码块上，锁指定对象

#### 2、原理

对应字节码指令为`monitorenter`和`monitorexit`，并且会有两个后者，因为出异常会自动释放锁，这也是`monitorexit`的一种方式

##### 锁升级

JDK1.6之前，`synchronized`是操作系统级别的锁，是重量级锁，效率较低

JDK1.6，官方进行了大量优化。变为四种锁，依次升级

- 无锁
  - 锁对象刚被new出来的时候，如果没有被访问，则相当于没加锁。`JVM`启动4秒后升级为偏向锁
- 偏向锁
  - 第一次被访问时，锁对象的对象头会记录线程ID（相当于贴了个标签，表示有人用这个代码块），如果该线程再次进入该代码块只需要看所对象记录的线程ID是否一致
  - 偏向锁可以关闭，默认是开启的，并且在`JVM`启动4秒后升级，这个时间参数也可以修改
- 轻量级锁 / 自旋锁
  - 一旦有竞争，撤销偏向锁，在自己的线程栈帧生成一个对象`Lock Record`，并通过`CAS`的方式让`Mark Word`指向`Lock Record`
  - `Lock Record`相当于`Mark Word`的拷贝
  - 注意：在有线程持有偏向锁的时候，升级为轻量级锁并不会导致该线程中断释放锁，只是外面抢锁的线程通过`CAS`把偏向锁改为了轻量级锁
- 重量级锁
  - 竞争加剧则升级为重量级锁，底层原理是操作系统的锁（`mutex`），涉及到用户态和内核态的转换。
  - 竞争加剧体现为，`CAS`时间长，过多的线程在`CAS`
    - 自旋超过10次
    - 等待线程数超过`CPU`核数一半
    - 上面两个可以在JVM参数中调整，但是已经被废弃了。目前是自适应自旋，`JVM`会自适应调整参数
  - 底层是一个任务队列，线程不需要像`CAS`一样循环抢锁，而是在队列中，不会占用`CPU`

**关于偏向锁的问题**

为什么需要延迟升级为偏向锁？

- 因为在JVM刚启动的时候，分配对象空间等操作本身就会产生大量竞争，没必要上偏向锁

开启偏向锁是否一定效率高？

- 否，正如上面问题的答案，如果明确知道一个资源会被大量竞争，则直接使用自旋锁会更好

没有线程访问，但是偏向锁已经启动，那么它偏向的是谁？

- 匿名偏向

##### 锁消除

锁消除是指虚拟机**JIT**在运行时，对一些代码要求同步，但是对被检测到不可能存在共享数据竞争的锁进行消除。锁消除的主要判定依据来源于逃逸分析的数据支持，如果判断到一段代码中，在堆上的所有数据都不会逃逸出去被其他线程访问到，那就可以把它们当作栈上数据对待，认为它们是线程私有的，同步加锁自然就无须再进行。

##### 锁粗化

如果一系列的连续操作都对同一个对象反复加锁和解锁，甚至加锁操作是出现在循环体之中的，那即使没有线程竞争，频繁地进行互斥同步操作也会导致不必要的性能损耗。如果虚拟机探测到有这样一串零碎的操作都对同一个对象加锁，将会把加锁同步的范围扩展（粗化）到整个操作序列的外部

### ReentrantLock

#### 使用

```java
public class T {
    ReentrantLock lock = new ReentrantLock();

    void task() {
        lock.lock();
        try {
            for (int i = 0; i < 10; i++) {
                System.out.print(i + " ");
            }
        } finally {
            lock.unlock();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        T t = new T();
        Thread t1 = new Thread(() -> t.task());
        Thread t2 = new Thread(() -> t.task());
        t1.start();
        t2.start();
    }
}
```

#### 原理

见后面的`AQS`

## 七、线程通信

###  wait() notify() notifyAll()

属于`Object`的方法，必须在`synchronzed`代码块内使用

- `wait()`会释放锁，能响应中断

###  await() signal() signalAll()

`JUC`包下`Condition`的方法，通过`Lock`获取，必须在`lock`代码块内使用

### 例子

生产者消费者模式，下面是错误的异常处理方式

```java
public class E {
    public static void main(String[] args) throws InterruptedException {
        ReentrantLock lock = new ReentrantLock();
        Condition c = lock.newCondition();
        ArrayDeque<String> platform = new ArrayDeque<>();
        Consumer consumer = new Consumer(platform, lock, c);
        Supplier supplier = new Supplier(platform, lock, c);
        ExecutorService es = Executors.newFixedThreadPool(2);
        es.execute(() -> {
            try {
                consumer.consume();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        es.execute(() -> {
            try {
                supplier.supply();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
    }
}

class Consumer {
    Deque<String> platform;
    Lock lock;
    Condition c;

    public Consumer(Deque<String> platform, Lock lock, Condition c) {
        this.platform = platform;
        this.lock = lock;
        this.c = c;
    }

    public void consume() throws InterruptedException {
        while (true) {
            lock.lock();
            if (platform.isEmpty()) {
                c.await();
            }
            String gd = platform.poll();
            System.out.println("consume : " + gd);
            TimeUnit.SECONDS.sleep(1);
            c.signal();
            lock.unlock();
        }
    }
}

class Supplier {
    Deque<String> platform;
    Lock lock;
    Condition c;

    public Supplier(Deque<String> platform, Lock lock, Condition c) {
        this.platform = platform;
        this.lock = lock;
        this.c = c;
    }

    public void supply() throws InterruptedException {
        while (true) {
            lock.lock();
            if (!platform.isEmpty()) {
                c.await();
            }
            String gd = UUID.randomUUID().toString().substring(0, 5);
            platform.offer(gd);
            System.out.println("supply : " + gd);
            TimeUnit.SECONDS.sleep(1);
            c.signal();
            lock.unlock();
        }
    }
}
```

## 八、Java内存模型

### 主内存与工作内存

所有的变量都存储在主内存中，每个线程还有自己的工作内存，工作内存保存了该线程使用的变量的主内存副本拷贝。

线程只能直接操作工作内存中的变量，不同线程之间的变量值传递需要通过主内存来完成。

![image-20201129160738208](https://gitee.com/p8t/picbed/raw/master/imgs/20201129160739.png)

`Java`内存模型定义了 8 个操作来完成主内存和工作内存的交互操作。

![image-20201129160256770](https://gitee.com/p8t/picbed/raw/master/imgs/20201129160257.png)

- `read`：把一个变量的值从主内存传输到工作内存中
- `load`：在 `read `之后执行，把 `read `得到的值放入工作内存的变量副本中
- `use`：把工作内存中一个变量的值传递给执行引擎
- `assign`：把一个从执行引擎接收到的值赋给工作内存的变量
- `store`：把工作内存的一个变量的值传送到主内存中
- `write`：在 `store `之后执行，把 `store `得到的值放入主内存的变量中
- `lock`：作用于主内存的变量，它把一个变量标识为一条线程独占的状态。 
- `unlock`：作用于主内存的变量，它把一个处于锁定状态的变量释放出来，释放后的变量
  才可以被其他线程锁定。

### 三大特性

#### 原子性

Java 内存模型保证了 read、load、use、assign、store、write、lock 和 unlock 操作具有原子性，例如对一个 int 类型的变量执行 assign 赋值操作，这个操作就是原子性的。但是 Java 内存模型允许虚拟机将没有被 volatile 修饰的 64 位数据（long，double）的读写操作划分为两次 32 位的操作来进行，即 load、store、read 和 write 操作可以不具备原子性。

#### 可见性

可见性指：当一个线程修改了共享变量的值时，其他线程能够立即得知这个修改。`Java`内存模型是通过在变量修改后将新值同步回主内存，在变量读取前从主内存刷新变量值来实现可见性的。

#### 有序性

有序性指：在本线程内观察，所有操作都是有序的。在一个线程观察另一个线程，所有操作都是无序的，无序是因为发生了指令重排序。在`Java`内存模型中，允许编译器和处理器对指令进行重排序，重排序过程不会影响到单线程程序的执行，却会影响到多线程并发执行的正确性。

### Happens-Before原则

下面是Java内存模型下一些“天然的”先行发生关系，这些先行发生关系无须任何同步器协助就已 经存在，可以在编码中直接使用。如果两个操作之间的关系不在此列，并且无法从下列规则推导出来，则它们就没有顺序性保障，虚拟机可以对它们随意地进行重排序。

- 程序次序规则（`Program Order Rule`）：在一个线程内，按照控制流顺序，书写在前面的操作先行发生于书写在后面的操作。
- 管程锁定规则（`Monitor Lock Rule`）：一个`unlock`操作先行发生于后面对同一个锁的lock操作。
- `volatile`变量规则（`Volatile Variable Rule`）：对一个`volatile`变量的写操作先行发生于后面对这个变量的读操作。
- 线程启动规则（`Thread Start Rule`）：`Thread`对象的`start()`方法先行发生于此线程的每一个动作。
- 线程终止规则（`Thread Termination Rule`）：线程中的所有操作都先行发生于对此线程的终止检测，我们可以通过`Thread::join()`方法是否结束、`Thread::isAlive()`的返回值等手段检测线程是否已经终止执行。
- 线程中断规则（`Thread Interruption Rule`）：对线程`interrupt()`方法的调用先行发生于被中断线程的代码检测到中断事件的发生，可以通过`Thread::interrupted()`方法检测到是否有中断发生。 
- 对象终结规则（`Finalizer Rule`）：一个对象的初始化完成（构造函数执行结束）先行发生于它的finalize()方法的开始。
- 传递性（`Transitivity`）：如果操作A先行发生于操作B，操作B先行发生于操作C，那就可以得出操作A先行发生于操作C的结论。

## 九、AQS与LockSupport

### AQS

> 原文链接：[Java并发之AQS详解](https://www.cnblogs.com/waterystone/p/4920797.html)

`AQS`定义了一套多线程访问共享资源的同步器框架，许多同步类实现都依赖于它，如常用的`ReentrantLock/Semaphore/CountDownLatch`...

#### 框架

`AQS`核心由以下两部分组成

- `volatile int state`：代表共享资源
- `FIFO`等待队列：多线程争用资源被阻塞时会进入此队列

![image-20201129182127965](https://gitee.com/p8t/picbed/raw/master/imgs/20201129182129.png)

`state`访问方法如下

```java
int getState()
void setState()
boolean compareAndSetState()
```

`AQS`定义了两种资源共享方式：

- `Exclusive`：独占，只有一个线程能执行，如`ReentrantLock`
- `Share`：共享，多个线程可同时执行，如`Semaphore/CountDownLatch`

不同的自定义同步器争用共享资源的方式也不同。**自定义同步器在实现时只需要实现共享资源state的获取与释放方式即可，至于具体线程等待队列的维护（如获取资源失败入队/唤醒出队等），AQS已经在顶层实现好了**。自定义同步器实现时主要实现以下几种方法：

```java
// 该线程是否正在独占资源。只有用到condition才需要去实现它。
boolean isHeldExclusively() 
//独占方式。尝试获取资源，成功则返回true，失败则返回false。
boolean tryAcquire() 
//独占方式。尝试释放资源，成功则返回true，失败则返回false。
boolean tryRelease() 
// 共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
int tryAcquireShared() 
// 共享方式。尝试释放资源，如果释放后允许唤醒后续等待结点返回true，否则返回false。
boolean tryReleaseShared() 
```

以`ReentrantLock`为例，`state`初始化为0，表示未锁定状态。A线程`lock()`时，会调用`tryAcquire()`独占该锁并将`state+1`。此后，其他线程再`tryAcquire()`时就会失败，直到A线程`unlock()`到`state=0`（即释放锁）为止，其它线程才有机会获取该锁。当然，释放锁之前，A线程自己是可以重复获取此锁的（`state`会累加），这就是可重入的概念。但要注意，获取多少次就要释放多么次，这样才能保证`state`是能回到零态的。

再以`CountDownLatch`以例，任务分为N个子线程去执行，`state`也初始化为N（注意N要与线程个数一致）。这N个子线程是并行执行的，每个子线程执行完后`countDown()`一次，`state`会`CAS`减1。等到所有子线程都执行完后(即`state=0`)，会`unpark()`主调用线程，然后主调用线程就会从`await()`函数返回，继续后余动作。

一般来说，自定义同步器要么是独占方法，要么是共享方式，他们也只需实现`tryAcquire-tryRelease、tryAcquireShared-tryReleaseShared`中的一种即可。但`AQS`也支持自定义同步器同时实现独占和共享两种方式，如`ReentrantReadWriteLock`。

#### 源码

**`Node节点`**

Node结点是对每一个等待获取资源的线程的封装，其包含了需要同步的线程本身及其等待状态，如是否被阻塞、是否等待唤醒、是否已经被取消等。变量waitStatus则表示当前Node结点的等待状态，共有5种取值：`CANCELLED、SIGNAL、CONDITION、PROPAGATE、0`。

- `CANCELLED(1) `：表示当前结点已取消调度。当timeout或被中断（响应中断的情况下），会触发变更为此状态，进入该状态后的结点将不会再变化。
- `SIGNAL(-1)`：表示后继结点在等待当前结点唤醒。后继结点入队时，会将前继结点的状态更新为SIGNAL。
- `CONDITION(-2)`：表示结点等待在`Condition`上，当其他线程调用了`Condition`的`signal()`方法后，`CONDITION`状态的结点将从等待队列转移到同步队列中，等待获取同步锁。
- `PROPAGATE(-3)`：共享模式下，前继结点不仅会唤醒其后继结点，同时也可能会唤醒后继的后继结点。
- `0`：新结点入队时的默认状态。

注意：负值表示结点处于有效等待状态，而正值表示结点已被取消。所以源码中很多地方用>0、<0来判断结点的状态是否正常。

**`acquire()`**

此方法是独占模式下线程获取共享资源的顶层入口。如果获取到资源，线程直接返回，否则进入等待队列，直到获取到资源为止，且整个过程忽略中断的影响。这也正是`lock()`的语义，当然不仅仅只限于`lock()`。获取到资源后，线程就可以去执行其临界区代码了。下面是`acquire()`的源码：

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

函数流程如下：

1. `tryAcquire()`尝试直接去获取资源，如果成功则直接返回（这里体现了非公平锁，每个线程获取锁时会尝试直接抢占一次，而`CLH`队列中可能还有别的线程在等待）；
2. `addWaiter()`将该线程加入等待队列的尾部，并标记为独占模式；
3. `acquireQueued()`使线程阻塞在等待队列中获取资源，一直获取到资源后才返回。如果在整个等待过程中被中断过，则返回`true`，否则返回`false`；
4. 如果线程在等待过程中被中断过，它是不响应的。只是获取资源后才再进行自我中断`selfInterrupt()`，将中断补上。

注意：线程入队时并不是简单的加到队尾，因为可能前面的节点已经处于`CANCELLED`状态，这时就需要找到最后一个正常状态的节点并加在它后面，然后`CANCELLED`状态的节点就被丢弃`GC`了

**`release()`**

`release()`是`acquire()`的反操作。此方法是独占模式下线程释放共享资源的顶层入口。它会释放指定量的资源，如果彻底释放了（即`state=0`）,它会唤醒等待队列里的其他线程来获取资源。这也正是`unlock()`的语义，当然不仅仅只限于`unlock()`。下面是`release()`的源码：

```java
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}

private void unparkSuccessor(Node node) {
    //这里，node一般为当前线程所在的结点。
    int ws = node.waitStatus;
    if (ws < 0)//置零当前线程所在的结点状态，允许失败。
        compareAndSetWaitStatus(node, ws, 0);

    Node s = node.next;//找到下一个需要唤醒的结点s
    if (s == null || s.waitStatus > 0) {//如果为空或已取消
        s = null;
        for (Node t = tail; t != null && t != node; t = t.prev) // 从后向前找。
            if (t.waitStatus <= 0)//从这里可以看出，<=0的结点，都是还有效的结点。
                s = t;
    }
    if (s != null)
        LockSupport.unpark(s.thread);//唤醒
}
```

一句话：用`unpark()`唤醒等待队列中最前边的那个未放弃线程

**`acquireShared()`与`releaseShared()`**

独占模式下，一个线程执行完任务后只会唤醒下一个线程。而共享模式则会在资源充足的情况下，递归唤醒后续线程。

### LockSupport

在`AQS`中挂起和恢复线程用到了`LockSupport`类的`park() / unpark()`，和`Object`的`wait() ` / `notify()`不同，它不需要限制在`synchronized`中使用

此外，`LockSupport`就是`Condition`实现的核心。在`AQS`内部有个类实现了`Condition`接口，并且里面线程阻塞唤醒用的就是`LockSupport`

## 十、工具类

### CountDownLatch

```java
public class E {
    public static void main(String[] args) {
        CountDownLatch cdl = new CountDownLatch(3);
        Thread t1 = new Thread(() -> {
            TimeUnit.SECONDS.sleep(1);
            System.out.println("first countdown");
            cdl.countDown();
        });
        Thread t2 = new Thread(() -> {
            TimeUnit.SECONDS.sleep(2);
            System.out.println("second countdown");
            cdl.countDown();
        });
        Thread t3 = new Thread(() -> {
            TimeUnit.SECONDS.sleep(3);
            System.out.println("third countdown");
            cdl.countDown();
        });
        t1.start();
        t2.start();
        t3.start();
        cdl.await();
        System.out.println("main thread");
        /*
         * first countdown
         * second countdown
         * third countdown
         * main thread
         */
    }
}
```

### CyclicBarrier

```java
public class E {
    public static void main(String[] args) {
        CyclicBarrier cb = new CyclicBarrier(3);
        Thread t1 = new Thread(() -> {
            TimeUnit.SECONDS.sleep(1);
            cb.await();
            System.out.println("t1");
        });
        Thread t2 = new Thread(() -> {
            TimeUnit.SECONDS.sleep(2);
            cb.await();
            System.out.println("t2");
        });
        Thread t3 = new Thread(() -> {
            TimeUnit.SECONDS.sleep(3);
            cb.await();
            System.out.println("t3");
        });
        t1.start();
        t2.start();
        t3.start();
        // 每个线程执行到cb.await()时挂起, 直到有3个都执行到这, 然后全部放行
    }
}
```

### Semaphore

## 十一、ReentrantReadWriteLock

读写锁内部含有读锁对象和写锁对象，但是这两个共享一个`AQS`，也就意味着共享一个资源变量**`state`**，具体共享方式实现如下图，高16位用于读锁，低16位用于写锁

![image-20201130203543036](https://gitee.com/p8t/picbed/raw/master/imgs/20201130203544.png)

剩下的操作基本上就是`AQS`的实现了，使用`Condition`进行读写同步

## 十二、强软弱虚

- 强
  - 就是正常new的对象引用方式
- 软
  - 内存不足时，无论是否为垃圾，GC都会回收
  - 可用于缓存
- 弱
  - 垃圾回收器看到就GC
  - **解决内存泄露问题**
- 虚
  - 和没有一样
  - 在被回收的时候会给一个队列发送消息
  - 用于管理堆外内存

## 十三、ThreadLocal

### 使用

```java
static ThreadLocal<String> tl = new ThreadLocal<>();

public static void main(String[] args) {
    // 往自己线程的map中放入value
    new Thread(() -> tl.set("TL")).start();
    new Thread(() ->{
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        // 在自己线程的map中获取value
        System.out.println(tl.get());	// null
    }).start();
}
```

### 源码

```java
public void set(T value) {
    Thread t = Thread.currentThread();
    // 这个map来自当前线程
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        // 在当前线程的一个map中设置value, key为这个ThreadLocal对象
        map.set(this, value);
    } else {
        // 在当前线程实例化ThreadLocalMap
        createMap(t, value);
    }
}

static class ThreadLocalMap {
    static class Entry extends WeakReference<ThreadLocal<?>> {
        Object value;
		// map中的key/value是一个entry
        Entry(ThreadLocal<?> k, Object v) {
            // 这里把key传给WeakReference说明key指向的ThreadLocal是弱引用
            super(k);
            value = v;
        }
    }
	// ...
}
```

### 内存泄漏

当在一个`ThreadLocal`中`set`一个值后，其引用关系如下图。`tl`指向`new`出来的`ThreadLocal`对象，并且因为ThreadLocalMap中的`key`也是`ThreadLocal`，因此这个`key`也指向那个`ThreadLocal`对象，并且这个指向是**弱引用**。

使用弱引用的原因：如果使用强引用，即使tl不再引用`ThreadLocal`对象了，那么它也不能被回收。因为`key`还指向着它，并且这个`key`的生命周期是与当前线程挂钩的，只要当前线程不死亡，那么就一直存在着`ThreadLocal`就会一直占着内存，导致内存泄漏。而如果使用若引用，只要`tl`不再指向`ThreadLocal`对象，那么`GC`一遇到它就会回收，不会有内存泄漏问题。

但还有一个问题，就是如果只`set`值，在`key`被`GC`置为`null`后，就无法获取到`value`值了，因此使用完后，需要调用`remove`方法把整个`entry`移除，否则就会造成内存泄漏

![image-20201203215950747](https://gitee.com/p8t/picbed/raw/master/imgs/20201203215952.png)
