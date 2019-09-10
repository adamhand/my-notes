# Java并发--阻塞队列
---

# 什么是阻塞队列？
阻塞队列（BlockingQueue）是一个支持两个附加操作的队列。这两个附加的操作是：**在队列为空时，获取元素的线程会等待队列变为非空。当队列满时，存储元素的线程会等待队列可用。阻塞队列常用于生产者和消费者的场景，**

生产者是往队列里添加元素的线程，消费者是从队列里拿元素的线程。阻塞队列就是生产者存放元素的容器，而消费者也只从容器里拿元素。

阻塞队列提供了四种处理方法:

|方法\处理方式	|抛出异常|	返回特殊值	|一直阻塞|	超时退出|
|-|-|-|-|-|
|插入方法|add(e)|	offer(e)|	put(e)	|offer(e,time,unit)|
|移除方法|	remove()|	poll()|	take()|	poll(time,unit)|
|检查方法	|element()	|peek()|	不可用|	不可用|


- 抛出异常：是指当阻塞队列满时候，再往队列里插入元素，会抛出`IllegalStateException(“Queue full”)`异常。当队列为空时，从队列里获取元素时会抛出`NoSuchElementException`异常 。
- 返回特殊值：插入方法会返回是否成功，成功则返回`true`。移除方法，则是从队列里拿出一个元素，如果没有则返回`null`。
- 一直阻塞：当阻塞队列满时，如果生产者线程往队列里`put`元素，队列会一直阻塞生产者线程，直到拿到数据，或者响应中断退出。当队列空时，消费者线程试图从队列里`take`元素，队列也会阻塞消费者线程，直到队列可用。
- 超时退出：当阻塞队列满时，队列会阻塞生产者线程一段时间，如果超过一定的时间，生产者线程就会退出。


# Java中的阻塞队列
JDK7提供了7个阻塞队列。分别是:

|队列|有界性|锁|数据结构|
|-|-|-|-|
|ArrayBlockingQueue |有界|加锁|数组|
|LinkedBlockingQueue|有界|加锁|单链表|
|PriorityBlockingQueue|无界|加锁|堆|
|DelayQueue|无界|加锁|堆|
|SynchronousQueue|有界|无锁(CAS)|-|
|LinkedTransferQueue|无界|无锁(CAS)|单链表|
|LinkedBlockingDeque|无界|加锁|双链表|


- **ArrayBlockingQueue**：是一个用数组实现的有界阻塞队列，此队列按照先进先出（FIFO）的原则对元素进行排序。支持公平锁和非公平锁。【注：每一个线程在获取锁的时候可能都会排队等待，如果在等待时间上，先获取锁的线程的请求一定先被满足，那么这个锁就是公平的。反之，这个锁就是不公平的。公平的获取锁，也就是当前等待时间最长的线程先获取锁】访问者的公平性是使用可重入锁实现的，`ReentrantLock`默认实现非公平锁，但是可以使用一个带参的构造函数实现公平锁。
- **LinkedBlockingQueue**：一个由**单链表**结构组成的有界队列，此队列的长度为Integer.MAX_VALUE。此队列按照先进先出的顺序进行排序。
- **PriorityBlockingQueue**：一个支持线程优先级排序的无界队列，默认自然序进行排序，也可以自定义实现comparator的compare()方法来指定元素排序规则，不能保证同优先级元素的顺序。
- **DelayQueue**:DelayQueue是一个支持延时获取元素的无界阻塞队列。队列使用PriorityQueue来实现。队列中的元素必须实现Delayed接口，在创建元素时可以指定多久才能从队列中获取当前元素。只有在延迟期满时才能从队列中提取元素。我们可以将DelayQueue运用在以下应用场景：
    - 缓存系统的设计：可以用DelayQueue保存缓存元素的有效期，使用一个线程循环查询DelayQueue，一旦能从DelayQueue中获取元素时，表示缓存有效期到了。
    - 定时任务调度。使用DelayQueue保存当天将会执行的任务和执行时间，一旦从DelayQueue中获取到任务就开始执行，从比如TimerQueue就是使用DelayQueue实现的。
