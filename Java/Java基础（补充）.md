# Java基础（补充）

---
# 一、数据类型
## 1. int和Integer的区别
### 1.1 int和Integer的基本使用对比
（1）Integer是int的包装类；int是基本数据类型； 
（2）Integer变量必须实例化后才能使用；int变量不需要； 
（3）Integer实际是对象的引用，指向此new的Integer对象；int是直接存储数据值 ； 
（4）Integer的默认值是null；int的默认值是0。
### 1.2 int和Integer的深入比较
（1）由于Integer变量实际上是对一个Integer对象的引用，所以两个通过new生成的Integer变量永远是不相等的（因为new生成的是两个对象，其内存地址不同）。
```java
Integer i = new Integer(100);
Integer j = new Integer(100);
System.out.print(i == j); //false
```
（2）Integer变量和int变量比较时，***只要两个变量的值是相等的***，则结果为true（因为包装类Integer和基本数据类型int比较时，java会自动拆包装为int，然后进行比较，实际上就变为两个int变量的比较）
```java
Integer i = new Integer(100);
int j = 100；
System.out.print(i == j); //true
```
（3）非new生成的Integer变量和new Integer()生成的变量比较时，结果为false。（因为非new生成的Integer变量指向的是java常量池中的对象(前提是在-128-127之间，超过这个会new对象)，而new Integer()生成的变量指向堆中新建的对象，两者在内存中的地址不同）
```java
Integer i = new Integer(100);
Integer j = 100;
System.out.print(i == j); //false
```
（4）对于两个非new生成的Integer对象，进行比较时，如果两个变量的值在区间-128到127之间，则比较结果为true，如果两个变量的值不在此区间，则比较结果为false。原因是Integer 缓存池的大小默认为 -128~127。
```java
Integer i = 100;
Integer j = 100;
System.out.print(i == j); //true

Integer i = 128;
Integer j = 128;
System.out.print(i == j); //false
```
### 1.3 自动装箱和自动拆箱
（1）自动装箱：将基本数据类型重新转化为对象
```java
public class Test {  
    public static void main(String[] args) {  
        //声明一个Integer对象
        Integer num = 9;

        //以上的声明就是用到了自动的装箱：解析为:Integer num = Integer.valueOf(9);
    }  
}  
```
(2)自动拆箱：将对象重新转化为基本数据类型
```java
 public class Test {  
        public static void main(String[] args) {  
            //声明一个Integer对象
            Integer num = 9;

            //进行计算时隐含的有自动拆箱
            System.out.print(num--);
        }  
}  
```
因为对象时不能直接进行运算的，而是要转化为基本数据类型后才能进行加减乘除。对比：
```java
//装箱
Integer num = 10;
//拆箱
int num1 = num;
```
## 小结1：使用Integer产生变量时有以下三种方式:
```java
Integer num = new Integer(100);   //堆中
Integer num = 100;                //可能在缓存池中(-128-127之间时；超过就new对象)
Integer num = Integer.valueOf(100)//可能在缓存池中(-128-127之间时；超过就new对象)
```
valueOf() 方法的实现比较简单，就是先判断值是否在缓存池中，如果在的话就直接返回缓存池的内容。
```java
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```

**注意，不仅是Integer变量能用缓存池,Byte,Short,Long,Character也能用。**

## 小结2：Integer缓存池和Java常量池
&emsp; 前面好像“缓存池”的概念和“常量池”混用了，其实缓存池就是常量池技术的一种实现。缓存池其实是利用一个Integer类型的数组来实现的，而这个数组被声明为final类型，所以放在常量池中。见下面的程序：
```java
private static class IntegerCache {
    static final int high;
    static final Integer cache[];
    static {
        final int low = -128;
        // high value may be configured by property
        int h = 127;
        if (integerCacheHighPropValue != null) {
        // Use Long.decode here to avoid invoking methods that
        // require Integer's autoboxing cache to be initialized
        int i = Long.decode(integerCacheHighPropValue).intValue();
        i = Math.max(i, 127);// Maximum array size is Integer.MAX_VALUE
        h = Math.min(i, Integer.MAX_VALUE - -low);
        }
        high = h;
        cache = new Integer[(high - low) + 1];
        int j = low;
        for(int k = 0; k < cache.length; k++)
            cache[k] = new Integer(j++);
    }
    private IntegerCache(){}
}
```
## 小结3：Integer和int在内存中的哪里
```java
int a= 200; 
Integer b = new Integer(200); 
Integer c= 200;
```
- 局部变量应该是放在栈中，即int a= 200;a和200都是栈中。
- 引用b在栈中，指向堆中new 的的Integer对象。
- Integer C=200;这种方式定义C的值时，如果C在+-128之间是在常量池中取的，而大于这个范围就是直接new。
# 二、String
## 1. 关于String不可变
### 1.1 什么是对象不可变
可以这样认为：如果一个对象，在它创建完成之后，不能再改变它的状态，那么这个对象就是不可变的。不能改变状态的意思是，不能改变对象内的成员变量，包括基本数据类型的值不能改变，引用类型的变量不能指向其他的对象，引用类型指向的对象的状态也不能改变。
### 1.2 区分对象和对象的引用
先看如下代码：
```java
String s = "ABCabc";
System.out.println("s = " + s);
 
s = "123456";
System.out.println("s = " + s);

```
打印结果为：
```java
s = ABCabc
s = 123456
```
从打印结果可以看出，s的值确实改变了。那么怎么还说String对象是不可变的呢？ 其实这里存在一个误区： s只是一个String对象的引用，并不是对象本身。对象在内存中是一块内存区，成员变量越多，这块内存区占的空间越大。引用只是一个4字节的数据，里面存放了它所指向的对象的地址，通过这个地址可以访问对象。如下图所示：
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/StringResistance.jpg" width="400" height="300">
</center>
### 1.3 String、StringBuffer和StringBuilder
#### 1.3.1 可变与不可变
String类中使用字符数组保存字符串，如下就是，因为有“final”修饰符，所以可以知道string对象是不可变的。
```java
private final char value[];
```
StringBuilder与StringBuffer都继承自AbstractStringBuilder类，在AbstractStringBuilder中也是使用字符数组保存字符串，如下就是，可知这两种对象都是可变的。
```java
char[] value;
```
#### 1.3.2 是否多线程安全
- String中的对象是不可变的，也就可以理解为常量，显然线程安全。
- AbstractStringBuilder是StringBuilder与StringBuffer的公共父类，定义了一些字符串的基本操作，如expandCapacity、append、insert、indexOf等公共方法。
- StringBuffer对方法加了同步锁或者对调用的方法加了同步锁，所以是线程安全的。

---
#### ***注意：***
对String的操作无论是sub操、concat还是replace操作都不是在原有的字符串上进行的，而是重新生成了一个新的字符串对象。也就是说进行这些操作后，最原始的字符串并没有被改变。
所以要永远记住一点：
***“对String对象的任何改变都不影响到原对象，相关的任何change操作都会生成新的对象”。***

---

### 1.3.3 深入理解String、StringBuffer和StringBuilder
#### (1)`String str="hello world"`和`String str=new String("hello world")`的区别
例子：
```java
public class Main {
         
    public static void main(String[] args) {
        String str1 = "hello world";
        String str2 = new String("hello world");
        String str3 = "hello world";
        String str4 = new String("hello world");
         
        System.out.println(str1==str2);
        System.out.println(str1==str3);
        System.out.println(str2==str4);
    }
}
```
输出结果:
```
false
true
false
```
原因：
&emsp; 在`class`文件中有一部分 来存储编译期间生成的 **字面常量以及符号引用**，这部分叫做**class文件常量池**，在运行期间对应着**方法区的运行时常量池**。

&emsp; 因此在上述代码中，`String str1 = "hello world";`和`String str3 = "hello world"`; 都在编译期间生成了字面常量和符号引用，运行期间字面常量`"hello world"`被存储在运行时常量池（当然只保存了一份）。通过这种方式来将`String`对象跟引用绑定的话，`JVM`执行引擎会先在运行时常量池查找是否存在相同的字面常量，如果存在，则直接将引用指向已经存在的字面常量；否则在运行时常量池开辟一个空间来存储该字面常量，并将引用指向该字面常量。

