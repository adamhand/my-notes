# Java--随机数

`Java`中常用的产生随机数的方法有：`Random`、`ThreadLocalRandom`和`UUID`。它们的作用如下：

- `Random`用来产生一个随机数，但是是伪随机的，因为通过相同的`seed`，产生的随机数是相同的。
- `ThreadLocalRamdom`可以解决`Random`中多个线程竞争同一个`seed`的情况，每个线程都存有一份独立的`seed`。
- `UUID`含义是通用唯一识别码 `(Universally Unique Identifier)`，目的是让分布式系统中的所有元素都能有唯一的辨识资讯，而不需要透过中央控制端来做辨识资讯的指定。

它们的具体原理和使用如下。

## Random
Random的简单用法如下：

```java
public class RandomTest {
    public static void main(String[] args) {

        //(1)创建一个默认种子的随机数生成器
        Random random = new Random();
        //(2)输出10个在0-5（包含0，不包含5）之间的随机数
        for (int i = 0; i < 10; ++i) {
            System.out.println(random.nextInt(5));
        }
    }
}
```
Random生成随机数时需要一个默认的“种子”(seed)，这是一个long类型的数字，可以在构造函数中指定。之后便可以使用nextInt()方法生成某个范围内的随机数。该方法的源码如下：

```java
public int nextInt(int bound) {
    //(3)参数检查
    if (bound <= 0)
        throw new IllegalArgumentException(BadBound);
    //(4)根据老的种子生成新的种子
    int r = next(31);
    //(5)根据新的种子计算随机数
    ...
    return r;
} 
```
可以看到生成随机数需要两个步骤：

- 首先需要根据老的种子生成新的种子。
- 然后根据新的种子来计算新的随机数。

上面的步骤(4)和步骤(5)可以看成是固定的函数，也就是说，如果多个线程拿到的`seed`是一样的，那么计算出来的随机数也是一样的，这是“多线程不安全”。为了解决这个问题，Java使用了一个AtomicLong类型的变量并配合CAS算法来解决。next()方法的源码如下：

```java
protected int next(int bits) {
    long oldseed, nextseed;
    AtomicLong seed = this.seed;
    do {
        //(6)
        oldseed = seed.get();
        //(7)
        nextseed = (oldseed * multiplier + addend) & mask;
        //(8)
    } while (!seed.compareAndSet(oldseed, nextseed));
    //(9)
    return (int)(nextseed >>> (48 - bits));
}
```

- 代码（6）获取当前原子变量种子的值
- 代码（7）根据当前种子值计算新的种子
- 代码（8）使用CAS操作，使用新的种子去更新老的种子，多线程下可能多个线程都同时执行到了代码（6）那么可能多个线程都拿到的当前种子的值是同一个，然后执行步骤（7）计算的新种子也都是一样的，但是步骤（8）的CAS操作会保证只有一个线程可以更新老的种子为新的，失败的线程会通过循环从新获取更新后的种子作为当前种子去计算老的种子，可见这里解决了上面提到的问题，也就保证了随机数的随机性。
- 代码（9）则使用固定算法根据新的种子计算随机数。

<strong>所以，Random是线程安全的，但是多个线程公用一个Random变量的时候可能会造成大量线程进行自旋重试，降低并发性能。</strong>

## 疑问
Random是多线程安全的，但是为什么下面的例子中，两个线程会同时产生相同的随机数？

```java
public class RandomDemo {
    private static class ProduceRandom implements Runnable {
        private Random random;
        private CountDownLatch latch;

        ProduceRandom(Random random, CountDownLatch latch) {
            this.random = random;
            this.latch = latch;
        }

        @Override
        public void run() {
            latch.countDown();
            System.out.println(">>> " + Thread.currentThread().getName() +
                    " " + random.nextInt(5));
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        final Random random = new Random();
        final CountDownLatch latch = new CountDownLatch(2);

        new Thread(new ProduceRandom(random, latch)).start();
        new Thread(new ProduceRandom(random, latch)).start();

        latch.await();
    }
}
```
多次运行上述程序，有可能产生类似下面的结果：

```java
>>> Thread-0 3
>>> Thread-1 3
```

## ThreadLocalRandom
ThreadLocalRandom能够解决多线程高并发下Random的缺陷。它的简单实用例子如下：

```java
public class RandomTest {

    public static void main(String[] args) {
        //(10)获取一个随机数生成器
        ThreadLocalRandom random =  ThreadLocalRandom.current();
        
        //(11)输出10个在0-5（包含0，不包含5）之间的随机数
        for (int i = 0; i < 10; ++i) {
            System.out.println(random.nextInt(5));
        }
    }
}
```

ThreadLocalRandom不是直接用new实例化，而是使用其静态方法current()。实用ThreadLocalRandom时，每一个线程都会存有一份独立的变量，所以不会再出现多个线程竞争同一个seed的情况，提高了效率。

## 问题
使用ThreadLocalRandom的时候也会出现多个线程产生一样随机数的情况，如下。

```java
public class ThreadLocalRandomDemo {
    private static class ProduceRandom implements Runnable {
        private CountDownLatch latch;

        ProduceRandom(CountDownLatch latch) {
            this.latch = latch;
        }

        @Override
        public void run() {
            latch.countDown();
            System.out.println(">>> " + Thread.currentThread().getName() +
                    " " + ThreadLocalRandom.current().nextInt(5));
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        final CountDownLatch latch = new CountDownLatch(2);

        new Thread(new ProduceRandom(latch)).start();
        new Thread(new ProduceRandom(latch)).start();

        latch.await();
    }
}
```

多次运行上述程序，有可能产生类似下面的结果：

```java
>>> Thread-0 3
>>> Thread-1 3
```

## UUID
UUID由32 个字母数字字符组成（不包括连字符），每一个字符是一个16进制的数字（0-f）。UUID 是有一定格式的，满足8-4-4-4-12这种格式，如:
```
6b349832-0470-4692-befd-6037b280bbc5
```

简单使用的例子如下：

```java
public class UUIDDemo {
    public static void main(String[] args) {
        String uuid = UUID.randomUUID().toString();
        System.out.println(uuid);
    }
}
```

## 参考
[并发包中ThreadLocalRandom类原理剖析](https://www.jianshu.com/p/9c2198586f9b)</br>
[Java Random、ThreadLocalRandom、UUID类中的方法应用（随机数）](https://www.cnblogs.com/jiangxifanzhouyudu/p/6659670.html?utm_source=itdadao&utm_medium=referral)</br>
[多线程下ThreadLocalRandom用法](https://www.jianshu.com/p/89dfe990295c)</br>
[java类uuid源码分析](https://blog.csdn.net/wobushixiaobailian/article/details/86065041#_94)</br>