队列中的元素必须实现Delayed接口，实现CompareTo方法。比如让延时时间最长的放在队列的末尾。实现代码如下：
```java
public int compareTo(Delayed other) {
   if (other == this) // compare zero ONLY if same object
        return 0;
    if (other instanceof ScheduledFutureTask) {
        ScheduledFutureTask x = (ScheduledFutureTask)other;
        long diff = time - x.time;
        if (diff < 0)
            return -1;
        else if (diff > 0)
            return 1;
    else if (sequenceNumber < x.sequenceNumber)
            return -1;
        else
            return 1;
    }
    long d = (getDelay(TimeUnit.NANOSECONDS) -
              other.getDelay(TimeUnit.NANOSECONDS));
    return (d == 0) ? 0 : ((d < 0) ? -1 : 1);
}
```
- **SynchronousQueue**：SynchronousQueue是一个不存储元素的阻塞队列。每一个put操作必须等待一个take操作，否则不能继续添加元素。SynchronousQueue可以看成是一个传球手，负责把生产者线程处理的数据直接传递给消费者线程。队列本身并不存储任何元素，非常适合于传递性场景,比如在一个线程中使用的数据，传递给另外一个线程使用，SynchronousQueue的吞吐量高于LinkedBlockingQueue 和 ArrayBlockingQueue。
SynchronousQueue 并没有使用锁来保证线程的安全，使用的是循环CAS方法。 
SynchronousQueue有两种模式： 
    - 公平模式 
所谓公平就是遵循先来先服务的原则，因此其内部使用了一个FIFO队列 来实现其功能。 
    - 非公平模式 
SynchronousQueue 中的非公平模式是默认的模式，其内部使用栈来实现其功能，也就是 后来的先服务。
- **LinkedTransferQueue**:LinkedTransferQueue 和SynchronousQueue 其实基本是差不多的，两者都是无锁带阻塞功能的队列，SynchronousQueue 通过内部类Transferer 来实现公平和非公平队列。
在LinkedTransferQueue 中没有公平与非公平的区分，LinkedTransferQueue 实现了TransferQueue接口，该接口定义的是带阻塞操作的操作，相比SynchronousQueue 中的Transferer 功能更丰富。 

LinkedTransferQueue是 SynchronousQueue 和 LinkedBlockingQueue 的合体，性能比 LinkedBlockingQueue 更高（没有锁操作），比 SynchronousQueue能存储更多的元素。

LinkedTransferQueue是基于链表的FIFO无界阻塞队列，它是JDK1.7才添加的阻塞队列，有4种操作模式：
```java
private static final int NOW   = 0; // for untimed poll, tryTransfer
private static final int ASYNC = 1; // for offer, put, add
private static final int SYNC  = 2; // for transfer, take
private static final int TIMED = 3; // for timed poll, tryTransfer
```
① NOW ：在取数据的时候，如果没有数据，则直接返回，无需阻塞等待。</br>
② ASYNC：入队的操作都不会阻塞，也就是说，入队后线程会立即返回，不需要等到消费者线程来取数据。</br>
③ SYNC ：取数据的时候，如果没有数据，则会进行阻塞等待。</br>
④ TIMED : 取数据的时候，如果没有数据，则会进行超时阻塞等待。

- **LinkedBlockingDeque**：LinkedBlockingDeque是基于双向链表的双端有界阻塞队列，默认使用非公平ReentrantLock实现线程安全，默认队列最大长度都为Integer.MAX_VALUE；不允许null元素添加；双端队列可以用来实现 **“窃取算法”**（关于窃取算法，参考另一篇笔记——《Java并发之J.U.C》） ,两头都可以操作队列，相对于单端队列可以减少一半的竞争。 

# 阻塞队列的实现原理
由上面的分析可以看到，阻塞队列有两种：**加锁**和**不加锁**。

**加锁的队列**是使用ReentrantLock的Condition的await()和signal()方法来实现生产者和消费者之间通信的，当生产者往满的队列里添加元素时会阻塞住生产者，当消费者消费了一个队列中的元素后，会通知生产者当前队列可用。而await()方法调用的是LockSupport.park()方法，这个park方法调用的又是调用的unsafe.park()方法实现队列的阻塞的。

**不加锁的队列**使用的是**CAS算法** + **LockSupport.park()/unpark()** 方法来实现的。

