# Java并发——从synchronized到锁优化
---
# synchronized(读音[ˈsɪŋkrənaɪzd])用法
synchronized关键字同步的三种用法:

- 同步实例方法，锁是当前实例对象
- 同步类方法，锁是当前类对象
- 同步代码块，锁是括号里面的对象

示例代码如下：
```java

public class SynchronizedTest {

    /**
     * 同步实例方法，锁实例对象
     */
    public synchronized void test() {
    }

    /**
     * 同步类方法，锁类对象
     */
    public synchronized static void test1() {
    }

    /**
     * 同步代码块
     */
    public void test2() {
        // 锁类对象
        synchronized (SynchronizedTest.class) {
            // 锁实例对象
            synchronized (this) {

            }
        }
    }
}
```

# synchronized实现
使用`javac`命令编译上述代码，然后使`javap -verbose`查看得到的`.class`文件:
<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/synchronized_javap.PNG">
</div>

可以看到，同步代码块是使用monitorenter和monitorexit指令实现的，同步方法依靠的是方法修饰符上的ACC_SYNCHRONIZED实现。具体如下：

**同步方法：** 方法级同步没有通过字节码指令来控制，它实现在方法调用和返回操作之中。当方法调用时，调用指令会检查方法ACC_SYNCHRONIZED访问标志是否被设置，若设置了则执行线程需要持有管程(Monitor)才能运行方法，当方法完成(无论是否出现异常)时释放管程。

**同步代码块:** synchronized关键字经过编译后，会在同步块的前后分别形成monitorenter和monitorexit两个字节码指令，每条monitorenter指令都必须执行其对应的monitorexit指令，为了保证方法异常完成时这两条指令依然能正确执行，编译器会**自动产生一个异常处理器**，其目的就是用来执行monitorexit指令(图中14-18、24-30为异常流程)。

# java对象头和monitor
Java对象头和monitor是实现synchronized的基础。
## java对象头
synchronized用的锁是存在Java对象头里的，Hotspot虚拟机的对象头主要包括两部分数据：**Mark Word（标记字段）**、**Klass Pointer（类型指针）**。其中Klass Point是是对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例，Mark Word用于存储对象自身的运行时数据，它是实现轻量级锁和偏向锁的关键。

### Mark Word
默认存储对象的HashCode，分代年龄和锁标志位信息。这些信息都是与对象自身定义无关的数据，所以Mark Word被设计成一个非固定的数据结构以便在极小的空间内存存储尽量多的数据。它会根据对象的状态复用自己的存储空间，也就是说在运行期间Mark Word里存储的数据会随着锁标志位的变化而变化。

### Klass Point
对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例。

## Monitor
每个对象都拥有自己的监视器(Monitor)，相当于操作系统中的互斥量（mutex）,它内置与每一个Object对象中。当这个对象由同步块或者这个对象的同步方法调用时，执行方法的线程必须先获取该对象的监视器才能进入同步块和同步方法，如果没有获取到监视器的线程将会被阻塞在同步块和同步方法的入口处，进入到BLOCKED状态，如图:
<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/monitorenterandexit.jpg">
</div>

在线程进入同步块的时候，如果同步对象锁状态为无锁状态，虚拟机首先将在当前线程的栈帧中建立一个**lock record**字段。每一个被锁住的对象都会和一个lock record关联(对象头的MarkWord中的LockWord指向lock record的起始地址)，同时lock record中有一个owner字段存放拥有该锁的线程的唯一标识，表示该锁被这个线程占用。其结构如下:

<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/monitorstructure.jpg">
</div>

- **Owner：** 初始时为NULL表示当前没有任何线程拥有该lock record，当线程成功拥有该锁后保存线程唯一标识，当锁被释放时又设置为NULL；
- **EntryQ:** 关联一个系统互斥锁（semaphore），阻塞所有试图锁住lock record失败的线程。
- **RcThis:** 表示blocked或waiting在该lock record上的所有线程的个数。
- **Nest:** 用来实现重入锁的计数。
- **HashCode:** 保存从对象头拷贝过来的HashCode值（可能还包含GC age）。
- **Candidate:** 用来避免不必要的阻塞或等待线程唤醒，因为每一次只有一个线程能够成功拥有锁，如果每次前一个释放锁的线程唤醒所有正在阻塞或等待的线程，会引起不必要的上下文切换（从阻塞到就绪然后因为竞争锁失败又被阻塞）从而导致性能严重下降。Candidate只有两种可能的值0表示没有需要唤醒的线程1表示要唤醒一个继任线程来竞争锁。

