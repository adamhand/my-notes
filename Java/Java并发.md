# Java并发

# 目录
- 1 引
    - 1.1 线程与进程
    - 1.2 线程的状态
    - 1.3 线程实现的方法
    - 1.4 线程常用的方法
    - 1.5 基础线程机制
    - 1.6 停止线程的方法
- 2 多线程基础
    - 2.1同步
        - synchronized机制
        - 同步锁（Lock）机制
        - volatile关键字实现同步
        - 使用阻塞队列LinkedBlockingQueue
        - 使用AtometicInteger
        - 小结
    - 2.2 死锁
        - 死锁的原因
        - 解决死锁的方法
- 3 多线程通信
    - join()机制
    - wait/notify机制
    - await() signal()机制

---

# 1. 引
## 1.1 线程与进程

基本概念:
  - 线程：进程中负责程序执行的执行单元。线程本身依靠程序进行运行。线程是程序中的顺序控制流，只能使用分配给程序的资源和环境。
  - 进程：执行中的程序，一个进程至少包含一个线程。
  - 单线程：程序中只存在一个线程，实际上主方法就是一个主线程。
  - 多线程：在一个程序中运行多个任务，目的是更好地使用CPU资源。

区别：
  - 最根本的区别：进程是操作系统资源分配的基本单位，而线程是任务调度和执行的基本单位。
  - 进程之间有自己独立的运行资源，比如内存和堆栈，但是一个进程中的所有线程的资源是共享的。
  - 进程之间的切换开销比较大，因为每个进程有独立的代码和数据空间，而线程之间的开销比较小。


  做一个形象的比喻，如果将应用程序比作一个工厂的话，进程就好比是工厂中的一个个车间，而线程就好像车间中的一个个流水线。一个车间的所有流水线共用车间的所有资源，而车间的资源是由工厂分配的，各个车间之间相互独立（有车间主任这个职位，却没听说过流水线主任）。

## 1.2 线程的状态

<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/javathread.jpg">
</div>

如上图所示，线程包含七种状态：
  - 创建（new）状态: 准备好了一个多线程的对象
  - 就绪（runnable）状态: 调用了start()方法, 等待CPU进行调度
  - 运行（running）状态: 执行run()方法
  - 阻塞（blocked）状态: 暂时停止执行, 可能将资源交给其它线程使用
  - 期限等待（time waiting）状态：不会被分配CPU执行权，但是无需等待其他进程显式唤醒，在一定时间后它们会由系统自动唤醒。
  - 无期限等待（waiting）状态：不会分配CPU执行权，需要其他进程显式唤醒。
  - 终止（terminated）状态: 线程销毁

下面具体来看一下：

- <strong>创建（new）状态:</strong>当需要新起一个线程来执行某个子任务时，就创建了一个线程。但是线程创建之后，不会立即进入就绪状态，因为线程的运行需要一些条件，只有线程运行需要的所有条件满足了，才进入就绪状态。<li>
<strong>就绪（runnable）状态：</strong>当线程进入就绪状态后，不代表立刻就能获取CPU执行时间，也许此时CPU正在执行其他的事情，因此它要等待。当得到CPU执行时间之后，线程便真正进入运行状态。<li>
<strong>运行（running）状态：</strong>线程在运行状态过程中，可能有多个原因导致当前线程不继续运行下去，比如用户主动让线程睡眠（睡眠一定的时间之后再重新执行）、用户主动让线程等待，或者被同步块给阻塞，此时就对应着多个状态：time waiting（睡眠或等待一定的事件）、waiting（等待被唤醒）、blocked（阻塞）。<li>
<strong>阻塞（blocked）状态</strong>：BLOCKED称为阻塞状态，或者说线程已经被挂起，它“睡着”了，原因通常是它在等待一个“锁”，当尝试进入一个synchronized语句块/方法时，锁已经被其它线程占有，就会被阻塞，直到另一个线程走完临界区或发生了相应锁对象的wait()操作后，它才有机会去争夺进入临界区的权利
在Java代码中，需要考虑synchronized的粒度问题，否则一个线程长时间占用锁，其它争抢锁的线程会一直阻塞，直到拥有锁的线程释放锁
处于BLOCKED状态的线程，即使对其调用 thread.interrupt()也无法改变其阻塞状态，因为interrupt()方法只是设置线程的中断状态，即做一个标记，不能唤醒处于阻塞状态的线程。（但是能将处于wait状态的线程强制唤醒）。
<strong>注意</strong>：ReentrantLock.lock()操作后进入的是WAITING状态，其内部调用的是LockSupport.park()方法。<li>
<strong>无期限等待（waiting）状态：</strong>处于无期限等待状态的线程不会被分配CPU执行时间，它们要等待显示的被其它线程唤醒。这种状态通常是指一个线程拥有对象锁后进入到相应的代码区域后，调用相应的“锁对象”的wait()方法操作后产生的一种结果。变相的实现还有LockSupport.park()、Thread.join()等，它们也是在等待另一个事件的发生，也就是描述了等待的意思。

|进入方法|退出方法|
|-|-|
|没有设置 Timeout 参数的 Object.wait() 方法|Object.notify() / Object.notifyAll()|
|没有设置 Timeout 参数的 Thread.join() 方法|被调用的线程执行完毕|
|LockSupport.park() 方法|-|

<strong>注意：</strong>
LockSupport.park(Object blocker) 会挂起当前线程，参数blocker是用于设置当前线程的“volatile Object parkBlocker 成员变量”

parkBlocker 是用于记录线程是被谁阻塞的，可以通过LockSupport.getBlocker()获取到阻塞的对象，用于监控和分析线程用的。

“阻塞”与“等待”的区别：

- “阻塞”状态是等待着获取到一个排他锁，进入“阻塞”状态都是被动的，离开“阻塞”状态是因为其它线程释放了锁，不阻塞了；
- “等待”状态是在等待一段时间 或者 唤醒动作的发生，进入“等待”状态是主动的。

<strong>期限等待（time waiting）状态：</strong>处于期限等待状态的线程也不会被分配CPU执行时间，不过无需等待被其它线程显示的唤醒，在一定时间之后它们会由系统自动的唤醒。

|进入方法|退出方法|
|-|-|
|Thread.sleep() 方法|时间结束|
|设置了 Timeout 参数的 Object.wait() 方法|时间结束 / Object.notify() / Object.notifyAll()|
|设置了 Timeout 参数的 Thread.join() 方法|时间结束 / 被调用的线程执行完毕|
|LockSupport.parkNanos() 方法|-|
|LockSupport.parkUntil() 方法|-|

<strong>消亡（terminated）状态:</strong>当由于突然中断或者子任务执行完毕，线程就会被消亡。

## 1.3 实现线程的方法

实现线程有三种方法：

- 继承 Thread 类。
- 实现 Runnable 接口；
- 实现 Callable 接口;

### 1.3.1 继承Tread类
(1)步骤：

  - 定义一个类(记为classA)继承Thread类。
  - 覆盖Thread类中的run方法。
  - 直接用classA创建对象。
  - 使用对象调用start()方法。

```java
/**
 * 通过继承Thread的方法创建线程
 */
class MyThread extends Thread
{
    @Override
    public void run() {
        System.out.println("创建的线程："+ Thread.currentThread().getName());
    }
}

public class ExtendThreadClass {
    public static void main(String[] args) {
        MyThread myThread = new MyThread();
        myThread.start();
    }
}

```

(2)注意：

在main中调用run（）方法和调用其他普通的方法一样，此时只有主线程一个线程，此时cpu被主线程占据，是在主线程中执行，currentThread（）返回的是主线程main，线程名和线程id是主线程的；而调用start（）方法是开始执行一个新线程，此时cpu被该线程占据，是在该线程中执行，currentThread（）返回的是该线程，线程名和线程id就是该线程自己的了。如下：

```java
class MyThread extends Thread
{
    @Override
    public void run() {
        System.out.println("创建的线程："+ Thread.currentThread().getName()+" "+Thread.currentThread().getId());
    }
}

public class ExtendThreadClass {
    public static void main(String[] args) {
        MyThread myThread = new MyThread();
        myThread.start();  //创建的线程：Thread-0 11
        myThread.run();    //创建的线程：main 1
    }
}
```

### 1.3.2实现Runnable接口
(1)步骤：

  - 定义类实现Runnable接口。
  - 覆盖街扩中的run方法，将线程的任务代码封装到run方法中。
  - 通过Thread类创建线程对象，并将Runnable接口的子类对象作为Thread类的构造函数的 参数进行传递。为什么？因为线程任务都封装在Runnable接口子类对象的run方法中。
  - 调用线程对象的star方法开启线程。

```java
/**
 * 实现Runnable接口创建线程
 */
class MyThread implements Runnable
{
    @Override
    public void run() {
        System.out.println("创建的线程: "+Thread.currentThread().getName()+" "+Thread.currentThread().getId());
    }
}

public class ImplRunnableInter {
    public static void main(String[] args) {
        MyThread myThread = new MyThread();

        Thread t0 = new Thread(myThread);

        t0.start();
    }
}
```

### 1.3.3 实现Callable接口：使用ExecutorService、Callable、Future实现有返回结果的多线程

可返回值的任务必须实现Callable接口，类似的，无返回值的任务必须实现Runnable接口。执行Callable任务后，可以获取一个Future的对象，在该对象上调用get就可以获取到Callable任务返回的Object了，再结合线程池接口ExecutorService就可以实现有返回结果的多线程了。

```java
/**
 *实现callable接口创建线程
 */
class MyCallable implements Callable<Object>
{
    private String taskNum;

    MyCallable(String taskNum)
    {
        this.taskNum = taskNum;
    }

    @Override
    public Object call() {
        System.out.println(">>>"+taskNum+"任务启动");
        Date dateTmp1 = new Date();
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        Date dateTmp2 = new Date();
        long time = dateTmp2.getTime() - dateTmp1.getTime();
        System.out.println(">>>"+taskNum + "任务终止");

        return taskNum + "任务返回运行结果，当前任务时间["+time+"毫秒]";
    }
}

public class ImplCallableInter {
    public static void main(String[] args) {
        System.out.println("程序开始运行..........");
        Date date1 = new Date();

        int taskSize = 5;   //线程池中可以容纳的线程数量
        ExecutorService pool = Executors.newFixedThreadPool(taskSize);  //创建一个线程池
        List<Future> list = new ArrayList<Future>(); //创建有多个返回值的任务

        for(int i = 0; i < taskSize; i++)
        {
            Callable c = new MyCallable(i + " ");
            Future f = pool.submit(c);  //执行任务并获取Future对象
            list.add(f);
        }
        pool.shutdown(); //关闭线程池

        /**
         * 获取所有并发任务的运行结果
         */
        for(Future f : list)
        {
            try {
                System.out.println(">>>"+f.get().toString()); //从future对象上获取任务的返回值，并输出到控制台
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (ExecutionException e) {
                e.printStackTrace();
            }
        }

        Date date2 = new Date();
        System.out.println("程序运行结束.......，程序运行时间["+(date2.getTime() - date1.getTime())+"毫秒]");
    }
}
```
代码说明：