下面以ArrayBlockingQueue为例看一下。通过查看JDK源码发现ArrayBlockingQueue使用了Condition来实现，代码如下：
```java
private final Condition notFull;
private final Condition notEmpty;

public ArrayBlockingQueue(int capacity, boolean fair) {
    //省略其他代码
    notEmpty = lock.newCondition();
    notFull =  lock.newCondition();
}

public void put(E e) throws InterruptedException {
    checkNotNull(e);
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        while (count == items.length)
            notFull.await();
        insert(e);
    } finally {
        lock.unlock();
    }
}

public E take() throws InterruptedException {
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        while (count == 0)
            notEmpty.await();
        return extract();
  } finally {
        lock.unlock();
    }
}

private void insert(E x) {
    items[putIndex] = x;
    putIndex = inc(putIndex);
    ++count;
    notEmpty.signal();
}
```
当往队列里插入一个元素时，如果队列不可用，阻塞生产者主要通过LockSupport.park(this);来实现：
```java
public final void await() throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    Node node = addConditionWaiter();
    int savedState = fullyRelease(node);
    int interruptMode = 0;
    while (!isOnSyncQueue(node)) {
        LockSupport.park(this);
        if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
            break;
    }
    if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
        interruptMode = REINTERRUPT;
    if (node.nextWaiter != null) // clean up if cancelled
        unlinkCancelledWaiters();
    if (interruptMode != 0)
        reportInterruptAfterWait(interruptMode);
}
```
继续进入源码，发现调用setBlocker先保存下将要阻塞的线程，然后调用unsafe.park阻塞当前线程。
```java
public static void park(Object blocker) {
    Thread t = Thread.currentThread();
    setBlocker(t, blocker);
    unsafe.park(false, 0L);
    setBlocker(t, null);
}
```
unsafe.park是个native方法，这个方法会阻塞当前线程，只有以下四种情况中的一种发生时，该方法才会返回。

- 与park对应的unpark执行或已经执行时。注意：已经执行是指unpark先执行，然后再执行的park。
- 线程被中断时。
- 如果参数中的time不是零，等待了指定的毫秒数时。
- 发生异常现象时。这些异常事先无法确定。

继续看一下JVM是如何实现park方法的，park在不同的操作系统使用不同的方式实现，在linux下是使用的是系统方法pthread_cond_wait实现。实现代码在JVM源码路径src/os/linux/vm/os_linux.cpp里的 os::PlatformEvent::park方法，代码如下：
```java
void os::PlatformEvent::park() {
 	 int v ;
     for (;;) {
	v = _Event ;
     if (Atomic::cmpxchg (v-1, &_Event, v) == v) break ;
     }
     guarantee (v >= 0, "invariant") ;
     if (v == 0) {
     // Do this the hard way by blocking ...
     int status = pthread_mutex_lock(_mutex);
     assert_status(status == 0, status, "mutex_lock");
     guarantee (_nParked == 0, "invariant") ;
     ++ _nParked ;
     while (_Event < 0) {
     status = pthread_cond_wait(_cond, _mutex);
     // for some reason, under 2.7 lwp_cond_wait() may return ETIME ...
     // Treat this the same as if the wait was interrupted
     if (status == ETIME) { status = EINTR; }
     assert_status(status == 0 || status == EINTR, status, "cond_wait");
     }
     -- _nParked ;

     // In theory we could move the ST of 0 into _Event past the unlock(),
     // but then we'd need a MEMBAR after the ST.
     _Event = 0 ;
     status = pthread_mutex_unlock(_mutex);
     assert_status(status == 0, status, "mutex_unlock");
     }
     guarantee (_Event >= 0, "invariant") ;
     }

 }
```
pthread_cond_wait是一个多线程的条件变量函数，cond是condition的缩写，字面意思可以理解为线程在等待一个条件发生，这个条件是一个全局变量。这个方法接收两个参数，一个共享变量_cond，一个互斥量_mutex。而unpark方法在linux下是使用pthread_cond_signal实现的。park 在windows下则是使用WaitForSingleObject实现的。

# 补充：各种阻塞队列使用举例
## 使用DelayQueue实现本地的延迟队列
在我们的业务中通常会有一些需求是这样的： 

- 淘宝订单业务:**下单之后如果三十分钟之内没有付款就自动取消订单**。 
- 饿了吗订餐通知:**下单成功后60s之后给用户发送短信通知**。

