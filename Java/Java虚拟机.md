# Java虚拟机
---
# 一、运行时数据区域
<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/jvm1.png" width="500">
</div>

## 程序计数器

程序计数器是一块较小的内存空间，他可以看做是当前线程所执行的字节码的行号指示器。

如果线程正在执行的是一个Java方法，这个计数器记录的是正在执行的虚拟机字节码指令的地址；如果正在执行的是一个Native方法，则这个计数器值为空（Undefined）。此内存区域是唯一一个在Java虚拟机规范中没有规定任何OutOfMemoryError情况的区域。

## Java虚拟机栈

每个方法在运行的时候都会创建一个栈帧（Stack Frame，栈帧是方法运行时的基础数据结构）用于存储局部变量表、操作数栈、动态链接和方法出口等信息。每一个方法从调用到执行结束的过程，对应着一个栈帧在虚拟机栈中入栈到出栈 的过程。通常所说的Java栈内存，都是指的Java虚拟机栈，或者说虚拟机栈中局部变量表部分。

局部变量表存放了编译时期可知的各种基本数据类型（boolean、byte、char、short、int、float、long、double）、对象引用（reference类型，它不等同与对象本身，可能是一个执行对象地址的引用指针，也可能是指向一个代表对象的句柄或者其他与此对象相关的位置）和returnAddress类型（指向了一条字节码指令的地址）。
<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/jvm2.png" width="500">
</div>

可以通过 -Xss 这个虚拟机参数来指定每个线程的 Java 虚拟机栈内存大小：
> java -Xss512M HackTheJava


该区域可能抛出以下异常：

- 当线程请求的栈深度超过设置的最大值，会抛出 StackOverflowError 异常；
- 如果虚拟机栈可以动态扩展（当前大部分的 Java 虚拟机都可动态扩展，只不过 Java 虚拟机规范中也允许固定长度的虚拟机栈），栈进行动态扩展时如果无法申请到足够内存，会抛出 OutOfMemoryError 异常。

## 本地方法栈

本地方法栈与 Java 虚拟机栈类似，它们之间的区别只不过是本地方法栈为本地方法服务。

本地方法一般是用其它语言（C、C++ 或汇编语言等）编写的，并且被编译为基于本机硬件和操作系统的程序，对待这些方法需要特别处理。
<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/jvm3.jpg" width="500">
</div>


与虚拟机栈一样，本地方法栈区域也会抛出StackOverflowError和OutOfMemoryError异常。


***补充:关于native关键字和本地方法***

*被native关键字修饰的方法叫做本地方法，本地方法和其它方法不一样，本地方法意味着和平台有关，因此使用了native的程序可移植性都不太高。native方法主要用于加载文件和动态链接库，由于Java语言无法访问操作系统底层信息（比如：底层硬件设备等），这时候就需要借助C语言来完成了。被native修饰的方法可以被C语言重写。*

## Java堆

Java虚拟机规范中有如下描述：所有的对象实例以及数组都要在堆上分配。但是随着**JIT编译器**的发展与**逃逸分析技术**的逐渐成熟，**栈上分配**、**标量替换优化技术**将会导致一些微妙的变化，所有的对象都分配在堆上也渐渐变得不是那么绝对了。

Java对是垃圾收集器管理的主要区域，现代的垃圾收集器基本都是采用分代收集算法，其主要的思想是针对不同类型的对象采取不同的垃圾回收算法，可以将堆分成两块：

- 新生代（Young Generation）
- 老年代（Old Generation）


再细致一点的还有Eden空间、From Survivor空间、To Survivor空间等。从内存分配的角度来看，线程共享的Java堆空间可能划分出许多个线程私有的分配缓冲区（Thread Local Allocation Buffer, TLAB）。但是不管怎么划分，都是为了更快地分配内存或更好地回收内存。

根据Java虚拟机规范的规定，Java堆可以处于物理上不连续的内存空间中，只要逻辑上是连续的即可。**如果在堆中没有内存完成实例分配，并且堆也无法再扩展时，将会抛出OutOfMemoryError异常。**

可以通过 -Xms 和 -Xmx 两个虚拟机参数来指定一个程序的堆内存大小，第一个参数设置初始值，第二个参数设置最大值。
> java -Xms1M -Xmx2M HackTheJava

## 方法区

用于存放已被加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。虽然Java虚拟机规范把方法区描述为堆的一个逻辑部分，但是它却又一个别名叫做Non-Heap（非堆），目的应该是与Java堆分开。

Java虚拟机规范对方法区的限制非常宽松，除了和Java堆一样不需要连续的内存和选择固定大小或者可扩展外(动态扩展失败一样会抛出 OutOfMemoryError 异常)，还可以选择不实现垃圾收集。

对这块区域进行垃圾回收的主要目标是**对常量池的回收**和**对类型的卸载**，但是一般比较难实现。

HotSpot 虚拟机把它当成永久代来进行垃圾回收。但是很难确定永久代的大小，因为它受到很多因素影响，并且每次 Full GC 之后永久代的大小都会改变，所以经常会抛出 OutOfMemoryError 异常。为了更容易管理方法区，从 JDK 1.8 开始，移除永久代，并把方法区移至**元空间**，它位于本地内存中，而不是虚拟机内存中。

## 运行时常量池

运行时常量池是方法区的一部分。Class 文件中的常量池（编译器生成的各种字面量和符号引用）会在类加载后被放入这个区域。

Java虚拟机对Class文件的每一部分(包括常量池)的格式都有严格的规定，每一个字节用于存储哪种数据都必须符合规范上的要求才能被虚拟机认可、装载和执行，但是对于运行时常量池，Java虚拟机规范没有做任何细节的要求，不同提供商时间的虚拟机可以安好自己的需求来实现这个内存区域。不过，一般来说，除了保存Class文件中描述的**符号引用**外，还会把翻译出来的**直接引用**也从存储在运行时常量池中。

运行时常量池相对于Class文件常量池的另外一个重要特性是具备动态性，除了在编译期生成的常量，还允许动态生成，例如 String 类的 intern()。

当常量池无法再申请到内存时会抛出OutOfMemoryError异常。

常量池中可以存储的信息具体如下图所示：

<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/constant_pool.jpg" width="500" height="500">
</div>