上述代码中Executors类，提供了一系列工厂方法用于创建线程池，返回的线程池都实现了ExecutorService接口。

ExecutoreService提供了submit()方法，传递一个Callable，或Runnable，返回Future。如果Executor后台线程池还没有完成Callable的计算，调用返回Future对象的get()方法，会阻塞直到计算完成

|方法|作用|
|-|-|
|public static ExecutorService newFixedThreadPool(int nThreads)|创建固定数目线程的线程池|
|public static ExecutorService newCachedThreadPool()|创建一个可缓存的线程池，调用execute 将重用以前构造的线程（如果线程可用）。如果现有线程没有可用的，则创建一个新线程并添加到池中。终止并从缓存中移除那些已有 60 秒钟未被使用的线程。|
|public static ExecutorService newSingleThreadExecutor()|创建一个单线程化的Executor|
|public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize)|创建一个支持定时及周期性的任务执行的线程池，多数情况下可用来替代Timer类|

### 1.3.4 实现接口VS继承Thread类

实现接口会更好一些，因为：

  - Java 不支持多重继承，因此继承了 Thread 类就无法继承其它类，但是可以实现多个接口；
  - 类可能只要求可执行就行，继承整个 Thread 类开销过大。

## 1.4 线程常用的方法
线程常用的方法有如下几个：

|编号|方法|功能|
|-|-|-|
|1|public void start()|使该线程开始执行；Java 虚拟机调用该线程的 run 方法。|
|2|public void run()|如果该线程是使用独立的 Runnable 运行对象构造的，则调用该 Runnable 对象的 run 方法；否则，该方法不执行任何操作并返回。|
|3|public final void setName(String name)|改变线程名称，使之与参数 name 相同。|
|4|public final void setPriority(int priority)|更改线程的优先级。|
|5|public final void setDaemon(boolean on)|将该线程标记为守护线程或用户线程。|
|6|public final void join(long millisec)|等待该线程终止的时间最长为 millis 毫秒。|
|7|public void interrupt()|中断线程。|
|8|public final boolean isAlive()|测试线程是否处于活动状态。|
|9|public static void yield()|暂停当前正在执行的线程对象，并执行其他线程。|
|10|public static void sleep(long millisec)|在指定的毫秒数内让当前正在执行的线程休眠（暂停执行），此操作受到系统计时器和调度程序精度和准确性的影响。|
|11|public static Thread currentThread()|返回对当前正在执行的线程对象的引用。|

### 1.4.1 静态方法
#### currentThread()方法

currentThread()方法可以返回代码段正在被哪个线程调用的信息。

#### sleep()方法

sleep()的作用是在指定的毫秒数内让当前“正在执行的线程”休眠（暂停执行）。这个“正在执行的线程”是指this.currentThread()返回的线程。<strong>注意：sleep方法不释放锁，但是释放CPU执行权。</strong>

#### yield()方法

调用yield方法会让当前线程交出CPU权限，让CPU去执行其他的线程。<strong>它跟sleep方法类似，同样不会释放锁。</strong>但是yield不能控制具体的交出CPU的时间，另外，yield方法只能让拥有相同优先级的线程有获取CPU执行时间的机会。

注意，调用yield方法并不会让线程进入阻塞状态，而是让线程重回就绪状态，它只需要等待重新获取CPU执行时间，这一点是和sleep方法不一样的。

<strong>使用yield()的目的是让具有相同优先级的线程之间能够适当的轮换执行。</strong>

### 1.4.2 静态方法
#### start()方法

start()用来启动一个线程，当调用start方法后，系统才会开启一个新的线程来执行用户定义的子任务，在这个过程中，会为相应的线程分配需要的资源。

#### run()方法

run()方法是不需要用户来调用的，当通过start方法启动一个线程之后，当线程获得了CPU执行时间，便进入run方法体去执行具体的任务。注意，继承Thread类必须重写run方法，在run方法中定义具体要执行的任务。

#### getId()

getId()的作用是取得线程的唯一标识。

#### isAlive()方法
方法isAlive()的功能是判断当前线程是否处于活动状态。方法isAlive()的作用是测试线程是否处于活动状态。什么是活动状态呢？活动状态就是线程已经启动且尚未终止。线程处于正在运行或准备开始运行的状态，就认为线程是“存活”的。

```java
/**
 * 测试isAlive函数
 */
class MyThread extends Thread
{
    @Override
    public void run() {
        System.out.println("run= "+this.isAlive());
    }
}

public class IsAliveTest {
    public static void main(String[] args) {
        MyThread myThread = new MyThread();
        System.out.println("begin: "+myThread.isAlive());
        myThread.start();
        System.out.println("end: "+myThread.isAlive());
    }
}
```

上述程序执行结果不确定，某次的执行结果为:

```java
begin: false
end: true
run= true
```

虽然上面的实例中end打印的值是true,但此值是不确定的。打印true值是因为myThread线程还未执行完毕，所以输出true。如果代码改成下面这样，加了个sleep休眠：

```java
public class IsAliveTest {
    public static void main(String[] args) {
        MyThread myThread = new MyThread();
        System.out.println("begin: "+myThread.isAlive());
        myThread.start();
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("end: "+myThread.isAlive());
    }
}
```
打印结果如下所示。`end: false`的原因是，主线程进行休眠的这段时间，线程`MyThread`已经执行完毕了。

```java
begin: false
run= true
end: false
```

#### join()函数

在很多情况下，主线程创建并启动了线程，如果子线程中要进行大量耗时运算，主线程往往将早于子线程结束之前结束。这时，如果主线程想等待子线程执行完成之后再结束，比如子线程处理一个数据，主线程要取得这个数据中的值，就要用到join()方法了。方法join()的作用是等待线程对象销毁。

更一般地说，在A线程中调用了B线程的join()方法时，表示只有当B线程执行完毕时，A线程才能继续执行；join方法中如果传入参数，则表示：如果A线程中调用B线程的join(10)，则表示A线程会等待B线程执行10毫秒，10毫秒过后，A、B线程并行执行。所以也可以说join()方法能够使得线程之间的并行执行变为串行执行。

那么join()方法的原理是什么呢？

其实，join方法是通过调用线程的wait方法来达到同步的目的的。例如，A线程中调用了B线程的join方法，则相当于A线程调用了B线程的wait方法，在调用了B线程的wait方法后，A线程就会进入阻塞状态，具体看下面的源码：

```java
public final synchronized void join(long millis)
throws InterruptedException {
    long base = System.currentTimeMillis();
    long now = 0;

    if (millis < 0) {
        throw new IllegalArgumentException("timeout value is negative");
    }

    if (millis == 0) {
        while (isAlive()) {
            wait(0);
        }
    } else {
        while (isAlive()) {
            long delay = millis - now;
            if (delay <= 0) {
                break;
            }
            wait(delay);
            now = System.currentTimeMillis() - base;
        }
    }
}
```

***问题：为什么wait()后等待的是主线程而不是调用join()函数的线程对象？？为什么自己写的程序用线程对象调用wait()函数时，等待的就是该线程对象？？？***

***这个应该是谁调用join()函数谁阻塞，而和调用的谁的join()函数无关。***

#### getName和setName

用来得到或者设置线程名称。

#### getPriority和setPriority

用来获取和设置线程优先级。

#### setDaemon和isDaemo

用来设置线程是否成为守护线程和判断线程是否是守护线程。

守护线程和用户线程的区别在于：**守护线程依赖于创建它的线程，而用户线程则不依赖(守护线程就像一个仆人，为主人(创建它的线程)服务，主人没了它也就没了)**。举个简单的例子：如果在main线程中创建了一个守护线程，当main方法运行完毕之后，守护线程也会随着消亡。而用户线程则不会，用户线程会一直运行直到其运行完毕。在JVM中，像垃圾收集器线程就是守护线程。

## 1.5 基础线程机制
### Executor

Executor 管理多个异步任务的执行，而无需程序员显式地管理线程的生命周期。这里的异步是指多个任务的执行互不干扰，不需要进行同步操作。

主要有三种 Executor

  - CachedThreadPool：一个任务创建一个线程；
  - FixedThreadPool：所有任务只能使用固定大小的线程；
  - SingleThreadExecutor：相当于大小为 1 的 FixedThreadPool。

```java
public static void main(String[] args) {
    ExecutorService executorService = Executors.newCachedThreadPool();
    for (int i = 0; i < 5; i++) {
        executorService.execute(new MyRunnable());
    }
    executorService.shutdown();
}
```

### Daemon
在Java线程中有两种线程，一种是User Thread（用户线程），另一种是Daemon Thread(守护线程)。

Daemon的作用是为其他线程的运行提供服务，比如说GC线程。其实User Thread线程和Daemon Thread守护线程本质上来说去没啥区别的，唯一的区别之处就在虚拟机的离开：如果User Thread全部撤离，那么Daemon Thread也就没啥线程好服务的了，所以虚拟机也就退出了。

守护线程并非虚拟机内部可以提供，用户也可以自行的设定守护线程，方法：public final void setDaemon(boolean on) ；但是有几点需要注意：

  - thread.setDaemon(true)必须在thread.start()之前设置，否则会抛出一个IllegalThreadStateException异常。不能把正在运行的常规线程设置为守护线程。 （备注：这点与守护进程有着明显的区别，守护进程是创建后，让进程摆脱原会话的控制+让进程摆脱原进程组的控制+让进程摆脱原控制终端的控制；所以说寄托于虚拟机的语言机制跟系统级语言有着本质上面的区别）
  - 在Daemon线程中产生的新线程也是Daemon的。 （这一点又是有着本质的区别了：守护进程fork()出来的子进程不再是守护进程，尽管它把父进程的进程相关信息复制过去了，但是子进程的进程的父进程不是init进程，所谓的守护进程本质上说就是“父进程挂掉，init收养，然后文件0,1,2都是/dev/null，当前目录到/”）
  - 不是所有的应用都可以分配给Daemon线程来进行服务，比如读写操作或者计算逻辑。因为在Daemon Thread还没来的及进行操作时，虚拟机可能已经退出了。

### sleep()

