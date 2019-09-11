# Java并发之wait()、notify()和notifyAll()和monitor实现原理

## 简介
wait()、notify()和notifyAll()这三个方法都是Object类中的方法。wait()方法的作用是让当前线程进入等待状态，同时释放锁。notify()方法的作用是随机唤醒持有锁的单个线程。notifyAll()的作用是唤醒持有锁的所有线程。

wait()方法还有两个重载的方法，如下：

- wait(long timeout)：传入一个超时时限，在超时之后会自动唤醒，也可以在达到超时时间之前使用notify()或notifyAll()方法唤醒。
- wait(long timeout, int nanos)：和上面方法的区别是可以提供更高的精度，总的超时时间(单位为ns) = 1000000 *timeout+ nanos。

## 使用时需要注意的问题

- 由于这三个方法是属于Object类的，所以所有对象都有这三个方法。同时也是因为每个对象都有一把看不见的锁(synchronized)
- 当需要调用以上的方法的时候，一定要对竞争资源进行加锁(synchronized)，如果不加锁的话(也就是说上述三个方法一定要用在synchonized语句块或方法中)，则会报 IllegalMonitorStateException 异常。同时，要注意多个线程使用的锁必须是一把

## 例子
## 两个线程存储元素
一个线程负责向一个集合中加入10个元素，当加入到第5个的时候，用线程2打印一下通知。

```java
public class TestCache {
    private static volatile List<Object> list = new LinkedList<>();

    public void add(Object o) {
        list.add(o);
    }

    public int size() {
        return list.size();
    }
}

class Main {
    public static void main(String[] args) {
        Object o = new Object();

        TestCache c1 = new TestCache();

        Thread t1 = new Thread(new Runnable() {
            int i = 0;
            @Override
            public void run() {
                synchronized (o) {
                    while (i++ < 10) {
                        c1.add(new Object());
                        System.out.println(c1.size());
                        if (c1.size() == 5) {
                            o.notify();
                            try {
                                o.wait();
                            } catch (InterruptedException e) {
                                e.printStackTrace();
                            }
                        }
                    }
                }
            }
        });

        Thread t2 =  new Thread(new Runnable() {
            @Override
            public void run() {
                synchronized (o) {
                    try {
                        o.wait();
                        System.out.println("five elements");
                        o.notify();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        });

        t2.start();
        t1.start();
    }
}
```
打印结果如下：
```
1
2
3
4
5
five elements
6
7
8
9
10
```

## 实现原理
wait()、notify()和notifyAll()的实现原理和monitor的实现原理分不开，因为要是用这三个方法必须获得对象的monitor才行，先看一下monitor的实现原理。

### monitor实现原理
Monitor是一种同步机制，它通常被描述为一个对象，能够保证多个线程访问同一对象时是互斥的。同时提供signal机制，允许正持有“许可”的线程暂时放弃，并“通知”其他线程来获得许可。

在Java虚拟机(HotSpot)中，monitor是基于C++实现的，它的主要数据结构如下：

```cpp
  ObjectMonitor() {
    _header       = NULL;
    _count        = 0;
    _waiters      = 0,
    _recursions   = 0;
    _object       = NULL;
    _owner        = NULL;
    _WaitSet      = NULL;
    _WaitSetLock  = 0 ;
    _Responsible  = NULL ;
    _succ         = NULL ;
    _cxq          = NULL ;
    FreeNext      = NULL ;
    _EntryList    = NULL ;
    _SpinFreq     = 0 ;
    _SpinClock    = 0 ;
    OwnerIsThread = 0 ;
  }
```
其中比较关键的几个属性有：

- _owner：指向持有ObjectMonitor对象的线程
- _WaitSet：**存放处于wait状态的线程队列**
- _EntryList：**存放处于等待锁block状态的线程队列**
- _recursions：锁的重入次数
- _count：用来记录该线程获取锁的次数

当多个线程同时访问一段同步代码时，**首先会进入_EntryList队列中**，当某个线程获取到对象的monitor后进入_Owner区域并把monitor中的_owner变量设置为当前线程，同时monitor中的计数器_count加1。即获得对象锁。

若持有monitor的线程调用wait()方法，将释放当前持有的monitor，`_owner`变量恢复为null，`_count`自减1，**同时该线程进入_WaitSet集合中等待被唤醒**。若当前线程执行完毕也将释放monitor(锁)并复位变量的值，以便其他线程进入获取monitor(锁)。如下图所示。

<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/monitor.png">
</div>

可以举一个形象的例子，可以把**监视器理解为包含一个特殊的房间的建筑物，这个特殊房间同一时刻只能有一个客人（线程）**。这个房间中包含了一些数据和代码。

