# Java并发-Executor框架和线程池
---

# Executor框架
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/executor_kuangjia.png">
</center>

# Executor接口
```java
public interface Executor {
     void execute(Runnable command);
 }
```
Executor接口是Executor框架中最基础的部分，定义了一个用于执行Runnable的execute方法，它没有实现类只有另一个重要的子接口ExecutorService

# ExecutorService接口
```java
//继承自Executor接口
public interface ExecutorService extends Executor {
    /**
    * 关闭方法，调用后执行之前提交的任务，不再接受新的任务
    */
    void shutdown();
    /**
    * 从语义上可以看出是立即停止的意思，将暂停所有等待处理的任务并返回这些任务的列表
    */
    List<Runnable> shutdownNow();
    /**
    * 判断执行器是否已经关闭
    */
    boolean isShutdown();
    /**
    * 关闭后所有任务是否都已完成
    */
    boolean isTerminated();
    /**
    * 中断
    */
    boolean awaitTermination(long timeout, TimeUnit unit)
        throws InterruptedException;
    /**
    * 提交一个Callable任务
    */
    <T> Future<T> submit(Callable<T> task);
    /**
    * 提交一个Runable任务，result要返回的结果
    */
    <T> Future<T> submit(Runnable task, T result);
    /**
    * 提交一个任务
    */
    Future<?> submit(Runnable task);
    /**
    * 执行所有给定的任务，当所有任务完成，返回保持任务状态和结果的Future列表
    */
    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
        throws InterruptedException;
    /**
    * 执行给定的任务，当所有任务完成或超时期满时（无论哪个首先发生），返回保持任务状态和结果的 Future 列表。
    */
    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks,
                                long timeout, TimeUnit unit)
        throws InterruptedException;
    /**
    * 执行给定的任务，如果某个任务已成功完成（也就是未抛出异常），则返回其结果。
    */
    <T> T invokeAny(Collection<? extends Callable<T>> tasks)
        throws InterruptedException, ExecutionException;
    /**
    * 执行给定的任务，如果在给定的超时期满前某个任务已成功完成（也就是未抛出异常），则返回其结果。
    */
    <T> T invokeAny(Collection<? extends Callable<T>> tasks,
                    long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
 }
```
ExecutorService接口继承自Executor接口，定义了终止、提交、执行任务、跟踪任务返回结果等方法。

ExecutorService的生命周期有三种状态：**运行**、**关闭**和**已终止**。

- execute（Runnable command）：履行Ruannable类型的任务,
- submit（task）：可用来提交Callable或Runnable任务，并返回代表此任务的Future对象
- shutdown（）：在完成已提交的任务后封闭办事，不再接管新任务,
- shutdownNow（）：停止所有正在履行的任务并封闭办事。
- isTerminated（）：测试是否所有任务都履行完毕了。,
- isShutdown（）：测试是否该ExecutorService已被关闭

# ThreadPoolExecutor类
ThreadPoolExecutor类是线程的主类，下面提到的工具类Executors中的建立线程池的方法，里面也是调用的ThreadPoolExecutor来创建的。ThreadPoolExecutor中定义了线程池最基本的参数和executor()、addWorker()两个核心方法。

它的构造函数如下所示：

```java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler) {...}
```
可以看到，共有7个参数，它们的功能分别如下：

- corePoolSize：常驻核心线程数。如果等于0，表示任务执行完之后，没有任何请求进入时销毁线程池的线程；如果大于0，及时本地任务执行完毕，核心线程也不会被销毁。
- maximumPoolSize：线程池能够容忍的同时执行的最大线程数，必须大于等于1，如果maximumPoolSize和corePoolSize大小相同，即是固定大小的线程池。
- keepAliveTime：线程池中的线程空闲时间，当空闲时间达到keepAliveTime值时，线程会被销毁，直到只剩下corePoolSize个线程为止，避免浪费内存和句柄资源，但是当ThreadPoolExecutor的allowCoreThreadTimeOut变量置为true时，核心线程超时后也会被回收。
- unit：keepAliveTime超时时间单位，
- workQueue：缓存队列。当请求的线程大于corePoolSize时，线程会进入workQueue等待。等到workQueue中的存活线程满了，才会在线程池中创建新线程。
- threadFactory：线程工厂。用来生产一组相同任务的线程。线程池的明明是通过给这个Factory增加组名前缀来实现的。
- handler：表示线程池采用的拒绝策略。当workQueue的任务缓存区到达上限后，并且活动线程数大于maximumPoolSize的时候，线程池通过该策略处理请求，是一种简单的限流保护。