那么这类业务我们可以总结出一个特点:需要延迟工作。由此的情况，就是我们的DelayQueue应用需求的产生。

比如，在网咖或者网吧上网时会用到一个网吧综合系统，其中有一个主要功能就是给每一位网民计时，用户充值一定金额会有相应的上网时常，这里我们用DelayQueue模拟实现一下：用DelayQueue存储网民（Wangmin类），每一个考生都有自己的名字和完成试卷的时间，Wangba线程对DelayQueue进行监控，从队列中取出到时间的网民执行下机操作。

### 实现了Delayed接口的网民类，并实现CompareTo()方法
```java
public class Wangmin implements Delayed {  

    private String name;  
    //身份证  
    private String id;  
    //截止时间  
    private long endTime;  
    //定义时间工具类
    private TimeUnit timeUnit = TimeUnit.SECONDS;
      
    public Wangmin(String name,String id,long endTime){  
        this.name=name;  
        this.id=id;  
        this.endTime = endTime;  
    }  
      
    public String getName(){  
        return this.name;  
    }  
      
    public String getId(){  
        return this.id;  
    }  
      
    /** 
     * 用来判断是否到了截止时间 
     */  
    @Override  
    public long getDelay(TimeUnit unit) { 
        //return unit.convert(endTime, TimeUnit.MILLISECONDS) - unit.convert(System.currentTimeMillis(), TimeUnit.MILLISECONDS);
        return endTime - System.currentTimeMillis();
    }  
  
    /** 
     * 相互批较排序用 
     */  
    @Override  
    public int compareTo(Delayed delayed) {  
        Wangmin w = (Wangmin)delayed;  
        return this.getDelay(this.timeUnit) - w.getDelay(this.timeUnit) > 0 ? 1:0;  
    }    
}
```

### 实现网吧类
```java
public class WangBa implements Runnable {  

    private DelayQueue<Wangmin> queue = new DelayQueue<Wangmin>();  
    
    public boolean yingye =true;  
    
    /**
     * 上机 
     */
    public void shangji(String name,String id,int money){  
        Wangmin man = new Wangmin(name, id, 1000 * money + System.currentTimeMillis());  
        System.out.println("网名"+man.getName()+" 身份证"+man.getId()+"交钱"+money+"块,开始上机...");  
        this.queue.add(man);  
    }  
    
    // 下机
    public void xiaji(Wangmin man){  
        System.out.println("网名"+man.getName()+" 身份证"+man.getId()+"时间到下机...");  
    }  
  
    @Override  
    public void run() {  
        while(yingye){  
            try {  
                Wangmin man = queue.take();  
                xiaji(man);  
            } catch (InterruptedException e) {  
                e.printStackTrace();  
            }  
        }  
    }  
      
    public static void main(String args[]){  
        try{  
            System.out.println("网吧开始营业");  
            WangBa siyu = new WangBa();  
            Thread shangwang = new Thread(siyu);  
            shangwang.start();  
              
            siyu.shangji("路人甲", "123", 1);  
            siyu.shangji("路人乙", "234", 10);  
            siyu.shangji("路人丙", "345", 5);  
        }  
        catch(Exception e){  
            e.printStackTrace();
        }  
  
    }  
}
```

## 使用ArrayBlockingQueue和LinkedBlockingQueue实现生产者消费者
使用ArrayBlockingQueue实现生产者消费者的代码如下：

```java
class Cookie {

}

class Productor implements Runnable {
    private ArrayBlockingQueue abq;

    Productor(ArrayBlockingQueue abq) {
        this.abq = abq;
    }
    @Override
    public void run() {
        while (true) {
            try {
                Thread.sleep(50);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            produce();
        }
    }

    private void produce() {
        Cookie cookie = new Cookie();
        try {
            abq.put(cookie);
            System.out.println(">>> produce " + cookie +" , count is " + abq.size());
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

class Consumer implements Runnable {
    private ArrayBlockingQueue abq;

    Consumer(ArrayBlockingQueue abq) {
        this.abq = abq;
    }

    @Override
    public void run() {
        while (true) {
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            consume();
        }
    }

    private void consume() {
        Cookie cookie = null;
        try {
            cookie = (Cookie) abq.take();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("<<< consume " + cookie + " , count is " + abq.size());
    }
}

public class ArrayBlockingQueueTest {
    public static void main(String[] args) {
        ArrayBlockingQueue abq = new ArrayBlockingQueue(10);
        new Thread(new Productor(abq)).start();
        new Thread(new Consumer(abq)).start();
    }
}
```

