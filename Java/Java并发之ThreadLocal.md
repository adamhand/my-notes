# Java并发之ThreadLocal
---

# ThreadLocal是什么
首先说明，**ThreadLocal与线程同步无关**。ThreadLocal虽然提供了一种解决多线程环境下成员变量的问题，但是**它并不是解决多线程共享变量的问题**。

ThreadLocal类提供了一种**线程局部变量(ThreadLocal)**，即每一个线程都会保存一份变量副本，每个线程都可以独立地修改自己的变量副本，而不会影响到其他线程，是一种**线程隔离**的思想。

# 实现原理
ThreadLocal提供四个方法：
```java
public T get() { }
public void set(T value) { }
public void remove() { }
protected T initialValue() { }
```
get()方法是用来获取ThreadLocal在当前线程中保存的变量副本，set()用来设置当前线程中变量的副本，remove()用来移除当前线程中变量的副本，initialValue()是一个protected方法，一般是用来在使用时进行重写的，它是一个延迟加载方法。这四种方法都是基于ThreadLocalMap的。

## ThreadLocalMap
ThreadLocal内部有一个静态内部类ThreadLocalMap，该内部类是实现线程隔离机制的关键。ThreadLocalMap提供了一种用键值对方式存储每一个线程的变量副本的方法，**key为当前ThreadLocal对象，value则是对应线程的变量副本**。该Map默认的大小是16，即能存储16个键值对，超过后会扩容。

具体源码如下：

### Entry类
ThreadLocalMap其内部利用Entry来实现key-value的存储，如下：
```java
 static class Entry extends WeakReference<ThreadLocal<?>> {
    /** The value associated with this ThreadLocal. */
    Object value;

    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
}
```
从上面代码中可以看出Entry的key就是ThreadLocal，而value就是值。同时，Entry也继承WeakReference，所以说Entry所对应key（ThreadLocal实例）的引用为一个弱引用。

`ThreadLocal`的内部结构如下图所示：

<div align="center">
<img src="https://github.com/adamhand/LeetCode-images/blob/master/threadlocal.jpg?raw=true">
</div>

### set方法
```java
 private void set(ThreadLocal<?> key, Object value) {

    ThreadLocal.ThreadLocalMap.Entry[] tab = table;
    int len = tab.length;

    // 根据 ThreadLocal 的散列值，查找对应元素在数组中的位置
    int i = key.threadLocalHashCode & (len-1);

    // 采用“线性探测法”，寻找合适位置
    for (ThreadLocal.ThreadLocalMap.Entry e = tab[i];
        e != null;
        e = tab[i = nextIndex(i, len)]) {

        ThreadLocal<?> k = e.get();

        // key 存在，直接覆盖
        if (k == key) {
            e.value = value;
            return;
        }

        // key == null，但是存在值（因为此处的e != null），说明之前的ThreadLocal对象已经被回收了
        if (k == null) {
            // 用新元素替换陈旧的元素
            replaceStaleEntry(key, value, i);
            return;
        }
    }

    // ThreadLocal对应的key实例不存在也没有陈旧元素，new 一个
    tab[i] = new ThreadLocal.ThreadLocalMap.Entry(key, value);

    int sz = ++size;

    // cleanSomeSlots 清楚陈旧的Entry（key == null）
    // 如果没有清理陈旧的 Entry 并且数组中的元素大于了阈值，则进行 rehash
    if (!cleanSomeSlots(i, sz) && sz >= threshold)
        rehash();
}
```
ThreadLocalMap的set方法和Map的put方法差不多，但是有一点区别是：put方法处理哈希冲突使用的是**链地址法**，而set方法使用的**开放地址法**。

set()操作除了存储元素外，还有一个很重要的作用，就是replaceStaleEntry()和cleanSomeSlots()，这两个方法可以清除掉key == null 的实例，防止内存泄漏。在set()方法中还有一个变量很重要：threadLocalHashCode，定义如下：
```java
private final int threadLocalHashCode = nextHashCode();
```
threadLocalHashCode是ThreadLocal的散列值，定义为final，表示ThreadLocal一旦创建其散列值就已经确定了，生成过程则是调用nextHashCode()：
```java
private static AtomicInteger nextHashCode = new AtomicInteger();

private static final int HASH_INCREMENT = 0x61c88647;

private static int nextHashCode() {
    return nextHashCode.getAndAdd(HASH_INCREMENT);
}
```
nextHashCode表示分配下一个ThreadLocal实例的threadLocalHashCode的值，HASH_INCREMENT则表示分配两个ThradLocal实例的threadLocalHashCode的增量。