这样，当某个线程获得对象锁时，它们之间的状态如下图所示：

<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/lock record.jpg" width="500">
</div>

所以，当线程进入同步块的时候，获得锁的过程应该包括如下过程(**此处还有待考究**)：

- 尝试通过`monitorenter`获得对象的`monitor`
- 成功获得之后会在栈帧中建立`lock record`字段，并与锁对象进行关联

# 锁优化
阻塞或唤醒一个Java线程需要操作系统切换CPU状态来完成，这种状态转换需要耗费处理器时间。如果同步代码块中的内容过于简单，状态转换消耗的时间有可能比用户代码执行的时间还要长。这种方式就是synchronized最初实现同步的方式，这就是JDK 6之前synchronized效率低的原因。这种依赖于操作系统Mutex Lock所实现的锁我们称之为“重量级锁”，JDK 6中为了减少获得锁和释放锁带来的性能消耗，引入了“偏向锁”和“轻量级锁”。

所以目前锁一共有**4种**状态，级别从低到高依次是：**无锁**、**偏向锁**、**轻量级锁**和**重量级锁**。锁状态只能升级不能降级。

四种锁状态对应的的Mark Word内容如下：

|锁状态|存储内容|标志位|
|-|-|-|
|无锁|	对象的hashCode、对象分代年龄、是否是偏向锁（0）	|01|
|偏向锁	|偏向线程ID、偏向时间戳、对象分代年龄、是否是偏向锁（1）	|01|
|轻量级锁|	指向栈中锁记录的指针|	00|
|重量级锁|	指向互斥量（重量级锁）的指针|	10|

## 无锁
无锁没有对资源进行锁定，所有的线程都能访问并修改同一个资源，但同时只有一个线程能修改成功。

无锁的特点就是修改操作在循环内进行，线程会不断的尝试修改共享资源。如果没有冲突就修改成功并退出，否则就会继续循环尝试。如果有多个线程修改同一个值，必定会有一个线程能修改成功，而其他修改失败的线程会不断重试直到修改成功。CAS即是无锁的实现。无锁无法全面代替有锁，但无锁在某些场合下的性能是非常高的。

## 偏向锁

偏向锁是指一段同步代码一直被一个线程所访问，那么该线程会自动获取锁，降低获取锁的代价。

在大多数情况下，锁总是由同一线程多次获得，不存在多线程竞争，所以出现了偏向锁。其目标就是在只有一个线程执行同步代码块时能够提高性能。

当一个线程访问同步代码块并获取锁时，会在Mark Word里存储锁偏向的线程ID。在线程进入和退出同步块时不再通过CAS操作来加锁和解锁，而是检测Mark Word里是否存储着指向当前线程的偏向锁。引入偏向锁是为了在无多线程竞争的情况下尽量减少不必要的轻量级锁执行路径，因为轻量级锁的获取及释放依赖多次CAS原子指令，而偏向锁只需要在置换ThreadID的时候依赖一次CAS原子指令即可。

偏向锁只有遇到其他线程尝试竞争偏向锁时，持有偏向锁的线程才会释放锁，线程不会主动释放偏向锁。偏向锁的撤销，需要等待全局安全点（在这个时间点上没有字节码正在执行），它会首先暂停拥有偏向锁的线程，判断锁对象是否处于被锁定状态。撤销偏向锁后恢复到无锁（标志位为“01”）或轻量级锁（标志位为“00”）的状态。

偏向锁在JDK 6及以后的JVM里是默认启用的。可以通过JVM参数关闭偏向锁：-XX:-UseBiasedLocking=false，关闭之后程序默认会进入轻量级锁状态。

## 轻量级锁

是指当锁是偏向锁的时候，被另外的线程所访问，偏向锁就会升级为轻量级锁，其他线程会通过自旋的形式尝试获取锁，不会阻塞，从而提高性能。

在代码进入同步块的时候，如果同步对象锁状态为无锁状态（锁标志位为“01”状态，是否为偏向锁为“0”），虚拟机首先将在当前线程的栈帧中建立一个名为锁记录（Lock Record）的空间，用于存储锁对象目前的Mark Word的拷贝，然后拷贝对象头中的Mark Word复制到锁记录中。