上面的workQueue、threadFactory和handler三个参数不能为空，但是一般很少对它们进行手动初始化，而是通过Executors这个工具类来实现。这个类下面介绍，先来看一下ThreadPoolExecutor中的两个核心方法：execute()和addWorker()。

## execute()方法

execute()方法如下：
```java
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();
    // 返回包含线程数及线程状态的Integer类型数据
    int c = ctl.get();
    // 如果工作线程数小于核心线程数，则创建线程任务并执行
    if (workerCountOf(c) < corePoolSize) {
        if (addWorker(command, true))
            return;
        c = ctl.get();
    }
    // 如果已经达到核心线程数，将线程任务置入队列，但是前提是线程池处于RUNNING态
    if (isRunning(c) && workQueue.offer(command)) {
        int recheck = ctl.get();
        // 再次判断一下，如果线程池不是RUNNING状态，则将刚加入队列的任务移除
        if (! isRunning(recheck) && remove(command))
            reject(command);
        // 如果之前的线程已经被消费完，创建一个新线程
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false);
    }
    // 核心线程池和队列均已满，尝试创建一个新线程
    else if (!addWorker(command, false))
        // 如果创建失败，唤醒拒绝策略
        reject(command);
}
```

## addWorder()方法
addWorder()方法的作用是根据当前线程池状态检查是否可以添加新的任务线程，如果可以则创建并启动任务。如果成功则返回true，否则返回false。返回false的可能性有两种：

- 线程池没有处于RUNNING状态
- 线程工厂创建新的任务线程失败

addWorder()的源码如下：

```java
private boolean addWorker(Runnable firstTask, boolean core) {
    // 类似于goto语句的标志，用于快速推出多层循环
    retry:
    for (;;) {
        // 得到线程池状态和工作线程数
        int c = ctl.get();
        int rs = runStateOf(c);

        // 如果线程池状态是STOP及之上的状态(TIDYING、TERMINATED)，或者fitstTask初试线程不为空或者队列为空，都会直接返回创建失败
        if (rs >= SHUTDOWN &&
            ! (rs == SHUTDOWN &&
                firstTask == null &&
                ! workQueue.isEmpty()))
            return false;

        for (;;) {
            int wc = workerCountOf(c);
            // 如果超过最大允许线程数则不能再创建新的线程
            if (wc >= CAPACITY ||
                wc >= (core ? corePoolSize : maximumPoolSize))
                return false;
            // 将当前活动线程数+1，是原子操作。然后跳出和retry相邻的循环体。 ---(1)
            if (compareAndIncrementWorkerCount(c))
                break retry;
            // 线程池的状态和工作线程数是可变的，需要经常提取这个值
            c = ctl.get();  
            // 如果工作线程数改变了，说明CAS失败，重新进入循环
            if (runStateOf(c) != rs)
                continue retry;
        }
    }

    // 开始创建工作线程
    boolean workerStarted = false;
    boolean workerAdded = false;
    Worker w = null;
    try {
        // 利用Worker构造方法中的线程工厂创建线程，并封装成工作线程Worker对象
        w = new Worker(firstTask);
        final Thread t = w.thread;
        if (t != null) {
            // 避免添加或启动线程时被干扰，需要加上锁
            final ReentrantLock mainLock = this.mainLock;
            mainLock.lock();
            try {
                // 重新检查一下，如果线程池状态为RUNNING，或者为SHUTDOWN并且firtTask为空，才允许添加worker
                int rs = runStateOf(ctl.get());

                if (rs < SHUTDOWN ||
                    (rs == SHUTDOWN && firstTask == null)) {
                    if (t.isAlive()) // precheck that t is startable
                        throw new IllegalThreadStateException();
                    workers.add(w);
                    int s = workers.size();
                    if (s > largestPoolSize)
                        largestPoolSize = s;
                    workerAdded = true;
                }
            } finally {
                mainLock.unlock();
            }
            if (workerAdded) {
                t.start();
                workerStarted = true;
            }
        }
    } finally {
        // 如果创建Worder失败，将上面(1)处增加的线程数在减掉
        if (! workerStarted)
            addWorkerFailed(w);
    }
    return workerStarted;
}
```
可以看到，创建Worker的逻辑是先用CAS将Worker的数量加1，然后才真正开始创建Worker，如果创建失败再将Worker的数量减回去。这是因为compareAndIncrementWorkerCount(c)方法执行失败的概率非常低，即使失败，再次执行是成功的概率也是极高的。而如果先建立Worker，成功之后再加1，当发现超出限制之后再销毁线程，代价会比较大。