Thread.sleep(millisec) 方法会休眠当前正在执行的线程，millisec 单位为毫秒。

sleep() 可能会抛出 InterruptedException，因为异常不能跨线程传播回 main() 中，因此必须在本地进行处理。线程中抛出的其它异常也同样需要在本地进行处理。

### yield()

对静态方法 Thread.yield()的调用声明了当前线程已经完成了生命周期中最重要的部分，可以切换给其它线程来执行。该方法只是对线程调度器的一个建议，而且也只是建议具有相同优先级的其它线程可以运行。

## 1.6 停止线程的方法
### 使用stop方法强行终止线程

不推荐使用这个方法，因为stop和suspend及resume一样，都是作废过期的方法，使用他们可能产生不可预料的结果。

### 使用退出标志，使线程正常退出，也就是当run方法完成后线程终止

要想使线程在某一特定条件下退出，最直接的方法就是设一个boolean类型的标志，并通过设置这个标志为true或false来控制while循环是否退出：

```java
/**
 * 使用标志位来退出线程
 */
class MyThread extends Thread
{
    public volatile boolean exitFlag = false;

    @Override
    public void run() {
        while(! exitFlag)
        {
            System.out.println("我还没退出 "+Thread.currentThread().getName()+" "+Thread.currentThread().getId());
        }
    }
}

public class ExitThread {
    public static void main(String[] args) {
        MyThread myThread = new MyThread();

        myThread.start();
        try {
            Thread.sleep(1000);    //主线程睡1s
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        myThread.exitFlag = true;
        try {
            myThread.join();         //等待线程退出
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("我已退出");
    }
}
```

在定义exit时，使用了一个Java关键字volatile，这个关键字的目的是使exitFlag同步，也就是说在同一时刻只能由一个线程来修改exitFlag的值。

但是有一种情况下使用标志也是退不出线程的，比如下面：

```java
/**
 * 使用标志位来退出线程
 */
class MyThread extends Thread
{
    public volatile boolean exitFlag = false;

    @Override
    public synchronized void run() {
        while(! exitFlag)
        {
            try {
                wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("我还没退出 "+Thread.currentThread().getName()+" "+Thread.currentThread().getId());
        }
    }
}

public class ExitThread {
    public static void main(String[] args) {
        MyThread myThread0 = new MyThread();
        MyThread myThread1 = new MyThread();

        myThread0.start();
        myThread1.start();
        try {
            Thread.sleep(1000);    //主线程睡1s
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        myThread0.exitFlag = true;
        myThread1.exitFlag = true;
        System.out.println("我已退出");
    }
}
```
该程序中，在run方法上加了同步锁，并且加了wait函数，这种情况下，两个子线程都停不下来，因为它们都进入`wait`状态且没有别的线程唤醒它们，但是主线程可以停。

### 使用interrupt方法中断线程，但这个不会终止一个正在运行的线程，还需要加入一个判断才可以完成线程的停止

使用interrupt()方法来中断线程有两种情况：

**(1)线程处于阻塞状态**：如使用了sleep,同步锁的wait,socket中的receiver,accept等方法时，会使线程处于阻塞状态。当调用线程的interrupt()方法时，会抛出InterruptException异常。阻塞中的那个方法抛出这个异常，通过代码捕获该异常，然后break跳出循环状态或者修改标志位，从而让我们有机会结束这个线程的执行。<strong>也就是说，interrupt()方法其实是将线程强制唤醒，让它们具有执行资格，然后再将其停止。但强制动作发生时会产生InterruptedException，所以要处理一下。</strong>

通常很多人认为只要调用interrupt方法线程就会结束，实际上是错的， 一定要先捕获InterruptedException异常之后通过break来跳出循环，才能正常结束run方法。例子如下：

```java
/**
 * interrupt方法结束线程
 */
class MyThread extends Thread
{
    @Override
    public void run() {
        while (true)
        {
            System.out.println("子线程还没退出");
            try {
                sleep(200);
            } catch (InterruptedException e) {
                e.printStackTrace();
                break;
            }
        }
    }
}

public class InterruptExitThread {
    public static void main(String[] args) {
        MyThread myThread0 = new MyThread();

        myThread0.start();
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        myThread0.interrupt();

        System.out.println("主线程退出");
    }
}
```
本例中，在主线程中调用myThread0线程的interrupt()并且在run方法中用break跳出循环，最终结果是打印InterruptedException信息，并且程序能够终止。如下：

```java
子线程还没退出
子线程还没退出
子线程还没退出
子线程还没退出
子线程还没退出
子线程还没退出
主线程退出
//以下是打印的InterruptedException信息，并不是报错。
java.lang.InterruptedException: sleep interrupted
	at java.lang.Thread.sleep(Native Method)
	at thread.MyThread.run(ExitThreadInterrupt.java:45)
```

**（2）线程未处于阻塞状态**：使用isInterrupted()判断线程的中断标志来退出循环。当使用interrupt()方法时，中断标志就会置true，和使用自定义的标志来控制循环是一样的道理。如下例：

```java
public class ThreadSafe extends Thread {
    public void run() {
        while (!isInterrupted()){
            //do something, but no throw InterruptedException
        }
    }
}
```

为什么要区分进入阻塞状态和和非阻塞状态两种情况了，是因为当阻塞状态时，如果有interrupt()发生，系统除了会抛出InterruptedException异常外，还会调用interrupted()函数，调用时能获取到中断状态是true的状态，调用完之后会复位中断状态为false，所以异常抛出之后通过isInterrupted()是获取不到中断状态是true的状态，从而不能退出循环。

因此在线程未进入阻塞的代码段时是可以通过isInterrupted()来判断中断是否发生来控制循环，在进入阻塞状态后要通过捕获异常来退出循环。因此使用interrupt()来退出线程的最好的方式应该是两种情况都要考虑：

```java
public class ThreadSafe extends Thread {
    public void run() {
        while (!isInterrupted()){ //非阻塞过程中通过判断中断标志来退出
            try{
                Thread.sleep(5*1000);//阻塞过程捕获中断异常来退出
            }catch(InterruptedException e){
                e.printStackTrace();
                break;//捕获到异常之后，执行break跳出循环。
            }
        }
    }
}
```

<strong>需要注意的是，`isInterrupted()`是`Thread`类中的方法，使用`isInterrupted()`方法的线程类必须继承`Thread`，实现`Runnable`的类是用不了这个方法的。</strong>

# 2. 多线程基础
## 2.1 同步

同步的方法大概有以下几种：

### synchronized机制
#### (1) 同步代码块

它只作用于同一个对象，如果调用两个对象上的同步代码块，就不会进行同步。同步代码块的格式如下：

```java
synchronized(对象)
{
	需要被同步的代码;
}
```
```java
/**
 * 卖票案例：同步代码块
 */
class Ticket implements Runnable
{
    private int num = 100;

    @Override
    public void run() {
        while (true)
        {
            synchronized (this)
            {
                if(num > 0)
                {
                    try {
                        Thread.sleep(10);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println(Thread.currentThread().getName()+"...sale...."+num--);
                }
                else
                {
                    break;
                }
            }
        }
    }
}

public class TicketDemo {
    public static void main(String[] args) {
        Ticket ticket = new Ticket();

        Thread t0 = new Thread(ticket);
        Thread t1 = new Thread(ticket);

        t0.start();
        t1.start();
    }
}
```

#### (2)同步函数

同步函数就是将同步关键字synchronized加载需要被同步的函数上，函数内部是需要被同步的代码，示例如下：

```java
/**
 * 多线程同步：存钱问题
 */
class Bank{
    private int sum;

    public synchronized void add(int num){
        sum += num;

        try {
            Thread.sleep(10);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("sum= :"+sum);
    }
}

class Customer implements Runnable{
    private Bank bank = new Bank();

    @Override
    public void run() {
        for(int i = 0; i < 3; i++){
            bank.add(100);
        }
    }
}

public class BandDemo {
    public static void main(String[] args) {
        Customer customer = new Customer();

        Thread t1 = new Thread(customer);
        Thread t2 = new Thread(customer);

        t1.start();
        t2.start();
    }
}
```

#### (3)同步一个类
作用于整个类，也就是说两个线程调用同一个类的不同对象上的这种同步语句，也会进行同步。

```java
public class SynchronizedExample {

    public void func2() {
        synchronized (SynchronizedExample.class) {
            for (int i = 0; i < 10; i++) {
                System.out.print(i + " ");
            }
        }
    }
}
public static void main(String[] args) {
    SynchronizedExample e1 = new SynchronizedExample();
    SynchronizedExample e2 = new SynchronizedExample();
    ExecutorService executorService = Executors.newCachedThreadPool();
    executorService.execute(() -> e1.func2());
    executorService.execute(() -> e2.func2());
}
结果为：0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9
```
### 同步锁机制（Lock）
使用显式的lock来进行同步，一个lock上面可以挂多个监视器。示例如下：

```java
/**
 * 多生产者和消费者问题:lock
 */

class Resource
{
    private String name;
    private int count;
    private boolean flag = false;

    Lock lock = new ReentrantLock();
    Condition producerLock = lock.newCondition();
    Condition customerLock = lock.newCondition();

    public void set(String name)
    {
        lock.lock();
        try {
            while (flag)//flag为1，表示还没消费完
            {
                try {
                    producerLock.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            this.name = name + count;
            count++;
            System.out.println(Thread.currentThread().getName() + " 生产者...... " + this.name);
            flag = true;
            customerLock.signal();
        }
        finally {
            lock.unlock();
        }
    }

    public void out()
    {
        lock.lock();
        try {
            while(! flag)
            {
                try {
                    customerLock.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            //this.name = name + count;
            //count--;
            System.out.println(Thread.currentThread().getName()+" 消费者............... " +this.name);
            flag = false;
            producerLock.signal();
        }
        finally {
            lock.unlock();
        }
    }
}

class Producer implements Runnable
{
    private Resource resource;

    public Producer(Resource resource)
    {
        this.resource = resource;
    }

    @Override
    public void run() {
        while(true)
        {
            resource.set("烤鸭");
        }
    }
}

class Customer implements Runnable
{
    private Resource resource;
    public Customer(Resource resource)
    {
        this.resource = resource;
    }

    @Override
    public void run() {
        while(true)
        {
            resource.out();
        }
    }
}

public class ProducerAndCustomer {
    public static void main(String[] args) {
        Resource resource = new Resource();
        Producer producer = new Producer(resource);
        Customer customer = new Customer(resource);

        Thread t0 = new Thread(producer);
        Thread t1 = new Thread(producer);
        Thread t2 = new Thread(customer);
        Thread t3 = new Thread(customer);

        t0.start();
        t1.start();
        t2.start();
        t3.start();
    }
}
```
### volatile关键字实现同步
volatile关键字为域变量的访问提供了一种免锁机制，使用volatile修饰域相当于告诉虚拟机该域可能会被其他线程更新，因此每次使用该域就要重新计算，而不是使用寄存器中的值，volatile不会提供任何原子操作，它也不能用来修饰final类型的变量。