更加详细的可以看[这里](https://blog.csdn.net/wangtaomtk/article/details/52267548)。需要注意的是，静态变量在方法区中，但是不是在运行时常量池中。


***补充：[关于class文件](https://www.cnblogs.com/yaoyinglong/p/JavaClass.html)***


***补充：符号引用和直接引用***

在JVM中，类从被加载到虚拟机内存中开始，到卸载出内存为止，它的整个生命周期包括：加载、验证、准备、解析、初始化、使用和卸载7个阶段。而解析阶段即是虚拟机将常量池内的符号引用替换为直接引用的过程。

- 符号引用（Symbolic References）：符号引用以一组符号来描述所引用的目标，符号可以是任何形式的字面量，只要使用时能够无歧义的定位到目标即可。例如，在Class文件中它以CONSTANT_Class_info、CONSTANT_Fieldref_info、CONSTANT_Methodref_info等类型的常量出现。符号引用与虚拟机的内存布局无关，引用的目标并不一定加载到内存中。在Java中，一个java类将会编译成一个class文件。在编译时，java类并不知道所引用的类的实际地址，因此只能使用符号引用来代替。比如org.simple.People类引用了org.simple.Language类，在编译时People类并不知道Language类的实际内存地址，因此只能使用符号org.simple.Language（假设是这个，当然实际中是由类似于CONSTANT_Class_info的常量来表示的）来表示Language类的地址。各种虚拟机实现的内存布局可能有所不同，但是它们能接受的符号引用都是一致的，因为符号引用的字面量形式明确定义在Java虚拟机规范的Class文件格式中。
- 直接引用：
直接引用可以是
    - 直接指向目标的指针（比如，指向“类型”【Class对象】、类变量、类方法的直接引用可能是指向方法区的指针）
    - 相对偏移量（比如，指向实例变量、实例方法的直接引用都是偏移量）
    - 一个能间接定位到目标的句柄

直接引用是和虚拟机的布局相关的，同一个符号引用在不同的虚拟机实例上翻译出来的直接引用一般不会相同。如果有了直接引用，那引用的目标必定已经被加载入内存中了。

---
小结：关于**字符串常量池**、**运行时常量池**、**方法区**和**移除永久代**的一些理解

### **方法区和永久代的区别**
**方法区**（Method Area）是jvm规范里面的运行时数据区的一个组成部分，注意，它只是一种规范，并不是实际的实现。

**永久代**又叫Perm区，只存在于hotspot jvm中(原因是hotspot设计团队选择把GC分带收集扩展到方法区，或者说使用永久代来实现方法区)，并且只存在于jdk7和之前的版本中，jdk8中已经彻底移除了永久代，jdk8中引入了一个新的内存区域叫metaspace。

因此，可以说，**永久代是方法区的一种实现**，当然，在hotspot jdk8中metaspace可以看成是方法区的一种实现。

### **字符串常量池在运行时常量池中吗**
以下还是针对hotspot虚拟机来说。

在JDK1.6及之前版本，字符串常量池是放在Perm Gen区(也就是方法区)中，**此时的字符串常量池是运行时常量池的物理上的一部分**；

在JDK1.7版本，开始了**逐步**移除永久代事件，字符串常量池被移到了堆中了，**但是此时字符串常量池还是被认为是运行时常量池逻辑上的一部分**。

到了JDK1.8版本，永久代已经被完全移除，取而代之的是**元空间**。**此时，原先存放在方法区的class文件信息被放到元空间，而原先存放在方法区的运行时常量池被放到了堆中**。

### 参考
[java jdk1.7常量池移到哪去了？](https://blog.csdn.net/u014039577/article/details/50377805/)</br>
[字符串常量池、class常量池和运行时常量池](https://blog.csdn.net/qq_26222859/article/details/73135660)</br>
[JVM-String常量池与运行时常量池](https://blog.csdn.net/Sugar_Rainbow/article/details/68150249)</br>
[Java中的常量池(字符串常量池、class常量池和运行时常量池)](https://blog.csdn.net/zm13007310400/article/details/77534349)</br>
[JVM的方法区和永久带是什么关系？](https://blog.csdn.net/goldenfish1919/article/details/81216560)</br>
[jdk1.6 1.7 1.8 运行时常量池位置的变化](https://blog.csdn.net/ychenfeng/article/details/77413206)</br>

---

## StackOverflowError和OutOfMemoryError
首先看一下StackOverflowError和OutOfMemoryError的区别：
### **stackoverflow：**
每当java程序启动一个新的线程时，java虚拟机会为他分配一个栈，java栈以帧为单位保持线程运行状态；当线程调用一个方法是，jvm压入一个新的栈帧到这个线程的栈中，只要这个方法还没返回，这个栈帧就存在。 

如果方法的嵌套调用层次太多(如递归调用),随着java栈中的帧的增多，最终导致这个线程的栈中的所有栈帧的大小的总和大于-Xss设置的值，而产生StackOverflowError溢出异常。

### **outofmemory：**
#### **栈内存溢出**

java程序启动一个新线程时，没有足够的空间为该线程分配java栈，一个线程java栈的大小由-Xss设置决定；JVM则抛出OutOfMemoryError异常。

#### **堆内存溢出**

java堆用于存放对象的实例，当需要为对象的实例分配内存时，而堆的占用已经达到了设置的最大值(通过-Xmx设置最大值)，则抛出OutOfMemoryError异常。

#### **方法区内存溢出**

方法区用于存放java类的相关信息，如类名、访问修饰符、常量池、字段描述、方法描述等。在类加载器加载class文件到内存中的时候，JVM会提取其中的类信息，并将这些类信息放到方法区中。

当需要存储这些类信息，而方法区的内存占用又已经达到最大值（通过-XX:MaxPermSize）；将会抛出OutOfMemoryError异常。

**也就是说，StackOverflowError只发生在栈区，原因是线程的请求深度大于所允许的栈深度；OutOfMemoryError可以发生在堆、栈和方法区，原因是申请的内存大于虚拟机所允许的最大值。**

参考[java stackoverflowerror与outofmemoryerror区别](https://blog.csdn.net/chenchaofuck1/article/details/51144223)

### Java堆溢出

堆里放的是new出来的对象，所以这部分很简单不断的new对象就可以了，但是为了防止对象new出来之后被GC，所以把对象new出来的对象放到一个List中去即可。为了有更好的效果，可以在运行前，调整堆的参数。
```java
import java.util.ArrayList;
import java.util.List;

/**
 * -Xms20m -Xms20m -XX:+HeapDumpOnOutOfMemoryError
 */
public class HeapOOM {
    static class OOMObject{

    }

    public static void main(String[] args) {
        List<OOMObject> list = new ArrayList<>();

        while (true)
            list.add(new OOMObject());
    }
}
```

运行结果：
```java
java.lang.OutOfMemoryError: Java heap space
Dumping heap to java_pid9944.hprof ...
```

### 虚拟机栈和本地方法栈溢出
- 在单线程的堆中我们不断的让一个成员变量自增，容纳这个变量的单元无法承受这个变量了，就抛出StackOverflowError了。
- 可以开尽量多的线程，并在每个线程里调用native的方法，就自然会抛出 OutOfMemoryError了。
```java
/**
 * VM Args: - Xss128k
 */
public class JavaVMStackOF {
    private int stackLength = 1;
    public void stackLeak(){
        stackLength++;
        stackLeak();
    }

    public static void main(String[] args) throws Throwable {
        JavaVMStackOF oom = new JavaVMStackOF();
        try {
            oom.stackLeak();
        }catch (Throwable e){
            System.out.println("stack length:" + oom.stackLength);
            throw e;
        }
    }
}
```

运行结果：
```java
stack length:992
Exception in thread "main" java.lang.StackOverflowError
	at JVM.StackOF.JavaVMStackOF.stackLeak(JavaVMStackOF.java:9)
	at JVM.StackOF.JavaVMStackOF.stackLeak(JavaVMStackOF.java:10)
	at JVM.StackOF.JavaVMStackOF.stackLeak(JavaVMStackOF.java:10)
	at JVM.StackOF.JavaVMStackOF.stackLeak(JavaVMStackOF.java:10)
	...
```

```java
/**
 * VM args: -Xss2m
 */
public class JavaVMStackOOM {
    private void dontStop(){
        while (true){

        }
    }

    public void stackLeakByThread(){
        while (true){
            Thread thread = new Thread(new Runnable() {
                @Override
                public void run() {
                    dontStop();
                }
            });
            thread.start();
        }
    }

    public static void main(String[] args) {
        JavaVMStackOOM oom = new JavaVMStackOOM();
        oom.stackLeakByThread();
    }
}
```

*注意：在运行上述代码时要先保存当前工作，因为在Windows平台上的虚拟机中，Java的线程是映射到操作系统的内核线程上的，因此上述代码可能导致系统假死。*

执行结果为：
```java
Exception in thread "main" java.long.OutOfMemoryError:unable to create new native thread.
```

### 方法区和运行时常量池溢出

**运行时常量池溢出：** intern()方法，这是一个Native方法，作用是：如果字符串常量池中已经包含一个等于此String对象的字符串，则返回池中的该对象；否则，将此String对象包含的字符串添加到常量池中，并返回此String对象的引用。


由于在jdk1.6以及之前的版本中，运行时常量池被放在永久代中，所以可以使用-XX:PermSize和-XX:MaxPermSize限制大小；但是在jdk1.6之后的版本，已将开始“去永久代”，所以以下程序在jdk1.6之前和之后的版本上运行时会得到不同的结果。
```java
/**
 * -XX:PermSize=10m -XX:MaxPermSize=10m
 */
public class RuntimeConstantPoolOOM {
    public static void main(String[] args) {
        //使用List保持着常量池的引用，避免Full GC回收常量池行为
        List<String> list = new ArrayList<>();
        int i = 0;
        while (true){
            list.add(String.valueOf(i++).intern());
        }
    }
}
```

上述代码在jdk1.6以及之前的版本中运行的结果为：
```java
Exception in thread "main" java.lang.OutOfMemoryError: PermGen space
      at java.lang.String.intern(   Native Method  )
      at RuntimeConstantPoolOOM.main(   RuntimeConstantPoolOOM.java:14   )
```

但是在jdk1.7以及之后的版本上运行时，while循环会一直进行下去。


**方法区溢出：**

注意：使用Enhancer时需要一个jar文件：cglib-nodep-2.1_3.jar包。[下载链接](https://www.java2s.com/Code/Jar/c/Downloadcglibnodep213jar.htm)。
```java
import net.sf.cglib.proxy.Enhancer;
import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;

import java.lang.reflect.Method;

/**
 * VM args:-XX:PermSize=10m -XX:MaxPermSize=10m
 */
public class JavaMethodAreaOOM {
    static class OOMObject{

    }

    public static void main(String[] args) {
        while (true){
            Enhancer enhancer = new Enhancer();
            enhancer.setSuperclass(OOMObject.class);
            enhancer.setCallback(new MethodInterceptor() {
                @Override
                public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
                    return methodProxy.invokeSuper(o, args);
                }
            });
            enhancer.create();
        }
    }
}
```

# 二、垃圾收集

垃圾收集主要是针对堆和方法区进行。


程序计数器、虚拟机栈和本地方法栈这三个区域属于线程私有的，只存在于线程的生命周期内，线程结束之后也会消失，因此不需要对这三个区域进行垃圾回收。
## 判断一个对象是否可被回收
### 1. 引用计数算法

给对象添加一个引用计数器，当对象增加一个引用时计数器加 1，引用失效时计数器减 1。引用计数为 0 的对象可被回收。


但是两个对象出现循环引用的情况下，此时引用计数器永远不为 0，导致无法对它们进行回收。


正因为循环引用的存在，因此 Java 虚拟机不使用引用计数算法。
```java
public class ReferenceCountingGC {
    public Object instance = null;

    public static void main(String[] args) {
        ReferenceCountingGC objA = new ReferenceCountingGC();
        ReferenceCountingGC objB = new ReferenceCountingGC();

        objA.instance = objB;
        objB.instance = objA;
        
        objA = null;
        objB = null;
        
        //假如在这时候进行垃圾回收，objA和objB依然会被回收，这就说明JVM不是依靠引用计数算法来
        //判断对象是否存活的。
        System.gc();
    }
}
```
### 2. 可达性分析算法

这个算法的基本思想是，通过一系列的称为“GC Roots”的对象作为起始点，从这些节点开始向下搜索，搜索所走过的路径称为引用链(Reference Chain)，当一个对象到GC Roots没有任何引用链相连，就是说从GC Roots到这个对象不可达，则证明此对象是不可用的，可以进行回收。


更直白一点，Tracing GC就是遍历一张对象关系图：给定一个集合的引用作为根出发，通过引用关系遍历对象图，能被遍历到的（可到达的）对象就被判定为存活，其余对象（也就是没有被遍历到的）就自然被判定为死亡。

注意：Tracing GC的本质是通过找出所有活对象来把其余空间认定为“无用”，而不是找出所有死掉的对象并回收它们占用的空间。GC Roots这组引用是Tracing GC的起点。要实现语义正确的Tracing GC，就必须要能完整枚举出所有的GC Roots，否则就可能会漏扫描应该存活的对象，导致GC错误回收了这些被漏扫的活对象。


在Java中可以作为GC Roots的对象包括以下几种：
> - 虚拟机栈中局部变量表中引用的对象
> - 本地方法栈中 JNI(Java Native Interface，即一般所说的Native方法) 中引用的对象
> - 方法区中类静态属性引用的对象
> - 方法区中的常量引用的对象

**验证虚拟机栈（栈帧中的局部变量）中引用的对象 作为GC Roots**
```java
/**
 * GCRoots 测试：虚拟机栈（栈帧中的局部变量）中引用的对象作为GCRoots
 * -Xms1024m -Xmx1024m -Xmn512m -XX:+PrintGCDetails
 *
 * 扩展：虚拟机栈中存放了编译器可知的八种基本数据类型,对象引用,returnAddress类型（指向了一条字节码指令的地址）
 */

public class TestGCRoots01 {

    private int _10MB = 10 * 1024 * 1024;
    private byte[] memory = new byte[8 * _10MB];

    public static void main(String[] args) {
        method01();
        System.out.println("返回main方法");
        System.gc();
        System.out.println("第二次GC完成");
    }

    public static void method01() {
        TestGCRoots01 t = new TestGCRoots01();
        System.gc();
        System.out.println("第一次GC完成");
    }
}
```

控制台打印日志：
```java
[GC (System.gc()) [PSYoungGen: 97648K->840K(458752K)] 97648K->82768K(983040K), 0.0587865 secs] [Times: user=0.13 sys=0.03, real=0.06 secs] 
[Full GC (System.gc()) [PSYoungGen: 840K->0K(458752K)] [ParOldGen: 81928K->82571K(524288K)] 82768K->82571K(983040K), [Metaspace: 3440K->3440K(1056768K)], 0.0298338 secs] [Times: user=0.00 sys=0.00, real=0.03 secs] 
第一次GC完成
返回main方法
[GC (System.gc()) [PSYoungGen: 7864K->160K(458752K)] 90436K->82731K(983040K), 0.0007510 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (System.gc()) [PSYoungGen: 160K->0K(458752K)] [ParOldGen: 82571K->647K(524288K)] 82731K->647K(983040K), [Metaspace: 3441K->3441K(1056768K)], 0.0037680 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
第二次GC完成
Heap
 PSYoungGen      total 458752K, used 23593K [0x00000000e0000000, 0x0000000100000000, 0x0000000100000000)
  eden space 393216K, 6% used [0x00000000e0000000,0x00000000e170a568,0x00000000f8000000)
  from space 65536K, 0% used [0x00000000fc000000,0x00000000fc000000,0x0000000100000000)
  to   space 65536K, 0% used [0x00000000f8000000,0x00000000f8000000,0x00000000fc000000)
 ParOldGen       total 524288K, used 647K [0x00000000c0000000, 0x00000000e0000000, 0x00000000e0000000)
  object space 524288K, 0% used [0x00000000c0000000,0x00000000c00a1e38,0x00000000e0000000)
 Metaspace       used 3448K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 376K, capacity 388K, committed 512K, reserved 1048576K
```
- 第一次GC：t为局部变量，引用了new出的对象（80M），作为GC Roots，在Minor GC后被转移到老年代中，且Full GC也不会回收该对象，仍保留在老年代中。
- 第二次GC：method01方法执行完后，局部变量t跟随方法消失，不再有引用类型指向该对象，该对象在Full GC后，被完全回收，老年代腾出该对象之前所占的空间。


**测试方法区中的静态变量引用的对象作为GCRoots**
```java
/**
 * 测试方法区中的静态变量引用的对象作为GCRoots
 * -Xms1024m -Xmx1024m -Xmn512m -XX:+PrintGCDetails
 *
 * 扩展：方法区存与堆一样,是各个线程共享的内存区域,用于存放已被虚拟机加载的类信息,常量,静态变量,即时编译器编译后的代码等数据。
 * @author ljl
 * */

public class TestGCRoots02 {
    private static int _10MB = 10 * 1024 * 1024;
    private byte[] memory;
    private static TestGCRoots02 t;

    public TestGCRoots02(int size) {
        memory = new byte[size];
    }

    public static void main(String[] args) {
        TestGCRoots02 t2 = new TestGCRoots02(4 * _10MB);
        t2.t = new TestGCRoots02(8 * _10MB);
        t2 = null;
        System.gc();
    }
}
```

控制台打印日志为：
```java
[GC (System.gc()) [PSYoungGen: 138608K->776K(458752K)] 138608K->82704K(983040K), 0.0445024 secs] [Times: user=0.16 sys=0.02, real=0.04 secs] 
[Full GC (System.gc()) [PSYoungGen: 776K->0K(458752K)] [ParOldGen: 81928K->82571K(524288K)] 82704K->82571K(983040K), [Metaspace: 3439K->3439K(1056768K)], 0.0305056 secs] [Times: user=0.08 sys=0.00, real=0.03 secs] 
Heap
 PSYoungGen      total 458752K, used 3932K [0x00000000e0000000, 0x0000000100000000, 0x0000000100000000)
  eden space 393216K, 1% used [0x00000000e0000000,0x00000000e03d7218,0x00000000f8000000)
  from space 65536K, 0% used [0x00000000f8000000,0x00000000f8000000,0x00000000fc000000)
  to   space 65536K, 0% used [0x00000000fc000000,0x00000000fc000000,0x0000000100000000)
 ParOldGen       total 524288K, used 82571K [0x00000000c0000000, 0x00000000e0000000, 0x00000000e0000000)
  object space 524288K, 15% used [0x00000000c0000000,0x00000000c50a2f18,0x00000000e0000000)
 Metaspace       used 3446K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 376K, capacity 388K, committed 512K, reserved 1048576K
```
- t2被置为null，Minor GC后t2之前引用的对象（40M）被完全回收；t为静态变量，存放于方法区中，引用了对象（80M），在Minor GC后，被转移到老年代中，且在Full GC后，也不会被回收，继续保留在老年代中。 

**验证方法区中常量引用对象作为GC Roots**
```java
/**
 * 测试常量引用对象作为GCRoots
 * 注意：t修饰符如果只是final会被回收，static final不会被回收，所以static final 才是常量的正确写法
 * -Xms1024m -Xmx1024m -Xmn512m -XX:+PrintGCDetails
 */

public class TestGCRoots03 {

    private static int _10MB = 10 * 1024 * 1024;
    private static final TestGCRoots03 t = new TestGCRoots03(8 * _10MB);
    private byte[] memory;

    public TestGCRoots03(int size) {
        memory = new byte[size];
    }

    public static void main(String[] args) {
        TestGCRoots03 t3 = new TestGCRoots03(4 * _10MB);
        t3 = null;
        System.gc();
    }
}
```

控制台打印信息如下：
```java
[GC (System.gc()) [PSYoungGen: 138608K->808K(458752K)] 138608K->82736K(983040K), 0.0538510 secs] [Times: user=0.20 sys=0.03, real=0.05 secs] 
[Full GC (System.gc()) [PSYoungGen: 808K->0K(458752K)] [ParOldGen: 81928K->82571K(524288K)] 82736K->82571K(983040K), [Metaspace: 3439K->3439K(1056768K)], 0.0316808 secs] [Times: user=0.11 sys=0.00, real=0.03 secs] 
Heap
 PSYoungGen      total 458752K, used 3932K [0x00000000e0000000, 0x0000000100000000, 0x0000000100000000)
  eden space 393216K, 1% used [0x00000000e0000000,0x00000000e03d7218,0x00000000f8000000)
  from space 65536K, 0% used [0x00000000f8000000,0x00000000f8000000,0x00000000fc000000)
  to   space 65536K, 0% used [0x00000000fc000000,0x00000000fc000000,0x0000000100000000)
 ParOldGen       total 524288K, used 82571K [0x00000000c0000000, 0x00000000e0000000, 0x00000000e0000000)
  object space 524288K, 15% used [0x00000000c0000000,0x00000000c50a2f18,0x00000000e0000000)
 Metaspace       used 3446K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 376K, capacity 388K, committed 512K, reserved 1048576K
```
- t3被置为null，Minor GC后t3之前引用的对象（40M）被完全回收；t为常量，存放于方法区中，引用了对象（80M），在Minor GC后，被转移到老年代中，且在Full GC后，也不会被回收，继续保留在老年代中。

**测试成员变量引用对象是否可作为GCRoots**
```java
/**
 * 测试成员变量引用对象是否可作为GCRoots
 * -Xms1024m -Xmx1024m -Xmn512m -XX:+PrintGCDetails
 */

public class TestGCRoots04 {
    private static int _10MB = 10 * 1024 * 1024;
    private TestGCRoots04 t;
    private byte[] memory;

    public TestGCRoots04(int size) {
        memory = new byte[size];
    }

    public static void main(String[] args) {
        TestGCRoots04 t4 = new TestGCRoots04(4 * _10MB);
        t4.t = new TestGCRoots04(8 * _10MB);
        t4 = null;
        System.gc();
    }
}
```

控制台打印结果为：
```java
[GC (System.gc()) [PSYoungGen: 138608K->840K(458752K)] 138608K->848K(983040K), 0.0008446 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (System.gc()) [PSYoungGen: 840K->0K(458752K)] [ParOldGen: 8K->651K(524288K)] 848K->651K(983040K), [Metaspace: 3439K->3439K(1056768K)], 0.0040585 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
Heap
 PSYoungGen      total 458752K, used 3932K [0x00000000e0000000, 0x0000000100000000, 0x0000000100000000)
  eden space 393216K, 1% used [0x00000000e0000000,0x00000000e03d7218,0x00000000f8000000)
  from space 65536K, 0% used [0x00000000f8000000,0x00000000f8000000,0x00000000fc000000)
  to   space 65536K, 0% used [0x00000000fc000000,0x00000000fc000000,0x0000000100000000)
 ParOldGen       total 524288K, used 651K [0x00000000c0000000, 0x00000000e0000000, 0x00000000e0000000)
  object space 524288K, 0% used [0x00000000c0000000,0x00000000c00a2ef8,0x00000000e0000000)
 Metaspace       used 3446K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 376K, capacity 388K, committed 512K, reserved 1048576K
```
- t4被置为null，Minor GC后t4之前引用的对象（40M）被完全回收；t为成员变量，也叫实例变量，不同于类变量（静态变量），前面讲到类变量是存储在方法区中，而成员变量是存储在堆内存的对象中的，和对象共存亡，所以是不能作为GC Roots的，从日志中也可看出t在MinorGC后，跟随t4一起被完全回收。不再占用任何空间。

**小结：**

- 要知道哪些对象是可作为GC Roots的，在实际开发过程中要特别注意这些对象，不要让无谓的大对象消耗了资源，拖累了性能。

### 3. 回收方法区

因为方法区主要存放永久代对象，而永久代对象的回收率比新生代低很多，因此在方法区上进行回收性价比不高，主要是**对常量池的回收**和**对类的卸载**。

回收废弃常量和回收Java堆中的对象非常相似。以回收常量池中字面量为例，加入一个字符串“abc”已经进入了常量池，但是当前系统没有任何一个String对象是叫做“abc”的，换句话说，没有任何一个对象引用了“abc”，也没有其他任何引用了这个字面量，如果这是发生了垃圾回收，而且必要的话，这个字面量就会被清除常量池。

类的卸载条件很多，需要满足以下三个条件，并且满足了也不一定会被卸载：

> - 该类所有的实例都已经被回收，也就是堆中不存在该类的任何实例。
> - 加载该类的 ClassLoader 已经被回收。
> - 该类对应的 Class 对象没有在任何地方被引用，也就无法在任何地方通过反射访问该类方法。


可以通过 -Xnoclassgc 参数来控制是否对类进行卸载。

在大量使用反射、动态代理、CGLib 等 ByteCode 框架、动态生成 JSP 以及 OSGi 这类频繁自定义 ClassLoader 的场景都需要虚拟机具备类卸载功能，以保证不会出现内存溢出。

### 4. finalize()

**两次标记：** 一个对象从被判定为死亡对象到被垃圾收集器回收掉还要经历两次标记的过程，该过程可以认为是该对象在死刑的缓刑阶段。**第一次标记：** 当可达性分析确认该对象没有引用链与GC Roots相连，则对其进行第一次标记和筛选，筛选的条件是重写了finalize()方法并没有执行过，对于重写了且并没有执行finalize()方法的对象这将其放置在一个F-Queue队列中，并在稍后由一个由虚拟机自动建立的低优先级的Finalizer线程去执行它。此处执行只保证执行该方法，但是不保证等待该方法执行结束，之所以这样子设计是为了系统的稳定性和健壮性考虑，以免该方法执行时间较长或者死循环导致系统崩溃。在此之后，系统会对对象进行**第二次标记**，如果在第一次标记之后的对象在执行finalize()方法时没有被引用到一个新的变量，这该对象将被回收掉。


finalize() 类似 C++ 的析构函数，用来做关闭外部资源等工作。但是 try-finally 等方式可以做的更好，并且该方法运行代价高昂，不确定性大，无法保证各个对象的调用顺序，因此最好不要使用。


当一个对象可被回收时，如果需要执行该对象的 finalize() 方法，那么就有可能在该方法中让对象重新被引用，从而实现自救。自救只能进行一次，如果回收的对象之前调用了 finalize() 方法自救，后面回收时不会调用 finalize() 方法。
```java
/**
 * 下面的代码演示了两点：
 * 1. 对象可以在GC时自我拯救
 * 2. 这种自救的机会只有一次，因为一个对象的finalize()方法最多只能被系统自动调用一次
 */
public class FinalizeEscapeGC {
    public static FinalizeEscapeGC SAVE_HOOK = null;

    public void isAlive(){
        System.out.println("yes, i am still alive :)");
    }

    @Override
    protected void finalize() throws Throwable {
        super.finalize();
        System.out.println("finalize method executed!");
        //自救，把自己(this关键字)赋值给一个类变量
        FinalizeEscapeGC.SAVE_HOOK = this;
    }

    public static void main(String[] args) throws InterruptedException {
        SAVE_HOOK = new FinalizeEscapeGC();

        //对象第一次进行自救
        SAVE_HOOK = null;
        System.gc();
        //因为finalize()方法优先级很低，暂停0.5s等待
        Thread.sleep(500);
        if(SAVE_HOOK != null)
            SAVE_HOOK.isAlive();
        else
            System.out.println("no, i am dead :(");

        //与上面的代码相同，但是自救失败，因为finalize()方法只能被系统自动调用一次
        SAVE_HOOK = null;
        System.gc();
        Thread.sleep(500);
        if(SAVE_HOOK != null)
            SAVE_HOOK.isAlive();
        else
            System.out.println("no, i am dead :(");
    }
}
```

执行结果为：
```java
finalize method executed!
yes, i am still alive :)
no, i am dead :(
```

## 引用类型

无论是通过引用计算算法判断对象的引用数量，还是通过可达性分析算法判断对象是否可达，判定对象是否可被回收都与引用有关。


Java 提供了四种强度不同的引用类型：强引用(Strong Reference)、软引用(Soft Reference)、弱引用(Weak Reference)和虚引用(Phantom Reference)。

### 1. 强引用

强引用就是在程序代码中普遍存在的，只要强引用还在，垃圾收集器永远不会回收掉被引用的对象。使用 new 一个新对象的方式来创建强引用。
```java
Object obj = new Object();
```

### 2. 软引用

软引用是用来描述一些还有用但是并非必需的对象。在系统将要发生内存溢出之前，将会把这些对象回收，如果还没有足够的内存，才会抛出内存异常。使用 SoftReference 类来创建软引用。
```java
Object obj = new Object();
SoftReference<Object> sf = new SoftReference<Object>(obj);
obj = null;  // 使对象只被软引用关联
```

### 3. 弱引用

弱引用也是用来描述非必须对象的，但是它的强度比软引用更弱一些。被弱引用关联的对象一定会被回收，也就是说它只能存活到下一次垃圾回收发生之前。使用 WeakReference 类来实现弱引用。
```java
Object obj = new Object();
WeakReference<Object> wf = new WeakReference<Object>(obj);
obj = null;
```

### 4. 虚引用

又称为幽灵引用或者幻影引用。一个对象是否有虚引用的存在，完全不会对其生存时间构成影响，也无法通过虚引用取得一个对象。


为一个对象设置虚引用关联的唯一目的就是能在这个对象被回收时收到一个系统通知。


使用 PhantomReference 来实现虚引用。
```java
Object obj = new Object();
PhantomReference<Object> pf = new PhantomReference<Object>(obj);
obj = null;
```

## 垃圾收集算法
### 1. 标记-清除(Mark-Sweep)算法
<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/mark1.jpg">
</div>


基本思路是：将存活的对象进行标记，然后清理掉未被标记的对象。

不足：

- 标记和清除过程效率都不高；
- 会产生大量不连续的内存碎片，导致无法给大对象分配内存。

### 2. 复制(Copying)算法
<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/mark2.jpg">
</div>


将内存划分为大小相等的两块，每次只使用其中一块，当这一块内存用完了就将还存活的对象复制到另一块上面，然后再把使用过的内存空间进行一次清理。


主要不足是只使用了内存的一半。


现在的商业虚拟机都采用这种收集算法来回收新生代，但是并不是将新生代划分为大小相等的两块，而是分为一块较大的 Eden 空间和两块较小的 Survivor 空间，每次使用 Eden 空间和其中一块 Survivor。在回收时，将 Eden 和 Survivor 中还存活着的对象一次性复制到另一块 Survivor 空间上，最后清理 Eden 和使用过的那一块 Survivor。


HotSpot 虚拟机的 Eden 和 Survivor 的大小比例默认为 8:1，保证了内存的利用率达到 90%。如果每次回收有多于 10% 的对象存活，那么一块 Survivor 空间就不够用了，此时需要依赖于老年代进行分配担保，也就是借用老年代的空间存储放不下的对象。

### 3. 标记-整理(Mark-Compact)算法
<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/mark3.jpg">
</div>


复制收集算法在对象存活率较高的时候就要进行较多的复制，效率就会变低，所以不适合老年代。有人针对老年代的特点提出了一种“标记-整理”算法。


“标记-整理”算法和“标记-清除”算法相似，但是后续步骤不是直接对可回收对象进行整理，而是让所有存活的对象都向一端移动，然后直接清理掉端边界以外的内存。

### 4. 分代收集(Generational Collection)算法

现在的商业虚拟机采用分代收集算法，它根据对象存活周期将内存划分为几块，不同块采用适当的收集算法。


一般将堆分为新生代和老年代。

- 新生代使用：复制算法
- 老年代使用：标记 - 清除 或者 标记 - 整理 算法

## 垃圾收集器
<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/mark4.jpg">
</div>


以上是 HotSpot 虚拟机中的`7`个垃圾收集器，连线表示垃圾收集器可以配合使用。

这`7`个垃圾收集器可以按照下面的顺序记忆：

- 先记住一个`Serial`收集器
- `Serial`收集器的老年代版本`Serial Old`收集器
- `Serial`收集器的多线程版本`ParNew`收集器
- `Parallel Scavenge`收集器
- `Parallel Scavenge`收集器的老年代版本`Parallel Old`
- 两个特殊的收集器`CMS`和`G1`

- 单线程与多线程：单线程指的是垃圾收集器只使用一个线程进行收集，而多线程使用多个线程；
- 串行、并行和并发：
    - 串行(Serial)指的是垃圾收集器与用户程序交替执行，这意味着在执行垃圾收集的时候需要停顿用户程序；
    - 并行(Parallel)指的是多条垃圾收集线程并行工作，但此时用户线程仍然处于等待状态。
    - 并发(Concurrent)指的是用户线程与垃圾收集线程同时执行（但是并不一定是并行的，可能会交替执行），用户程序在继续运行，而垃圾收集程序运行与另一个CPU上。

### 1. Serial 收集器
<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/mark5.jpg">
</div>


Serial 翻译为串行，也就是说它以串行的方式执行。在垃圾收集工作开始时，必须停止所有其他的工作线程直到收集结束，叫做“Stop The World”。

它是单线程的收集器，只会使用一个线程进行垃圾收集工作。

它的优点是简单高效，对于单个 CPU 环境来说，由于没有线程交互的开销，因此拥有最高的单线程收集效率。

它是 Client 模式下的默认新生代收集器，因为在该应用场景下，分配给虚拟机管理的内存一般来说不会很大。Serial 收集器收集几十兆甚至一两百兆的新生代停顿时间可以控制在一百多毫秒以内，只要不是太频繁，这点停顿是可以接受的。

### 2.  ParNew 收集器
<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/mark6.jpg">
</div>


它是 Serial 收集器的多线程版本。

是 Server 模式下的虚拟机首选新生代收集器，除了性能原因外，主要是因为除了 Serial 收集器，只有它能与 CMS 收集器配合工作。

默认开启的线程数量与 CPU 数量相同，可以使用 -XX:ParallelGCThreads 参数来设置线程数。

### 3.  Parallel Scavenge[ˈskævɪndʒ] 收集器

与 ParNew 一样是多线程收集器。

其它收集器关注点是尽可能缩短垃圾收集时用户线程的停顿时间，而它的目标是达到一个可控制的吞吐量，它被称为“吞吐量优先”收集器。这里的吞吐量指 CPU 用于运行用户代码的时间占总时间的比值，即`吞吐量=运行用户代码时间 / (运行用户代码时间 + 垃圾收集时间)`。

停顿时间越短就越适合需要与用户交互的程序，良好的响应速度能提升用户体验。而高吞吐量则可以高效率地利用 CPU 时间，尽快完成程序的运算任务，适合在后台运算而不需要太多交互的任务。

缩短停顿时间是以牺牲吞吐量和新生代空间来换取的：新生代空间变小，垃圾回收变得频繁，导致吞吐量下降。

可以通过一个开关参数打开 GC 自适应的调节策略（GC Ergonomics），就不需要手工指定新生代的大小（-Xmn）、Eden 和 Survivor 区的比例、晋升老年代对象年龄等细节参数了。虚拟机会根据当前系统的运行情况收集性能监控信息，动态调整这些参数以提供最合适的停顿时间或者最大的吞吐量。

### 4. Serial Old 收集器
<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/mark7.jpg">
</div>


是 Serial 收集器的老年代版本，也是给 Client 模式下的虚拟机使用。如果用在 Server 模式下，它有两大用途：

- 在 JDK 1.5 以及之前版本（Parallel Old 诞生以前）中与 Parallel Scavenge 收集器搭配使用。
- 作为 CMS 收集器的后备预案，在并发收集发生 Concurrent Mode Failure 时使用。

### 5. Parallel Old 收集器
<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/mark8.jpg">
</div>


是 Parallel Scavenge 收集器的老年代版本。

在注重吞吐量以及 CPU 资源敏感的场合，都可以优先考虑 Parallel Scavenge 加 Parallel Old 收集器。

### 6. CMS 收集器
<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/mark9.jpg">
</div>


CMS（Concurrent Mark Sweep），Mark Sweep 指的是标记 - 清除算法。分为以下四个流程：

- 初始标记：仅仅只是标记一下 GC Roots 能直接关联到的对象，速度很快，需要停顿。
- 并发标记：进行 GC Roots Tracing 的过程，它在整个回收过程中耗时最长，不需要停顿。
- 重新标记：为了修正并发标记期间因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录，需要停顿。
- 并发清除：不需要停顿。


在整个过程中耗时最长的并发标记和并发清除过程中，收集器线程都可以与用户线程一起工作，不需要进行停顿。

具有以下缺点：

- 吞吐量低：低停顿时间是以牺牲吞吐量为代价的，导致 CPU 利用率不够高。
- 无法处理浮动垃圾，可能出现 Concurrent Mode Failure。浮动垃圾是指并发清除阶段由于用户线程继续运行而产生的垃圾，这部分垃圾只能到下一次 GC 时才能进行回收。由于浮动垃圾的存在，因此需要预留出一部分内存，意味着 CMS 收集不能像其它收集器那样等待老年代快满的时候再回收。如果预留的内存不够存放浮动垃圾，就会出现 Concurrent Mode Failure，这时虚拟机将临时启用 Serial Old 来替代 CMS。
- 标记 - 清除算法导致的空间碎片，往往出现老年代空间剩余，但无法找到足够大连续空间来分配当前对象，不得不提前触发一次 Full GC。

### 7. G1收集器

G1（Garbage-First），它是一款面向服务端应用的垃圾收集器，在多 CPU 和大内存的场景下有很好的性能。HotSpot 开发团队赋予它的使命是未来可以替换掉 CMS 收集器。它有如下特点：
- 并行与并发：G1能充分利用多CPU、多核的硬件优势，在不得不停掉主线程的时候采用并行模式来缩短Stop-The-World停顿时间；并且“并发标记”和“筛选回收”的过程是并发的。
- 分代收集：与其他收集器一样，分代概念在G1中依然得以保留。虽然G1可以不需其他收集器配合就能独立管理整个GC堆，但它能够采用不同的方式去处理新创建的对象和已经存活了一段时间、熬过多次GC的旧对象以获取更好的收集效果。
- 空间整合：与CMS的“标记-清理”算法不同，G1从整体看来是基于“标记-整理”算法实现的收集器，从局部（两个Region之间）上看是基于“复制”算法实现，这两种算法都意味着G1运作期间不会产生内存空间碎片，收集后能提供规整的可用内存。这种特性有利于程序长时间运行，分配大对象时不会因为无法找到连续内存空间而提前触发下一次GC。
- 可预测的停顿：这是G1相对于CMS的另外一大优势，降低停顿时间是G1和CMS共同的关注点，但G1除了追求低停顿外，还能建立可预测的停顿时间模型，能让使用者明确指定在一个长度为M毫秒的时间片段内，消耗在垃圾收集上的时间不得超过N毫秒，这几乎已经是实时Java（RTSJ）的垃圾收集器特征了。


堆被分为新生代和老年代，其它收集器进行收集的范围都是整个新生代或者老年代，而 G1 可以直接对新生代和老年代一起回收。
<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/mark10.png" width="500">
</div>


G1 把堆划分成多个大小相等的独立区域（Region），新生代和老年代不再物理隔离。

<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/mark11.png" width="500">
</div>


通过引入 Region 的概念，从而将原来的一整块内存空间划分成多个的小空间，使得每个小空间可以单独进行垃圾回收。这种划分方法带来了很大的灵活性，使得可预测的停顿时间模型成为可能。通过记录每个 Region 垃圾回收时间以及回收所获得的空间（这两个值是通过过去回收的经验获得），并维护一个优先列表，每次根据允许的收集时间，优先回收价值最大的 Region。

每个 Region 都有一个 Remembered Set，用来记录该 Region 对象的引用对象所在的 Region。通过使用 Remembered Set，在做可达性分析的时候就可以避免全堆扫描。
<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/mark12.jpg">
</div>


如果不计算维护 Remembered Set 的操作，G1 收集器的运作大致可划分为以下几个步骤：

- 初始标记：参见CMS。
- 并发标记：参见CMS。
- 最终标记：为了修正在并发标记期间因用户程序继续运作而导致标记产生变动的那一部分标记记录，虚拟机将这段时间对象变化记录在线程的 Remembered Set Logs 里面，最终标记阶段需要把 Remembered Set Logs 的数据合并到 Remembered Set 中。这阶段需要停顿线程，但是可并行执行。
- 筛选回收：首先对各个 Region 中的回收价值和成本进行排序，根据用户所期望的 GC 停顿时间来制定回收计划。此阶段其实也可以做到与用户程序一起并发执行，但是因为只回收一部分 Region，时间是用户可控制的，而且停顿用户线程将大幅度提高收集效率。

**只有第二步不需要停顿。**

### 补充：如何查看JVM当前使用的垃圾回收算法
使用`java -XX:+PrintCommandLineFlags -version`命令，比如在`jdk1.8`中使用该命令打印的结果如下：

```
-XX:InitialHeapSize=132165184 -XX:MaxHeapSize=2114642944 -XX:+PrintCommandLineFlags -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:-UseLargePagesIndividualAllocation -XX:+UseParallelGC
java version "1.8.0_181"
Java(TM) SE Runtime Environment (build 1.8.0_181-b13)
Java HotSpot(TM) 64-Bit Server VM (build 25.181-b13, mixed mode)
```
之后再根据《深入理解Java虚拟机：JVM高级特性与最佳实践》第二版第`90`页表`3-2`的介绍，可以看到，`jdk1.8`使用的默认垃圾收集器为`新生代（Parallel Scavenge）+ 老年代（Serial Old）`。

### 补充：堆中老年代和年轻代的划分：
<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/Eden%20and%20survivor.jpg" width="450">
</div>


如上图所示，整个堆被分为两部分：新生代和老年代；新生代又分为1个Eden区和两个Survivor区。

- 新生代(Young Generation)：新生代中的对象都是“朝生夕死”的，所以采用的是“复制”算法。
    - Eden Space字面意思是伊甸园，对象被创建的时候首先放到这个区域，进行垃圾回收后，不能被回收的对象被放入到空的survivor区域，即to区。
    - Survivor Space幸存者区，用于保存在eden space内存区域中经过垃圾回收后没有被回收的对象。在GC开始的时候，对象只会存在于Eden区和名为“From”的Survivor区，Survivor区“To”是空的。紧接着进行GC，Eden区中所有存活的对象都会被复制到“To”，而在“From”区中，仍存活的对象会根据他们的年龄值来决定去向。年龄达到一定值(年龄阈值，可以通过-XX:MaxTenuringThreshold来设置)的对象会被移动到年老代中，没有达到阈值的对象会被复制到“To”区域。经过这次GC后，Eden区和From区已经被清空。这个时候，“From”和“To”会交换他们的角色，也就是新的“To”就是上次GC前的“From”，新的“From”就是上次GC前的“To”。不管怎样，都会保证名为To的Survivor区域是空的。Minor GC会一直重复这样的过程，直到“To”区被填满，“To”区被填满之后，会将所有对象移动到年老代中。
- Old Gen老年代，用于存放新生代中经过多次垃圾回收仍然存活的对象，也有可能是新生代分配不了内存的大对象会直接进入老年代。经过多次垃圾回收都没有被回收的对象，这些对象的年代已经足够old了，就会放入到老年代。当老年代被放满的之后，虚拟机会进行垃圾回收，称之为Major GC。由于Major GC除并发GC外均需对整个堆进行扫描和回收，因此又称为Full GC。

# 三、内存分配与回收策略
## Minor GC 和 Full GC

- Minor GC：发生在新生代上，因为新生代对象存活时间很短，因此 Minor GC 会频繁执行，执行的速度一般也会比较快。
- Full GC(Major GC)：发生在老年代上，老年代对象其存活时间长，因此 Full GC 很少执行，执行速度会比 Minor GC 慢很多。

## 内存分配策略
### 1. 对象优先在 Eden 分配

大多数情况下，对象在新生代 Eden 区分配，当 Eden 区空间不够时，发起 Minor GC。

### 2. 大对象直接进入老年代

大对象是指需要连续内存空间的对象，最典型的大对象是那种很长的字符串以及数组。大对象对JVM来说是个坏消息，更坏的消息是遇到一群“朝生夕灭”的“短命大对象”，应该尽量避免，因为经常出现大对象会提前触发垃圾收集以获取足够的连续空间分配给大对象。

-XX:PretenureSizeThreshold，大于此值的对象直接在老年代分配，避免在 Eden 区和 Survivor 区之间的大量内存复制。对象优先在 Eden 分配

### 3.  长期存活的对象进入老年代

为对象定义年龄计数器，对象在 Eden 出生并经过 Minor GC 依然存活，将移动到 Survivor 中，年龄变为 1 岁，对象在Survivor中每“熬过”一次Minor GC，年龄就增加1岁，增加到一定年龄则移动到老年代中。

使用-XX:MaxTenuringThreshold 用来定义年龄的阈值。

### 4. 动态对象年龄判定

虚拟机并不是永远地要求对象的年龄必须达到 MaxTenuringThreshold 才能晋升老年代，如果在 Survivor 中相同年龄所有对象大小的总和大于 Survivor 空间的一半(注意这里的“Survivor 空间的一半”指的是一块servivor空间，比如from空间为1M，则这里的“一般是指”0.5M)，则年龄大于或等于该年龄的对象可以直接进入老年代，无需等到 MaxTenuringThreshold 中要求的年龄。

### 5. 空间分配担保

在发生 Minor GC 之前，虚拟机先检查老年代最大可用的连续空间是否大于新生代所有对象总空间，如果条件成立的话，那么 Minor GC 可以确认是安全的。


如果不成立的话虚拟机会查看 HandlePromotionFailure 设置值是否允许担保失败，如果允许那么就会继续检查老年代最大可用的连续空间是否大于历次晋升到老年代对象的平均大小，如果大于，将尝试着进行一次 Minor GC；如果小于，或者 HandlePromotionFailure 设置不允许冒险，那么就要进行一次 Full GC。


 需要注意的是，在jdk6 uptate 24之后，HandlePromotionFailure已经不起作用了，规则变为只要老年代的连续空间大于新生代对象总大小或者历次晋升的平均大小就会进行Minor GC，否则将进行Full GC。

## 测试
### 1. 对象优先在 Eden 分配测试
```java
/**
 * 对象优先在 Eden 分配测试
 * VM args: -verbose:gc -Xms20m -Xmx20m -Xmn10m -XX:+PrintGCDetails -XX:SurvivorRatio=8 -XX:+UseSerialGC
 */
public class TestAllocation {
    private static final int _1MB = 1024 * 1024;

    public static void testAllocation(){
        byte[] allocation1, allocation2, allocation3, allocation4;
        allocation1 = new byte[2 * _1MB];
        allocation2 = new byte[2 * _1MB];
        allocation3 = new byte[2 * _1MB];
        allocation4 = new byte[4 * _1MB];   //出现一次Minor GC
    }

    public static void main(String[] args) {
        testAllocation();
    }
}
```

控制台输出结果为：
```java
[GC (Allocation Failure) [DefNew: 8144K->643K(9216K), 0.0046612 secs] 8144K->6787K(19456K), 0.0047032 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
Heap
 def new generation   total 9216K, used 4821K [0x00000000fec00000, 0x00000000ff600000, 0x00000000ff600000)
  eden space 8192K,  51% used [0x00000000fec00000, 0x00000000ff014930, 0x00000000ff400000)
  from space 1024K,  62% used [0x00000000ff500000, 0x00000000ff5a0cb8, 0x00000000ff600000)
  to   space 1024K,   0% used [0x00000000ff400000, 0x00000000ff400000, 0x00000000ff500000)
 tenured generation   total 10240K, used 6144K [0x00000000ff600000, 0x0000000100000000, 0x0000000100000000)
   the space 10240K,  60% used [0x00000000ff600000, 0x00000000ffc00030, 0x00000000ffc00200, 0x0000000100000000)
 Metaspace       used 3285K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 359K, capacity 388K, committed 512K, reserved 1048576K
```

需要注意的是，idea+jdk1.8默认使用的是Parallel Scavenge回收器，而书上使用的是Serial/Serial Old，所以要想达到和书上一样的结果，要添加条件：-XX:+UseSerialGC。

结果解释：代码中通过-Xms20m -Xmx20m -Xmn10m三个参数限制了Java堆大小为20M不可扩展，其中10M分配给新生代，10M分配给老年代。-XX:SurvivorRatio=8限制了新生代中Eden区和一个Survivor区的空间比例为8:1。当JVM给allocation4分配内存时，发现Eden已经被占用了6M，剩余空间不足以分配allocation4所申请的4M，因此发生Minor GC。GC期间JVM又发现已有的3个2M大小的对象全部无法放入Survivor空间，所以只能通过分配担保机制转移到老年代中区。所以，GC完成之后，4M的allocation4被分配在Eden中，6M的allocation1、allocation2和allocation3被分配到老年代中。

### 2. 大对象直接进入老年代测试
```java
/**
 * 大对象直接进入老年代测试
 * VM args: -verbose:gc -Xmx20m -Xms20m -Xmn10m -XX:+PrintGCDetails -XX:SurvivorRatio=8 -XX:PretenureSizeThreshold=3145728 -XX:+UseSerialGC
 */
public class TestPerSizeThreshold {
    private static final int _1MB = 1024 * 1024;

    public static void testPerSizeThreshold(){
        byte[] allocation;
        allocation = new byte[4 * _1MB];
    }

    public static void main(String[] args) {
        testPerSizeThreshold();
    }
}
```

控制台输出结果为：
```java
Heap
 def new generation   total 9216K, used 2165K [0x00000000fec00000, 0x00000000ff600000, 0x00000000ff600000)
  eden space 8192K,  26% used [0x00000000fec00000, 0x00000000fee1d638, 0x00000000ff400000)
  from space 1024K,   0% used [0x00000000ff400000, 0x00000000ff400000, 0x00000000ff500000)
  to   space 1024K,   0% used [0x00000000ff500000, 0x00000000ff500000, 0x00000000ff600000)
 tenured generation   total 10240K, used 4096K [0x00000000ff600000, 0x0000000100000000, 0x0000000100000000)
   the space 10240K,  40% used [0x00000000ff600000, 0x00000000ffa00010, 0x00000000ffa00200, 0x0000000100000000)
 Metaspace       used 3284K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 359K, capacity 388K, committed 512K, reserved 1048576K。
```

同样需要设置一下-XX:+UseSerialGC。

解释：通过-XX:PretenureSizeThreshold=3145728设置超过3M的对象就算大对象，所以allocation会直接被放入老年代中。

### 3. 长期存活的对象进入老年代测试
```java
/**
 * 长期存活的对象进入老年代测试
 * VM args: -verbose:gc -Xmx20m -Xms20m -Xmn10m -XX:+PrintGCDetails -XX:SurvivorRatio=8 -XX:MaxTenuringThreshold=1
 * -XX:+PrintTenuringDistribution -XX:+UseSerialGC
 */
public class TestTenuringThreshold {
    private static final int _1MB = 1024 * 1024;

    public static void testTenuringThreshold(){
        byte[] allocation1, allocation2, allocation3;
        allocation1 = new byte[_1MB / 4];
        allocation2 = new byte[_1MB * 4];
        allocation3 = new byte[_1MB * 4];
        allocation3 = null;
        allocation3 = new byte[_1MB * 4];
    }

    public static void main(String[] args) {
        testTenuringThreshold();
    }
}
```

控制台输出的结果为：
```java
Desired survivor size 524288 bytes, new threshold 1 (max 1)
- age   1:     895136 bytes,     895136 total
: 6353K->874K(9216K), 0.0035154 secs] 6353K->4970K(19456K), 0.0035540 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [DefNew
Desired survivor size 524288 bytes, new threshold 1 (max 1)
- age   1:      25232 bytes,      25232 total
: 5054K->24K(9216K), 0.0013032 secs] 9150K->4991K(19456K), 0.0013257 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
Heap
 def new generation   total 9216K, used 4202K [0x00000000fec00000, 0x00000000ff600000, 0x00000000ff600000)
  eden space 8192K,  51% used [0x00000000fec00000, 0x00000000ff014930, 0x00000000ff400000)
  from space 1024K,   2% used [0x00000000ff400000, 0x00000000ff406290, 0x00000000ff500000)
  to   space 1024K,   0% used [0x00000000ff500000, 0x00000000ff500000, 0x00000000ff600000)
 tenured generation   total 10240K, used 4966K [0x00000000ff600000, 0x0000000100000000, 0x0000000100000000)
   the space 10240K,  48% used [0x00000000ff600000, 0x00000000ffad99b0, 0x00000000ffad9a00, 0x0000000100000000)
 Metaspace       used 3284K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 359K, capacity 388K, committed 512K, reserved 1048576K
```

依然通过-XX:+UseSerialGC来设置。

结果解释：allocation1对象需要256KB内存，Survivor空间可以容纳。MaxTenuringThreshold=1时，allocation1对象在第二次GC时进入老年代，Survivor中被使用的内存变为0KB；如果将MaxTenuringThreshold设置为一个比较大的数，比如15，那么第二次GC后，Survivor中的allocation1还在。

### 4. 动态对象年龄判定测试
```java
/**
 * 动态对象年龄判定测试
 * VM args: -verbose:gc -Xmx20m -Xms20m -Xmn10m -XX:+PrintGCDetails -XX:SurvivorRatio=8 -XX:MaxTenuringThreshold=15
 * -XX:+PrintTenuringDistribution -XX:+UseSerialGC
 */
public class TestTenuringThreshold2 {
    private static final int _1MB = 1024 * 1024;

    public static void testTenuringThreshold2(){
        byte[] allocation1, allocation2, allocation3, allocation4;
        //allocation1和allocation2和大于Survivor一半
        allocation1 = new byte[_1MB / 4];
        allocation2 = new byte[_1MB / 4];

        allocation3 = new byte[_1MB * 4];
        allocation4 = new byte[_1MB * 4];
        allocation4 = null;
        allocation4 = new byte[_1MB * 4];
    }

    public static void main(String[] args) {
        testTenuringThreshold2();
    }
}
```

控制台打印结果为：
```java
[GC (Allocation Failure) [DefNew
Desired survivor size 524288 bytes, new threshold 1 (max 15)
- age   1:    1048576 bytes,    1048576 total
: 6609K->1024K(9216K), 0.0034656 secs] 6609K->5250K(19456K), 0.0035106 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [DefNew
Desired survivor size 524288 bytes, new threshold 15 (max 15)
: 5120K->0K(9216K), 0.0125447 secs] 9346K->5250K(19456K), 0.0125669 secs] [Times: user=0.00 sys=0.00, real=0.02 secs] 
Heap
 def new generation   total 9216K, used 4178K [0x00000000fec00000, 0x00000000ff600000, 0x00000000ff600000)
  eden space 8192K,  51% used [0x00000000fec00000, 0x00000000ff014930, 0x00000000ff400000)
  from space 1024K,   0% used [0x00000000ff400000, 0x00000000ff400000, 0x00000000ff500000)
  to   space 1024K,   0% used [0x00000000ff500000, 0x00000000ff500000, 0x00000000ff600000)
 tenured generation   total 10240K, used 5250K [0x00000000ff600000, 0x0000000100000000, 0x0000000100000000)
   the space 10240K,  51% used [0x00000000ff600000, 0x00000000ffb20b30, 0x00000000ffb20c00, 0x0000000100000000)
 Metaspace       used 3284K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 359K, capacity 388K, committed 512K, reserved 1048576K
```

可以看到，虽然MaxTenuringThreshold=15，但是Survivor空间仍然为被使用，这是因为allocation1和allocation2的和大于Survivor空间的一半，直接进入了老年代。

### 5. 空间分配担保测试
```java
/**
 * 空间分配担保测试
 * VM args: -verbose:gc -Xmx20m -Xms20m -Xmn10m -XX:+PrintGCDetails -XX:SurvivorRatio=8 -XX:-HandlePromotionFailure
 * -XX:+UseSerialGC
 */
public class TestHandlePromotion {
    private static final int _1MB = 1024 * 1024;

    public static void testHandlePromotion(){
        byte[] allocation1, allocation2, allocation3, allocation4, allocation5, allocation6, allocation7;
        //allocation1和allocation2和大于Survivor一半
        allocation1 = new byte[_1MB * 2];
        allocation2 = new byte[_1MB * 2];
        allocation3 = new byte[_1MB * 2];

        allocation1 = null;
        allocation4 = new byte[_1MB * 2];
        allocation5 = new byte[_1MB * 2];
        allocation6 = new byte[_1MB * 2];

        allocation4 = null;
        allocation5 = null;
        allocation6 = null;

        allocation7 = new byte[_1MB * 2];
    }

    public static void main(String[] args) {
        testHandlePromotion();
    }
}
```

控制台输出结果为：
```java
Error: Could not create the Java Virtual Machine.
Error: A fatal exception has occurred. Program will exit.
Unrecognized VM option 'HandlePromotionFailure'
Did you mean '(+/-)PromotionFailureALot'?
```

目前该错误的原因还没找到。

需要注意的是，在jdk6 uptate 24之后，HandlePromotionFailure已经不起作用了，规则变为***只要老年代的连续空间大于新生代对象总大小或者历次晋升的平均大小就会进行Minor GC，否则将进行Full GC。***

## jdk1.7和jdk1.8的变化
### 1. 对比

JDK 1.7 及以往的 JDK 版本中，Java **类信息、常量池、静态变量**都存储在 Perm（永久代）里。类的元数据和静态变量在类加载的时候分配到 Perm，当类被卸载的时候垃圾收集器从 Perm 处理掉类的元数据和静态变量。当然常量池的东西也会在 Perm 垃圾收集的时候进行处理。

JDK 1.8 的对 JVM 架构的改造将**类元数据放到本地内存中(Meta Space)**，另外，**将常量池和静态变量放到 Java 堆里**。在这种架构下，类元信息就突破了原来 -XX:MaxPermSize 的限制，现在可以使用更多的本地内存。这样就从一定程度上解决了原来在运行时生成大量类的造成经常 Full GC 问题，如运行时使用反射、代理等。

元空间(Meta Space)并不在虚拟机中，而是使用本地内存。因此，默认情况下，元空间的大小仅受本地内存限制。

### 2. 元空间相关参数
> - -XX:MetaspaceSize，初始空间大小，达到该值就会触发垃圾收集进行类型卸载，同时GC会对该值进行调整：如果释放了大量的空间，就适当降低该值；如果释放了很少的空间，那么在不超过MaxMetaspaceSize时，适当提高该值。
> - -XX:MaxMetaspaceSize，最大空间，默认是没有限制的。


除了上面两个指定大小的选项以外，还有两个与 GC 相关的属性：

> - -XX:MinMetaspaceFreeRatio，在GC之后，最小的Metaspace剩余空间容量的百分比，减少为分配空间所导致的垃圾收集
> - -XX:MaxMetaspaceFreeRatio，在GC之后，最大的Metaspace剩余空间容量的百分比，减少为释放空间所导致的垃圾收集

<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/metaspace.png" width="600">
</div>

## Full GC 的触发条件

对于 Minor GC，其触发条件非常简单，当 Eden 空间满时，就将触发一次 Minor GC。而 Full GC 则相对复杂，有以下条件：

### 1. 调用 System.gc()

只是建议虚拟机执行 Full GC，但是虚拟机不一定真正去执行。不建议使用这种方式，而是让虚拟机管理内存。

### 2. 老年代空间不足
&emsp;老年代空间不足的常见场景为前文所讲的大对象直接进入老年代、长期存活的对象进入老年代等。

为了避免以上原因引起的 Full GC，应当尽量不要创建过大的对象以及数组。除此之外，可以通过 -Xmn 虚拟机参数调大新生代的大小，让对象尽量在新生代被回收掉，不进入老年代。还可以通过 -XX:MaxTenuringThreshold 调大对象进入老年代的年龄，让对象在新生代多存活一段时间。

### 3. 空间分配担保失败

使用复制算法的 Minor GC 需要老年代的内存空间作担保，如果担保失败会执行一次 Full GC。

### 4. JDK 1.7 及以前的永久代空间不足

在 JDK 1.7 及以前，HotSpot 虚拟机中的方法区是用永久代实现的，永久代中存放的为一些 Class 的信息、常量、静态变量等数据。

当系统中要加载的类、反射的类和调用的方法较多时，永久代可能会被占满，在未配置为采用 CMS GC 的情况下也会执行 Full GC。如果经过 Full GC 仍然回收不了，那么虚拟机会抛出 java.lang.OutOfMemoryError。

为避免以上原因引起的 Full GC，可采用的方法为增大永久代空间或转为使用 CMS GC。

### 5.  Concurrent Mode Failure

执行 CMS GC 的过程中同时有对象要放入老年代，而此时老年代空间不足（可能是 GC 过程中浮动垃圾过多导致暂时性的空间不足），便会报 Concurrent Mode Failure 错误，并触发 Full GC。

## GC 日志分析

日志示例：
```java

4.231: [GC 4.231: [DefNew: 4928K->512K(4928K), 0.0044047 secs] 6835K->3468K(15872K), 0.0045291 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]
4.445: [Full GC (System) 4.445: [Tenured: 2956K->3043K(10944K), 0.1869806 secs] 4034K->3043K(15872K), [Perm : 3400K->3400K(12288K)], 0.1870847 secs] [Times: user=0.05 sys=0.00, real=0.19 secs]
```
> - 4.231和4.445表示发生GC的时间戳,是从jvm启动开始计时的。
> - [GC 和 [Full GC 是垃圾回收的停顿类型，如果有Full 说明发生了 Stop-The-World ，是Full GC 。同时，如果是调用System.gc() 触发的，那么将显示的是 [Full GC (System) 。
> - [DefNew , [Tenured ,[Perm 表示 GC 发生的区域，区域的名称与使用的 GC 收集器相关。Serial 收集器中新生代名为 "Default New Generation"，显示的名字为 "[DefNew"。对于ParNew收集器，显示的是 "[ParNew"，表示 “Parallel New Generation”。 对于 Parallel Scavenge 收集器，新生代名为 "PSYoungGen"。年老代和永久代也相同，名称都由收集器决定。
> - 方括号内部显示的 “4928K->512K(4928K)” 表示 “GC 前该区域已使用容量 -> GC 后该区域已使用容量 (该区域内存总容量) 。
> -  “0.0044047 secs” 表示该区域GC所用时间，单位是秒。
> - “6835K->3468K(15872K)” 表示 “GC 前Java堆已使用容量 -> GC后Java堆已使用容量 （Java堆总容量）”。
> -  “0.0045291 secs” 是Java堆GC所用的总时间。
> -  “[Times: user=0.00 sys=0.00, real=0.00 secs]” 分别代表 用户态消耗的CPU时间、内核态消耗的CPU时间 和 操作从开始到结束所经过的墙钟时间。墙钟时间包括各种非运算的等待耗时，如IO等待、线程阻塞。CPU时间不包括等待时间，当系统有多核时，多线程操作会叠加这些CPU时间，所以user或sys时间会超过real时间。

# 类加载机制

类是在运行期间第一次使用时动态加载的，而不是编译时期一次性加载。因为如果在编译时期一次性加载，那么会占用很多的内存

## 类的生命周期
<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/classloader.jpg">
</div>


包括以下 7 个阶段：
> - **加载（Loading）**
> - **验证（Verification）**
> - **准备（Preparation）**
> - **解析（Resolution）**
> - **初始化（Initialization）**
> - 使用（Using）
> - 卸载（Unloading）

## 类加载过程

包含了加载、验证、准备、解析和初始化这 5 个阶段。

### 1. 加载

加载是类加载的一个阶段，注意不要混淆。

加载过程完成以下三件事：
> - 通过一个类的全限定名来获取定义此类的二进制字节流。
> - 将这个字节流所代表的静态存储结构转化为方法区的运行时存储结构。
> - 在内存中生成一个代表这个类的 Class 对象，作为方法区这个类的各种数据的访问入口。


其中二进制字节流可以从以下方式中获取：
> - 从 ZIP 包读取，成为 JAR、EAR、WAR 格式的基础。
> - 从网络中获取，最典型的应用是 Applet。
> - 运行时计算生成，例如动态代理技术，在 java.lang.reflect.Proxy 使用 ProxyGenerator.generateProxyClass 的代理类的二进制字节流。
> - 由其他文件生成，例如由 JSP 文件生成对应的 Class 类。


对于类的加载，可以分为数组类型和非数组类型，对于非数组类型可以通过系统的引导类加载器进行加载，也可以通过自定义的类加载器进行加载(只要重写一个类加载器的loadClass()方法)。

数组类本身不通过类加载器进行加载，而是通过Java虚拟机直接进行加载的，但数组去除所有维度之后的类型(数组元素类型)最终还是要依靠类加载器进行加载的，所以数组类型的类与类加载器的关系还是很密切的。通常一个数组类型的类进行加载需要遵循以下的原则：
> - 如果数组的组件类型（Component Type，也就是数组类去除一个维度之后的类型，比如对于二维数组，去除一个维度之后是一个一维数组）是引用类型，那么递归采用上面的过程加载这个组件类型
> - 如果数组类的组件类型不是引用类型，比如是基本数据类型(比如int[] 数组)，Java虚拟机将把数组类标记为与引导类加载器关联
> - 数组类的可见性与组件类型的可见性是一致的。如果组件类型不是引用类型，那么数组类的可见性是public，意味着组件类型的可见性也是public。

### 2. 验证

验证阶段的目的是为了确保Class字节流中包含的信息符合当前虚拟机的要求，并且不会危害虚拟机的安全。


Java语言具有相对的安全性（这里的安全性体现为两个方面：一是Java语言本身特性，比如Java去除指针，这点可以避免对内存的直接操作；二是Java所提供的沙箱运行机制，Java保证所运行的机制都是在沙箱之内运行的，而沙箱之外的操作都不可以运行）。但是需要注意的是Java虚拟机处理的Class文件并不一定是是从Java代码编译而来，完全可能是来自其他的语言，甚至可以直接通过十六进制编辑器书写Class文件（当然前提是编写的Class文件符合规范）。从这个角度讲，其他来源的Class文件是不可能都保证其安全性的。所以如果Java虚拟机都信任其加载进来的Class文件，那么很有可能会造成对虚拟机自身的危害。

虚拟机的验证阶段主要完后以下4项验证：
> - 文件格式验证
> - 元数据验证
> - 字节码验证
> - 符号引用验证

#### 2.1 文件格式验证

这里的文件格式是指Class的文件规范，这一步的验证主要保证加载的字节流（在计算机中不可能是整个Class文件，只有0和1，也就是字节流）符合Class文件的规范（根据前面对Class类文件的描述，Class文件的每一个字节表示的含义都是确定的。比如前四个字节是否是一个魔数等）以及保证这个字节流可以被虚拟机接受处理。

#### 2.2 元数据验证

首先需要对元数据进行一点解释：元数据可以理解为描述数据的数据，更通俗的说，元数据是描述类之间的依赖关系的数据，比如Java语言中的注解使用（使用@interface创建一个注解）。**验证阶段的主要目的是对类的元数据信息进行语义校验，保证不存在不符合Java语言规范的元数据信息**。

上面的语义指的是Java语言中的语义，说白了就是Java的语法。具体的验证信息包括以下几个方面：
> - 这个类是否有父类（除了java.lang.Object外其余的类都应该有父类）；
> - 这个类的父类是否继承了不允许被继承的类（比如被final修饰的类）
> - 如果这个类不是抽象类，是否实现了其父类或者接口中要求实现的方法
> - 类中的字段、方法是否与父类产生矛盾（比如是否覆盖了父类的final字段）

#### 2.3 字节码验证

这个阶段主要对类的方法体进行校验分析。通过了字节码的验证并不代表就是没有问题的，但是如果没有通过验证就一定是有问题的。整个字节码的验证过程比这个复杂的多，由于字节码验证的高度复杂性，在jdk1.6版本之后的虚拟机增加了一项优化。

#### 2.4 符号引用验证

这个验证是最后阶段的验证，符号引用是Class文件的逻辑符号，直接引用指向的方法区中某一个地址，这个转化阶段是在连接的第三个阶段也就是解析阶段完成的。符号引用验证主要是对类自身以外(常量池中的 各种 符号引用)的信息进行匹配性校验，通常包涵以下方面：
> - 符号引用是否通过字符串描述的全限定名是否能够找到对应点类;
> - 符号引用中的类、字段、方法的访问属性(private、protected、public、default)是否可被当前类访问。

### 3. 准备

类变量是被 static 修饰的变量，准备阶段为类变量分配内存并设置初始值，使用的是方法区的内存。实例变量不会在这阶段分配内存，它将会在对象实例化时随着对象一起分配在堆中。

***注意，实例化不是类加载的一个过程，类加载发生在所有实例化操作之前，并且类加载只进行一次，实例化可以进行多次。***

初始值一般为 0 值，例如下面的类变量 value 在准备阶段结束后被初始化为 0 而不是 123。
```java
public static int value = 123;
```

如果类变量是常量，那么会按照表达式来进行初始化，而不是赋值为 0。
```java
public static final int value = 123;
```

### 4. 解析

将常量池的符号引用替换为直接引用的过程。

其中解析过程在某些情况下可以在初始化阶段之后再开始，这是为了支持 Java 的动态绑定。
未完，参考https://blog.csdn.net/u011116672/article/details/49887585
https://blog.csdn.net/noaman_wgs/article/details/74489549


### 5. 初始化

初始化阶段才真正开始执行类中定义的 Java 程序代码。初始化阶段即虚拟机执行类构造器 `<clinit>()` 方法的过程。

在准备阶段，类变量已经赋过一次系统要求的初始值，而在初始化阶段，根据程序员通过程序制定的主观计划去初始化类变量和其它资源。

`<clinit>()` 方法具有以下特点：

- 是由编译器自动收集类中所有类变量的赋值动作和静态语句块中的语句合并产生的，编译器收集的顺序由语句在源文件中出现的顺序决定。特别注意的是，静态语句块只能访问到定义在它之前的类变量，定义在它之后的类变量只能赋值，不能访问。例如以下代码：
```java
public class Test {
    static {
        i = 0;                // 给变量赋值可以正常编译通过
        System.out.print(i);  // 这句编译器会提示“非法向前引用”
    }
    static int i = 1;
}
```
- 与类的构造函数（或者说实例构造器 `<init>()`）不同，不需要显式的调用父类的构造器。虚拟机会自动保证在子类的 `<clinit>()` 方法运行之前，父类的 `<clinit>()` 方法已经执行结束。因此虚拟机中第一个执行 `<clinit>()` 方法的类肯定为 java.lang.Object。
- 由于父类的 `<clinit>()` 方法先执行，也就意味着父类中定义的静态语句块的执行要优先于子类。例如以下代码：
```java
static class Parent {
    public static int A = 1;
    static {
        A = 2;
    }
}

static class Sub extends Parent {
    public static int B = A;
}

public static void main(String[] args) {
     System.out.println(Sub.B);  // 2
}
```
- `<clinit>()` 方法对于类或接口不是必须的，如果一个类中不包含静态语句块，也没有对类变量的赋值操作，编译器可以不为该类生成 `<clinit>()` 方法。
- 接口中不可以使用静态语句块，但仍然有类变量初始化的赋值操作，因此接口与类一样都会生成 `<clinit>()` 方法。但接口与类不同的是，执行接口的 `<clinit>()` 方法不需要先执行父接口的 `<clinit>()` 方法。只有当父接口中定义的变量使用时，父接口才会初始化。另外，接口的实现类在初始化时也一样不会执行接口的 `<clinit>()` 方法。
- 虚拟机会保证一个类的 `<clinit>()` 方法在多线程环境下被正确的加锁和同步，如果多个线程同时初始化一个类，只会有一个线程执行这个类的 `<clinit>()` 方法，其它线程都会阻塞等待，直到活动线程执行 `<clinit>()` 方法完毕。如果在一个类的 `<clinit>()` 方法中有耗时的操作，就可能造成多个线程阻塞，在实际过程中此种阻塞很隐蔽。

## 类初始化时机
### 1. 主动引用

虚拟机规范中并没有强制约束何时进行加载，但是规范严格规定了有且只有下列五种情况必须对类进行初始化（加载、验证、准备都会随之发生）：

 - 遇到 new、getstatic、putstatic、invokestatic 这四条字节码指令时，如果类没有进行过初始化，则必须先触发其初始化。最常见的生成这 4 条指令的场景是：使用 new 关键字实例化对象的时候；读取或设置一个类的静态字段（被 final 修饰、已在编译期把结果放入常量池的静态字段除外）的时候；以及调用一个类的静态方法的时候。
- 使用 java.lang.reflect 包的方法对类进行反射调用的时候，如果类没有进行初始化，则需要先触发其初始化。
当初始化一个类的时候，如果发现其父类还没有进行过初始化，则需要先触发其父类的初始化。
- 当虚拟机启动时，用户需要指定一个要执行的主类（包含 main() 方法的那个类），虚拟机会先初始化这个主类；
- 当使用 JDK 1.7 的动态语言支持时，如果一个 java.lang.invoke.MethodHandle 实例最后的解析结果为 REF_getStatic, REF_putStatic, REF_invokeStatic 的方法句柄，并且这个方法句柄所对应的类没有进行过初始化，则需要先触发其初始化；

### 2. 被动引用

以上 5 种场景中的行为称为对一个类进行主动引用。除此之外，所有引用类的方式都不会触发初始化，称为被动引用。被动引用的常见例子包括：

- 通过子类引用父类的静态字段，不会导致子类初始化。
```java
/**
 * 通过子类引用父类的静态字段，不会导致子类初始化
 */
public class NotInitialization_1 {
    public static class superClass{
        static {
            System.out.println("superclass init");
        }
        public static int value = 123;
    }

    public static class subClass extends superClass{
        static {
            System.out.println("subclass init");
        }
    }

    public static void main(String[] args) {
        System.out.println(subClass.value);
    }
}
```

上述代码结果为：
```java
superclass init
123
```

对于静态段，之后直接定义这个字段的类才会被初始化，因此superClass会被初始化而subClass不会被初始化。

- 通过数组定义来引用类，不会触发此类的初始化。该过程会对数组类进行初始化，数组类是一个由虚拟机自动生成的、直接继承自 Object 的子类，其中包含了数组的属性和方法。
```java
/**
 * 通过数组定义来引用类，不会触发此类的初始化
 */
public class NotInitialization_2 {
    public static class superClass{
        static {
            System.out.println("superclass init");
        }
        public static int value = 123;
    }

    public static void main(String[] args) {
        superClass[] superClasses = new superClass[10];
    }
}
```

- 常量在编译阶段会存入调用类的常量池中，本质上并没有直接引用到定义常量的类，因此不会触发定义常量的类的初始化。
```java
/**
 * 使用常量不会触发定义此常量的类的初始化
 */
public class NotInitialization_3 {
    public static class constClass{
        static{
            System.out.println("const class init");
        }

        public static final String HELLOWORLD = "hello world";
    }

    public static void main(String[] args) {
        System.out.println(constClass.HELLOWORLD);
    }
}
```

## 类与类加载器

两个类相等需要类本身相等，并且使用同一个类加载器进行加载。这是因为每一个类加载器都拥有一个独立的类名称空间。

这里的相等，包括类的 Class 对象的 equals() 方法、isAssignableFrom() 方法、isInstance() 方法的返回结果为 true，也包括使用 instanceof 关键字做对象所属关系判定结果为 true。

## 类加载器分类

从 Java 虚拟机的角度来讲，只存在以下两种不同的类加载器：

> - 启动类加载器（Bootstrap ClassLoader），这个类加载器用 C++ 实现，是虚拟机自身的一部分；
> - 所有其他类的加载器，这些类由 Java 实现，独立于虚拟机外部，并且全都继承自抽象类 java.lang.ClassLoader。


从 Java 开发人员的角度看，类加载器可以划分得更细致一些：

> - 启动类加载器（Bootstrap ClassLoader）此类加载器负责将存放在 <JRE_HOME>\lib 目录中的，或者被 -Xbootclasspath 参数所指定的路径中的，并且是虚拟机识别的（仅按照文件名识别，如 rt.jar，名字不符合的类库即使放在 lib 目录中也不会被加载）类库加载到虚拟机内存中。启动类加载器无法被 Java 程序直接引用，用户在编写自定义类加载器时，如果需要把加载请求委派给启动类加载器，直接使用 null 代替即可。
> - 扩展类加载器（Extension ClassLoader）这个类加载器是由 ExtClassLoader（sun.misc.Launcher$ExtClassLoader）实现的。它负责将 <JAVA_HOME>/lib/ext 或者被 java.ext.dir 系统变量所指定路径中的所有类库加载到内存中，开发者可以直接使用扩展类加载器。

> - 应用程序类加载器（Application ClassLoader）这个类加载器是由 AppClassLoader（sun.misc.Launcher$AppClassLoader）实现的。由于这个类加载器是 ClassLoader 中的 getSystemClassLoader() 方法的返回值，因此一般称为系统类加载器。它负责加载用户类路径（ClassPath）上所指定的类库，开发者可以直接使用这个类加载器，如果应用程序中没有自定义过自己的类加载器，一般情况下这个就是程序中默认的类加载器。

## 双亲委派模型

应用程序都是由三种类加载器相互配合进行加载的，如果有必要，还可以加入自己定义的类加载器。

下图展示的类加载器之间的层次关系，称为类加载器的双亲委派模型（Parents Delegation Model）。该模型要求除了顶层的启动类加载器外，其余的类加载器都应有自己的父类加载器。这里类加载器之间的父子关系一般通过组合（Composition）关系来实现，而不是通过继承（Inheritance）的关系实现。

<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/parentsclassloader.png">
</div>

### 1. 工作过程

一个类加载器首先将类加载请求传送到父类加载器，只有当父类加载器无法完成类加载请求时子类加载器才尝试加载。

### 2. 好处

安全。
黑客自定义一个java.lang.String类，该String类具有系统的String类一样的功能，只是在某个函数稍作修改。比如equals函数，这个函数经常使用，如果在这这个函数中，黑客加入一些“病毒代码”。并且通过自定义类加载器加入到JVM中。此时，如果没有双亲委派模型，那么JVM就可能误以为黑客自定义的java.lang.String类是系统的String类，导致“病毒代码”被执行。

而有了双亲委派模型，黑客自定义的java.lang.String类永远都不会被加载进内存。因为首先是最顶端的类加载器加载系统的java.lang.String类，最终自定义的类加载器无法加载java.lang.String类。


使得 Java 类随着它的类加载器一起具有一种带有优先级的层次关系，从而使得基础类得到统一。

例如 java.lang.Object 存放在 rt.jar 中，如果编写另外一个 java.lang.Object 的类并放到 ClassPath 中，程序可以编译通过。由于双亲委派模型的存在，所以在 rt.jar 中的 Object 比在 ClassPath 中的 Object 优先级更高，这是因为 rt.jar 中的 Object 使用的是启动类加载器，而 ClassPath 中的 Object 使用的是应用程序类加载器。rt.jar 中的 Object 优先级更高，那么程序中所有的 Object 都是这个 Object。

### 3. 实现

以下是抽象类 java.lang.ClassLoader 的代码片段，其中的 loadClass() 方法运行过程如下：先检查类是否已经加载过，如果没有则让父类加载器去加载。当父类加载器加载失败时抛出 ClassNotFoundException，此时尝试自己去加载。
```java
public abstract class ClassLoader {
    // The parent class loader for delegation
    private final ClassLoader parent;

    public Class<?> loadClass(String name) throws ClassNotFoundException {
        return loadClass(name, false);
    }

    protected Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException {
        synchronized (getClassLoadingLock(name)) {
            // First, check if the class has already been loaded
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                try {
                    if (parent != null) {
                        c = parent.loadClass(name, false);
                    } else {
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }

                if (c == null) {
                    // If still not found, then invoke findClass in order
                    // to find the class.
                    c = findClass(name);
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }

    protected Class<?> findClass(String name) throws ClassNotFoundException {
        throw new ClassNotFoundException(name);
    }
}
```

## 自定义类加载器实现

FileSystemClassLoader 是自定义类加载器，继承自 java.lang.ClassLoader，用于加载文件系统上的类。它首先根据类的全名在文件系统上查找类的字节代码文件（.class 文件），然后读取该文件内容，最后通过 defineClass() 方法来把这些字节代码转换成 java.lang.Class 类的实例。

java.lang.ClassLoader 的 loadClass() 实现了双亲委派模型的逻辑，因此自定义类加载器一般不去重写它，但是需要重写 findClass() 方法。
```java
public class FileSystemClassLoader extends ClassLoader {

    private String rootDir;

    public FileSystemClassLoader(String rootDir) {
        this.rootDir = rootDir;
    }

    protected Class<?> findClass(String name) throws ClassNotFoundException {
        byte[] classData = getClassData(name);
        if (classData == null) {
            throw new ClassNotFoundException();
        } else {
            return defineClass(name, classData, 0, classData.length);
        }
    }

    private byte[] getClassData(String className) {
        String path = classNameToPath(className);
        try {
            InputStream ins = new FileInputStream(path);
            ByteArrayOutputStream baos = new ByteArrayOutputStream();
            int bufferSize = 4096;
            byte[] buffer = new byte[bufferSize];
            int bytesNumRead;
            while ((bytesNumRead = ins.read(buffer)) != -1) {
                baos.write(buffer, 0, bytesNumRead);
            }
            return baos.toByteArray();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return null;
    }

    private String classNameToPath(String className) {
        return rootDir + File.separatorChar
                + className.replace('.', File.separatorChar) + ".class";
    }
}
```

---
参考：
[Java自定义类加载器与双亲委派模型](http://www.cnblogs.com/wxd0108/p/6681618.html)

---

# 虚拟机性能监测与故障处理工具
## JDK命令行工具

### java

### javac

### javap

### jps(JVM Process Status Tool)
虚拟机进程状况工具，主要用来显示指定系统内所有的HotSpot虚拟机进程，类似于UNIX的ps命令。还可以显示虚拟机执行主类(Main Class，main()函数所在的类)和进程的本地虚拟机唯一ID(Local Cirtual Machine Identifier, LVMID)。对于本地虚拟机来说，LVMID就是操作系统的进程ID，也就是PID。

jps命令格式为：
```
jps [ options ] [ hostid ]
```
常用选项有：
```
jps -q        只显示LVMID
jps -m        显示传递给主函数的参数
jps -l        主类全名，如果是jar则输出Jar包路径
jps -v        显示虚拟机启动时的JVM参数
```
例子：
```
jps -l localhost
35196 sun.tools.jps.Jps
```

### jstat(JVM Statistics Monitoring Tool)
虚拟机统计信息监视工具。它可以显示本地或者远程虚拟机(需要远程主机提供RMI支持，可以借助jstatd工具建立远程RMI服务器)的类装载，内存，垃圾收集，JIT编译等信息。

命令格式为：
```
jstat [ option vmid [interval [s|ms] [count]]]
```
如果是本地虚拟机进程,VMID与LVMID一致，如果是远程虚拟机进程，那么VMID的格式为：
```
[protocol:][//]lvmid [@hostname [:port]/servername]
```
interval: 执行每次的 间隔时间，单位为 毫秒。count: 用于指定输出记录的 次数，缺省只查询一次。

option代表需要查询的虚拟机信息，主要分为三类：类装载、垃圾收集、运行期编译状况。
```
class        显示 类加载 ClassLoad 的相关信息；
compiler     显示 JIT 编译 的相关信息；
gc           显示和 gc相关的 堆信息；
gccapacity   显示 各个代 的 容量 以及 使用情况；
gcmetacapacity   显示 元空间metaspace 的大小；
gcnew        显示 新生代 信息； gcnewcapacity: 显示 新生代大小 和 使用情况；
gcold        显示 老年代 和 永久代 的信息；
gcoldcapacity   显示 老年代 的大小；
gcutil       显示垃圾回收信息；
gccause      显示 垃圾回收 的相关信息（同 -gcutil），同时显示 最后一次 或 当前 正在发生的垃圾回收的诱
printcompilation  输出 JIT 编译 的方法信息
```

比如，每250ms查询一次12500进程的垃圾收集状况，一共查询3次，命令和结果如下：
```
jstat -gc 12500 250 3
 S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT
3392.0 3392.0  0.0   3392.0 27328.0   3154.8   68288.0    21724.0   4864.0 3450.1 512.0  375.9       1    0.020   0      0.000    0.020
3392.0 3392.0  0.0   3392.0 27328.0   3474.9   68288.0    21724.0   4864.0 3450.1 512.0  375.9       1    0.020   0      0.000    0.020
3392.0 3392.0  0.0   3392.0 27328.0   3730.9   68288.0    21724.0   4864.0 3450.1 512.0  375.9       1    0.020   0      0.000    0.020
```
其中S0，S1表示新生代两个Survivor区，E代表的是新生代的 Eden区，C的 意思是容量，u表示已经使用的意思，O表示 的老年代，M表示方法区。F和Y则 表示 fullGC和 minorGC（即年轻代GC）

### jinfo(Configuration Info for Java)
Java配置信息工具。作用是实时查看和调整 虚拟机运行参数。jinfo 命令格式 
```
jinfo [options] pid
```
例如，需要查看`CMSInitiatingOccupancyFranction`参数值的命令为：
```
jinfo -flag CMSInitiatingOccupancyFranction 1444

-XX:CMSInitiatingOccupancyFranction=85
```

### jmap(Memory Map for Java)
Java内存映射工具，主要用于生成堆存储快照(一般称为heapdump或dump文件)。此外还可以查询finalize执行队列，Java对和永久代详细信息，如空间利用率当前使用的收集器类型等。

命令格式 
```
jmap [options] pid
```
常见选项参数的意思： 
```
-heap            显示 Java堆中的详细信息 
-histo           显示对象的统计 消息 
-clstats         显示 类加载 的统计信息
```
示例
```
jmap -heap 25440
Attaching to process ID 25440, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.181-b13

using thread-local object allocation.
Mark Sweep Compact GC

Heap Configuration:
   MinHeapFreeRatio         = 40
   MaxHeapFreeRatio         = 70
   MaxHeapSize              = 104857600 (100.0MB)
   NewSize                  = 34930688 (33.3125MB)
   MaxNewSize               = 34930688 (33.3125MB)
   OldSize                  = 69926912 (66.6875MB)
   NewRatio                 = 2
   SurvivorRatio            = 8
   MetaspaceSize            = 21807104 (20.796875MB)
   CompressedClassSpaceSize = 1073741824 (1024.0MB)
   MaxMetaspaceSize         = 17592186044415 MB
   G1HeapRegionSize         = 0 (0.0MB)

Heap Usage:
New Generation (Eden + 1 Survivor Space):
   capacity = 31457280 (30.0MB)
   used     = 21844000 (20.832061767578125MB)
   free     = 9613280 (9.167938232421875MB)
   69.44020589192708% used
Eden Space:
   capacity = 27983872 (26.6875MB)
   used     = 21844000 (20.832061767578125MB)
   free     = 6139872 (5.855438232421875MB)
   78.05924784104215% used
From Space:
   capacity = 3473408 (3.3125MB)
   used     = 0 (0.0MB)
   free     = 3473408 (3.3125MB)
   0.0% used
To Space:
   capacity = 3473408 (3.3125MB)
   used     = 0 (0.0MB)
   free     = 3473408 (3.3125MB)
   0.0% used
tenured generation:
   capacity = 69926912 (66.6875MB)
   used     = 0 (0.0MB)
   free     = 69926912 (66.6875MB)
   0.0% used

1787 interned Strings occupying 159472 bytes.
```

### jhat(JVM Heap Anylysis Tool)
虚拟机堆转存快照分析工具。与jmap搭配使用，主要用来分析jmap生成的dump文件，但是这个工具比较简陋，所以用的比较少。

### jstack(Stack Trace for Java)
Java堆栈跟踪工具。 该命令用于生成 java 虚拟机当前时刻的**线程快照**(一般称为threaddump或者javacore文件)。线程快照是当前 java 虚拟机内 每一条线程正在执行的方法堆栈的集合。生成线程快照的主要目的是定位线程出现 长时间停顿 的原因，如 线程间死锁、死循环、请求外部资源 导致的 长时间等待等等。

命令格式： 
```
jstack [options] pid
```
option值选项：
```
f         当正常输出请求 不被响应 时，强制输出 线程堆栈
l         输出锁信息
m         当调用本地方法时，可以显示C++堆栈
```
在JDK1.5中，java.lang.Thread类新增了一个getAllStackTraces()方法用来获取虚拟机中的素有线程的StackTraceElement对象。使用这个方法可以通过简单的几行代码就完成jstack的大部分功能，在实际项目中可以使用这个方法做一个管理员界面，可以随时使用浏览器来查看线程堆栈。
```html
<%@ page import="java.util.Map"%>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>服务器线程信息</title>
</head>
<body>
<pre>
<%
    for (Map.Entry<Thread, StackTraceElement[]> stackTrace : Thread.getAllStackTraces().entrySet()){
        Thread thread = (Thread) stackTrace.getKey();
        StackTraceElement[] stack = (StackTraceElement[]) stackTrace.getValue();
        if(thread.equals(Thread.currentThread())){
            continue;
        }
        System.out.println("线程： "+ thread.getName());
        System.out.println();
        for (StackTraceElement element : stack){
            System.out.println(element);
        }
    }
%>
</pre>
</body>
</html>
```
## 可视化工具
## JConsole

## VisualVM