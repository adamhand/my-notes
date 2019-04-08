# Java并发之ReentrantReadWriteLock
---

# 概念
顾名思义，ReentrantReadWriteLock名为“可重入读写锁”，它维护两个锁：读锁和写锁。在没有写锁的情况下，读锁允许多个线程同时访问，而写锁是独占的，

ReentrantReadWriteLock主要特性有以下几个：

- **公平性**。支持公平锁和非公平锁，默认是非公平锁。非公平锁比公平锁有更高的吞吐量。
- **可重入**。允许读锁可写锁可重入。写锁可以获得读锁，读锁不能获得写锁。读写锁最多支持65535个递归写入锁和65535个递归读取锁(这里有个需要注意的地方，ReentrantLock的可重入次数为2^32-1，而ReentrantReadWriteLock可重入次数为2^16-1=65535，为啥呢？这是因为ReentrantReadWriteLock将AQS中的state域分成了两部分，读锁和写锁各占16位，具体可以往下看)。
- **锁降级**。在获得写锁的情况下再获得读锁(这和写锁是独占的并不冲突，这里是指一个线程可以活得写锁再获得读锁，独占是指多个线程之间)，然后释放写锁称为锁降级，反之称为锁升级。允许写锁降低为读锁，反之不允许。

# 使用
ReentrantReadWriteLock可以用来提高某些集合的并发性能。当集合比较大，并且读比写频繁时，可以使用该类。

读写锁的简单示例如下：

```java
class Cache{
    static Map<String, Object> map = new HashMap<String, Object>();
    static ReentrantReadWriteLock rwLock = new ReentrantReadWriteLock();
    static Lock r = rwLock.readLock();
    static Lock w = rwLock.writeLock();
    //获取一个key对应的value
    public static final Object get(String key){
        r.lock();
        try {
            return map.get(key);
        }finally {
            r.unlock();
        }
    }
    //设置key对应的value值
    public static final Object put(String key, Object value){
        w.lock();
        try {
            return map.put(key, value);
        }finally {
            w.unlock();
        }
    }
    //清空所有内容
    public static final void clear(){
        w.lock();
        try {
            map.clear();
        }finally {
            w.unlock();
        }
    }
}
```

锁降级的示例如下：

```java
class LockDec{
    static ReentrantReadWriteLock rwLock = new ReentrantReadWriteLock();
    static Lock readLock = rwLock.readLock();
    static Lock writeLock = rwLock.writeLock();
    public volatile boolean update = false;
    public void processData(){
        readLock.lock();
        if (!update){
            //必须先释放读锁
            readLock.unlock();
            //锁降级从写锁获取到开始
            writeLock.lock();
            try {
                if (!update){
                    //准备数据的流程
                    update = true;
                }
                readLock.lock();
            }finally {
                writeLock.unlock();
            }
            //锁降级完成，写锁降级为读锁
        }
        try {
            //使用数据的流程
        }finally {
            readLock.unlock();
        }
    }
}
```
当数据发生变更后 ，update变量被设置为true，此时所有访问processData()方法的线程都能感知到变化。

# 基本原理
ReentrantReadWriteLock底层也是使用AQS实现的，但是AQS中只有一个state域，这样就有了几个问题：

- AQS只有一个状态，那么如何表示 多个读锁 与 单个写锁 呢？
- ReentrantLock 里，状态值表示重入计数，现在如何在AQS里表示每个读锁、写锁的重入次数呢？
- 如何实现读锁、写锁的公平性呢？

ReentrantReadWriteLock的解决办法总结起来有以下几个：

- 将state域按位分成两部分，高位部分表示读锁，低位表示写锁，由于写锁只有一个，所以写锁的重入计数也解决了，这也会导致写锁可重入的次数减小(前面提到的2^16-1)。
- 读锁是共享锁，可以同时有多个，那么只靠一个state来计算锁的重入次数是不行的。ReentrantReadWriteLock是通过一个HoldCounter的类来实现的，这个类中有一个count计数器，同时该类通过ThreadLocal关键字被修饰为线程私有变量，那么每个线程都保留一份对读锁的重入次数。

未完待续...


# 参考
[ReentrantReadWriteLock读写锁详解](https://www.cnblogs.com/xiaoxi/p/9140541.html)
[JUC 可重入 读写锁 ReentrantReadWriteLock](http://ifeve.com/juc-reentrantreadwritelock/)
[【死磕Java并发】—–J.U.C之读写锁：ReentrantReadWriteLock](http://cmsblogs.com/?p=2213#i-2)
[Java并发编程--ReentrantReadWriteLock](https://www.cnblogs.com/zaizhoumo/p/7782941.html)
[深入理解读写锁—ReadWriteLock源码分析](https://blog.csdn.net/qq_19431333/article/details/70568478)
[读写锁 ReetrantReadWriteLock](https://www.cnblogs.com/xcyz/p/8025276.html)