注：多线程中的非同步问题主要出现在对域的读写上，如果让域自身避免这个问题，则就不需要修改操作该域的方法。用final域，有锁保护的域和volatile域可以避免非同步的问题。
示例如下：

```java
/**
 * 卖票案例：volatile关键字
 */
class Ticket implements Runnable
{
    private volatile int num = 100;

    @Override
    public void run() {
        while (true)
        {
            if(num > 0)
            {
                System.out.println(Thread.currentThread().getName()+"...sale...."+num--);
            }
            else
            {
                break;
            }
        }
    }
}
public class VolatileDemo {
    public static void main(String[] args) {
        Ticket ticket = new Ticket();

        Thread t0 = new Thread(ticket);
        Thread t1 = new Thread(ticket);

        t0.start();
        t1.start();
    }
}
```

### 使用阻塞队列LinkedBlockingQueue实现线程同步
阻塞队列的特点是，当队列是空的时，从队列中获取元素的操作将会被阻塞，或者当队列是满时，往队列里添加元素的操作会被阻塞。

LinkedBlockingQueue<E>是一个基于已连接节点的，范围任意的blocking queue。 

LinkedBlockingQueue 类常用方法：

|名称|作用|
|-|-|
|LinkedBlockingQueue()|创建一个容量为Integer.MAX_VALUE的LinkedBlockingQueue|
|put(E e)|在队尾添加一个元素，如果队列满则阻塞|
|size()|返回队列中的元素个数|
|take()|移除并返回队头元素，如果队列空则阻塞|

BlockingQueue<E>定义了阻塞队列的常用方法，尤其是三种添加元素的方法，我们要多加注意，当队列满时：add()方法会抛出异常；offer()方法返回false；put()方法会阻塞。

```java
/**
 * 用阻塞队列实现线程同步 LinkedBlockingQueue的使用
 */
public class BlockingSynchronizedThread {
    /**
     * 定义一个阻塞队列用来存储生产出来的商品
     */
    private LinkedBlockingQueue<Integer> queue = new LinkedBlockingQueue<>();
    /**
     * 定义生产商品个数
     */
    private static final int size = 10;
    /**
     * 定义启动线程的标志，为0时，启动生产商品的线程；为1时，启动消费商品的线程
     */
    private int flag = 0;

    private class LinkBlockThread implements Runnable {
        @Override
        public void run() {
            int new_flag = flag++;
            System.out.println("启动线程 " + new_flag);
            if (new_flag == 0) {
                for (int i = 0; i < size; i++) {
                    System.out.println("生产商品：" + i + "号");
                    try {
                        queue.put(i);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println("仓库中还有商品：" + queue.size() + "个");
                    try {
                        Thread.sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            } else {
                for (int i = 0; i < size / 2; i++) {
                    try {
                        int n = queue.take();
                        System.out.println("消费者买去了" + n + "号商品");
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println("仓库中还有商品：" + queue.size() + "个");
                    try {
                        Thread.sleep(100);
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                }
            }
        }
    }

    public static void main(String[] args) {
        BlockingSynchronizedThread bst = new BlockingSynchronizedThread();
        LinkBlockThread lbt = bst.new LinkBlockThread();
        Thread thread1 = new Thread(lbt);
        Thread thread2 = new Thread(lbt);
        thread1.start();
        thread2.start();

    }
}
```

### 使用AtometicInteger定义原子变量实现同步
需要使用线程同步的根本原因在于对普通变量的操作不是原子的。那么什么是原子操作呢？原子操作就是指将读取变量值、修改变量值、保存变量值看成一个整体来操作。即-**这几种行为要么同时完成，要么都不完成。**

在java的util.concurrent.atomic包中提供了创建了原子类型变量的工具类，使用该类可以简化线程同步。其中AtomicInteger可以用原子方式更新int的值，可用在应用程序中(如以原子方式增加的计数器)，但不能用于替换Integer；可扩展Number，允许那些处理机遇数字类的工具和实用工具进行统一访问。

AtomicInteger类常用方法：

|方法|作用|
|-|-|
|AtomicInteger(int initialValue)|创建具有给定初始值的新的AtomicInteger|
|addAddGet(int dalta) |以原子方式将给定值与当前值相加|
|get()|获取当前值|

```java
/**
 * 卖票案例：AtomicInteger
 */
class Ticket implements Runnable
{
    private AtomicInteger num = new AtomicInteger(100);

    @Override
    public void run() {
        while (true)
        {
            if(num.get() > 0)
            {
                System.out.println(Thread.currentThread().getName()+"...sale...."+num.getAndDecrement());//注意这里
            }
            else
            {
                break;
            }
        }
    }
}
public class AutometicIntegerDemo {
    public static void main(String[] args) {
        Ticket ticket = new Ticket();

        Thread t0 = new Thread(ticket);
        Thread t1 = new Thread(ticket);

        t0.start();
        t1.start();
    }
}
```

### 小结
#### 同步函数和同步代码块的锁

- 所有的非静态同步方法用的都是同一把锁——实例对象本身，也就是说如果一个实例对象的非静态同步方法获取锁后，该实例对象的其他非静态同步方法必须等待获取锁的方法释放锁后才能获取锁，可是别的实例对象的非静态同步方法因为跟该实例对象的非静态同步方法用的是不同的锁，所以毋须等待该实例对象已获取锁的非静态同步方法释放锁就可以获取他们自己的锁。
- 所有的静态同步方法用的也是同一把锁——类对象本身，这两把锁是两个不同的对象，所以静态同步方法与非静态同步方法之间是不会有竞态条件的。但是一旦一个静态同步方法获取锁后，其他的静态同步方法都必须等待该方法释放锁后才能获取锁，而不管是同一个实例对象的静态同步方法之间，还是不同的实例对象的静态同步方法之间，只要它们同一个类的实例的对象！
- 对于同步块，由于其锁是可以选择的，所以只有使用同一把锁的同步块之间才有着竞争条件。
也就是说：
    - 同步代码块使用的锁是任意的对象，只要保证多个线程使用的是唯一的锁就行。
    - 非静态同步代码块使用的锁一般是this。
    - 静态同步代码块使用的锁一般是 类名.class或者this.getClass（比较别扭）。

#### ReentrantLock和synchronized比较

- 锁的实现
synchronized 是 JVM 实现的，而 ReentrantLock 是 JDK 实现的。
- 性能
新版本 Java 对 synchronized 进行了很多优化，例如自旋锁等，synchronized 与 ReentrantLock 大致相同。
-等待可中断
当持有锁的线程长期不释放锁的时候，正在等待的线程可以选择放弃等待，改为处理其他事情。
ReentrantLock 可中断，而 synchronized 不行。
- 公平锁
公平锁是指多个线程在等待同一个锁时，必须按照申请锁的时间顺序来依次获得锁。
synchronized 中的锁是非公平的，ReentrantLock 默认情况下也是非公平的，但是也可以是公平的。
- 锁绑定多个条件
一个 ReentrantLock 可以同时绑定多个 Condition 对象。

使用选择：除非需要使用 ReentrantLock 的高级功能，否则优先使用 synchronized。这是因为 synchronized 是 JVM 实现的一种锁机制，JVM 原生地支持它，而 ReentrantLock 不是所有的 JDK 版本都支持。并且使用 synchronized 不用担心没有释放锁而导致死锁问题，因为 JVM 会确保锁的释放。

## 2.2 死锁
### 死锁的示例：同步的嵌套

Java中死锁最简单的情况是，一个线程T1持有锁L1并且申请获得锁L2，而另一个线程T2持有锁L2并且申请获得锁L1，因为默认的锁申请操作都是阻塞的，所以线程T1和T2永远被阻塞了。导致了死锁。这是最容易理解也是最简单的死锁的形式。但是实际环境中的死锁往往比这个复杂的多。可能会有多个线程形成了一个死锁的环路，比如：线程T1持有锁L1并且申请获得锁L2，而线程T2持有锁L2并且申请获得锁L3，而线程T3持有锁L3并且申请获得锁L1，这样导致了一个锁依赖的环路：T1依赖T2的锁L2，T2依赖T3的锁L3，而T3依赖T1的锁L1。从而导致了死锁。

举个形象的例子，两个人在吃饺子，其中甲手里攥着醋，乙手里攥着辣椒油，甲对乙说：把辣椒油给我我就给你醋。乙对甲说：把醋给我我就给你辣椒油。就这样两个人就杠了起来，谁也吃不了。

所以，总结起来死锁产生的原因有两个:
  - 锁的嵌套;
  - 申请锁是默认阻塞的;


示例1如下：
```java
/**
 * 死锁：同步的嵌套
 */
class Test implements Runnable
{
    private boolean flag;

    Test(boolean flag)
    {
        this.flag = flag;
    }

    public void setFlag(boolean flag) {
        this.flag = flag;
    }

    @Override
    public void run() {
        if(flag)
        {
            while(true)
            {
                synchronized (MyLock.locka)
                {
                    System.out.println(Thread.currentThread().getName()+" if locka.....");
                    synchronized (MyLock.lockb)
                    {
                        System.out.println(Thread.currentThread().getName()+" if lockb......");
                    }
                }
            }
        }
        else
        {
            while(true)
            {
                synchronized (MyLock.lockb)
                {
                    System.out.println(Thread.currentThread().getName()+" else lockb...");
                    synchronized (MyLock.locka)
                    {
                        System.out.println(Thread.currentThread().getName()+" else locka...");
                    }
                }
            }
        }
    }
}

class MyLock{
    public static final Object locka = new Object();
    public static final Object lockb = new Object();
}

public class DeadLock {
    public static void main(String[] args) {
//        Test a = new Test(true);
//        Test b = new Test(false);
//
//        Thread t1 = new Thread(a);
//        Thread t2 = new Thread(b);
//
//        t1.start();
//        t2.start();

        Test a = new Test(true);

        Thread t1 = new Thread(a);
        Thread t2 = new Thread(a);

        t1.start();
        try {
            Thread.sleep(10);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        a.setFlag(false);
        t2.start();
    }
}
```

