# Java并发之CAS和Atomic
---

# 无锁
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/casssss.png" width="500">
</center>

“无锁”即不用锁保证程序的并发。无锁采用CAS(compare and swap)算法来处理线程冲突。

# CAS
CAS的全称为Compare-And-Swap，直译就是对比交换。是一条CPU的原子指令，其作用是让CPU先进行比较两个值是否相等，然后原子地更新某个位置的值，经过调查发现，其实现方式是基于硬件平台的汇编指令，就是说CAS是靠硬件实现的，JVM只是封装了汇编调用，那些AtomicInteger类便是使用了这些封装后的接口。

CAS包含3个参数CAS(V,E,N)。V表示要更新的变量, E表示预期值, N表示新值。仅当V值等于E值时, 才会将V的值设为N, 如果V值和E值不同, 则说明已经有其他线程做了更新, 则当前线程什么都不做。最后, CAS返回当前V的真实值。CAS操作是抱着乐观的态度进行的, 它总是认为自己可以成功完成操作。当多个线程同时使用CAS操作一个变量时, 只有一个会胜出, 并成功更新,其余均会失败。失败的线程不会被挂起,仅是被告知失败, 并且允许再次尝试,当然也允许失败的线程放弃操作。基于这样的原理, CAS操作即时没有锁,也可以发现其他线程对当前线程的干扰, 并进行恰当的处理。

CAS操作是原子性的，所以多线程并发使用CAS更新数据时，可以不使用锁。JDK中大量使用了CAS来更新数据而防止加锁（synchronized 重量级锁）来保持原子更新。

java.util.concurrent包都中的实现类都是基于volatile和CAS来实现的。尤其java.util.concurrent.atomic包下的原子类。

简单介绍下volatile特性(具体可以参见另一篇笔记"Java并发之volatile")：

> - 内存可见性（当一个线程修改volatile变量的值时，另一个线程就可以实时看到此变量的更新值）
> - 禁止指令重排（volatile变量之前的变量执行先于volatile变量执行，volatile之后的变量执行在volatile变量之后）

# AtomicInteger 源码解析
```java
public class AtomicInteger extends Number implements java.io.Serializable {
    private static final Unsafe unsafe = Unsafe.getUnsafe();
    private static final long valueOffset;
    static {
        try {
            //用于获取value字段相对当前对象的“起始地址”的偏移量
            valueOffset = unsafe.objectFieldOffset(AtomicInteger.class.getDeclaredField("value"));
        } catch (Exception ex) { throw new Error(ex); }
    }

    private volatile int value;

    //返回当前值
    public final int get() {
        return value;
    }

    //递增加detla
    public final int getAndAdd(int delta) {
        //三个参数，1、当前的实例 2、value实例变量的偏移量 3、当前value要加上的数（value+delta）。
        return unsafe.getAndAddInt(this, valueOffset, delta);
    }

    //递增加1
    public final int incrementAndGet() {
        return unsafe.getAndAddInt(this, valueOffset, 1) + 1;
    }
...
}
```
可以看到 AtomicInteger 底层用的是volatile的变量和CAS来进行更改数据的。
volatile保证线程的可见性，多线程并发时，一个线程修改数据，可以保证其它线程立马看到修改后的值

CAS 保证数据更新的原子性。

# Unsafe源码解析
Unsafe 类中代码反编译出来的结果如下：
```java
public final int getAndAddInt(Object paramObject, long paramLong, int paramInt)
  {
    int i;
    do
      i = getIntVolatile(paramObject, paramLong);
    while (!compareAndSwapInt(paramObject, paramLong, i, i + paramInt));
    return i;
  }

  public final long getAndAddLong(Object paramObject, long paramLong1, long paramLong2)
  {
    long l;
    do
      l = getLongVolatile(paramObject, paramLong1);
    while (!compareAndSwapLong(paramObject, paramLong1, l, l + paramLong2));
    return l;
  }

  public final int getAndSetInt(Object paramObject, long paramLong, int paramInt)
  {
    int i;
    do
      i = getIntVolatile(paramObject, paramLong);
    while (!compareAndSwapInt(paramObject, paramLong, i, paramInt));
    return i;
  }

  public final long getAndSetLong(Object paramObject, long paramLong1, long paramLong2)
  {
    long l;
    do
      l = getLongVolatile(paramObject, paramLong1);
    while (!compareAndSwapLong(paramObject, paramLong1, l, paramLong2));
    return l;
  }

  public final Object getAndSetObject(Object paramObject1, long paramLong, Object paramObject2)
  {
    Object localObject;
    do
      localObject = getObjectVolatile(paramObject1, paramLong);
    while (!compareAndSwapObject(paramObject1, paramLong, localObject, paramObject2));
    return localObject;
  }
```
从源码中发现，内部使用自旋的方式进行CAS更新（while循环进行CAS更新，如果更新失败，则循环再次重试）。