&emsp; 众所周知，通过`new`关键字来生成对象是在堆区进行的，而在堆区进行对象生成的过程是不会去检测该对象是否已经存在的。因此通过`new`来创建对象，创建出的一定是不同的对象，即使字符串的内容是相同的。
#### (2)`String`、`StringBuffer`以及`StringBuilder`的区别
&emsp; 既然在Java中已经存在了String类，那为什么还需要StringBuilder和StringBuffer类呢？看代码：
```java
public class Main {
         
    public static void main(String[] args) {
        String string = "";
        for(int i=0;i<10000;i++){
            string += "hello";
        }
    }
}
```
&emsp; 这句 `string += "hello";`的过程相当于将原有的string变量指向的对象内容取出与"hello"作字符串相加操作再存进另一个新的String对象当中，再让string变量指向新生成的对象。每次循环会new出一个StringBuilder对象，然后进行append操作，最后通过toString方法返回String对象。也就是说这个循环执行完毕new出了10000个对象，试想一下，如果这些对象没有被回收，会造成多大的内存资源浪费。
&emsp; 而使用StringBuilder时new操作只进行了一次，也就是说只生成了一个对象，append操作是在原有对象的基础上进行的。
#### (3)三者对比
&emsp; 1）对于直接相加字符串，效率很高，因为在编译器便确定了它的值，也就是说形如"I"+"love"+"java"; 的字符串相加，在编译期间便被优化成了"Ilovejava"。这个可以用javap -c命令反编译生成的class文件进行验证。

&emsp; 对于间接相加（即包含字符串引用），形如s1+s2+s3; 效率要比直接相加低，因为在编译器不会对引用变量进行优化。

&emsp; 2）String、StringBuilder、StringBuffer三者的执行效率：
```
StringBuilder > StringBuffer > String
```
#### (4)常见面试试题
- 下面这段代码的输出结果是什么？
```java
String a = "hello2"; 　　
String b = "hello" + 2; 　　
System.out.println((a == b));
```
&emsp; 输出结果为：`true`。原因很简单，"hello"+2在编译期间就已经被优化成"hello2"，因此在运行期间，变量a和变量b指向的是同一个对象。

- 下面这段代码的输出结果是什么？
```java
String a = "hello2"; 　  
String b = "hello";       
String c = b + 2;       
System.out.println((a == c));
```
&emsp; 输出结果为:`false`。由于有符号引用(即字符串引用)的存在，所以  String c = b + 2;不会在编译期间被优化，不会把b+2当做字面常量来处理的，因此这种方式生成的对象事实上是保存在堆上的。

- 下面这段代码的输出结果是什么？
```java
String a = "hello2";   　 
final String b = "hello";       
String c = b + 2;       
System.out.println((a == c));
```
&emsp; 输出结果为：`true`。对于被final修饰的变量，会在class文件常量池中保存一个副本，也就是说不会通过连接而进行访问，对final变量的访问在编译期间都会直接被替代为真实的值。那么String c = b + 2;在编译期间就会被优化成：String c = "hello" + 2; 

- 下面这段代码输出结果是什么？
```java
public class Main {
    public static void main(String[] args) {
        String a = "hello";
        String b =  new String("hello");
        String c =  new String("hello");
        String d = b.intern();
         
        System.out.println(a==b);
        System.out.println(b==c);
        System.out.println(b==d);
        System.out.println(a==d);
    }
}
```
结果为（jdk1.6、jdk1.8）：
```java
false
false
false
true
```
&emsp; 这里面涉及到的是String.intern方法的使用。在String类中，intern方法是一个本地方法，在JAVA SE6之前，intern方法会在运行时常量池中查找是否存在内容相同的字符串，如果存在则返回指向该字符串的引用，如果不存在，则会将该字符串入池，并返回一个指向该字符串的引用。因此，a和d指向的是同一个对象。

- String str = new String("abc")创建了多少个对象？
&emsp; 这个问题在很多书籍上都有说到比如《Java程序员面试宝典》，包括很多国内大公司笔试面试题都会遇到，大部分网上流传的以及一些面试书籍上都说是2个对象，这种说法是片面的。因为只调用了一次new，显然只创建了一个对象。

&emsp; 而这道题目让人混淆的地方就是这里，这段代码在运行期间确实只创建了一个对象，即在堆上创建了"abc"对象。而为什么大家都在说是2个对象呢，这里面要澄清一个概念  该段代码执行过程和类的加载过程是有区别的。在类加载的过程中，确实在运行时常量池中创建了一个"abc"对象，而在代码执行过程中确实只创建了一个String对象。

&emsp; 因此，这个问题如果换成 String str = new String("abc")涉及到几个String对象？合理的解释是2个。

&emsp; 个人觉得在面试的时候如果遇到这个问题，可以向面试官询问清楚”是这段代码执行过程中创建了多少个对象还是涉及到多少个对象“再根据具体的来进行回答。

- 下面这段代码1）和2）的区别是什么？
```java
public class Main {
    public static void main(String[] args) {
        String str1 = "I";
        //str1 += "love"+"java";        1)
        str1 = str1+"love"+"java";      //2)
         
    }
}
```

&emsp; 1）的效率比2）的效率要高，1）中的"love"+"java"在编译期间会被优化成"lovejava"，而2）中的不会被优化。在1）中只进行了一次append操作，而在2）中进行了两次append操作。可以使用`javap -c 类名`来查看字节码，比如`javap -c Main`。

### 1.4 String#intern
#### 1.4.1 intern实现原理
&emsp; “如果常量池中存在当前字符串, 就会直接返回当前字符串. 如果常量池中没有此字符串, 会将此字符串放入常量池中后, 再返回”。
#### 1.4.2 jdk6 和 jdk7 下 intern 的区别
&emsp; 看一段代码：
```java
public static void main(String[] args) {
    String s = new String("1");
    s.intern();
    String s2 = "1";
    System.out.println(s == s2);

    String s3 = new String("1") + new String("1");
    s3.intern();
    String s4 = "11";
    System.out.println(s3 == s4);
}
```
&emsp; 打印结果是：

- jdk6下`false false`
- jdk7下`false true`
&emsp; 然后将`s3.intern();`语句下调一行，放到`String s4 = "11";`后面。将`s.intern();` 放到`String s2 = "1";`后面。是什么结果呢
```java
public static void main(String[] args) {
    String s = new String("1");
    String s2 = "1";
    s.intern();
    System.out.println(s == s2);

    String s3 = new String("1") + new String("1");
    String s4 = "11";
    s3.intern();
    System.out.println(s3 == s4);
}
```
&ensp; 打印结果为：

- jdk6下`false false`
- jdk7下`false false`

#### ***jdk6中的解释***
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/jdk6.png">
***注：图中绿色线条代表 string 对象的内容指向。 黑色线条代表地址指向。***

&emsp; 在 jdk6中上述的所有打印都是 false 的，因为 jdk6中的常量池是放在 Perm 区中的，Perm 区和正常的 JAVA Heap 区域是完全分开的。上面说过如果是使用引号声明的字符串都是会直接在字符串常量池中生成，而 new 出来的 String 对象是放在 JAVA Heap 区域。所以拿一个 JAVA Heap 区域的对象地址和字符串常量池的对象地址进行比较肯定是不相同的，即使调用String.intern方法也是没有任何关系的。
#### ***jdk7中的解释***
&emsp; 在 Jdk6 以及以前的版本中，字符串的常量池是放在堆的 Perm 区的，Perm 区是一个类静态的区域，主要存储一些加载类的信息，常量池，方法片段等内容，默认大小只有4m，一旦常量池中大量使用 intern 是会直接产生`java.lang.OutOfMemoryError: PermGen space`错误的。 所以在 jdk7 的版本中，字符串常量池已经从 Perm 区移到正常的 Java Heap 区域了。为什么要移动，Perm 区域太小是一个主要原因，当然据消息称 jdk8 已经直接取消了 Perm 区域，而新建立了一个元区域。应该是 jdk 开发者认为 Perm 区域已经不适合现在 JAVA 的发展了。
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/jdk7_1.png">

- 在第一段代码中，先看 s3和s4字符串。`String s3 = new String("1") + new String("1");`，这句代码中现在生成了2最终个对象，是字符串常量池中的“1” 和 JAVA Heap 中的 s3引用指向的对象。中间还有2个匿名的`new String("1")`我们不去讨论它们。此时s3引用对象内容是"11"，但此时常量池中是没有 “11”对象的。
- 接下来`s3.intern();`这一句代码，是将 s3中的“11”字符串放入 String 常量池中，因为此时常量池中不存在“11”字符串，因此常规做法是跟 jdk6 图中表示的那样，在常量池中生成一个 "11" 的对象，关键点是 jdk7 中常量池不在 Perm 区域了，这块做了调整。常量池中不需要再存储一份对象了，可以直接存储堆中的引用。这份引用指向 s3 引用的对象。 也就是说引用地址是相同的。
- 最后`String s4 = "11";` 这句代码中"11"是显示声明的，因此会直接去常量池中创建，创建的时候发现已经有这个对象了，此时也就是指向 s3 引用对象的一个引用。所以 s4 引用就指向和 s3 一样了。因此最后的比较 `s3 == s4` 是 true。
- 再看 s 和 s2 对象。 `String s = new String("1");` 第一句代码，生成了2个对象。常量池中的“1” 和 JAVA Heap 中的字符串对象。`s.intern();` 这一句是 s 对象去常量池中寻找后发现 “1” 已经在常量池里了。
- 接下来`String s2 = "1";` 这句代码是生成一个 s2的引用指向常量池中的“1”对象。 结果就是 s 和 s2 的引用地址明显不同。图中画的很清晰。