示例2如下：
```java
/**
 * 死锁：同步的嵌套
 */
class Test implements Runnable
{
    private boolean flag;
    Lock locka = new ReentrantLock();
    Lock lockb = new ReentrantLock();

    Test(boolean flag)
    {
        this.flag = flag;
    }

    public void setFlag(boolean flag) {
        this.flag = flag;
    }

    @Override
    public void run() {
        if(flag)
        {
            while(true)
            {
                try {
                    locka.lock();
                    System.out.println(Thread.currentThread().getName()+" if locka.....");
                    try {
                        lockb.lock();
                        System.out.println(Thread.currentThread().getName()+" if lockb......");
                    }
                    finally {
                        lockb.unlock();
                    }
                }
                finally {
                    locka.unlock();
                }
            }
        }
        else
        {
            while(true)
            {
                try {
                    lockb.lock();
                    System.out.println(Thread.currentThread().getName()+" if locka.....");
                    try {
                        locka.lock();
                        System.out.println(Thread.currentThread().getName()+" if lockb......");
                    }
                    finally {
                        locka.unlock();
                    }
                }
                finally {
                    lockb.unlock();
                }
            }
        }
    }
}

public class DeadLock {
    public static void main(String[] args) {
        Test a = new Test(true);

        Thread t1 = new Thread(a);
        Thread t2 = new Thread(a);

        t1.start();
        try {
            Thread.sleep(10);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        a.setFlag(false);
        t2.start();
    }
}
```
### 死锁的解决办法
#### （1）如上死锁产生的原因所示，如果能够避免在一个同步方法中调用其他的同步方法，就能避免死锁。

#### （2）如果无法避免同步的嵌套，只要让方法申请锁的顺序一致，就能够避免死锁。示例如下（该程序和上面的死锁示例基本形同，只是变化了一下申请锁的顺序，两个方法都是先申请locka在申请lockb）：
```java
class Test implements Runnable
{
    private boolean flag;

    Test(boolean flag)
    {
        this.flag = flag;
    }

    public void setFlag(boolean flag) {
        this.flag = flag;
    }

    @Override
    public void run() {
        if(flag)
        {
            while(true)
            {
                synchronized (MyLock.locka)
                {
                    System.out.println(Thread.currentThread().getName()+" if locka.....");
                    synchronized (MyLock.lockb)
                    {
                        System.out.println(Thread.currentThread().getName()+" if lockb......");
                    }
                }
            }
        }
        else
        {
            while(true)
            {
                synchronized (MyLock.locka)
                {
                    System.out.println(Thread.currentThread().getName()+" else lockb...");
                    synchronized (MyLock.lockb)
                    {
                        System.out.println(Thread.currentThread().getName()+" else locka...");
                    }
                }
            }
        }
    }
}

class MyLock{
    public static final Object locka = new Object();
    public static final Object lockb = new Object();
}

public class DeadLock {
    public static void main(String[] args) {
//        Test a = new Test(true);
//        Test b = new Test(false);
//
//        Thread t1 = new Thread(a);
//        Thread t2 = new Thread(b);
//
//        t1.start();
//        t2.start();

        Test a = new Test(true);

        Thread t1 = new Thread(a);
        Thread t2 = new Thread(a);

        t1.start();
        try {
            Thread.sleep(10);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        a.setFlag(false);
        t2.start();
    }
}
```

#### （3）另一种方法就是申请锁的时候加上等待时间，如果等待时间到了还没拿到锁也不会被阻塞，相当于解决了死锁产生的第二个原因（死锁是默认阻塞的）。示例如下：
```java
/**
 * 死锁解决方法之一：申请锁的时候加上等待时间.
 */
class Test implements Runnable
{
    private boolean flag;
    Lock locka = new ReentrantLock();
    Lock lockb = new ReentrantLock();

    Test(boolean flag)
    {
        this.flag = flag;
    }

    public void setFlag(boolean flag) {
        this.flag = flag;
    }

    @Override
    public void run() {
        if(flag)
        {
            while(true)
            {
                try {
                    locka.lock();
                    System.out.println(Thread.currentThread().getName()+" if locka.....");
                    if(lockb.tryLock(100, TimeUnit.MILLISECONDS)) {
                        try {
                            System.out.println(Thread.currentThread().getName()+" if lockb......");
                        }
                        finally {
                            lockb.unlock();
                        }
                    }
                }
                catch (InterruptedException e)
                {
                    e.printStackTrace();
                }
                finally {
                    locka.unlock();
                }
            }
        }
        else
        {
            while(true)
            {
                try {
                    lockb.lock();
                    System.out.println(Thread.currentThread().getName()+" if locka.....");
                    if(locka.tryLock(100, TimeUnit.MILLISECONDS)) {
                       try {
                           System.out.println(Thread.currentThread().getName() + " if lockb......");
                       }
                       finally {
                           locka.unlock();
                       }
                    }
                }
                catch (InterruptedException e)
                {
                    e.printStackTrace();
                }
                finally {
                    lockb.unlock();
                }
            }
        }
    }
}

public class DeadLock {
    public static void main(String[] args) {
        Test a = new Test(true);

        Thread t1 = new Thread(a);
        Thread t2 = new Thread(a);

        t1.start();
        try {
            Thread.sleep(10);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        a.setFlag(false);
        t2.start();
    }
}
```

## 2.3 多线程通信
### join()机制

在线程中调用另一个线程的 join() 方法，会将当前线程挂起，而不是忙等待，直到目标线程结束。

对于以下代码，虽然 b 线程先启动，但是因为在 b 线程中调用了 a 线程的 join() 方法，b 线程会等待 a 线程结束才继续执行，因此最后能够保证 a 线程的输出先于 b 线程的输出。
```java
public class JoinExample {

    private class A extends Thread {
        @Override
        public void run() {
            System.out.println("A");
        }
    }

    private class B extends Thread {

        private A a;

        B(A a) {
            this.a = a;
        }

        @Override
        public void run() {
            try {
                a.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("B");
        }
    }

    public void test() {
        A a = new A();
        B b = new B(a);
        b.start();
        a.start();
    }
}
public static void main(String[] args) {
    JoinExample example = new JoinExample();
    example.test();
}
输出结果为：
A
B
```
### wait/notify机制

调用 wait() 使得线程等待某个条件满足，线程在等待时会被挂起，当其他线程的运行使得这个条件满足时，其它线程会调用 notify() 或者 notifyAll() 来唤醒挂起的线程。<strong>它们都属于 Object 的一部分，而不属于 Thread。</strong>

使用 wait() 挂起期间，线程会释放锁。这是因为，如果没有释放锁，那么其它线程就无法进入对象的同步方法或者同步控制块中，那么就无法执行 notify() 或者 notifyAll() 来唤醒挂起的线程，造成死锁。

wait() 和 sleep() 的区别
  - wait() 是 Object 的方法，而 sleep() 是 Thread 的静态方法；
  - wait() 会释放锁，sleep() 不会。


使用wait/notify机制时，要注意的是在“单生产者但消费者”模式的时候，仅用if与notify机制配合就行，因为不会存在多个生产者或消费者同时wait的情况，这样在被notify后就不用再次判断标记；在“多生产者多消费者”模式的情况下，需要用while和notifyAll配合，线程醒后还需要再判断标记。如果使用whlile和notify机制配合，可能会产生死锁，因为控制不了唤醒哪个具体的线程。示例如下：

单生产者但消费者模式：
```java
/**
 * 生产者和消费者问题（一个生产者一个消费者） if-notify()
 */
class Resource
{
    private String name;
    private int count;
    private boolean flag = false;

    public synchronized void set(String name)
    {
        if(flag)//flag为1，表示还没消费完
        {
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        this.name = name + count;
        count++;
        System.out.println(Thread.currentThread().getName()+" 生产者...... "+this.name);
        flag = true;
        notify();
    }

    public synchronized void out()
    {
        if(! flag)
        {
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        //this.name = name + count;
        //count--;
        System.out.println(Thread.currentThread().getName()+" 消费者............... " +this.name);
        flag = false;
        notify();
    }
}

class Producer implements Runnable
{
    private Resource resource;

    public Producer(Resource resource)
    {
        this.resource = resource;
    }

    @Override
    public void run() {
        while(true)
        {
            resource.set("烤鸭");
        }
    }
}

class Customer implements Runnable
{
    private Resource resource;
    public Customer(Resource resource)
    {
        this.resource = resource;
    }

    @Override
    public void run() {
        while(true)
        {
            resource.out();
        }
    }
}

public class ProducerAndCustomer {
    public static void main(String[] args) {
        Resource resource = new Resource();
        Producer producer = new Producer(resource);
        Customer customer = new Customer(resource);

        Thread t0 = new Thread(producer);
        Thread t1 = new Thread(customer);

        t0.start();
        t1.start();
    }
}
```

多生产者多消费者模式：
```java
/**
 * 多生产者和消费者问题:while-noifyAll
 * while+notify()可能会导致死锁
 */

class Resource
{
    private String name;
    private int count;
    private boolean flag = false;

    public synchronized void set(String name)
    {
        while(flag)//flag为1，表示还没消费完
        {
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        this.name = name + count;
        count++;
        System.out.println(Thread.currentThread().getName()+" 生产者...... "+this.name);
        flag = true;
        notifyAll();
    }

    public synchronized void out()
    {
        while(! flag)
        {
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        //this.name = name + count;
        //count--;
        System.out.println(Thread.currentThread().getName()+" 消费者............... " +this.name);
        flag = false;
        notifyAll();
    }
}

class Producer implements Runnable
{
    private Resource resource;

    public Producer(Resource resource)
    {
        this.resource = resource;
    }

    @Override
    public void run() {
        while(true)
        {
            resource.set("烤鸭");
        }
    }
}

class Customer implements Runnable
{
    private Resource resource;
    public Customer(Resource resource)
    {
        this.resource = resource;
    }

    @Override
    public void run() {
        while(true)
        {
            resource.out();
        }
    }
}

public class ProducerAndCustomer {
    public static void main(String[] args) {
        Resource resource = new Resource();
        Producer producer = new Producer(resource);
        Customer customer = new Customer(resource);

        Thread t0 = new Thread(producer);
        Thread t1 = new Thread(producer);
        Thread t2 = new Thread(customer);
        Thread t3 = new Thread(customer);

        t0.start();
        t1.start();
        t2.start();
        t3.start();
    }
}
```
### await() signal()机制

如果说synchronized是隐式同步，那么Lock就是显示同步，它允许一个锁上带多个监视器。

java.util.concurrent 类库中提供了 Condition 类来实现线程之间的协调，可以在 Condition 上调用 await() 方法使线程等待，其它线程调用 signal() 或 signalAll() 方法唤醒等待的线程。