# 线程池四种拒绝策略
线程池的拒绝策略，是指当任务添加到线程池中被拒绝，所采取的措施。当任务添加到线程池中之所以被拒绝，可能是由于：
- 线程池异常关闭
- 任务数量超过线程池的最大限制。

在ThreadPoolExecutor中提供了四个公开的内部静态类，实现了四种拒绝策略：

- AbortPolicy(默认)：丢弃任务并且抛出RejectedExecutionException异常
- DiscardPolicy：丢弃任务，但是不抛出异常，不建议这样做
- DiscardOldestPolicy：抛弃队列中等待最久的任务，然后把当前任务加入到队列中
- CallerRunsPolicy：调用任务的run()方法绕过线程池直接执行

AbortPolicy策略上面的例子已经看到了，就不再尝试了，下面看一下其他三个策略的实验。

## DiscardPolicy策略
例子如下：
```java
public class DiscardPolicyDemo {
    public static void main(String[] args) {
        ThreadPoolExecutor pool = new ThreadPoolExecutor(1, 2, 0L, TimeUnit.SECONDS,
                new LinkedBlockingQueue<>(2), new ThreadPoolExecutor.DiscardPolicy());

        Task task = new Task();
        for (int i = 0; i < 10; i++) {
            pool.execute(task);
        }
        pool.shutdown();
    }
}
```
结果如下：
```java
running_task: 0
running_task: 1
running_task: 2
running_task: 3
```
上面的例子中，最大线程数为2，阻塞队列的长度也为2，所以线程池允许的最大工作线程数为4，当超过这个数之后，就会直接将新任务丢弃，并且不会抛出异常。

## DiscardOldestPolicy策略
例子如下：

```java
public class DiscardOldestPolicyDemo {
    public static void main(String[] args) {
        ThreadPoolExecutor pool = new ThreadPoolExecutor(1, 2, 0L, TimeUnit.SECONDS,
                new LinkedBlockingQueue<>(2), new ThreadPoolExecutor.DiscardOldestPolicy());

        MyTask task = new MyTask();
        for (int i = 0; i < 10; i++) {
            pool.execute(task);
        }
        pool.shutdown();
    }
}
```
**这个地方貌似有个问题，打印的结果和上面DiscardPolicy策略相同**

## CallerRunsPolicy策略
例子如下：
```java
public class DiscardOldestPolicyDemo {
    public static void main(String[] args) {
        ThreadPoolExecutor pool = new ThreadPoolExecutor(1, 2, 0L, TimeUnit.SECONDS,
                new LinkedBlockingQueue<>(2), new ThreadPoolExecutor.CallerRunsPolicy());

        Task task = new Task();
        for (int i = 0; i < 10; i++) {
            pool.execute(task);
        }
        pool.shutdown();
    }
}
```
执行结果为：
```java
running_task: 0
running_task: 1
running_task: 3
running_task: 4
running_task: 5
running_task: 6
running_task: 7
running_task: 8
running_task: 9
running_task: 2
```

# 工具类Executors的静态方法
负责生成各种类型的ExecutorService线程池实例，共有5种。

- newFixedThreadPool(numberOfThreads:int):（固定线程池）输入的参数即是固定线程数，既是核心线程数也是最大线程数，不存在空闲线程，所以keepAliveTime等于0
- newSingleThreadExecutor();（单线程执行器）线程池中只有一个线程，按任务的提交顺序执行任务
- newCachedThreadPool():（可缓存线程池）maximumPoolSize最大可以至Integer.MAX_VALUE，keepAliveTime默认为60，工作线程处于空闲状态则会被回收，如果任务数增加，再次创建出新线程处理任务
- newScheduledThreadPool()：和newCachedThreadPool相同，线程数最大可以达到Integer.MAX_VALUE，支持定时和周期性任务执行，不会对工作线程进行回收
- newWorkStealingPool()：jdk8引入的，创建持有足够线程的线程池支持给定的并行度，并通过使用多个队列减少竞争，默认并行度为CPU数量