使用LinkedBlockingQueue差不多，将上述程序中的ArrayBlockingQueue换成LinkedBlockingQueue就行。

## 使用PriorityBlockingQueue模拟银行的VIP通道
在银行排队办理业务,通常会有一个VIP通道,让一些有VIP贵宾卡的优先办理业务,而不需要排队。假设在这么一个场景下,银行开始办理业务之前,已经来了20个客户,而且银行认为谁钱多,谁就优先办理业务。

代码如下：

```java
class Person {
    private String name;
    private int money;

    Person(String name, int money) {
        this.name = name;
        this.money = money;
    }

    private String getName() {
        return this.name;
    }

    public int getMoney() {
        return this.money;
    }

    @Override
    public String toString() {
        return getName() + "[" + "存款" + getMoney() + "]";
    }
}

class MoneyComparator implements Comparator<Person> {
    @Override
    public int compare(Person o1, Person o2) {
        return o2.getMoney() - o1.getMoney();
    }
}

class ProducerRunnable implements Runnable {
    private static final String name = "明刚红李刘吕赵黄王孙朱曾游丽吴昊周郑秦丘";
    private Random random = new Random();
    private PriorityBlockingQueue<Person> queue;

    ProducerRunnable(PriorityBlockingQueue<Person> queue) {
        this.queue = queue;
    }

    @Override
    public void run() {
        for (int i = 0; i < 20; i++) {
            Person person = new Person("小" + name.charAt(i), random.nextInt(1000));
            queue.put(person);
            System.out.println(person + "开始排队");
        }
    }
}

class ConsumerRunnable implements Runnable {
    private PriorityBlockingQueue<Person> queue;

    ConsumerRunnable(PriorityBlockingQueue<Person> queue) {
        this.queue = queue;
    }

    @Override
    public void run() {
        while (true) {
            Person person = queue.poll();
            if (person == null) {
                break;
            }
            System.out.println(person + "办理业务");
        }
    }
}

public class PriorityBlockingQueueTest {
    public static void main(String[] args) throws InterruptedException {
        PriorityBlockingQueue<Person> queue = new PriorityBlockingQueue<>(100, new MoneyComparator());
        Thread thread = new Thread(new ProducerRunnable(queue));
        thread.start();
        thread.join();
        new Thread(new ConsumerRunnable(queue)).start();
    }
}
```
结果如下：

```java
小明[存款804]开始排队
小刚[存款372]开始排队
小红[存款933]开始排队
小李[存款904]开始排队
小刘[存款322]开始排队
小吕[存款92]开始排队
小赵[存款160]开始排队
小黄[存款508]开始排队
小王[存款608]开始排队
小孙[存款76]开始排队
小朱[存款772]开始排队
小曾[存款506]开始排队
小游[存款214]开始排队
小丽[存款70]开始排队
小吴[存款853]开始排队
小昊[存款960]开始排队
小周[存款970]开始排队
小郑[存款642]开始排队
小秦[存款955]开始排队
小丘[存款960]开始排队
小周[存款970]办理业务
小昊[存款960]办理业务
小丘[存款960]办理业务
小秦[存款955]办理业务
小红[存款933]办理业务
小李[存款904]办理业务
小吴[存款853]办理业务
小明[存款804]办理业务
小朱[存款772]办理业务
小郑[存款642]办理业务
小王[存款608]办理业务
小黄[存款508]办理业务
小曾[存款506]办理业务
小刚[存款372]办理业务
小刘[存款322]办理业务
小游[存款214]办理业务
小赵[存款160]办理业务
小吕[存款92]办理业务
小孙[存款76]办理业务
小丽[存款70]办理业务
```

## 使用SynchronizedBlockingQueue模拟玩具生产流水线
在玩具生产流水线上，一个工人安装好自己负责的配件后，会将玩具传递给下一个工人安装其他配件，中间不需要存储，这比较适合使用SynchronizedBlockingQueue来做。代码如下：

