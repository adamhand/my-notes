# Java并发之CountDownLatch、CyclicBarrier和Semaphore
---

# CountDownLatch
## 是什么
先看一下官方的解释：
```
CountDownLatch:A synchronization aid that allows one or more threads to wait until a set of operations being performed in other threads completes.
```
大概的意思是，CountDownLatch允许一个或多个线程等待其他一组线程完成后，再继续执行。其实这从CountDownLatch的名字也可以大概看出它的用途，count-计数，down-减，latch-门栓，当计数器减为零之后，门栓就打开了，线程可以继续执行。

可以举个例子：在玩LOL英雄联盟时会出现十个人不同加载状态，但是最后一个人由于各种原因始终加载不了100%，于是游戏系统自动等待所有玩家的状态都准备好，才展现游戏画面。

## 怎么用
在使用时关注的方法主要有三个：
```java
await();     // 使当前线程在锁存器倒计数至零之前一直等待
await(long timeout, TimeUnit unit) throws InterruptedException { };  //和await()类似，只不过等待timeout的时间后计数器值还没变为0的话就会继续执行
countDown(); // 递减锁存器的计数，如果计数到达零，则释放所有等待的线程
```

下面以上面提到的的游戏的场景，模拟一下CountDownLatch的用法。

```java
public class CountDownLatchGame {
    public static void main(String[] args) {
        ExecutorService executor = Executors.newCachedThreadPool();
        final CountDownLatch latch = new CountDownLatch(3);

        executor.execute(new Runnable() {
            @Override
            public void run() {
                try {
                    latch.await();
                    System.out.println("所有玩家都准备好，可以开始游戏了");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });

        executor.execute(new Runnable() {
            @Override
            public void run() {
                System.out.println("玩家1已经准备好");
                latch.countDown();
            }
        });

        executor.execute(new Runnable() {
            @Override
            public void run() {
                System.out.println("玩家2已经准备好");
                latch.countDown();
            }
        });

        executor.execute(new Runnable() {
            @Override
            public void run() {
                System.out.println("玩家3已经准备好");
                latch.countDown();
            }
        });
    }
}
```
打印结果如下：

```java
玩家1已经准备好
玩家2已经准备好
玩家3已经准备好
所有玩家都准备好，可以开始游戏了
```

## 待续
源码分析

# CyclicBarrier
## 是什么
先看一下官方的解释：
```
CyclicBarrier：A synchronization aid that allows a set of threads to all wait for each other to reach a common barrier point. 
```

也就是说，CyclicBarrier允许一组线程相互之间等待，达到一个共同点(Barrier，栅栏)，再继续执行。

举一个打游戏的例子，打游戏的时候需要等待玩家都准备好之后才会开始游戏，所以游戏玩家之间需要相互等待。

## 怎么用
下面就上面的打游戏的例子写一下代码。

```java
public class CyclicBarrierDemo {
    public static void main(String[] args) {
        ExecutorService executor = Executors.newCachedThreadPool();
        final CyclicBarrier barrier = new CyclicBarrier(5, new Runnable() {
            @Override
            public void run() {
                System.out.println("玩家准备好，游戏开始");
            }
        });

        playerWait(barrier, executor);
        /**
         * 可重用
         */
        //playerWait(barrier, executor);
        //playerWait(barrier, executor);
    }

    public static void playerWait(final CyclicBarrier barrier, ExecutorService executor){
        for (int i = 0; i < 5; i++){
            final int j = i;
            executor.execute(new Runnable() {
                @Override
                public void run() {
                    try {
                        System.out.println("玩家"+j+"准备好，等待其他玩家准备");
                        barrier.await();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    } catch (BrokenBarrierException e) {
                        e.printStackTrace();
                    }
                }
            });
        }
    }
}
```
打印结果为：
```java
玩家1准备好，等到其他玩家准备
玩家4准备好，等到其他玩家准备
玩家3准备好，等到其他玩家准备
玩家0准备好，等到其他玩家准备
玩家2准备好，等到其他玩家准备
玩家准备好，游戏开始
```
另外，CyclicBarrier是可以重用的，当线程执行结束后，又可以用来进行新一轮的使用。而CountDownLatch无法进行重复使用。将上面程序中的注释打开，即可以重用CyclicBarrier。