又从Unsafe类中发现，原子操作其实只支持下面三个方法：
```java
public final native boolean compareAndSwapObject(Object paramObject1, long paramLong, Object paramObject2, Object paramObject3);

public final native boolean compareAndSwapInt(Object paramObject, long paramLong, int paramInt1, int paramInt2);

public final native boolean compareAndSwapLong(Object paramObject, long paramLong1, long paramLong2, long paramLong3);
```
Unsafe只提供了3种CAS方法：compareAndSwapObject、compareAndSwapInt和compareAndSwapLong。都是native方法。

native方法中使用了CPU原子指令：cmpxchg。它的作用是“比较并交换”。

如：CMPXCHG r/m,r 将累加器AL/AX/EAX/RAX中的值与首操作数（目的操作数）比较，如果相等，第2操作数（源操作数）的值装载到首操作数，zf置1。如果不等， 首操作数的值装载到AL/AX/EAX/RAX并将zf清0
该指令只能用于486及其后继机型。第2操作数（源操作数）只能用8位、16位或32位寄存器。第1操作数（目地操作数）则可用寄存器或任一种存储器寻址方式。


# AtomicBoolean 源码解析
```java
public class AtomicBoolean implements java.io.Serializable {
    private static final Unsafe unsafe = Unsafe.getUnsafe();
    private static final long valueOffset;

    static {
        try {
            valueOffset = unsafe.objectFieldOffset
                (AtomicBoolean.class.getDeclaredField("value"));
        } catch (Exception ex) { throw new Error(ex); }
    }

    private volatile int value;

    public AtomicBoolean(boolean initialValue) {
        value = initialValue ? 1 : 0;
    }
    public final boolean compareAndSet(boolean expect, boolean update) {
        int e = expect ? 1 : 0;
        int u = update ? 1 : 0;
        return unsafe.compareAndSwapInt(this, valueOffset, e, u);
    }
    ...
}
```
从AtomicBoolean源码，发现他底层也是使用volatile类型的int 变量，跟AtomicInteger 实现方式一样，只不过是把Boolean转换成 0和1进行操作。

所以原子更新char、float和double变量也可以转换成int 或long来实现CAS的操作。

# CAS缺点
- ABA问题。因为CAS需要在操作值的时候检查下值有没有发生变化，如果没有发生变化则更新，但是如果一个值原来是A，变成了B，又变成了A，那么使用CAS进行检查时会发现它的值没有发生变化，但是实际上却变化了。ABA问题的解决思路就是使用版本号。在变量前面追加上版本号，每次变量更新的时候把版本号加一，那么A-B-A 就会变成1A-2B-3A。
从Java1.5开始JDK的atomic包里提供了一个类AtomicStampedReference来解决ABA问题。这个类的compareAndSet方法作用是首先检查当前引用是否等于预期引用，并且当前标志是否等于预期标志，如果全部相等，则以原子方式将该引用和该标志的值设置为给定的更新值。
- 循环时间长开销大。自旋CAS如果长时间不成功，会给CPU带来非常大的执行开销。

# 总结
Atomic使用volatile保证可见性和有序性，使用unsafe类下面的CAS方法保证原子性。

# 参考
> - https://www.jianshu.com/p/a533cbb740c6
> - https://www.cnblogs.com/my376908915/p/6758415.html
> - https://www.cnblogs.com/xdecode/p/9022525.html
> - http://ifeve.com/java-atomic/