```java
class Toy {
    private int num;

    Toy(int num) {
        this.num = num;
    }

    @Override
    public String toString() {
        return "toy number is " + num;
    }
}

class ToyProductor implements Runnable{
    private BlockingQueue<Toy> queue;
    private Random random = new Random();

    ToyProductor(BlockingQueue<Toy> queue) {
        this.queue = queue;
    }

    @Override
    public void run() {
        while (true) {
            Toy toy = new Toy(random.nextInt(1000));
            try {
                System.out.println(">>> productor : " + toy.toString());
                queue.put(toy);
                Thread.sleep(200);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

class ToyConsumer implements Runnable {
    private BlockingQueue<Toy> queue;

    ToyConsumer(BlockingQueue<Toy> queue) {
        this.queue = queue;
    }

    @Override
    public void run() {
        while (true)    {
            try {
                Toy toy = queue.take();
                System.out.println("<<<<<< consumer : " + toy.toString());
                Thread.sleep(300);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

public class SynchronizedBlockingQueueTest {
    public static void main(String[] args) {
        BlockingQueue<Toy> queue = new SynchronousQueue<>();

        ToyProductor productor = new ToyProductor(queue);
        ToyConsumer consumer = new ToyConsumer(queue);

        new Thread(productor).start();
        new Thread(consumer).start();
    }
}
```
结果如下：

```java
>>> productor : toy number is 90
<<<<<< consumer : toy number is 90
>>> productor : toy number is 707
<<<<<< consumer : toy number is 707
>>> productor : toy number is 875
<<<<<< consumer : toy number is 875
>>> productor : toy number is 737
<<<<<< consumer : toy number is 737
>>> productor : toy number is 435
<<<<<< consumer : toy number is 435
>>> productor : toy number is 128
<<<<<< consumer : toy number is 128
>>> productor : toy number is 806
<<<<<< consumer : toy number is 806
>>> productor : toy number is 636
<<<<<< consumer : toy number is 636
...
```

## 使用LinkedTransferQueue模拟生产者消费者
LinkedTransferQueue和SynchronizedBlockingQueue作用很相似，下面是使用LinkedBlockingQueue模拟生产者-消费者的代码。

```java
public class LinkedTransferQueueDemo {
    private static LinkedTransferQueue<String> queue = new LinkedTransferQueue<>();

    private class Productor implements Runnable {
        @Override
        public void run() {
            for (int i = 0; i < 3; i++) {
                try {
                    System.out.println("Producer is waiting to transfer...");
                    System.out.println("producer transfered element: A "+i);
                    queue.transfer("A " + i);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    private class Consumer implements Runnable {
        @Override
        public void run() {
            for (int i = 0; i < 3; i++) {
                try {
                    System.out.println("Consumer is waiting to take element...");
                    String s= queue.take();
                    System.out.println("Consumer received Element: "+s);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    public static void main(String[] args) {
        new Thread(new LinkedTransferQueueDemo().new Productor()).start();
        new Thread(new LinkedTransferQueueDemo().new Consumer()).start();
    }
}
```
结果如下：

```java
Producer is waiting to transfer...
producer transfered element: A 0
Consumer is waiting to take element...
Producer is waiting to transfer...
producer transfered element: A 1
Consumer received Element: A 0
Consumer is waiting to take element...
Consumer received Element: A 1
Consumer is waiting to take element...
Producer is waiting to transfer...
producer transfered element: A 2
Consumer received Element: A 2
```

# 参考
[聊聊并发（七）——Java中的阻塞队列](http://ifeve.com/java-blocking-queue/)</br>
[Java 并发 --- 阻塞队列总结](https://blog.csdn.net/u014634338/article/details/78915965)</br>
[Java并发编程-阻塞队列(BlockingQueue)的实现原理](https://blog.csdn.net/chenchaofuck1/article/details/51660119)</br>
[java并发之SynchronousQueue实现原理](https://blog.csdn.net/yanyan19880509/article/details/52562039)</br>
[使用delayedQueue实现你本地的延迟队列](https://blog.csdn.net/u011001723/article/details/51882887)</br>
[DelayedQueue学习笔记](https://www.jianshu.com/p/5b48180bafce)</br>
[(十六)java多线程之优先队列PriorityBlockingQueue](https://segmentfault.com/a/1190000009111757)</br>
[线程池工作窃取实例](https://segmentfault.com/a/1190000011120556)</br>