### getEntry()
```java
private Entry getEntry(ThreadLocal<?> key) {
    int i = key.threadLocalHashCode & (table.length - 1);
    Entry e = table[i];
    if (e != null && e.get() == key)
        return e;
    else
        return getEntryAfterMiss(key, i, e);
}
```
由于采用了开放定址法，所以当前key的散列值和元素在数组的索引并不是完全对应的，首先取一个探测数（key的散列值），如果所对应的key就是我们所要找的元素，则返回，否则调用getEntryAfterMiss()，如下：
```java
private Entry getEntryAfterMiss(ThreadLocal<?> key, int i, Entry e) {
    Entry[] tab = table;
    int len = tab.length;

    while (e != null) {
        ThreadLocal<?> k = e.get();
        if (k == key)
            return e;
        if (k == null)
            expungeStaleEntry(i);
        else
            i = nextIndex(i, len);
        e = tab[i];
    }
    return null;
}
```
这里有一个重要的地方，当key == null时，调用了expungeStaleEntry()方法，该方法用于处理key == null，有利于GC回收，能够有效地避免内存泄漏。

## get()方法
```java
public T get() {
    // 获取当前线程
    Thread t = Thread.currentThread();

    // 获取当前线程的成员变量 threadLocal
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        // 从当前线程的ThreadLocalMap获取相对应的Entry
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")

            // 获取目标值        
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}
```
首先通过当前线程获取所对应的成员变量ThreadLocalMap，然后通过ThreadLocalMap获取当前ThreadLocal的Entry，最后通过所获取的Entry获取目标值result。

getMap()方法可以获取当前线程所对应的ThreadLocalMap，如下：
```java
ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}
```

##  set(T value)
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
获取当前线程所对应的ThreadLocalMap，如果不为空，则调用ThreadLocalMap的set()方法，key就是当前ThreadLocal，如果不存在，则调用createMap()方法新建一个，如下：
```java
void createMap(Thread t, T firstValue) {
    t.threadLocals = new ThreadLocalMap(this, firstValue);
}
```

## initialValue()
```java
protected T initialValue() {
    return null;
}
```
该方法定义为protected级别且返回为null，很明显是要子类实现它的，所以我们在使用ThreadLocal的时候一般都应该覆盖该方法。

注意：**如果想在get之前不需要调用set就能正常访问的话，必须重写initialValue()方法。**

因为在上面的代码分析过程中，我们发现如果没有先set的话，即在map中查找不到对应的存储，则会通过调用setInitialValue方法返回i，而在setInitialValue方法中，有一个语句是T value = initialValue()， 而默认情况下，initialValue方法返回的是null。

## remove()
```java
 public void remove() {
    ThreadLocalMap m = getMap(Thread.currentThread());
    if (m != null)
        m.remove(this);
}
```
该方法的目的是减少内存的占用。为了避免内存泄露，使用完`ThreadLocal`变量之后都要调用`remove`方法。

# ThreadLocal使用示例
```java
public class SeqCount {

    private static ThreadLocal<Integer> seqCount = new ThreadLocal<Integer>(){
        // 实现initialValue()
        public Integer initialValue() {
            return 0;
        }
    };

    public int nextSeq(){
        seqCount.set(seqCount.get() + 1);

        return seqCount.get();
    }

    public void removeSeq(){
        seqCount.remove();
    }

    public static void main(String[] args){
        SeqCount seqCount = new SeqCount();

        SeqThread thread1 = new SeqThread(seqCount);
        SeqThread thread2 = new SeqThread(seqCount);
        SeqThread thread3 = new SeqThread(seqCount);
        SeqThread thread4 = new SeqThread(seqCount);

        thread1.start();
        thread2.start();
        thread3.start();
        thread4.start();
    }

    private static class SeqThread extends Thread{
        private SeqCount seqCount;

        SeqThread(SeqCount seqCount){
            this.seqCount = seqCount;
        }

        public void run() {
            for(int i = 0 ; i < 3 ; i++){
                System.out.println(Thread.currentThread().getName() + " seqCount :" + seqCount.nextSeq());
            }
            seqCount.removeSeq();
        }
    }
}
```
结果如下：
```java
Thread-1 seqCount :1
Thread-3 seqCount :1
Thread-2 seqCount :1
Thread-0 seqCount :1
Thread-2 seqCount :2
Thread-3 seqCount :2
Thread-1 seqCount :2
Thread-3 seqCount :3
Thread-2 seqCount :3
Thread-0 seqCount :2
Thread-1 seqCount :3
Thread-0 seqCount :3
```

# ThreadLocal与内存泄漏
## 为什么会出现内存泄漏
首先看一下运行时ThreadLocal变量的内存图：
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/ThreadLocalMap.jpg">
</center>