除了newWorkStealingPool之外，其他四个创建方式都会存在资源耗尽的风险，具体如下：

- newFixedThreadPool和newSingleThreadExecutor中允许的请求队列长度为Integer.MAX_VALUE，可能会堆积大量请求，从而导致OOM
- newCacheThreadPool和newScheduledThreadPool允许创建线程数为Integer.MAX_VALUE，可能会创建大量线程，从而导致OOM

所以，不建议使用过Executors来创建线程，，而是使用ThreadPoolExecutor来创建，同时自定义线程工厂和拒绝策略等参数。一个例子如下：

自定义线程工厂和任务：
```java
public class MyThreadFactory implements ThreadFactory{
    private final String namePrefix;
    private final AtomicInteger nextId = new AtomicInteger(1);

    MyThreadFactory(String whatFeatureOfGroup) {
        namePrefix = "MyThreadFactory's " + whatFeatureOfGroup + " -Worder-";
    }

    @Override
    public Thread newThread(Runnable r) {
        String name = namePrefix + nextId.getAndIncrement();
        // 参数分别为：ThreadGroup, Runnable, name, stackSize
        Thread thread = new Thread(null, r, name, 0);
        System.out.println(thread.getName());
        return thread;
    }
}

class Task implements Runnable {
    private final AtomicLong count = new AtomicLong(0L);
    @Override
    public void run() {
        System.out.println("running_task: " + count.getAndIncrement());
    }
}
```

自定义任务拒绝策略：
```java
public class MyRejectHandler implements RejectedExecutionHandler{
    @Override
    public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
        System.out.println("task rejected : " + executor.toString());
    }
}
```

```java
public class MyThreadPool {
    public static void main(String[] args) {
        BlockingQueue queue = new LinkedBlockingQueue(2);

        MyThreadFactory f1 = new MyThreadFactory(" 第1机房 ");
        MyThreadFactory f2 = new MyThreadFactory(" 第2机房 ");

        MyRejectHandler handler = new MyRejectHandler();

        ThreadPoolExecutor poolFirst = new ThreadPoolExecutor(1, 2,
                60, TimeUnit.SECONDS, queue, f1, handler);
        ThreadPoolExecutor poolSecond = new ThreadPoolExecutor(1, 2,
                60, TimeUnit.SECONDS, queue, f2, handler);


        Runnable task = new Task();
        for (int i = 0; i < 200; i++) {
            poolFirst.execute(task);
            poolSecond.execute(task);
        }
    }
}
```

执行结果入下所示：

```java
MyThreadFactory's  第1机房  -Worder-1
MyThreadFactory's  第2机房  -Worder-1
MyThreadFactory's  第1机房  -Worder-2
MyThreadFactory's  第2机房  -Worder-2
running_task: 0
running_task: 1
running_task: 2
task rejected : java.util.concurrent.ThreadPoolExecutor@45ee12a7[Running, pool size = 2, active threads = 1, queued tasks = 0, completed tasks = 3]
task rejected : java.util.concurrent.ThreadPoolExecutor@330bedb4[Running, pool size = 2, active threads = 2, queued tasks = 2, completed tasks = 0]
...
```

除此之外，也可使用开源类库建立线程池，比如使用guava包建立的例子如下：

```java
public class GuavaThreadPool {
    private static ThreadFactory namedThreadFactory =
            new ThreadFactoryBuilder().setNameFormat("thread-pool-%d").build();

    private static ExecutorService pool = new ThreadPoolExecutor(1, 2, 0L, TimeUnit.SECONDS,
            new LinkedBlockingQueue<>(2), namedThreadFactory, new ThreadPoolExecutor.AbortPolicy());

    public static void main(String[] args) {
        for (int i = 0; i < 100; i++) {
            pool.execute(new Task());
        }
    }
}
```

执行结果如下：