## 待续
源码分析

# Semaphore
## 是什么
Semaphore翻译成字面意思为 信号量，Semaphore可以控同时访问的线程个数，通过 acquire() 获取一个许可，如果没有就等待，而 release() 释放一个许可。

Semaphore提供了2个构造器：
```java
public Semaphore(int permits) {          //参数permits表示许可数目，即同时可以允许多少线程进行访问
    sync = new NonfairSync(permits);
}
public Semaphore(int permits, boolean fair) {    //这个多了一个参数fair表示是否是公平的，即等待时间越久的越先获取许可
    sync = (fair)? new FairSync(permits) : new NonfairSync(permits);
}
```

Semaphore类中比较重要的几个方法，首先是acquire()、release()方法：
```java
public void acquire() throws InterruptedException {  }     //获取一个许可，若无许可能够获得，则会一直等待，直到获得许可。
public void acquire(int permits) throws InterruptedException { }    //获取permits个许可
public void release() { }          //释放一个许可
public void release(int permits) { }    //释放permits个许可
```

这4个方法都会被阻塞，如果想立即得到执行结果，可以使用下面几个方法：
```java
public boolean tryAcquire() { };    //尝试获取一个许可，若获取成功，则立即返回true，若获取失败，则立即返回false
public boolean tryAcquire(long timeout, TimeUnit unit) throws InterruptedException { };  //尝试获取一个许可，若在指定的时间内获取成功，则立即返回true，否则则立即返回fals
public boolean tryAcquire(int permits) { }; //尝试获取permits个许可，若获取成功，则立即返回true，若获取失败，则立即返回false
public boolean tryAcquire(int permits, long timeout, TimeUnit unit) throws InterruptedException { }; //尝试获取permits个许可，若在指定的时间内获取成功，则立即返回true，否则则立即返回false
```
另外还可以通过availablePermits()方法得到可用的许可数目。

## 怎么用
下面通过一个例子来看一下Semaphore的具体使用：

假若一个工厂有5台机器，但是有10个工人，一台机器同时只能被一个工人使用，只有使用完了，其他工人才能继续使用。
```java
public class SemaphoreDemo {
    public static void main(String[] args) {
        int workerNum = 10;
        final Semaphore semaphore = new Semaphore(5);

        ExecutorService executor = Executors.newCachedThreadPool();

        for(int i = 0; i < workerNum; i++){
            final int j = i;
            executor.execute(new Runnable() {
                @Override
                public void run() {
                    try {
                        semaphore.acquire();
                        System.out.println("工人"+j+"正在使用机器");
                        Thread.sleep(1000);
                        System.out.println("工人"+j+"释放机器");
                        semaphore.release();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }

                }
            });
        }
    }
}
```
打印结果为：
```java
工人0正在使用机器
工人3正在使用机器
工人2正在使用机器
工人1正在使用机器
工人4正在使用机器
工人2释放机器
工人0释放机器
工人3释放机器
工人7正在使用机器
工人1释放机器
工人4释放机器
工人9正在使用机器
工人8正在使用机器
工人6正在使用机器
工人5正在使用机器
工人8释放机器
工人7释放机器
工人9释放机器
工人6释放机器
工人5释放机器
```

## 待续
源码分析


# 参考
[CountDownLatch用法](https://blog.csdn.net/cyantide/article/details/50947356)
[你真的理解CountDownLatch与CyclicBarrier使用场景吗？](https://blog.csdn.net/zzg1229059735/article/details/61191679)
[Java并发编程：CountDownLatch、CyclicBarrier和Semaphore](https://www.cnblogs.com/dolphin0520/p/3920397.html)
[Java多线程编程-（8）-两种常用的线程计数器CountDownLatch和循环屏障CyclicBarrier](https://blog.csdn.net/bntX2jSQfEHy7/article/details/78237208)
[Java并发编程：CountDownLatch、CyclicBarrier和Semaphore](https://www.cnblogs.com/dolphin0520/p/3920397.html)