<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/jdk7_2.png">

- 来看第二段代码，从上边第二幅图中观察。第一段代码和第二段代码的改变就是 `s3.intern();` 的顺序是放在`String s4 = "11";`后了。这样，首先执行`String s4 = "11";`声明 s4 的时候常量池中是不存在“11”对象的，执行完毕后，“11“对象是 s4 声明产生的新对象。然后再执行`s3.intern();`时，常量池中“11”对象已经存在了，因此 s3 和 s4 的引用是不同的。
- 第二段代码中的 s 和 s2 代码中，`s.intern();`，这一句往后放也不会有什么影响了，因为对象池中在执行第一句代码`String s = new String("1");`的时候已经生成“1”对象了。下边的s2声明都是直接从常量池中取地址引用的。 s 和 s2 的引用地址是不会相等的。

#### ***小结***
&emsp; 从上述的例子代码可以看出 jdk7 版本对 intern 操作和常量池都做了一定的修改。主要包括2点：

- 将String常量池 从 Perm 区移动到了 Java Heap区
- String#intern 方法时，如果存在堆中的对象，会直接保存对象的引用，而不会重新创建对象。

### 1.5 从字符串比较到==和equals方法区别 
首先看一段代码：
```java
String str1 = new String("hello");
String str2 = "hello";
 
System.out.println("str1==str2： " + (str1==str2));  \\1
System.out.println("str1.equals(str2)： " + str1.equals(str2));  \\2
```
打印结果为：
`false`
`true`
原因：
String类中，`==`比较的是两个元素的地址，代码中一个在堆中，一个在常量池中，显然不同；equals()方法首先比较两个字符串的长度，长度不同则返回`false`，否则再比较字符串的每一位，不同返回`false`，若每一位都相同，则返回`true`。“比较每一位”的操作是通过一个`char`类型的数组完成的，如下图所示：
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/String%23equals.jpg">

说完了`String`，再看一下通用的`==`和`equals`的区别。
java中的数据类型，可分为两类： 
1.基本数据类型，也称原始数据类型。byte,short,char,int,long,float,double,boolean 
它们之间的比较，应用双等号（==）,比较的是它们的值。 
2.复合数据类型(类) 
当它们用（==）进行比较的时候，比较的是它们在内存中的存放地址，所以，除非是同一个new出来的对象，比较后的结果为true，否则比较后结果为false。 JAVA当中所有的类都是继承于Object这个基类的，在Object中的基类中定义了一个equals的方法，这个方法的初始行为是比较对象的内存地址，但在一些类库当中这个方法被覆盖掉了，如String,Integer,Date在这些类当中equals有其自身的实现，而不再是比较类在堆内存中的存放地址了。
对于复合数据类型之间进行equals比较，在没有覆写equals方法的情况下，他们之间的比较还是基于他们在内存中的存放位置的地址值的，因为Object的equals方法也是用双等号（==）进行比较的，所以比较后的结果跟双等号（==）的结果相同。

# 三、运算
## 1. switch
switch支持的类型：

- 基本数据类型：byte, short, char, int
- 包装数据类型：Byte, Short, Character, Integer
- 枚举类型：Enum
- 字符串类型：String（Jdk 7+ 开始支持）

## 参数传递
首先说明，**Java 语言的参数传递只有「按值传递」**

然而我们经常看到对于对象（数组，类，接口）的传递似乎有点像引用传递，可以改变对象中某个属性的值。但是不要被这个假象所蒙蔽，实际上这个传入函数的值是**对象引用的拷贝，即传递的是引用的地址值，是将对象的地址以值的方式传递到形参中，方法得到的是所有参数值的一个拷贝，所以还是按值传递。**

方法参数共有两种类型：

- 基本数据类型
- 对象引用

### 基本数据类型为参数
先看一段代码：
```java
public static void main(String[] args) {
    int num = 1;
    System.out.println(num);

    changeNumber(num);

    System.out.println(num);
}

private static void changeNumber(int x){
    x++;
}
```
打印结果为：
```java
1
1
```
以上程序的执行过程为：

- x被初始化为num的一个拷贝，值为1；
- x执行`++`操作，但是num并没有改变；
- 方法调用完成之后，参数不再使用，num的值从始至终没有改变。

### 引用为参数
```java
public static void main(String[] args) {
    int[] nums = {3,2,1};

    changeNums(nums);
    
    for(int i = 0; i < nums.length; i++)
        System.out.print(nums[i] + " ");
}

private static void changeNums(int[] nums){
    for(int i = 0; i < nums.length; i++)
        nums[i]++;
}
```
打印结果为：
```java
4 3 2
```
可以看到，调用`changeNums`函数之后，nums数组的值被改变了。

以上程序的执行过程为：

- `changeNums`方法的参数nums初始化为数组nums的拷贝，参数nums的值为数组nums的地址，所以，这两个指向同一个内存空间；
- 对参数nums指向的内存空间的值进行`++`操作，可以看到，原nums数组也被修改。

再看另一个例子：
```java
public class Dog {

    String name;

    Dog(String name) {
        this.name = name;
    }

    String getName() {
        return this.name;
    }

    void setName(String name) {
        this.name = name;
    }

    String getObjectAddress() {
        return super.toString();
    }
}
public class PassByValueExample {
    public static void main(String[] args) {
        Dog dog = new Dog("A");
        System.out.println(dog.getObjectAddress()); // Dog@4554617c
        func(dog);
        System.out.println(dog.getObjectAddress()); // Dog@4554617c
        System.out.println(dog.getName());          // A
    }

    private static void func(Dog dog) {             //调用该方法后，dog被初始化为A第一个拷贝
        System.out.println(dog.getObjectAddress()); // Dog@4554617c
        dog = new Dog("B");                         //将A的拷贝指向另一个对象，对A没有任何影响
        System.out.println(dog.getObjectAddress()); // Dog@74a14482
        System.out.println(dog.getName());          // B
    }
}
```
如果在方法中改变对象的字段值会改变原对象该字段值，因为改变的是同一个地址指向的内容。
```java
class PassByValueExample {
    public static void main(String[] args) {
        Dog dog = new Dog("A");
        func(dog);
        System.out.println(dog.getName());          // B
    }

    private static void func(Dog dog) {
        dog.setName("B");
    }
}
```