```java
running_task: 0
running_task: 0
running_task: 0
running_task: 0
Exception in thread "main" java.util.concurrent.RejectedExecutionException: Task thread.thread_pool.Task@5e2de80c rejected from java.util.concurrent.ThreadPoolExecutor@1d44bcfa[Running, pool size = 2, active threads = 2, queued tasks = 0, completed tasks = 1]
	at java.util.concurrent.ThreadPoolExecutor$AbortPolicy.rejectedExecution(ThreadPoolExecutor.java:2047)
	at java.util.concurrent.ThreadPoolExecutor.reject(ThreadPoolExecutor.java:823)
	at java.util.concurrent.ThreadPoolExecutor.execute(ThreadPoolExecutor.java:1369)
	at thread.thread_pool.GuavaThreadPool.main(GuavaThreadPool.java:16)
```

# Runnable、Callable、Future接口

## Runnable接口
```java
// 实现Runnable接口的类将被Thread执行，表示一个基本的任务
  public interface Runnable {
      // run方法就是它所有的内容，就是实际执行的任务
      public abstract void run();
  }
```
## Callable接口
与Runnable接口的区别在于它接收泛型，同时它执行任务后带有返回内容
```java
// Callable同样是任务，与Runnable接口的区别在于它接收泛型，同时它执行任务后带有返回内容
  public interface Callable<V> {
      // 相对于run方法的带有返回值的call方法
      V call() throws Exception;
}
```

Runnable接口和Callable接口的实现类，都可以被ThreadPoolExecutor和ScheduledThreadPoolExecutor执行，他们之间的区别是Runnable不会返回结果，而Callable可以返回结果。

Executors可以把一个Runnable对象转换成Callable对象：
```java
public static Callable<Object> callable(Runnbale task);
```

当把一个Callable对象(Callable1,Callable2)提交给ThreadPoolExecutor和ScheduledThreadPoolExecutor执行时，submit(...)会返回一个FutureTask对象。我们执行FutureTask.get()来等待任务执行完成，当任务完成后，FutureTask.get()将返回任务的结果。

## Future接口
```java
// Future代表异步任务的执行结果
public interface Future<V> {

    /**
    * 尝试取消一个任务，如果这个任务不能被取消（通常是因为已经执行完了），返回false，否则返回true。
    */
    boolean cancel(boolean mayInterruptIfRunning);

    /**
    * 返回代表的任务是否在完成之前被取消了
    */
    boolean isCancelled();

    /**
    * 如果任务已经完成，返回true
    */
    boolean isDone();

    /**
    * 获取异步任务的执行结果（如果任务没执行完将等待）
    */
    V get() throws InterruptedException, ExecutionException;

    /**
    * 获取异步任务的执行结果（有最常等待时间的限制）
    *
    *  timeout表示等待的时间，unit是它时间单位
    */
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```
Future就是对于具体的Runnable或者Callable任务的执行结果进行取消、查询是否完成、获取结果。必要时可以通过get方法获取执行结果，该方法会阻塞直到任务返回结果

- cancel方法用来取消任务，如果取消任务成功则返回true，如果取消任务失败则返回false。
- isCancelled方法表示任务是否被取消成功，如果在任务正常完成前被取消成功，则返回 true。
- isDone方法表示任务是否已经完成，若任务完成，则返回true；
- get()方法用来获取执行结果，这个方法会产生阻塞，会一直等到任务执行完毕才返回；
- get(long timeout, TimeUnit unit)用来获取执行结果，如果在指定时间内，还没获取到结果，就直接返回null。

也就是说Future提供了三种功能：

- 判断任务是否完成；
- 能够中断任务；
- 能够获取任务执行结果。

# 线程池的处理流程
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/excutor.png">
</center>

线程池优先要创建出基本线程池大小（corePoolSize）的线程数量，没有达到这个数量时，每次提交新任务都会直接创建一个新线程，当达到了基本线程数量后，又有新任务到达，优先放入等待队列，如果队列满了，才去创建新的线程（不能超过线程池的最大数maxmumPoolSize）。

# 向线程池提交任务的两种方式
## 通过execute()方法
```java
ExecutorService threadpool= Executors.newFixedThreadPool(10);  
threadpool.execute(new Runnable(){...});
```
这种方式提交没有返回值，也就不能判断任务是否被线程池执行成功。