拷贝成功后，虚拟机将使用CAS操作尝试将对象的Mark Word更新为指向Lock Record的指针，并将Lock Record里的owner指针指向对象的Mark Word。

如果这个更新动作成功了，那么这个线程就拥有了该对象的锁，并且对象Mark Word的锁标志位设置为“00”，表示此对象处于轻量级锁定状态。

如果轻量级锁的更新操作失败了，虚拟机首先会检查对象的Mark Word是否指向当前线程的栈帧，如果是就说明当前线程已经拥有了这个对象的锁，那就可以直接进入同步块继续执行，否则说明多个线程竞争锁。

若当前只有一个等待线程，则该线程通过自旋进行等待。但是当自旋超过一定的次数，或者一个线程在持有锁，一个在自旋，又有第三个来访时，轻量级锁升级为重量级锁。

## 重量级锁

升级为重量级锁时，锁标志的状态值变为“10”，此时Mark Word中存储的是指向重量级锁的指针，此时等待锁的线程都会进入阻塞状态。

重量级锁通过对象内部的监视器(monitor)实现，其中monitor的本质是依赖于底层操作系统的Mutex Lock实现，操作系统实现线程之间的切换需要从用户态到内核态的切换，切换成本非常高。

## 总结
偏向锁通过对比Mark Word解决加锁问题，避免多次执行CAS操作。而轻量级锁是通过用CAS操作和自旋来解决加锁问题，避免线程阻塞和唤醒而影响性能。重量级锁是将除了拥有锁的线程以外的线程都阻塞。

---
# 补丁
第一部分提到了静态方法和非静态方法的锁是不同的，这里再使用示例进行一下验证，观察在实际使用时它们有什么不同。

这里需要验证五个方面：

- 两个线程访问同一对象的(不同/相同)非静态同步方法会不会互斥；
- 两个线程访问不同对象的同一非静态同步方法会不会互斥；
- 两个线程访问同一类的(不同/相同)静态同步方法会不会互斥；
- 两个线程通过静态对象访问同步方法会不会互斥；
- 两个线程分别调用同一对象的静态同步方法和非静态同步方法会不会互斥；

首先给出包含同步函数的类和主类。主类在测试过程中不需要修改，SynchronizedClass只需要在new对象的时候注释掉单例模式的代码即可：
```java
public class SynchronizedClass {
    private static final SynchronizedClass instance = new SynchronizedClass();

    private SynchronizedClass(){}

    public static SynchronizedClass getInstance(){
        return instance;
    }

    /**
     * 静态对象，用来调用方法
     */
    public static SynchronizedClass synchronizedClass = new SynchronizedClass();

    public synchronized void method1(){
        for(int i = 0; i < 3; i++){
            System.out.println("method1 is running.......");
            try {
                Thread.sleep(1000);
            }catch (Exception e){
                e.printStackTrace();
            }
        }
    }

    public synchronized void method2(){
        for(int i = 0; i < 3; i++){
            System.out.println("method2 is running.......");
            try {
                Thread.sleep(1000);
            }catch (Exception e){
                e.printStackTrace();
            }
        }
    }

    public static synchronized void staticMethod1(){
        for(int i = 0; i < 3; i++){
            System.out.println("staticMethod1 is running.......");
            try {
                Thread.sleep(1000);
            }catch (Exception e){
                e.printStackTrace();
            }
        }
    }

    public static synchronized void staticMethod2(){
        for(int i = 0; i < 3; i++){
            System.out.println("staticMethod2 is running.......");
            try {
                Thread.sleep(1000);
            }catch (Exception e){
                e.printStackTrace();
            }
        }
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        Thread t1 = new Thread(new Thread1());
        Thread t2 = new Thread(new Thread2());
        t1.start();
        t2.start();
    }
}
```

## 验证1：两个线程访问同一对象(不同/相同)的非静态同步方法
给出测试类，这里只给出不同非静态同步方法的测试，调用同一非静态同步方法类似。

```java
public class Thread1 implements Runnable {
    @Override
    public void run() {
        SynchronizedClass sClass = SynchronizedClass.getInstance();
        sClass.method1();
    }
}

public class Thread2 implements Runnable {
    @Override
    public void run() {
        SynchronizedClass sClass = SynchronizedClass.getInstance();
        sClass.method2();
    }
}
```
打印结果如下，可以看到产生了互斥。
```java
method1 is running.......
method1 is running.......
method1 is running.......
method2 is running.......
method2 is running.......
method2 is running.......
```