---
参考：
[java的值传递和引用传递的问题](https://www.cnblogs.com/coderising/p/5697986.html)
[java值传递还是引用传递](https://www.cnblogs.com/xiaoxiaoyihan/p/4883770.html)
[Java的参数传递是「按值传递」还是「按引用传递」？](https://www.cnblogs.com/nnngu/p/8299724.html)

---

# 四、继承
## 1. 继承关系图
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/JavaExtends.png" width="500">
</center>
## 2. 抽象类和接口
### 2.1 为什么接口中的成员变量必须是`public static final`的？
&emsp; 首先明白一个原理，就是接口的存在意义。接口就是为了实现多继承的抽象类，是一种高度抽象的模板、标准或者说协议。规定了什么东西该是这样，如果你继承了我这接口，就必须这样。比如USB接口，就是小方口，两根电源线和两根数据线，不能多不能少。
（1）public
&emsp; 既然是公共的模板或者协议，那么如果定义成private就没有意义了，因为所有继承了你这接口的类都不能用，并且接口中的方法是不能够被具体实现的，因此，接口内部中也没有任何方法可使用。为了让所有实现了该接口的类能够使用，就必须是public的。接口中定义的所有东西就应该是对所有用户开放的东西。
 
（2）static
&emsp; 如果接口中的成员变量是非静态的，那么每一个实现了该接口的类都会有这么一个变量。那么，因为接口是多继承的，那么如果另一个接口也是有同样这样一个变量呢，那你用哪一个？所以，因为是标准，所以我规定从一开始，这个东西只能有一份，只能放在静态存储区，如果第二个接口也想同命名这么一个变量，那么存储时候就会报错，因为我静态存储区已经有一份了。你改名吧。
 
（3）final
&emsp; 想想，如果不是final的，那么意味着每一个实现了该接口的子类都可以去修改这个变量。我们开头说了，接口就是标准规范，也改也只能是制定该接口的架构师来改，如果某类随便改的话，那么其他也继承了该接口的类就会受到影响。牵一发而动全身！！因此，既然是标准，那么就不能改，方便管理。
 
最后归纳：

- public是因为接口是标准，必须对外完全开放，自己藏着掖着没意义；
- static是因为要确保该变量只有一份，避免重名；
- final是因为接口的东西是大家共用的，不能随便修改，因此干脆不然你有修改的权限！

## 3. super()和this()

- super()是引用父类的构造函数
- super(形参)是引用父类的有参构造函数
- this(形参)是引用自己的有参构造函数

这三个语句都必须放在第一行。如果什么都不写，会默认调用super()；如果写了this(形参)，那么会调用自己的带参构造函数，在带参构造函数中又会调用super()。例子如下：

```
public class SuperClass {
    private String name;
    public SuperClass(){
        this("super");
        System.out.println("super class ()");
    }

    public SuperClass(String name){
        this.name = name;
        System.out.println("super class ("+name+")");
    }
}

public class SonClass extends SuperClass {
    public SonClass(){
        this("son");
        System.out.println("son class ()");
    }

    public SonClass(String name){
        System.out.println("son class ("+name+")");
    }
}

public class main {
    public static void main(String[] args) {
        SonClass sonClass = new SonClass();
    }
}
```
结果如下：
```
super class (super)
super class ()
son class (son)
son class ()
```

经过上面的例子可以看出，进行子类构造函数初始化之前需要先调用父类的构造哈数，但是如果在调用子类构造函数的时候执行了一条语句，那么这条语句是先于父类构造函数调用之前执行呢还是在父类构造函数调用之后执行？**答案是会先执行这条语句，然后再调用父类的构造函数**。例子如下，是根据《Java编程思想》上的一个例子扩展的。
```
public class Flower extends SuperFlower {
    int petalCount = 0;
    String s = "init value";

    //3
    Flower(int petais){
        this(new soutClass());  //4
        petalCount = petais;
        System.out.println("Constructor W/ int arg only. petalCount= "+ petalCount);
    }

    //2
    Flower(String s, int petals){
        this(petals);
        this.s = s;
        System.out.println("string & int args");
    }

    //1
    Flower(){
        this("hi", 47);
        System.out.println("default constructor (no args)");
    }

    //5
    Flower(soutClass sc){

    }

    void printPetalCount(){
        System.out.println("petalCount = "+ petalCount + " s = "+s);
    }

    public static void main(String[] args) {
        Flower f = new Flower();
        f.printPetalCount();
    }

    private static class soutClass {
        soutClass(){
            System.out.println("i am sout class");
        }
    }
}
```
这个例子中，main函数执行时首先会调用无参构造函数1,1会调用2,2会调用3,3会调用5。3调用5的时候需要执行一条new语句示例一个soutClass的对象，所以会调用soutClass的构造函数，输出一个i am sout class语句。

这个例子的执行结果如下，可以看到，i am sout class语句先于i am super flower执行，也就是说，如果子类在调用构造函数的时候执行了一条语句，这条语句会先于父类的构造函数执行。

需要注意的是，soutClass类要被声明为static，否则会报Cannot reference XXX before supertype constructor has been called错。原因是父类构造函数初始化早于子类非静态变量的初始化，晚于子类静态变量的初始化。这条语句先于父类构造函数执行，soutClass必须为静态的。
```
i am sout class
i am super flower
Constructor W/ int arg only. petalCount= 47
string & int args
default constructor (no args)
petalCount = 47 s = hi
```

# 五、反射
JAVA反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法；这种动态获取的信息以及动态调用对象的方法的功能称为java语言的反射机制。
Java反射机制主要提供了以下功能： 在运行时判断任意一个对象所属的类；在运行时构造任意一个类的对象；在运行时判断任意一个类所具有的成员变量和方法；在运行时调用任意一个对象的方法；生成动态代理。

## 1. 反射的三种实现方式
### 1.1 对象名.getClass()
```java
public static void getClassDemo() {
    Person p = new Person();
    Class clazz1 = p.getClass();
    
    Person p1 = new Person();
    Class clazz2 = p.getClass();

    System.out.println(clazz1 == clazz2); //true
}
```
### 1.2 类名.class
```java
public static void getClassDemo() {
    Class clazz1 = Person.class;
    Class clazz2 = Person.class;
    System.out.println(clazz1 == clazz2); //true
}
```
### 1.3 类所在的绝对路径
```java
public static void getClassDemo() throws ClassNotFoundException {
    String className = "Test.Person";
    Class clazz = Class.forName(className);
    System.out.println(clazz);
}
```

## 2. Java反射可以实现的功能：

- 在运行时判断任意一个对象所属的类
- 在运行时构造任意一个类的对象
- 在运行时判断任意一个类所具有的方法和属性
- 在运行时调用任意一个对象的方法
- 生成动态代理

## 3. JAVA反射API：反射API用来生成当前JAVA虚拟机中的类、接口或对象的信息

|-|-|
|-|-|
|Class类|反射的核心类，可以获取类的属性、方法等内容信息|
|Field类|Field和Method和Constructor类都定义于Java.lang.reflect包下，Field表示类的属性，可以获取和设置类中属性的值|
|Method类|表示类的方法，它可以用来获取类中方法的信息，或者执行方法|
|Constructor类|表示类的构造方法|

### 使用反射获得所有构造方法（包括私有的，非私有的）

- public Constructor getConstructor(Class… parameterTypes)	获得指定的构造方法，注意只能获得 public 权限的构造方法，其他访问权限的获取不到
- public Constructor getDeclaredConstructor(Class… parameterTypes)	获得指定的构造方法，注意可以获取到任何访问权限的构造方法。
- public Constructor[] getConstructors() throws SecurityException	获得所有 public 访问权限的构造方法
- public Constructor[] getDeclaredConstructors() throws SecurityException	获得所有的构造方法，包括（public, private,protected,默认权限的）

### 使用反射获得所有的 Filed 变量

- getDeclaredFileds() 获取所有字段包括private字段
- getFileds() 获取所有可以访问的字段，包括父类的
- getFiled(String) 根据名字获取字段，包括其父类，如果不存在抛出异常
- getDeclaredFiled(String) 根据名字其声明获取字段，如果名字不存在抛出异常

### 使用反射执行相应的 Method

- public Method[] getDeclaredMethods()
- public Method[] getMethods() throws SecurityException
- public Method getDeclaredMethod()
- public Method getMethod(String name, Class

## Constructor类
调用newInstance()方法会去使用该类的空参数构造函数进行初始化，如果指定的类中没有空参数的构造函数，或者要创建的类对象需要通过指定的构造函数进行初始化。这时就用到Constructor类了。Constructor<?> 表示的是?类的构造器，调用Constructor中的newInstance()方法其实是调用的?类的构造方法。比如下面这样写，其实调用的是?类的构造函数进行的初始化：

```
private final Constructor<? extends T> construcor;
...
constructor.newInstance();
```

参考：
[Java反射详解](https://cloud.tencent.com/developer/article/1163271)
[Java 反射机制详解](https://blog.csdn.net/gdutxiaoxu/article/details/68947735)
[JAVA反射详解](https://blog.csdn.net/qwert2190/article/details/81256607)
[黑马程序员——Java基础---反射Class类、Constructor类、Field类](https://www.cnblogs.com/ktlshy/p/4716838.html)

---

关于反射的进一步知识，参考下面几个链接
https://www.importnew.com/23902.html
https://mp.weixin.qq.com/s/5H6UHcP6kvR2X5hTj_SBjA
https://www.cnblogs.com/chanshuyi/p/head_first_of_reflection.html
https://www.cnblogs.com/dongguacai/p/6535417.html

# 六、对象创建的过程
&emsp;在存在继承时，并且子类和父类中包括静态变量、静态代码块和普通代码块的时候，变量的初始过程如下：
> 
&emsp; 1. 启动`JVM`；
&emsp; 2. 将主函数所在的`Test.class`文件加载进入方法区中，加载过程静态内容要加载进入静态区；
&emsp; 3. 执行`main`方法。`JVM`会将`main`方法加载到栈中，从第一行开始执行；
&emsp; 4. 执行`new Child()`。`JVM`会在方法区中查找是否有`Child`文件，如果没有就加载`Child.class`文件(从哪里加载呢？参见类加载过程)；如果`Child`有直接父类，首先查找父类是否存在方法区中，如果不在要先加载父类；
&emsp; `Child.class`和`Parent.class`中的所有的非静态内容会加载到非静态的区域中，而静态的内容会加载到静态区中。静态内容（静态变量，静态代码块，静态方法）按照书写顺序加载；
&emsp; *说明：类的加载只会执行一次。下次再创建对象时，可以直接在方法区中获取`class`信息。*
&emsp; 5. 开始给静态区中的所有静态的成员变量默认初始化。默认初始化完成之后，给所有的静态成员变量显示初始化；
&emsp; 6. 所有静态成员变量显示初始化完成之后，开始执行静态的代码块。先执行父类的静态代码块，再执行子类的静态代码块；
*&emsp; 说明：静态代码块是在类加载的时候执行的，类的加载只会执行一次所以静态代码块也只会执行一次；
&emsp; 非静态代码块和构造函数中的代码是在对象创建的时候执行的，因此对象创建(`new`)一次，它们就会执行一次。*
&emsp; 这时`Parent.class`文件 和 `Child.class`文件加载完成；
&emsp; 7. 开始在堆中创建`Child`对象。给`Child`对象分配内存空间，其实就是分配内存地址；
&emsp; 8. 为对象的成员变量进行默认初始化；
&emsp; 9. 调用对象的构造方法，执行**隐式三步**：

>> &emsp; (1)隐式`super()`，首先需要执行父类的构造函数，对父类进行构造函数初始化；
&emsp; (2)子类对象成员变量的显示初始化；
&emsp; (3)执行子类的构造代码块；
&emsp; 接着才会执行子类构造代码块的剩余代码。
&emsp; 在第(1)步中，父类构造函数初始化的时候也有隐式三步，因为父类继承自超类`Object`。

> &emsp; 10. 将地址赋值给引用变量，对象初始化结束。
&emsp; 下面是一个例子：
```java
package NewAObject;

class Parent{
    int num = 100;
    static int staticNum = 101;

    static {
        System.out.println("Parent静态代码块：staticNum="+staticNum);
        staticNum++;
    }

    {
        System.out.println("Parent普通代码块：num="+num+" "+"staticNum="+staticNum);
        num++;
        staticNum++;
    }

    Parent(){
        super();
        System.out.println("Parent构造函数：num="+num+" "+"staticNum="+staticNum);
        num++;
        staticNum++;
        show();
        return;
    }
    void show(){
        System.out.println("Parent show函数：num="+num+" "+"staticNum="+staticNum);
    }
}

class Child extends Parent {
    int num = 1;
    static int staticNum = 2;

    static {
        System.out.println("Child静态代码块：staticNum="+staticNum);
        staticNum++;
    }

    {
        System.out.println("Child普通代码块：num+"+num+" "+"staticNum="+staticNum);
        num++;
        staticNum++;
    }

    Child(){
        super();
        //通过super初始化父类内容时，子类的成员并未显式初始化，而是父类初始化完毕时候才会进行显示初始化。
        System.out.println("Child构造函数：num="+num+" "+"staticNum="+staticNum);
        num++;
        staticNum++;
        return;
    }
    void show(){
        System.out.println("Child show函数：num="+ num+" "+"staticNum="+staticNum);
    }
}

public class NewAObject{
    public static void main(String[] args) {
        Child child = new Child();
        child.show();
    }
}
```
&emsp; 上述代码的执行结果为：
```java
Parent静态代码块：staticNum=101
Child静态代码块：staticNum=2
Parent普通代码块：num=100 staticNum=102
Parent构造函数：num=101 staticNum=103
Child show函数：num=0 staticNum=3
Child普通代码块：num+1 staticNum=3
Child构造函数：num=2 staticNum=4
Child show函数：num=3 staticNum=5
```

上述过程可简化为：

- 父类（静态变量、静态语句块）
- 子类（静态变量、静态语句块）
- 父类（实例变量、普通语句块）
- 父类（构造函数）
- 子类（实例变量、普通语句块）
- 子类（构造函数）

---
**补充：为何java中static静态数据无法访问非static数据，但是反过来却可以**

由上述过程可知，类在加载的时候会初始化static变量，但是没有对非static变量声明和初始化，如果我们在static方法中调用类非static变量的话，就极有可能出错，当然java是不允许的。所以在编译阶段，就会报错。

## 待定的一个问题
**补充**：好像有一个例外，**静态对象可以调用非静态方法**。这是为什么呢？现在还没搞懂。

---

# 七、成员变量和局部变量
## 定义
&emsp; 可以将变量分为**成员变量**、**静态变量**和**局部变量**，成员变量和静态变量是在类范围内定义的变量，其中成员变量又叫实例变量，不用static关键字修饰，静态变量又叫类变量，需要用static关键字修饰。局部变量是在一个方法内定义的变量。
&emsp; 局部变量可分为：形参（形式参数）、方法局部变量、代码块局部变量。
&emsp; 如下图所示：
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/javaviriable1.png">
</center>

## 区别
### 成员变量和局部变量的区别
- 在类中的位置不同
成员变量：在类中方法外
局部变量：在方法定义中或者方法声明上
- 在内存中的位置不同
成员变量：在堆内存
局部变量：如果是基本类型，会把值直接存储在栈；如果是引用类型，比如String s = new String("william");会把其对象存储在堆，而把这个对象的引用（指针）存储在栈。
- 生命周期不同
成员变量：随着对象的创建而存在，随着对象的消失而消失
局部变量：随着方法的调用而存在，随着方法的调用完毕而消失
- 初始化值不同
成员变量：有默认初始化值
局部变量：没有默认初始化值，必须定义，赋值，然后才能使用。
### 成员变量和静态变量的区别
- 成员变量所属于对象。所以也称为实例变量。
静态变量所属于类。所以也称为类变量。
- 成员变量，基本类型和引用类型的成员变量都在这个对象的空间中，作为一个整体存储在堆。
静态变量存在于方法区中，只有被调用的时候才会被压栈，放入栈中。
- 成员变量随着对象创建而存在。随着对象被回收而消失。
静态变量随着类的加载而存在。随着类的消失而消失。
- 成员变量只能被对象所调用 。
静态变量可以被对象调用，也可以被类名调用。

# 八、异常
## 1. throws和throw的区别
- throws用在函数上；throw用在函数内。
- throws抛出的是异常类，可以抛出多个，用逗号隔开；throw抛出的是异常对象。

***为什么有的异常throw之后还要再函数上throws一下，要不然就编译不通过而有些异常不用这样呢？***
&emsp; 如果一个异常是Exception，也就是编译时异常，那throw之后要么在函数上用throws一下，要么在本方法中catch一下。但是如果一个异常是RuntimeException，就不用。也就是说，如果自己定义的异常继承来自Exception，在函数中抛出时就必须声明或捕捉，如果是继承自RuntimeException，就不需要。
&emsp; 可以认为checked exception就是要强制去处理这个异常（不管throws多少层，终归要在某个地方catch它）；而runtime exception则没有这个限制，可以自由选择是否catch。
&emsp; 例子如下：
```java
public class MyException {
    public static void main(String[] args) {
        int[] arr = {1, 2, 3, 4, 5};

        Demo d = new Demo();
        d.method(arr, -1);
    }
}

/**
 * 继承自Exception，如果在Demo中的method方法里抛出这个异常，那么必须在Demo的method方法上声明
 * 或在method方法中捕捉；如果采用声明的解决办法，那么所有调用method的方法都需要声明
 */
class FuShuIndexException1 extends Exception{
    FuShuIndexException1(){

    }

    FuShuIndexException1(String msg){
        super(msg);
    }
}

/**
 * 继承自RuntimeException，如果在Demo中抛出这个异常，那么既不需要声明也不需要捕捉
 */
class FuShuIndexException2 extends RuntimeException{
    FuShuIndexException2(){

    }

    FuShuIndexException2(String msg){
        super(msg);
    }
}

class Demo{
    public int method(int[] arr, int index){
        if(arr ==null)
            throw new NullPointerException("数组引用不能为空");
        if(index > arr.length)
            throw new ArrayIndexOutOfBoundsException("数组指针越界");
        if(index < 0)
            throw new FuShuIndexException2("角标为如数==负数");

        return arr[index];
    }
}
```

# 九、泛型
## 泛型简介
泛型泛型，泛泛的类型。泛型就是一种不确定的类型。那么为什么要使用泛型？

使用泛型的好处大概有两个：

- 类型安全。将运行时的异常转变为编译时的异常。
- 代码重用。泛型合并了同类型的处理代码，使得代码可以很好重用。

**对于第一种好处**，可以举个例子：
```java
public static void main(String[] args) {
    List list = new ArrayList();
    list.add(10);
    list.add("adamhand");

    String num = (String) list.get(0);  //1
}
```
上面的代码在编译时期不会出现任何问题，但是运行的时候在1处却会出现类型转换错误：
```
Exception in thread "main" java.lang.ClassCastException: java.lang.Integer cannot be cast to java.lang.String
```
而使用泛型可以避免这个错误，使得错误在编译时期就会被检查，如下：
```java
public static void main(String[] args) {
    List<String> list = new ArrayList<>();
    list.add(10);   //1
    list.add("adamhand");

    String num = (String) list.get(0);
}
```
上述代码在编译的时候，1处就会报错。

**对于第二种好处**，使用Object似乎也可以实现，比如下面的代码：
```java
public class Stove {
    public static Object heat(Object food){
        System.out.println(food +"is done");
        return food;
    }

    public static void main(String[] args) {
        Meat meat = new Meat();
        meat = (Meat) Stove.heat(meat);

        Soup soup = new Soup();
        soup = (Soup) Stove.heat(soup);
    }
}
```
定义了一个微波炉的类，使用微波炉的加热方法对不同的食物进行加热。这种写法看似没问题，但是存在小瑕疵：如果客户端不知道加热的具体内容是什么，在取出的时候进行强制类型转换，可能会出现错误。

而使用泛型可以解决这个问题，上面的代码可以改为如下：
```java
public class Stove {
    public static <T> T heat(T food){
        System.out.println(food +"is done");
        return food;
    }

    public static void main(String[] args) {
        Meat meat = new Meat();
        meat = Stove.heat(meat);

        Soup soup = new Soup();
        soup = Stove.heat(soup);
    }
}
```
这样就可以免去强制类型转换了。

## 泛型类和泛型方法
需要注意的是，定义泛型方法的时候，需要将泛型定义在返回值之前；且静态方法无法使用类上定义的泛型
```java
/**
 * 泛型类
 * @param <T>
 */
public class GenericTest<T> {
    private T value;

    public T getObject(){
        return value;
    }

    /**
     * 泛型方法，将泛型定义现在返回值之前。
     * @param w
     * @param <W>
     */
    public <W> void method(W w){
        System.out.println("method:" + w);
    }

    /**
     * 当方法静态时，不能访问类上定义的泛型。如果静态方法使用泛型，
     * 只能将泛型定义在方法上。
     * @param y
     * @param <Y>
     */
    public static <Y> void staticMethod(Y y){
        System.out.println("staticmethod" + y);
    }
}
```

## compareTo
如果需要比较两个泛型数的大小，需要使用Compareable接口：
```java
public static <M extends Comparable<M>> int countGreaterThan(M[] array, M elem){
    int count = 0;
    for(M e : array){
        if(e.compareTo(elem) > 0){
            count++;
        }
    }
    return count;
}
```
调用方式如下：
```java
public static void main(String[] args) {
    Integer[] array = {1,2,3,4,5,6,7,8,9};
    GenericTest.countGreaterThan(array, 3);
}
```

## 通配符
在java中，数组是可以协变的，比如dog extends Animal，那么Animal[] 与dog[]是兼容的。而集合是不能协变的，也就是说`List<Animal>`不是`List<dog>`的父类，这时候就可以用到通配符了。

主要有三种通配符的使用：

- 无边界通配符：`List<?>`
- 上界通配符：`List<? extends E>`
- 下界通配符：`List<? super E>`

### 通配符 与 T 的区别

T：表示一个确定的类型。作用于模板上，用于将数据类型进行参数化，不能用于实例化对象。 
?：表示一个不确定的类型。在实例化对象的时候，不确定泛型参数的具体类型时，可以使用通配符进行对象定义。
```java
< T > 等同于 < T extends Object>
< ? > 等同于 < ? extends Object>
```
例一：定义泛型类，将key，value的数据类型进行< K, V >参数化，而不可以使用通配符。
```java
public class Container<K, V> {
    private K key;
    private V value;

    public Container(K k, V v) {
        key = k;
        value = v;
    }
}
```
例二：实例化泛型对象，我们不能够确定eList存储的数据类型是Integer还是Long，因此我们使用List<? extends Number>定义变量的类型。
```java
List<? extends Number> eList = null;
eList = new ArrayList<Integer>();
eList = new ArrayList<Long>();
```

### 无界通配符
等同于`<? extends Object>`。

### 上界类型通配符（? extends E）
为什么叫上界？

`？`表示E或者E的子类，那么E就是上限。E是父类，父类可接受子类。比如Object可以接收所有类型。
```java
List<? extends Number> eList = null;
eList = new ArrayList<Integer>();
Number numObject = eList.get(0);  //语句1，正确

Integer intObject = eList.get(0);  //语句2，错误

eList.add(new Integer(1));  //语句3，错误
```
语句1：List<? extends Number>eList存放Number及其子类的对象，语句1取出Number（或者Number子类）对象直接赋值给Number类型的变量是符合java规范的。 

语句2：List<? extends Number>eList存放Number及其子类的对象，语句2取出Number（或者Number子类）对象直接赋值给Integer类型（Number子类）的变量是不符合java规范的。

语句3：List<? extends Number>eList不能够确定实例化对象的具体类型，因此无法add具体对象至列表中，可能的实例化对象如下。
```java
eList = new ArrayList<Integer>();
eList = new ArrayList<Long>();
eList = new ArrayList<Float>();
```
总结：上界类型通配符add方法受限，但可以获取列表中的各种类型的数据，并赋值给父类型（extends Number）的引用。**也就是说，上界通配符允许取元素不允许存元素。**

### 下界类型通配符（? super E）
为什么叫下界：

`？`表示E或者E的父类，那么E就是下限。E是子类，子类不可以接收父类
```java
List<? super Integer> sList = null;
sList = new ArrayList<Number>();

Number numObj = sList.get(0);  //语句1，错误

Integer intObj = sList.get(0);  //语句2，错误

sList.add(new Integer(1));  //语句3，正确
```

语句1：List<? super Integer> 无法确定sList中存放的对象的具体类型，因此sList.get获取的值存在不确定性，子类对象的引用无法赋值给兄弟类的引用，父类对象的引用无法赋值给子类的引用，因此语句错误。 

语句2：同语句1。 

语句3：子类对象的引用可以赋值给父类对象的引用，因此语句正确。 

**总结：下界类型通配符get方法受限，但可以往列表中添加各种数据类型的对象。因此如果你想把对象写入一个数据结构里，使用 ? super 通配符。限定通配符总是包括自己。**

---
参考：
[Java泛型三：通配符详解extends super](https://blog.csdn.net/claram/article/details/51943742)
[三句话总结JAVA泛型通配符（PECS）](https://blog.csdn.net/yiifaa/article/details/73433623)
[JAVA泛型通配符T，E，K，V区别，T以及Class<T>，Class<?>的区别](https://www.jianshu.com/p/95f349258afb)

---

## 类型擦除
泛型是 Java 1.5 版本才引进的概念，在这之前是没有泛型的概念的，但显然，**泛型代码能够很好地和之前版本的代码很好地兼容**。

这是因为，泛型信息只存在于代码编译阶段，在进入 JVM 之前，**与泛型相关的信息会被擦除掉，专业术语叫做类型擦除**。

看下面的代码：
```java
public class Erasure <T>{
    T object;

    public Erasure(T object) {
        this.object = object;
    }
}
```
使用`javap -c`命令查看反编译的结果:
```java
  public Generic.Erasure.Erasure(T);
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: aload_0
       5: aload_1
       6: putfield      #2                  // Field object:Ljava/lang/Object;
       9: return
```
可以看到，在编译的时候`T`全被替换成了`Object`。

那可不可以说，泛型类被类型擦除后，相应的类型就被替换成 Object 类型呢？

非也，看下面的代码(给上面代码中的T加上了上限)：
```java
public class Erasure <T extends String>{
    T object;

    public Erasure(T object) {
        this.object = object;
    }
}
```
反编译的结果为：
```java
  public Generic.Erasure.Erasure(T);
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: aload_0
       5: aload_1
       6: putfield      #2                  // Field object:Ljava/lang/String;
       9: return
```
可以看到，`T`没有被替换成Object而是被替换成String。于是可以得到结论：

**在泛型类被类型擦除的时候，之前泛型类中的类型参数部分如果没有指定上限，如 `<T>` 则会被转译成普通的 Object 类型，如果指定了上限如 `<T extends String>` 则类型参数就被替换成类型上限。**

## 类型擦除带来的局限性
类型擦除，是泛型能够与之前的 java 版本代码兼容共存的原因。但也因为类型擦除，它会抹掉很多继承相关的特性，这是它带来的局限性。

理解类型擦除有利于我们绕过开发当中可能遇到的雷区，同样理解类型擦除也能让我们绕过泛型本身的一些限制。比如 
```java
public class ToolTest {
    public static void main(String[] args) {
        List<Integer> list = new ArrayList<>();
        list.add(3);
        list.add("123");   //1
    }
}
```
正常情况下，因为泛型的限制，编译器不让最后一行代码编译通过，因为类似不匹配，但是，基于对类型擦除的了解，利用反射，我们可以绕过这个限制。
```java
public interface List<E> extends Collection<E>{

     boolean add(E e);
}
```
上面是 List 和其中的 add() 方法的源码定义。

因为 E 代表任意的类型，所以类型擦除时，add 方法其实等同于
```java
boolean add(Object obj);
```
那么，利用反射，我们绕过编译器去调用 add 方法。
```java
public class ToolTest {
    public static void main(String[] args) {
        List<Integer> ls = new ArrayList<>();
        ls.add(23);
//      ls.add("text");
        try {
            //getDeclaredMethods(),该方法是获取本类中的所有方法
            Method method = ls.getClass().getDeclaredMethod("add",Object.class);

            method.invoke(ls,"test");
            method.invoke(ls,42.9f);
        } catch (Exception e) {
            e.printStackTrace();
        }

        for ( Object o: ls){
            System.out.println(o);
        }
    }
}
```
打印结果是：
```java
23
test
42.9
```
可以看到，利用类型擦除的原理，用反射的手段就绕过了正常开发中编译器不允许的操作限制。

---
参考：
[Java 泛型，你了解类型擦除吗？](https://blog.csdn.net/briblue/article/details/76736356)
[Java泛型详解](http://www.importnew.com/24029.html)
《码出高效：Java开发手册》

---

# 十、关键字
## final
- 使用Final修饰符修饰的类**不能被继承**；
- 使用Final修饰符修饰的对象**引用地址不能改变**(即不可以再将该引用指向其他对象，但对象本身可以改变)；
- 使用Final修饰符修饰的方法**不能被重写**；
- 使用Final修饰符修饰的变量**不能被修改**。

# 十一、内部类
在Java中，可以将一个类定义在另一个类里面或者一个方法里面，这样的类称为内部类。内部类有四种类型：

- 成员内部类，如：`private class InstanceInnerClass{}`
- 静态内部类，如：`static class StaticClass{}`
- 局部内部类，定义在方法或表达式内部
- 匿名内部类，如：`new Thread(){}.start()`

如下所示：
```java
public class OuterClass {
    //成员内部类
    private class InstanceInnerClass{}
    //静态内部类
    private class StaticInnerClass{}

    public static void main(String[] args) {
        //匿名内部类
        new Thread(){}.start();
        new Thread(){}.start();
        //方法内部类
        class MethodClass_1{}
        class MethodClass_2{}
    }
}
```

## 局部内部类和匿名内部类
需要注意的一点是，局部内部类和匿名内部类只能访问局部final变量，如下程序所示。b和a都是final变量。

**需要注意的是，即使将final修饰符去掉，编译也不会出错，因为java会默认它们是final变量。可以验证的是，如果b不加final修饰，而试图在test方法中修改b的值，会编译出错。**
```java
public class MethodInnerClass {
    public static void main(String[] args)  {
        test(3);
    }

    public static void test(final int b) {
        final int a = 10;
        new Thread(){
            public void run() {
                System.out.println(a);
                System.out.println(b);
            }
        }.start();
    }
}
```
那么，为什么它们只能访问final修饰的变量呢？

首先明白一点，当外部类的方法(上面程序中的test方法)结束时，局部变量就会被销毁了，但是内部类对象可能还存在(只有不再被引用时，才会死亡)。这里就出现了一个矛盾：**内部类对象访问了一个不存在的变量**。为了解决这个问题，就将局部变量复制了一份作为内部类的成员变量，**也就是说，内部类在访问局部变量时，会复制一份副本，内部类访问的其实是这个副本。**这样当局部变量死亡后，内部类仍可以访问它，实际访问的是局部变量的”copy”。这样就好像延长了局部变量的生命周期。

但是新的问题又出现了：将局部变量复制为内部类的成员变量时，必须保证这两个变量是一样的，**也就是说，必须保证这两个变量的同步，**如果我们在内部类中修改了成员变量，方法中的局部变量也得跟着改变，怎么解决问题呢？**就将局部变量设置为final**。

当变量被final修饰时：

- 若是基本类型，其值是不能改变的，就保证了copy与原始的局部变量的值是一样的；
- 若是引用类型，其引用是不能改变的，保证了copy与原始的变量引用的是同一个对象。

**总结一下，为了解决生命周期不同的问题，匿名内部类备份了变量，为了解决备份变量引出的变量同步问题，外部变量要被定义成final。**

---
参考：
《码出高效：Java编程手册》
[为什么局部内部类和匿名内部类只能访问final的局部变量?](https://blog.csdn.net/sf_climber/article/details/78326984)
[匿名内部类访问方法成员变量需要加final的原因及证明](https://blog.csdn.net/wjw521wjw521/article/details/77333820)

---

## 为什么要使用内部类
- 每个内部类都能独立的继承一个接口的实现，所以无论外部类是否已经继承了某个(接口的)实现，对于内部类都没有影响。内部类使得多继承的解决方案变得完整。
- 内部类可以很好的实现隐藏和封装。

---
参考：
[Java内部类详解](http://www.cnblogs.com/dolphin0520/p/3811445.html)
[Java中的内部类（成员内部类、静态内部类、局部内部类、匿名内部类）](https://www.cnblogs.com/shen-hua/p/5440285.html)

---

# 十二、Object中的常用方法
## clone()
clone() 是 Object 的 protected 方法，它不是 public，一个类不显式去重写 clone()，其它类就不能直接去调用该类实例的 clone() 方法。

需要使用clone()函数的类必须实现Cloneable()接口，但clone() 方法并不是 Cloneable 接口的方法，而是 Object 的一个 protected 方法。Cloneable 接口只是规定，如果一个类没有实现 Cloneable 接口又调用了 clone() 方法，就会抛出 CloneNotSupportedException。

## 浅拷贝
被复制对象的所有变量都含有与原来的对象相同的值，而所有的对其他对象的引用仍然指向原来的对象。即对象的浅拷贝会对“主”对象进行拷贝，但不会复制主对象里面的对象。”里面的对象“会在原来的对象和它的副本之间共享。

简而言之，浅拷贝仅仅复制所考虑的对象，而不复制它所引用的对象

例子：
```java
public class Student implements Cloneable {
    private String name;
    private int age;
    private Teacher teacher;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public Teacher getTeacher() {
        return teacher;
    }

    public void setTeacher(Teacher teacher){
        this.teacher = teacher;
    }

    @Override
    protected Object clone() throws CloneNotSupportedException {
        //浅复制
        return super.clone();
    }
}
```
```java
public class Teacher implements Cloneable {
    private String name;
    private int age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}
```
```java
public class Main {
    public static void main(String[] args) throws CloneNotSupportedException {
        Teacher teacher = new Teacher();
        teacher.setAge(30);
        teacher.setName("Bob");

        Student student1 = new Student();
        student1.setAge(15);
        student1.setName("Alice");
        student1.setTeacher(teacher);

        Student student2 = (Student) student1.clone();
        student2.setAge(16);
        student2.setName("Couche");

        System.out.println("student1 "+student1.getName());
        System.out.println("student1 "+student1.getAge());
        System.out.println("student1 "+student1.getTeacher().getName());
        System.out.println("student1 "+student1.getTeacher().getAge());

        System.out.println();

        System.out.println("student2 "+student2.getName());
        System.out.println("student2 "+student2.getAge());
        System.out.println("student2 "+student2.getTeacher().getName());
        System.out.println("student2 "+student2.getTeacher().getAge());

        student1.getTeacher().setName("Jam");

        System.out.println();
        System.out.println("修改老师信息后：");
        System.out.println("student1 "+student1.getTeacher().getName());
        System.out.println("student2 "+student2.getTeacher().getName());
    }
}
```
结果为：
```java
student1 Alice
student1 15
student1 Bob
student1 30

student2 Couche
student2 16
student2 Bob
student2 30

修改老师信息后：
student1 Jam
student2 Jam
```
可以看到，在student1中修改了teacher的名字后，student2中的teacher的名字也跟着改变了。可以得出结论：**student1和student2中的teacher指向的是一个teacher对象**。示意图如下：
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/shallocopy.png">
</center>

## 深拷贝
深拷贝是一个整个独立的对象拷贝，深拷贝会拷贝所有的属性,并拷贝属性指向的动态分配的内存。当对象和它所引用的对象一起拷贝时即发生深拷贝。深拷贝相比于浅拷贝速度较慢并且花销较大。

简而言之，深拷贝把要复制的对象所引用的对象都复制了一遍。

将上述程序中Student类中的clone()函数改为如下，就实现了深拷贝。
```java
    @Override
    protected Object clone() throws CloneNotSupportedException {
        //深复制
        Student student = (Student) super.clone();
        student.setTeacher((Teacher) student.getTeacher().clone());
        return student;
    }
```
结果如下：
```java
student1 Alice
student1 15
student1 Bob
student1 30

student2 Couche
student2 16
student2 Bob
student2 30

修改老师信息后：
student1 Jam
student2 Bob
```
内存示意图如下：
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/deepcopy.png">
</center>

## clone()的替代方案
使用 clone() 方法来拷贝一个对象即复杂又有风险，它会抛出异常，并且还需要类型转换。Effective Java 书上讲到，最好不要去使用 clone()，可以使用**拷贝构造函数**、**拷贝工厂**或者**序列化**来拷贝一个对象。


### 使用拷贝构造函数
```java
public class CloneConstructorExample {

    private int[] arr;

    public CloneConstructorExample() {
        arr = new int[10];
        for (int i = 0; i < arr.length; i++) {
            arr[i] = i;
        }
    }

    public CloneConstructorExample(CloneConstructorExample original) {
        arr = new int[original.arr.length];
        for (int i = 0; i < original.arr.length; i++) {
            arr[i] = original.arr[i];
        }
    }

    public void set(int index, int value) {
        arr[index] = value;
    }

    public int get(int index) {
        return arr[index];
    }
}
```
```java
CloneConstructorExample e1 = new CloneConstructorExample();
CloneConstructorExample e2 = new CloneConstructorExample(e1);
e1.set(2, 222);
System.out.println(e2.get(2)); // 2
```

### 使用序列化
```java
package CloneTest.SerializableClone;

import java.io.*;

public class Student implements Serializable {
    private String name;
    private int age;
    private Teacher teacher;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public void setTeacher(Teacher teacher){
        this.teacher = teacher;
    }

    public Teacher getTeacher(){
        return teacher;
    }

    public Object deepClone() throws IOException, ClassNotFoundException {
        ByteArrayOutputStream bos = new ByteArrayOutputStream();
        ObjectOutputStream oos = new ObjectOutputStream(bos);

        oos.writeObject(this);

        ByteArrayInputStream bis = new ByteArrayInputStream(bos.toByteArray());
        ObjectInputStream ois = new ObjectInputStream(bis);

        return ois.readObject();

    }
}
```
```java
package CloneTest.SerializableClone;

import java.io.Serializable;

public class Teacher implements Serializable {
    private String name;
    private int age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```
```java
package CloneTest.SerializableClone;

import java.io.IOException;

public class Main {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        Teacher teacher = new Teacher();
        teacher.setAge(50);
        teacher.setName("Bob");

        Student student1 = new Student();
        student1.setAge(15);
        student1.setName("Alice");
        student1.setTeacher(teacher);

        Student student2 = (Student) student1.deepClone();

        System.out.println();
        System.out.println("student2 "+student2.getName());
        System.out.println("student2 "+student2.getAge());
        System.out.println("student2 "+student2.getTeacher().getName());
        System.out.println("student2 "+student2.getTeacher().getAge());

        System.out.println();
        System.out.println("修改老师信息：");
        student1.getTeacher().setName("Job");

        System.out.println("student1 "+student1.getTeacher().getName());
        System.out.println("student2 "+student2.getTeacher().getName());
    }
}
```

---
参考：
[【Java深入】深拷贝与浅拷贝详解](https://blog.csdn.net/baiye_xing/article/details/71788741)

---

# 十三、Lambda表达式
Lambda表达式是Java8中引入的新特性，要理解Lambda表达式，首先要理解什么是**函数式接口**。

## 函数式接口
函数式接口（`functional interface` 也叫功能性接口）。简单来说，函数式接口是只包含一个方法的接口。比如Java标准库中的`java.lang.Runnable`和 `java.util.Comparator`都是典型的函数式接口。

而Lambda表达式的作用，就是可以将函数式接口中的方法以一种简单的形式表达出来。

举个例子，在`Arrays.sort()`方法中自定义比较器，Java8之前的做法和使用Lambda表达式的做法分别是：
```
Arrays.sort(nums, new Comparator<Integer>() {
        @Override
        public int compare(Integer o1, Integer o2) {
            if(o1 < o2) 
                return -1;
            return 0;
        }
    });
```
```
Arrays.sort(nums, (o1, o2) -> {
        if(o1 < o2) 
            return -1;
        return 0;
    });
```

## Lambda表达式的具体语法
Lambda表达式的基本语法：

- `(parameters) -> expression `
- `(parameters) -> { statements; }`

也就是说，当只有一条`expression`时，`{}`和`return`可以省略，但是有多条时，不能省略。

除此之外，还有以下几条规则：

- 若只有一个参数，`（）`可以省略不写
- lambda表达式的参数列表的数据类型可以省略不写

参考：
[java函数式编程之lambda表达式](https://www.cnblogs.com/mahang/p/3247017.html)
[Java 8 Lambda表达式，让你的代码更简洁](https://www.cnblogs.com/pkufork/p/java_8_lambda.html)
[java8新特性（拉姆达表达式lambda）](https://blog.csdn.net/qq_35805528/article/details/53264301)
[【java8新特性】兰姆达表达式-1](https://blog.csdn.net/equaker/article/details/81635633)
[Java8 Lambda表达式和流操作如何让你的代码变慢5倍](http://www.importnew.com/17262.html)

# 十五、System类中的getProperties()和getenv()

- getenv是获取系统的环境变更，对于windows对在系统属性-->高级-->环境变量中设置的变量将显示在此(对于linux,通过export设置的变量将显示在此) 
- getProperties是获取系统的相关属性,包括文件编码,操作系统名称,区域,用户名等。

getProperty(String str) 中的 str参数如下：

|变量名|作用|
|-|-|
|java.version |Java运行时环境版本|
|java.vendor |Java运行时环境供应商|
|java.vendor.url|  Java供应商的 URL|
|java.home   |Java安装目录|
|java.vm.specification.version  |Java虚拟机规范版本|
|java.vm.specification.vendor  |Java虚拟机规范供应商|
|java.vm.specification.name |Java虚拟机规范名称|
|java.vm.version |Java虚拟机实现版本|
|java.vm.vendor| Java虚拟机实现供应商|
|java.vm.name  |Java虚拟机实现名称|
|java.specification.version | Java运行时环境规范版本|
|java.specification.vendor| Java运行时环境规范供应商|
|java.specification.name  |Java运行时环境规范名称|
|java.class.version|  Java类格式版本号|
|java.class.path  |Java类路径|
|java.library.path|  加载库时搜索的路径列表|
|java.io.tmpdir | 默认的临时文件路径|
|java.compiler | 要使用的 JIT 编译器的名称|
|java.ext.dirs | 一个或多个扩展目录的路径|
|os.name|  操作系统的名称|
|os.arch | 操作系统的架构|
|os.version|  操作系统的版本|
|file.separator | 文件分隔符（在 UNIX 系统中是“/”）|
|path.separator | 路径分隔符（在 UNIX 系统中是“:”）|
|line.separator | 行分隔符（在 UNIX 系统中是“/n”）|
|user.name| 用户的账户名称|
|user.home | 用户的主目录|
|user.dir | 用户的当前工作目录|

例子：
```
public class GetProTest {
    public static void main(String[] args) {
        Properties properties = System.getProperties();
        for (String property : properties.stringPropertyNames()){
            System.out.println(property+"  "+properties.get(property));
        }

        System.out.println(System.getProperty("java.version"));

        System.setProperty("jdbc.Driver", "hahahahah");
        System.out.println(System.getProperty("jdbc.Driver"));
    }
}
```

```
public class GetEnvTest {
    public static void main(String[] args) {
        Map<String, String> map = System.getenv();
        Set<Map.Entry<String, String>> entries = map.entrySet();
        for (Map.Entry<String, String> entry : entries){
            System.out.println(entry.getKey() +"  "+entry.getValue());
        }

        System.out.println(System.getenv("JAVA_HOME"));
    }
}
```

参考：
[System.getProperty()方法获取系统变量(转 阿进的写字台)](https://blog.csdn.net/qq_42875051/article/details/85990870)
[System.getProperty()方法获取系统变量](https://blog.csdn.net/weixin_37139197/article/details/78877766)
[JAVA System.getProperty() System.getenv()区别及示例](https://yiranwuqing.iteye.com/blog/720180)