## 通过submit()方法
```java
Future<?> future = threadpool.submit(new Runnable(){...});  
    try {  
            Object res = future.get();//获取任务执行结果  
        } catch (InterruptedException e) {  
            // 处理中断异常  
            e.printStackTrace();  
        } catch (ExecutionException e) {  
            // 处理无法执行任务异常  
            e.printStackTrace();  
        }finally{  
            // 关闭线程池  
            executor.shutdown();  
        }  
```
使用submit 方法来提交任务，它会返回一个Future对象，通过future的get方法来获取返回值，get方法会阻塞住直到任务完成，而使用get(long timeout, TimeUnit unit)方法则会阻塞一段时间后立即返回，这时有可能任务没有执行完。

# 线程池本身的状态
线程池本身有5中状态，Running、ShutDown、Stop、Tidying、Terminated。
```java
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
private static final int COUNT_BITS = Integer.SIZE - 3;
private static final int CAPACITY = (1 << COUNT_BITS) - 1;

private static final int RUNNING = -1 << COUNT_BITS;
private static final int SHUTDOWN = 0 << COUNT_BITS;
private static final int STOP = 1 << COUNT_BITS;
private static final int TIDYING = 2 << COUNT_BITS;
private static final int TERMINATED = 3 << COUNT_BITS;
private static int ctlOf(int rs, int wc) { return rs | wc; }
```
ctl是一个AtomicInteger类型的原子对象。ctl记录了"线程池中的任务数量"和"线程池状态"2个信息。ctl共包括32位。其中，高3位表示"线程池状态"，低29位表示"线程池中的任务数量"。
```java
RUNNING    -- 对应的高3位值是111。
SHUTDOWN   -- 对应的高3位值是000。
STOP       -- 对应的高3位值是001。
TIDYING    -- 对应的高3位值是010。
TERMINATED -- 对应的高3位值是011。
```
各个状态的转换如下：

<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/threadpoolstate.jpg">
</div>

- 当创建线程池后，初始时，线程池处于RUNNING状态
- 如果调用了shutdown()方法，则线程池处于SHUTDOWN状态，此时线程池不能够接受新的任务，它会等待所有任务执行完毕，最后终止
- 如果调用了shutdownNow()方法，则线程池处于STOP状态，此时线程池不能接受新的任务，并且会去尝试终止正在执行的任务，返回没有执行的任务列表
- 当线程池处于SHUTDOWN或STOP状态，并且所有工作线程已经销毁，任务缓存队列已经清空或执行结束后，线程池被设置为TIDYING状态
- 处于TIDYING状态的线程池会调用terminated()方法，该方法执行完之后，线程池就会变成TERMINATED状态

# ForkJoinPool
ForkJoinPool是jdk1.7新引入的线程池，基于ForkJoin框架，使用了“分治”的思想。关于ForkJoin框架参考另一篇笔记“Java并发之J.U.C”。

ThreadPoolExecutor中每个任务都是由单个线程独立处理的，如果出现一个非常耗时的大任务(比如大数组排序)，就可能出现线程池中只有一个线程在处理这个大任务，而其他线程却空闲着，这会导致**CPU负载不均衡**：空闲的处理器无法帮助工作繁忙的处理器。

ForkJoinPool就是用来解决这种问题的：将一个大任务拆分成多个小任务后，使用fork可以将小任务分发给其他线程同时处理，使用join可以将多个线程处理的结果进行汇总；这实际上就是**分治思想的并行版本**。

## ForkJoinPool基本原理
ForkJoinPool 类是Fork/Join 框架的核心，和ThreadPoolExecutor一样它也是ExecutorService接口的实现类。

所以上面的线程池框架图是比较老的了，新的框架图如下图所示：
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/newexcuterframwork.PNG">
</center>

（未完待续）
https://www.jianshu.com/p/32a15ef2f1bf
https://www.jianshu.com/p/de025df55363
https://blog.csdn.net/Holmofy/article/details/82714665
https://www.cnblogs.com/lixuwu/p/7979480.html`

# 参考
[Java的Executor框架和线程池实现原理](https://blog.csdn.net/tuke_tuke/article/details/51353925)</br>
[Java并发编程：线程池的使用](http://www.cnblogs.com/dolphin0520/p/3932921.html)</br>
《码出高效-Java开发手册》</br>
[Java中线程池，你真的会用吗？](https://www.hollischuang.com/archives/2888)</br>
[JUC线程池（5）：线程池拒绝策略](https://www.jianshu.com/p/9145672b358a)</br>