相比于 wait() 这种等待方式，await() 可以指定等待的条件，因此更加灵活。
```java
/**
 * 多生产者和消费者问题:lock
 */

class Resource
{
    private String name;
    private int count;
    private boolean flag = false;

    Lock lock = new ReentrantLock();
    Condition producerLock = lock.newCondition();
    Condition customerLock = lock.newCondition();

    public void set(String name)
    {
        lock.lock();
        try {
            while (flag)//flag为1，表示还没消费完
            {
                try {
                    producerLock.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            this.name = name + count;
            count++;
            System.out.println(Thread.currentThread().getName() + " 生产者...... " + this.name);
            flag = true;
            customerLock.signal();
        }
        finally {
            lock.unlock();
        }
    }

    public void out()
    {
        lock.lock();
        try {
            while(! flag)
            {
                try {
                    customerLock.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            //this.name = name + count;
            //count--;
            System.out.println(Thread.currentThread().getName()+" 消费者............... " +this.name);
            flag = false;
            producerLock.signal();
        }
        finally {
            lock.unlock();
        }
    }
}

class Producer implements Runnable
{
    private Resource resource;

    public Producer(Resource resource)
    {
        this.resource = resource;
    }

    @Override
    public void run() {
        while(true)
        {
            resource.set("烤鸭");
        }
    }
}

class Customer implements Runnable
{
    private Resource resource;
    public Customer(Resource resource)
    {
        this.resource = resource;
    }

    @Override
    public void run() {
        while(true)
        {
            resource.out();
        }
    }
}

public class ProducerAndCustomer {
    public static void main(String[] args) {
        Resource resource = new Resource();
        Producer producer = new Producer(resource);
        Customer customer = new Customer(resource);

        Thread t0 = new Thread(producer);
        Thread t1 = new Thread(producer);
        Thread t2 = new Thread(customer);
        Thread t3 = new Thread(customer);

        t0.start();
        t1.start();
        t2.start();
        t3.start();
    }
}
```
### 管道机制

管道流是JAVA中线程通讯的常用方式之一，基本流程如下：
  - 创建管道输出流PipedOutputStream pos和管道输入流PipedInputStream pis;
  - 将pos和pis匹配，pos.connect(pis);
  - 将pos赋给信息输入线程，pis赋给信息获取线程，就可以实现线程间的通讯了。
```java
/**
 * 通道方式实现线程同步
 */
class Producer implements Runnable
{
    private PipedOutputStream pos;

    public Producer(PipedOutputStream pos)
    {
        this.pos = pos;
    }

    @Override
    public void run() {
        int i = 0;
        while(true)
        {
            try {
                 Thread.sleep(100);
                 pos.write(i++);
            }
            catch (Exception e)
            {
                e.printStackTrace();
            }
        }
    }
}

class Customer implements Runnable
{
    private PipedInputStream pis;
    public Customer(PipedInputStream pis)
    {
        this.pis = pis;
    }

    @Override
    public void run() {
        while(true)
        {
            try {
                System.out.println("customer "+pis.read());
            } catch (IOException e) {
                e.printStackTrace();
            }
            ;
        }
    }
}

public class TestPipedConnection {
    public static void main(String[] args) {
        PipedOutputStream pos = new PipedOutputStream();;
        PipedInputStream pis = new PipedInputStream();

        try {
            pos.connect(pis);
        } catch (IOException e) {
            e.printStackTrace();
        }

        Producer producer = new Producer(pos);
        Customer customer = new Customer(pis);

        Thread t0 = new Thread(producer);
        Thread t1 = new Thread(customer);

        t0.start();
        t1.start();
    }
}
```

管道流虽然使用起来方便，但是也有一些缺点
  - 管道流只能在两个线程之间传递数据
线程consumer1和consumer2同时从pis中read数据，当线程producer往管道流中写入一段数据后，每一个时刻只有一个线程能获取到数据，并不是两个线程都能获取到producer发送来的数据，因此一个管道流只能用于两个线程间的通讯。不仅仅是管道流，其他IO方式都是一对一传输。
  - 管道流只能实现单向发送，如果要两个线程之间互通讯，则需要两个管道流
 可以看到上面的例子中，线程producer通过管道流向线程consumer发送数据，如果线程consumer想给线程producer发送数据，则需要新建另一个管道流pos1和pis1，将pos1赋给consumer1，将pis1赋给producer。

# 3. Java内存模型

Java 内存模型试图屏蔽各种硬件和操作系统的内存访问差异，以实现让 Java 程序在各种平台下都能达到一致的内存访问效果。

## 主内存与工作内存

处理器上的寄存器的读写的速度比内存快几个数量级，为了解决这种速度矛盾，在它们之间加入了高速缓存。

加入高速缓存带来了一个新的问题：缓存一致性。如果多个缓存共享同一块主内存区域，那么多个缓存的数据可能会不一致，需要一些协议来解决这个问题。
<center>
<img src="https://raw.githubusercontent.com/CyC2018/CS-Notes/master/pics/68778c1b-15ab-4826-99c0-3b4fd38cb9e9.png" width="600">
</center>


所有的变量都存储在主内存中，每个线程还有自己的工作内存，工作内存存储在高速缓存或者寄存器中，保存了该线程使用的变量的主内存副本拷贝。

线程只能直接操作工作内存中的变量，不同线程之间的变量值传递需要通过主内存来完成。
<center>
<img src="https://raw.githubusercontent.com/CyC2018/CS-Notes/master/pics/47358f87-bc4c-496f-9a90-8d696de94cee.png" width="600">
</center>

## 内存间交互操作

Java 内存模型定义了 8 个操作来完成主内存和工作内存的交互操作。
<center>
<img src="https://raw.githubusercontent.com/CyC2018/CS-Notes/master/pics/536c6dfd-305a-4b95-b12c-28ca5e8aa043.png">
</center>

  - read(读取)：把一个变量的值从主内存传输到工作内存中
  - load(载入)：在 read 之后执行，把 read 得到的值放入工作内存的变量副本中
  - use(使用)：把工作内存中一个变量的值传递给执行引擎
  - assign(赋值)：把一个从执行引擎接收到的值赋给工作内存的变量
  - store(存储)：把工作内存的一个变量的值传送到主内存中
  - write(写入)：在 store 之后执行，把 store 得到的值放入主内存的变量中
  - lock(锁定)：作用于主内存的变量，它把一个变量标识为一条线程独占的状态。
  - unlock(解锁)：它把一个处于锁定状态的变量释放出来，释放后的变量才可以被其他线程锁定。

## 内存模型三大特性
### 1. 原子性

Java 内存模型保证了 read、load、use、assign、store、write、lock 和 unlock 操作具有原子性，例如对一个 int 类型的变量执行 assign 赋值操作，这个操作就是原子性的。但是 Java 内存模型允许虚拟机将没有被 volatile 修饰的 64 位数据（long，double）的读写操作划分为两次 32 位的操作来进行，即 load、store、read 和 write 操作可以不具备原子性。

有一个错误认识就是，int 等原子性的类型在多线程环境中不会出现线程安全问题。

为了方便讨论，将内存间的交互操作简化为 3 个：load、assign、store。

下图演示了两个线程同时对 cnt 进行操作，load、assign、store 这一系列操作整体上看不具备原子性，那么在 T1 修改 cnt 并且还没有将修改后的值写入主内存，T2 依然可以读入旧值。可以看出，这两个线程虽然执行了两次自增运算，但是主内存中 cnt 的值最后为 1 而不是 2。因此对 int 类型读写操作满足原子性只是说明 load、assign、store 这些单个操作具备原子性。
<center>
<img src="https://raw.githubusercontent.com/CyC2018/CS-Notes/master/pics/ef8eab00-1d5e-4d99-a7c2-d6d68ea7fe92.png" width="400">
</center>


AtomicInteger 能保证多个线程修改的原子性。
<center>
<img src="https://raw.githubusercontent.com/CyC2018/CS-Notes/master/pics/952afa9a-458b-44ce-bba9-463e60162945.png" width="400">
</center>


使用 AtomicInteger 得到线程安全实现：
```java
public class AtomicExample {
    private AtomicInteger cnt = new AtomicInteger();

    public void add() {
        cnt.incrementAndGet();
    }

    public int get() {
        return cnt.get();
    }
}
```
```java
public static void main(String[] args) throws InterruptedException {
    final int threadSize = 1000;
    AtomicExample example = new AtomicExample(); // 只修改这条语句
    final CountDownLatch countDownLatch = new CountDownLatch(threadSize);
    ExecutorService executorService = Executors.newCachedThreadPool();
    for (int i = 0; i < threadSize; i++) {
        executorService.execute(() -> {
            example.add();
            countDownLatch.countDown();
        });
    }
    countDownLatch.await();
    executorService.shutdown();
    System.out.println(example.get());
}
```
```java
1000
```

除了使用原子类之外，也可以使用 synchronized 互斥锁来保证操作的原子性。它对应的内存间交互操作为：lock 和 unlock，在虚拟机实现上对应的字节码指令为 monitorenter 和 monitorexit。
```java
public class AtomicSynchronizedExample {
    private int cnt = 0;

    public synchronized void add() {
        cnt++;
    }

    public synchronized int get() {
        return cnt;
    }
}
```
```java
public static void main(String[] args) throws InterruptedException {
    final int threadSize = 1000;
    AtomicSynchronizedExample example = new AtomicSynchronizedExample();
    final CountDownLatch countDownLatch = new CountDownLatch(threadSize);
    ExecutorService executorService = Executors.newCachedThreadPool();
    for (int i = 0; i < threadSize; i++) {
        executorService.execute(() -> {
            example.add();
            countDownLatch.countDown();
        });
    }
    countDownLatch.await();
    executorService.shutdown();
    System.out.println(example.get());
}
```
```java
1000
```
所以，有两种方法可是先后实现原子性 ：

- Atomic关键字
- synchronized关键字

### 2. 可见性

可见性指当一个线程修改了共享变量的值，其它线程能够立即得知这个修改。Java 内存模型是通过在变量修改后将新值同步回主内存，在变量读取前从主内存刷新变量值来实现可见性的。

主要有有三种实现可见性的方式：
  - volatile
  - synchronized，对一个变量执行 unlock 操作之前，必须把变量值同步回主内存。
  - final，被 final 关键字修饰的字段在构造器中一旦初始化完成，并且没有发生 this 逃逸（其它线程通过 this 引用访问到初始化了一半的对象），那么其它线程就能看见 final 字段的值。


对线程不安全示例中的 cnt 变量使用 volatile 修饰，有时不能解决线程不安全问题，因为 volatile 并不能保证操作的原子性。关于volatile关键字的理解详见另一篇笔记——《Java并发之volatile》。

