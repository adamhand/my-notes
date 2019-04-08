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

# 工具类Executors的静态方法
负责生成各种类型的ExecutorService线程池实例，共有四种。

- newFixedThreadPool(numberOfThreads:int):（固定线程池）ExecutorService 创建一个固定线程数量的线程池，并行执行的线程数量不变，线程当前任务完成后，可以被重用执行另一个任务
- newCachedThreadPool():（可缓存线程池）ExecutorService 创建一个线程池，按需创建新线程，如果线程的当前规模超过了处理需求时，阿么将回收空闲的线程，而当需求增加时，则可以添加新的线程，线程池的规模不存在任何限制
- new SingleThreadExecutor();（单线程执行器）线程池中只有一个线程，依次执行任务
- new ScheduledThreadPool()：线程池按时间计划来执行任务，允许用户设定执行任务的时间，类似于timer

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

当把一个Callable对象(Callable1,Callable2)提交给ThreadPoolExecutor和ScheduledThreadPoolExecutor执行时，submit(...)会向我们返回一个FutureTask对象。我们执行FutureTask.get()来等待任务执行完成，当任务完成后，FutureTask.get()将返回任务的结果。

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
```java
volatile int runState;   
static final int RUNNING = 0;   //运行状态
static final int SHUTDOWN = 1;   //关闭状态
static final int STOP = 2;       //停止
static final int TERMINATED = 3; //终止，终结
```
- 当创建线程池后，初始时，线程池处于RUNNING状态；
- 如果调用了shutdown()方法，则线程池处于SHUTDOWN状态，此时线程池不能够接受新的任务，它会等待所有任务执行完毕，最后终止；
- 如果调用了shutdownNow()方法，则线程池处于STOP状态，此时线程池不能接受新的任务，并且会去尝试终止正在执行的任务，返回没有执行的任务列表；
- 当线程池处于SHUTDOWN或STOP状态，并且所有工作线程已经销毁，任务缓存队列已经清空或执行结束后，线程池被设置为TERMINATED状态。

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
https://www.cnblogs.com/lixuwu/p/7979480.html

# 参考
[Java的Executor框架和线程池实现原理](https://blog.csdn.net/tuke_tuke/article/details/51353925)
[Java并发编程：线程池的使用](http://www.cnblogs.com/dolphin0520/p/3932921.html)