## 验证2：两个线程访问不同对象的非静态同步方法
测试类如下：

```java
public class Thread1 implements Runnable {
    @Override
    public void run() {
        SynchronizedClass sClass = new SynchronizedClass();
        sClass.method1();
    }
}

public class Thread2 implements Runnable {
    @Override
    public void run() {
        SynchronizedClass sClass = new SynchronizedClass();
        sClass.method2();
    }
}
```
结果如下，可以看到并没有产生互斥。
```java
method1 is running.......
method2 is running.......
method1 is running.......
method2 is running.......
method1 is running.......
method2 is running.......
```

## 验证3：两个线程访问同一类的(不同/相同)静态同步方法
这里只给出访问不同静态同步方法的示例，相同同步方法类似。
```java
public class Thread1 implements Runnable {
    @Override
    public void run() {
        SynchronizedClass.staticMethod2();;
    }
}

public class Thread2 implements Runnable {
    @Override
    public void run() {
        SynchronizedClass.staticMethod1();;
    }
}
```
打印结果如下，可以看到，产生了互斥。
```java
staticMethod1 is running.......
staticMethod1 is running.......
staticMethod1 is running.......
staticMethod2 is running.......
staticMethod2 is running.......
staticMethod2 is running.......
```

## 验证4：两个线程通过静态对象访问同步方法
这里只给出调用非静态同步方法的例子，调用静态同步方法类似。

```java
public class Thread1 implements Runnable {
    @Override
    public void run() {
        SynchronizedClass.synchronizedClass.method1();
    }
}

public class Thread1 implements Runnable {
    @Override
    public void run() {
        SynchronizedClass.synchronizedClass.method2();
    }
}
```
打印结果如下，可以看到，会产生互斥。
```java
method1 is running.......
method1 is running.......
method1 is running.......
method2 is running.......
method2 is running.......
method2 is running.......
```

## 验证5：两个线程分别调用同一对象的静态同步方法和非静态同步方法
```java
public class Thread1 implements Runnable {
    @Override
    public void run() {
        SynchronizedClass.synchronizedClass.method1();
    }
}

public class Thread2 implements Runnable {
    @Override
    public void run() {
        SynchronizedClass.synchronizedClass.staticMethod1();
    }
}
```
打印结果如下，可以看到，不会发生互斥。
```java
method1 is running.......
staticMethod1 is running.......
method1 is running.......
staticMethod1 is running.......
method1 is running.......
staticMethod1 is running.......
```

## 解释

### 验证1：
非静态同步方法的锁是对象本身，既然是调用的同一对象的同步方法，那么不管这两个同步方法是不是一个，锁都是相同的，所以会互斥。

### 验证2：
既然是不同的对象，锁肯定是不一样的，即使是同一个非静态方法。所以不会互斥。

### 验证3：
静态同步方法的锁是类对象本身，只要是同一个类的静态同步方法，用的锁都是相同的，所以会互斥。

### 验证4：
首先了解，类对象是唯一的。通过静态对象调用两个静态同步方法时，锁是当前静态对象，是相同的，所以会互斥。

调用两个非静态同步方法时类似，锁是当前类对象。会互斥。

### 验证5：
因为锁不一样，一个是当前对象，一个是当前类对象，所以不会互斥。

## 小结
分析多个线程调用同步方法时，只需要关注的是，**被调用的同步函数的锁是不是相同的**，如果是，就会产生互斥，否则就不会。从这个出发点去分析，就会比较清楚。

# 参考
[Java并发——关键字synchronized解析](https://juejin.im/post/5b42c2546fb9a04f8751eabc)
[【死磕Java并发】—–深入分析synchronized的实现原理](http://cmsblogs.com/?p=2071)
[Java中synchronized的实现原理与应用](https://blog.csdn.net/u012465296/article/details/53022317)
[JVM源码分析之synchronized实现](https://www.jianshu.com/p/c5058b6fe8e5)
[JVM源码分析之java对象头实现](https://www.jianshu.com/p/9c19eb0ea4d8)
[不可不说的Java“锁”事](https://tech.meituan.com/Java_Lock.html)
[JAVA并发：多线程编程之同步“监视器monitor”（三）](https://blog.csdn.net/jingzi123456789/article/details/69951057)
[Synchronized同步静态方法和非静态方法总结](https://blog.csdn.net/u010842515/article/details/65443084)