### 3. 有序性

有序性是指：在本线程内观察，所有操作都是有序的。在一个线程观察另一个线程，所有操作都是无序的，无序是因为发生了指令重排序。在 Java 内存模型中，允许编译器和处理器对指令进行重排序，重排序过程不会影响到单线程程序的执行，却会影响到多线程并发执行的正确性。

可以通过下面两种方式实现有序性：
  - volatile 关键字通过添加内存屏障的方式来禁止指令重排，即重排序时不能把后面的指令放到内存屏障之前。
  - 以通过 synchronized 来保证有序性，它保证每个时刻只有一个线程执行同步代码，相当于是让线程顺序执行同步代码。

## 先行发生(happens-before)原则

上面提到了可以用 volatile 和 synchronized 来保证有序性。除此之外，JVM 还规定了先行发生原则，让一个操作无需控制就能先于另一个操作完成。下面是Java内存模型下一些“天然”的先行发生关系。

### 1. 单一线程原则(Single Thread rule)

在一个线程内，在程序前面的操作先行发生于后面的操作。
<center>
<img src="https://raw.githubusercontent.com/CyC2018/CS-Notes/master/pics/single-thread-rule.png" >
</center>

### 2. 管程锁定规则(Monitor Lock Rule)

一个 unlock 操作先行发生于后面对同一个锁的 lock 操作。
<center>
<img src="https://raw.githubusercontent.com/CyC2018/CS-Notes/master/pics/monitor-lock-rule.png" width="500">
</center>

### 3. volatile 变量规则(Volatile Variable Rule)

对一个 volatile 变量的写操作先行发生于后面对这个变量的读操作。
<center>
<img src="https://raw.githubusercontent.com/CyC2018/CS-Notes/master/pics/volatile-variable-rule.png" width="500">
</center>

### 4. 线程启动规则(Thread Start Rule)

Thread 对象的 start() 方法调用先行发生于此线程的每一个动作。
<center>
<img src="https://raw.githubusercontent.com/CyC2018/CS-Notes/master/pics/thread-start-rule.png" width="500">
</center>

### 5. 线程加入规则(Thread Join Rule)

Thread 对象的结束先行发生于 join() 方法返回。
<center>
<img src="https://raw.githubusercontent.com/CyC2018/CS-Notes/master/pics/thread-join-rule.png" width="500">
</center>

### 6. 线程中断规则(Thread Interruption Rule)

对线程 interrupt() 方法的调用先行发生于被中断线程的代码检测到中断事件的发生，可以通过 interrupted() 方法检测到是否有中断发生。

### 7. 对象终结规则(Finalizer Rule)

一个对象的初始化完成（构造函数执行结束）先行发生于它的 finalize() 方法的开始。

### 8. 传递性(Transitivity)

如果操作 A 先行发生于操作 B，操作 B 先行发生于操作 C，那么操作 A 先行发生于操作 C。

# 4. 线程安全

线程安全是指多个线程不管以何种方式访问某个类，并且在主调代码中不需要进行同步，都能表现正确的行为。

## 线程安全分类

按照线程安全的“安全程度”由强至若来排序，可以将Java语言中各种操作共享数据分为以下５类：不可变、绝对线程安全、相对线程安全、线程兼容和线程对立。
### 不可变

不可变（Immutable）的对象一定是线程安全的，不需要再采取任何的线程安全保障措施。只要一个不可变的对象被正确地构建出来，永远也不会看到它在多个线程之中处于不一致的状态。多线程环境下，应当尽量使对象成为不可变，来满足线程安全。

不可变的类型：
  - final 关键字修饰的基本数据类型
  - String
  - 枚举类型
  - Number 部分子类，如 Long 和 Double 等数值包装类型，BigInteger 和 BigDecimal 等大数据类型。但同为 Number 的原子类 AtomicInteger 和 AtomicLong 则是可变的。


对于集合类型，可以使用 Collections.unmodifiableXXX() 方法来获取一个不可变的集合。
```java
public class ImmutableExample {
    public static void main(String[] args) {
        Map<String, Integer> map = new HashMap<>();
        Map<String, Integer> unmodifiableMap = Collections.unmodifiableMap(map);
        unmodifiableMap.put("a", 1);
    }
}
```
```java
Exception in thread "main" java.lang.UnsupportedOperationException
    at java.util.Collections$UnmodifiableMap.put(Collections.java:1457)
    at ImmutableExample.main(ImmutableExample.java:9)
```

Collections.unmodifiableXXX() 先对原始的集合进行拷贝，需要对集合进行修改的方法都直接抛出异常。
```java
public V put(K key, V value) {
    throw new UnsupportedOperationException();
}
```

### 绝对线程安全 
### 相对线程安全
### 线程兼容
### 线程对立

## 线程安全实现方式
### 互斥同步

同步是指在多个线程并发访问共享数据时，保证共享数据在同一时刻只被一个(或者是一些，使用信号量的时候)线程使用。而互斥是实现同步的一种手段，临界区(Critical Section)、互斥量(Mutex)和信号量(Semaphore)都是主要的互斥实现方式。因此，互斥是因，同步是果；互斥是方法，同步是目的。

Java主要使用synchronized 和 ReentrantLock实现互斥同步。

相比于synchronized，ReentrantLock增加了一些高级功能，主要有三个：等待可中断、可实现公平锁、可以绑定多个条件。

  - 等待可中断是指当吃持有锁的线程长期不释放锁的时候，正在等待的线程可以选择放弃等待，改为处理其他事情，可中断特性对处理执行时间非常长的同步快很有帮助。
  - 公平锁是指在多个线程等待同一个锁时，必须按照申请锁的时间顺序来依次获得锁；而非公平锁则不保证这一点，在锁被释放时，任何一个等待锁的线程都有机会获得锁。synchronized中的锁是非公平的，ReentrantLock默认情况下也是非公平的，但是可以通过带布尔值的构造函数要求使用公平锁。
  - 绑定多个条件是指一个ReentrantLock对象可以同时绑定多个Condition对象。

### 非阻塞同步

互斥同步最主要的问题就是线程阻塞和唤醒所带来的性能问题，因此这种同步也称为阻塞同步。

互斥同步属于一种悲观的并发策略，总是认为只要不去做正确的同步措施，那就肯定会出现问题。无论共享数据是否真的会出现竞争，它都要进行加锁（这里讨论的是概念模型，实际上虚拟机会优化掉很大一部分不必要的加锁）、用户态核心态转换、维护锁计数器和检查是否有被阻塞的线程需要唤醒等操作。

#### 1. CAS

随着硬件指令集的发展，我们可以使用基于冲突检测的乐观并发策略：先进行操作，如果没有其它线程争用共享数据，那操作就成功了，否则采取补偿措施（不断地重试，直到成功为止）。这种乐观的并发策略的许多实现都不需要将线程阻塞，因此这种同步操作称为非阻塞同步。

乐观锁需要操作和冲突检测这两个步骤具备原子性，这里就不能再使用互斥同步来保证了，只能靠硬件来完成。硬件支持的原子性操作最典型的是：比较并交换（Compare-and-Swap，CAS）。CAS 指令需要有 3 个操作数，分别是内存地址 V、旧的预期值 A 和新值 B。当执行操作时，只有当 V 的值等于 A，才将 V 的值更新为 B。

#### 2. AtomicInteger

J.U.C 包里面的整数原子类 AtomicInteger 的方法调用了 Unsafe 类的 CAS 操作。

以下代码使用了 AtomicInteger 执行了自增的操作。
```java
private AtomicInteger cnt = new AtomicInteger();

public void add() {
    cnt.incrementAndGet();
}
```

以下代码是 incrementAndGet() 的源码，它调用了 Unsafe 的 getAndAddInt() 。
```java
public final int incrementAndGet() {
    return unsafe.getAndAddInt(this, valueOffset, 1) + 1;
}
```

以下代码是 getAndAddInt() 源码，var1 指示对象内存地址，var2 指示该字段相对对象内存地址的偏移，var4 指示操作需要加的数值，这里为 1。通过 getIntVolatile(var1, var2) 得到旧的预期值，通过调用 compareAndSwapInt() 来进行 CAS 比较，如果该字段内存地址中的值等于 var5，那么就更新内存地址为 var1+var2 的变量为 var5+var4。

可以看到 getAndAddInt() 在一个循环中进行，发生冲突的做法是不断的进行重试。
```java
public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do {
        var5 = this.getIntVolatile(var1, var2);
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

    return var5;
}
```
##### 3. ABA

如果一个变量初次读取的时候是 A 值，它的值被改成了 B，后来又被改回为 A，那 CAS 操作就会误认为它从来没有被改变过。

J.U.C 包提供了一个带有标记的原子引用类 AtomicStampedReference 来解决这个问题，它可以通过控制变量值的版本来保证 CAS 的正确性。大部分情况下 ABA 问题不会影响程序并发的正确性，如果需要解决 ABA 问题，改用传统的互斥同步可能会比原子类更高效。

### 无同步方案

要保证线程安全，并不是一定就要进行同步。如果一个方法本来就不涉及共享数据，那它自然就无须任何同步措施去保证正确性。

#### 栈封闭

多个线程访问同一个方法的局部变量时，不会出现线程安全问题，因为局部变量存储在虚拟机栈中，属于线程私有的。
```java
public class StackClosedExample {
    public void add100() {
        int cnt = 0;
        for (int i = 0; i < 100; i++) {
            cnt++;
        }
        System.out.println(cnt);
    }
}
```
```java
public static void main(String[] args) {
    StackClosedExample example = new StackClosedExample();
    ExecutorService executorService = Executors.newCachedThreadPool();
    executorService.execute(() -> example.add100());
    executorService.execute(() -> example.add100());
    executorService.shutdown();
}
```
```java
100
100
```

#### 线程本地存储（Thread Local Storage）

如果一段代码中所需要的数据必须与其他代码共享，那就看看这些共享数据的代码是否能保证在同一个线程中执行。如果能保证，我们就可以把共享数据的可见范围限制在同一个线程之内，这样，无须同步也能保证线程之间不出现数据争用的问题。

符合这种特点的应用并不少见，大部分使用消费队列的架构模式（如“生产者-消费者”模式）都会将产品的消费过程尽量在一个线程中消费完。其中最重要的一个应用实例就是经典 Web 交互模型中的“一个请求对应一个服务器线程”（Thread-per-Request）的处理方式，这种处理方式的广泛应用使得很多 Web 服务端应用都可以使用线程本地存储来解决线程安全问题。