如果一个顾客想要进入这个特殊的房间，他首先需要在**走廊（Entry Set）**排队等待。调度器将基于某个标准（比如 FIFO）来选择排队的客户进入房间。如果，因为某些原因，该客户客户暂时因为其他事情无法脱身（线程被挂起），那么他将被送到另外一间**专门用来等待的房间（Wait Set）**，这个房间的可以可以在稍后再次进入那件特殊的房间。

**获得锁的代码如下**：

```cpp
void ATTR ObjectMonitor::enter(TRAPS) {
  Thread * const Self = THREAD ;
  void * cur ;
  //通过CAS尝试把monitor的`_owner`字段设置为当前线程
  cur = Atomic::cmpxchg_ptr (Self, &_owner, NULL) ;
  //获取锁失败
  if (cur == NULL) {         
     assert (_recursions == 0   , "invariant") ;
     assert (_owner      == Self, "invariant") ;
     // CONSIDER: set or assert OwnerIsThread == 1
     return ;
  }
  // 如果旧值和当前线程一样，说明当前线程已经持有锁，此次为重入，_recursions自增，并获得锁。
  if (cur == Self) { 
     // TODO-FIXME: check for integer overflow!  BUGID 6557169.
     _recursions ++ ;
     return ;
  }

  // 如果当前线程是第一次进入该monitor，设置_recursions为1，_owner为当前线程
  if (Self->is_lock_owned ((address)cur)) { 
    assert (_recursions == 0, "internal state error");
    _recursions = 1 ;
    // Commute owner from a thread-specific on-stack BasicLockObject address to
    // a full-fledged "Thread *".
    _owner = Self ;
    OwnerIsThread = 1 ;
    return ;
  }

  // 省略部分代码。
  // 通过自旋执行ObjectMonitor::EnterI方法等待锁的释放
  for (;;) {
    jt->set_suspend_equivalent();

    EnterI (THREAD) ;

    if (!ExitSuspendEquivalent(jt)) break ;

    _recursions = 0 ;
    _succ = NULL ;
    exit (Self) ;

    jt->java_suspend_self();
    }
}
```

流程图如下所示：

<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/lockenter.png">
</div>

**释放锁的代码如下**：

```cpp
void ATTR ObjectMonitor::exit(TRAPS) {
   Thread * Self = THREAD ;
   //如果当前线程不是Monitor的所有者
   if (THREAD != _owner) { 
    if (THREAD->is_lock_owned((address) _owner)) { // 
        // Transmute _owner from a BasicLock pointer to a Thread address.
        // We don't need to hold _mutex for this transition.
        // Non-null to Non-null is safe as long as all readers can
        // tolerate either flavor.
        assert (_recursions == 0, "invariant") ;
        _owner = THREAD ;
        _recursions = 0 ;
        OwnerIsThread = 1 ;
    } else {
        // NOTE: we need to handle unbalanced monitor enter/exit
        // in native code by throwing an exception.
        // TODO: Throw an IllegalMonitorStateException ?
        TEVENT (Exit - Throw IMSX) ;
        assert(false, "Non-balanced monitor enter/exit!");
        if (false) {
            THROW(vmSymbols::java_lang_IllegalMonitorStateException());
        }
        return;
    }
   }
   // 如果_recursions次数不为0.自减
   if (_recursions != 0) {
        _recursions--;        // this is simple recursive enter
        TEVENT (Inflated exit - recursive) ;
        return ;
   }
   //省略部分代码，根据不同的策略（由QMode指定），从cxq或EntryList中获取头节点，通过ObjectMonitor::ExitEpilog方法唤醒该节点封装的线程，唤醒操作最终由unpark完成。
}
```

流程图如下所示：

<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/lockexit.png">
</div>

### wait()、notify()和notifyAll()实现原理
#### **wait()实现**
wait()的实现是`void ObjectMonitor::wait(jlong millis, bool interruptible, TRAPS)`函数。它的主要内容如下：

```cpp
void ObjectMonitor::wait(jlong millis, bool interruptible, TRAPS) {
   Thread * const Self = THREAD ;
   assert(Self->is_Java_thread(), "Must be Java thread!");
   JavaThread *jt = (JavaThread *)THREAD;

   DeferredInitialize () ;

   // Throw IMSX or IEX.
   CHECK_OWNER();

   // check for a pending interrupt
   if (interruptible && Thread::is_interrupted(Self, true) && !HAS_PENDING_EXCEPTION) {
     // post monitor waited event.  Note that this is past-tense, we are done waiting.
     if (JvmtiExport::should_post_monitor_waited()) {
        // Note: 'false' parameter is passed here because the
        // wait was not timed out due to thread interrupt.
        JvmtiExport::post_monitor_waited(jt, this, false);
     }
     TEVENT (Wait - Throw IEX) ;
     THROW(vmSymbols::java_lang_InterruptedException());
     return ;
   }
   TEVENT (Wait) ;

   assert (Self->_Stalled == 0, "invariant") ;
   Self->_Stalled = intptr_t(this) ;
   jt->set_current_waiting_monitor(this);

   // create a node to be put into the queue
   // Critically, after we reset() the event but prior to park(), we must check
   // for a pending interrupt.
   ObjectWaiter node(Self);
   node.TState = ObjectWaiter::TS_WAIT ;
   Self->_ParkEvent->reset() ;
   OrderAccess::fence();          // ST into Event; membar ; LD interrupted-flag

   Thread::SpinAcquire (&_WaitSetLock, "WaitSet - add") ;
   AddWaiter (&node) ;           // 1
   Thread::SpinRelease (&_WaitSetLock) ;
   ...
}
```
关键代码在上面的`1`处，通过AddWaiter方法将node添加到_WaitSet列表中，关键代码如下：