运行时，会在栈中产生两个引用，指向堆中相应的对象。

可以看到，ThreadLocalMap使用ThreadLocal的弱引用作为key，这样一来，当ThreadLocal ref和ThreadLocal之间的强引用断开 时候，即ThreadLocal ref被置为null，下一次GC时，threadLocal对象势必会被回收，这样，ThreadLocalMap中就会出现key为null的Entry，就没有办法访问这些key为null的Entry的value，如果当前线程再迟迟不结束的话，**比如使用线程池**，线程使用完成之后会被放回线程池中，不会被销毁，这些key为null的Entry的value就会一直存在一条强引用链：Thread Ref -> Thread -> ThreaLocalMap -> Entry -> value永远无法回收，造成内存泄漏。

其实，ThreadLocalMap的设计中已经考虑到这种情况，也加上了一些防护措施：**在ThreadLocal的get(),set(),remove()的时候都会清除线程ThreadLocalMap里所有key为null的value。**

但是这些被动的预防措施并不能保证不会内存泄漏：

- 使用static的ThreadLocal，延长了ThreadLocal的生命周期，可能导致的内存泄漏。
- 分配使用了ThreadLocal又不再调用get(),set(),remove()方法，那么就会导致内存泄漏。

## 为什么要使用弱引用？
使用弱引用，是为了更好地对ThreadLocal对象进行回收。如果使用强引用，当ThreadLocal ref = null的时候，意味着ThreadLocal对象已经没用了，ThreadLocal对象应该被回收，但由于Entry中还存着这对ThreadLocal对象的强引用，导致ThreadLocal对象不能回收，可能会发生内存泄漏。

## 为什么不将value也设置成弱引用？
为什么呢？

## 如何避免内存泄漏？
**每次使用完ThreadLocal，都调用它的remove()方法，清除数据。**

# ThreadLocal与脏读
前面说了，ThreadLocal中的set()、get()和remove()方法都会对key==null的value进行处理，其中set()和get()方法是将key==null的value置为null。但是如果ThreadLocal是static类型的，并且配合线程池使用，线程池会重用Thread对象，同时会重用与Thread绑定的ThreadLocal变量。倘若下一个线程不调用set()方法重新设置初始值，也不调用remove()方法处理旧值，直接调用get()方法获取，就会出现脏读问题。

例子如下。
```java
public class DirtyDataInThreadLocal {
    public static ThreadLocal<String> threadLocal = new ThreadLocal<>();

    public static void main(String[] args) {
        //使用固定大小为1的线程池，说明上一个线程属性会被下一个线程属性复用
        ExecutorService pool = Executors.newFixedThreadPool(1);

        for(int i = 0; i < 2; i++){
            MyThread thread = new MyThread();
            pool.execute(thread);
        }
    }

    private static class MyThread extends Thread{
        private static boolean flag = true;

        @Override
        public void run() {
            if(flag){
                //第一个线程set后，没有remove，第二个线程也没有进行set操作
                threadLocal.set(this.getName() + ", session info.");
                flag = false;
            }
            System.out.println(this.getName() + " 线程是 " + threadLocal.get());
        }
    }
}
```
打印结果如下：
```java
Thread-0线程是 Thread-0, session info.
Thread-1线程是 Thread-0, session info.
```

# ThreadLocal使用场景
## 数据连接和Session管理
最常见的ThreadLocal使用场景为 用来解决 数据库连接、Session管理等。

如：
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

## ThreadLocal在Spring中的应用


# 参考
[【死磕Java并发】—–深入分析ThreadLocal](http://cmsblogs.com/?p=2442)</br>
[深入分析 ThreadLocal内存泄漏问题](http://blog.xiaohansong.com/2016/08/06/ThreadLocal-memory-leak/)</br>
[Java并发编程：深入剖析ThreadLocal](https://www.cnblogs.com/dolphin0520/p/3920407.html)</br>
[Java多线程编程-（8）-多图深入分析ThreadLocal原理](https://blog.csdn.net/xlgen157387/article/details/78297568)</br>
[ThreadLocal类详解与源码分析](https://blog.csdn.net/shenlei19911210/article/details/50060223)</br>
[ThreadLocal解决什么问题](https://www.cnblogs.com/jasongj/p/8079718.html)</br>
[对ThreadLocal实现原理的一点思考](https://www.jianshu.com/p/ee8c9dccc953)</br>
[ThreadLocalMap的enrty的key为什么要设置成弱引用](https://blog.csdn.net/qq646040754/article/details/82493409)</br>
《码出高效 Java开发手册》