可以使用 java.lang.ThreadLocal 类来实现线程本地存储功能。**ThreadLocal为变量在每个线程中都创建了一个副本，那么每个线程可以访问自己内部的副本变量。**

对于以下代码，thread1 中设置 threadLocal 为 1，而 thread2 设置 threadLocal 为 2。过了一段时间之后，thread1 读取 threadLocal 依然是 1，不受 thread2 的影响。
```java
public class ThreadLocalExample {
    public static void main(String[] args) {
        ThreadLocal threadLocal = new ThreadLocal();
        Thread thread1 = new Thread(() -> {
            threadLocal.set(1);
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(threadLocal.get());
            threadLocal.remove();
        });
        Thread thread2 = new Thread(() -> {
            threadLocal.set(2);
            threadLocal.remove();
        });
        thread1.start();
        thread2.start();
    }
}
```
```java
1
```

为了理解 ThreadLocal，先看以下代码：
```java
public class ThreadLocalExample1 {
    public static void main(String[] args) {
        ThreadLocal threadLocal1 = new ThreadLocal();
        ThreadLocal threadLocal2 = new ThreadLocal();
        Thread thread1 = new Thread(() -> {
            threadLocal1.set(1);
            threadLocal2.set(1);
        });
        Thread thread2 = new Thread(() -> {
            threadLocal1.set(2);
            threadLocal2.set(2);
        });
        thread1.start();
        thread2.start();
    }
}
```

它所对应的底层结构图为：
<center>
<img src="https://raw.githubusercontent.com/CyC2018/CS-Notes/master/pics/3646544a-cb57-451d-9e03-d3c4f5e4434a.png">
</center>


每个 Thread 都有一个 ThreadLocal.ThreadLocalMap 对象。
```java
/* ThreadLocal values pertaining to this thread. This map is maintained
 * by the ThreadLocal class. */
ThreadLocal.ThreadLocalMap threadLocals = null;
```

当调用一个 ThreadLocal 的 set(T value) 方法时，先得到当前线程的 ThreadLocalMap 对象，然后将 ThreadLocal->value 键值对插入到该 Map 中。
```java
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}
```


get() 方法类似。
```java
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}
```

ThreadLocal 从理论上讲并不是用来解决多线程并发问题的，因为根本不存在多线程竞争。

在一些场景 (尤其是使用线程池) 下，由于 ThreadLocal.ThreadLocalMap 的底层数据结构导致 ThreadLocal 有内存泄漏的情况，应该尽可能在每次使用 ThreadLocal 后手动调用 remove()，以避免出现 ThreadLocal 经典的内存泄漏甚至是造成自身业务混乱的风险。

最常见的ThreadLocal使用场景为 用来解决数据库连接、Session管理等。如：

数据库连接：
```java
private static ThreadLocal<Connection> connectionHolder = new ThreadLocal<Connection>() {  
    public Connection initialValue() {  
        return DriverManager.getConnection(DB_URL);  
    }  
};  
  
public static Connection getConnection() {  
    return connectionHolder.get();  
}  
```

Session管理：
```java
private static final ThreadLocal threadSession = new ThreadLocal();  
  
public static Session getSession() throws InfrastructureException {  
    Session s = (Session) threadSession.get();  
    try {  
        if (s == null) {  
            s = getSessionFactory().openSession();  
            threadSession.set(s);  
        }  
    } catch (HibernateException ex) {  
        throw new InfrastructureException(ex);  
    }  
    return s;  
}  
```

#### 可重入代码（Reentrant Code）

这种代码也叫做纯代码（Pure Code），可以在代码执行的任何时刻中断它，转而去执行另外一段代码（包括递归调用它本身），而在控制权返回后，原来的程序不会出现任何错误。

可重入代码有一些共同的特征，例如不依赖存储在堆上的数据和公用的系统资源、用到的状态量都由参数中传入、不调用非可重入的方法等。

# 5. 锁优化

这里的锁优化主要是指 JVM 对 synchronized 的优化。

## 自旋锁

互斥同步进入阻塞状态的开销都很大，应该尽量避免。在许多应用中，共享数据的锁定状态只会持续很短的一段时间。自旋锁的思想是让一个线程在请求一个共享数据的锁时执行忙循环（自旋）一段时间，如果在这段时间内能获得锁，就可以避免进入阻塞状态。

自旋锁虽然能避免进入阻塞状态从而减少开销，但是它需要进行忙循环操作占用 CPU 时间，它只适用于共享数据的锁定状态很短的场景。

在 JDK 1.6 中引入了自适应的自旋锁。自适应意味着自旋的次数不再固定了，而是由前一次在同一个锁上的自旋次数及锁的拥有者的状态来决定。如果在同一个锁对象上，自旋等待刚刚成功获得锁，并且持有锁的线程正在运行中，那么虚拟机就会认为这次自旋也很有可能再次成功，今儿它讲允许自旋等待持续相对较长的时间；相反，如果对于某个锁，自选很少成功获得过，那在以后要获取这个锁时将可能省略掉自旋过程。

## 锁消除

锁消除是指对于被检测出不可能存在竞争的共享数据的锁进行消除。

锁消除主要是通过逃逸分析来支持，如果堆上的共享数据不可能逃逸出去被其它线程访问到，那么就可以把它们当成私有数据对待，也就可以将它们的锁进行消除。

对于一些看起来没有加锁的代码，其实隐式的加了很多锁。例如下面的字符串拼接代码就隐式加了锁：
```java
public static String concatString(String s1, String s2, String s3) {
    return s1 + s2 + s3;
}
```

String 是一个不可变的类，编译器会对 String 的拼接自动优化。在 JDK 1.5 之前，会转化为 StringBuffer 对象的连续 append() 操作(JDK1.5之后会转化为StringBuilder对象的连续append()操作)：
```java
public static String concatString(String s1, String s2, String s3) {
    StringBuffer sb = new StringBuffer();
    sb.append(s1);
    sb.append(s2);
    sb.append(s3);
    return sb.toString();
}
```

每个 append() 方法中都有一个同步块。虚拟机观察变量 sb，很快就会发现它的动态作用域被限制在 concatString() 方法内部。也就是说，sb 的所有引用永远不会逃逸到 concatString() 方法之外，其他线程无法访问到它，因此可以进行消除。

## 锁粗化

如果一系列的连续操作都对同一个对象反复加锁和解锁，频繁的加锁操作就会导致性能损耗。

上一节的示例代码中连续的 append() 方法就属于这类情况。如果虚拟机探测到由这样的一串零碎的操作都对同一个对象加锁，将会把加锁的范围扩展（粗化）到整个操作序列的外部。对于上一节的示例代码就是扩展到第一个 append() 操作之前直至最后一个 append() 操作之后，这样只需要加锁一次就可以了。

## 轻量级锁

JDK 1.6 引入了偏向锁和轻量级锁，从而让锁拥有了四个状态：无锁状态（unlocked）、偏向锁状态（biasble）、轻量级锁状态（lightweight locked）和重量级锁状态（inflated）。

以下是 HotSpot 虚拟机对象头的内存布局，这些数据被称为 Mark Word。其中 tag bits 对应了五个状态，这些状态在右侧的 state 表格中给出。除了 marked for gc 状态，其它四个状态已经在前面介绍过了。
<center>
<img src="https://raw.githubusercontent.com/CyC2018/CS-Notes/master/pics/bb6a49be-00f2-4f27-a0ce-4ed764bc605c.png" width="500">
</center>


下图左侧是一个线程的虚拟机栈，其中有一部分称为 Lock Record 的区域，这是在轻量级锁运行过程创建的，用于存放锁对象的 Mark Word。而右侧就是一个锁对象，包含了 Mark Word 和其它信息。
<center>
<img src="https://raw.githubusercontent.com/CyC2018/CS-Notes/master/pics/051e436c-0e46-4c59-8f67-52d89d656182.png" width="500">
</center>


轻量级锁是相对于传统的重量级锁而言，它使用 CAS 操作来避免重量级锁使用互斥量的开销。对于绝大部分的锁，在整个同步周期内都是不存在竞争的，因此也就不需要都使用互斥量进行同步，可以先采用 CAS 操作进行同步，如果 CAS 失败了再改用互斥量进行同步。

当尝试获取一个锁对象时，如果锁对象标记为 0 01，说明锁对象的锁未锁定（unlocked）状态。此时虚拟机在当前线程的虚拟机栈中创建 Lock Record，然后使用 CAS 操作将对象的 Mark Word 更新为 Lock Record 指针。如果 CAS 操作成功了，那么线程就获取了该对象上的锁，并且对象的 Mark Word 的锁标记变为 00，表示该对象处于轻量级锁状态。
<center>
<img src="https://raw.githubusercontent.com/CyC2018/CS-Notes/master/pics/baaa681f-7c52-4198-a5ae-303b9386cf47.png" width="500">
</center>

## 偏向锁

偏向锁的思想是偏向于让第一个获取锁对象的线程，这个线程在之后获取该锁就不再需要进行同步操作，甚至连 CAS 操作也不再需要。

当锁对象第一次被线程获得的时候，进入偏向状态，标记为 1 01。同时使用 CAS 操作将线程 ID 记录到 Mark Word 中，如果 CAS 操作成功，这个线程以后每次进入这个锁相关的同步块就不需要再进行任何同步操作。

当有另外一个线程去尝试获取这个锁对象时，偏向状态就宣告结束，此时撤销偏向（Revoke Bias）后恢复到未锁定状态或者轻量级锁状态。
<center>
<img src="https://raw.githubusercontent.com/CyC2018/CS-Notes/master/pics/390c913b-5f31-444f-bbdb-2b88b688e7ce.jpg" width="500">
</center>

# 6. 多线程开发良好的实践
  - 给线程起个有意义的名字，这样可以方便找 Bug。
  - 缩小同步范围，从而减少锁争用。例如对于 synchronized，应该尽量使用同步块而不是同步方法。
  - 多用同步工具少用 wait() 和 notify()。首先，CountDownLatch, CyclicBarrier, Semaphore 和 Exchanger 这些同步类简化了编码操作，而用 wait() 和 notify() 很难实现复杂控制流；其次，这些同步类是由最好的企业编写和维护，在后续的 JDK 中还会不断优化和完善。
  - 使用 BlockingQueue 实现生产者消费者问题。
  - 多用并发集合少用同步集合，例如应该使用 ConcurrentHashMap 而不是 Hashtable。
  - 使用本地变量和不可变类来保证线程安全。
  - 使用线程池而不是直接创建线程，这是因为创建线程代价很高，线程池可以有效地利用有限的线程来启动任务。

 
 
 
 