```cpp
inline void ObjectMonitor::AddWaiter(ObjectWaiter* node) {
  assert(node != NULL, "should not dequeue NULL node");
  assert(node->_prev == NULL, "node already in list");
  assert(node->_next == NULL, "node already in list");
  // put node at end of queue (circular doubly linked list)
  if (_WaitSet == NULL) {
    _WaitSet = node;
    node->_prev = node;
    node->_next = node;
  } else {
    ObjectWaiter* head = _WaitSet ;
    ObjectWaiter* tail = head->_prev;
    assert(tail->_next == head, "invariant check");
    tail->_next = node;
    head->_prev = node;
    node->_next = head;
    node->_prev = tail;
  }
}
```

最终是使用park()方法将线程挂起。

#### **notify()实现**
notify()的实现是`void ObjectMonitor::notify(TRAPS) `函数，关键代码如下：

```cpp
void ObjectMonitor::notify(TRAPS) {
  CHECK_OWNER();
  if (_WaitSet == NULL) {
     TEVENT (Empty-Notify) ;
     return ;
  }
  DTRACE_MONITOR_PROBE(notify, this, object(), THREAD);

  int Policy = Knob_MoveNotifyee ;

  Thread::SpinAcquire (&_WaitSetLock, "WaitSet - notify") ;
  ObjectWaiter * iterator = DequeueWaiter() ;  // 1
  ...
```
主要的是通过上面`1`处的`DequeueWaiter()`方法实现的。此方法的主要逻辑和代码如下：

- 如果当前_WaitSet为空，即没有正在等待的线程，则直接返回；
- 通过ObjectMonitor::DequeueWaiter方法，获取_WaitSet列表中的第一个ObjectWaiter节点。**需要注意的是，在jdk的notify方法注释是随机唤醒一个线程，其实是第一个ObjectWaiter节点**。

```cpp
inline ObjectWaiter* ObjectMonitor::DequeueWaiter() {
  // dequeue the very first waiter
  ObjectWaiter* waiter = _WaitSet;
  if (waiter) {
    DequeueSpecificWaiter(waiter);
  }
  return waiter;
}

inline void ObjectMonitor::DequeueSpecificWaiter(ObjectWaiter* node) {
  assert(node != NULL, "should not dequeue NULL node");
  assert(node->_prev != NULL, "node already removed from list");
  assert(node->_next != NULL, "node already removed from list");
  // when the waiter has woken up because of interrupt,
  // timeout or other spurious wake-up, dequeue the
  // waiter from waiting list
  ObjectWaiter* next = node->_next;
  if (next == node) {
    assert(node->_prev == node, "invariant check");
    _WaitSet = NULL;
  } else {
    ObjectWaiter* prev = node->_prev;
    assert(prev->_next == node, "invariant check");
    assert(next->_prev == node, "invariant check");
    next->_prev = prev;
    prev->_next = next;
    if (_WaitSet == node) {
      _WaitSet = next;
    }
  }
  node->_next = NULL;
  node->_prev = NULL;
}
```
之后在notify()方法中根据不同的策略，将取出来的ObjectWaiter节点，加入到_EntryList或则通过Atomic::cmpxchg_ptr指令进行自旋操作cxq。

#### **notifyALL()实现**
notifyAll()方法通过`void ObjectMonitor::notifyAll(TRAPS)`实现，和上面的notify()方法类似，通过for循环取出_WaitSet的ObjectWaiter节点，并根据不同策略，加入到_EntryList或则进行自旋操作。

## 参考
[深入理解多线程（四）—— Moniter的实现原理](https://www.hollischuang.com/archives/2030)</br>
[JVM源码分析之Object.wait/notify实现](https://www.jianshu.com/p/f4454164c017)</br>
[openjdk-mirror/jdk7u-hotspot/src/share/vm/runtime/objectMonitor.cpp](https://github.com/openjdk-mirror/jdk7u-hotspot/blob/50bdefc3afe944ca74c3093e7448d6b889cd20d1/src/share/vm/runtime/objectMonitor.cpp)</br>