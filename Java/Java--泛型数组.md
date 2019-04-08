# Java--泛型数组
---
# 问题来源
今天在刷题时，遇到了需要使用泛型数组的场景。题目是**按之字形打印二叉树**。这道题目需要交替使用两个栈来解决，我的初始代码为：
```java
ArrayDeque<TreeNode>[] stacks = new ArrayDeque<TreeNode>[2];    //1
stacks[0] = new ArrayDeque<TreeNode>();
stacks[1] = new ArrayDeque<TreeNode>();
```
而在编译时，代码1报出了如下编译错误：
```java
Cannot create a generic array of ArrayDeque<Integer>
```
原来Java不支持泛型数组。

# Java中的泛型做了什么
首先看一下Java中的泛型做了什么。看下面这段代码：
```java
public class GenTest<T> {
    T value;

    public T getValue() {
        return value;
    }

    public void setValue(T t) {
        value = t;
    }
}
```
使用javap命令反编译生成的GenTest类的class文件，可以得到下面的输出：
```java
javap -c -p GenTest
Compiled from "GenTest.java"
public class GenTest extends java.lang.Object{
java.lang.Object value;

public GenTest();
  Code:
   0:   aload_0
   1:   invokespecial   #12; //Method java/lang/Object."<init>":()V
   4:   return

public java.lang.Object getValue();
  Code:
   0:   aload_0
   1:   getfield        #23; //Field value:Ljava/lang/Object;
   4:   areturn

public void setValue(java.lang.Object);
  Code:
   0:   aload_0
   1:   aload_1
   2:   putfield        #23; //Field value:Ljava/lang/Object;
   5:   return

}
```
我们清楚的看到，**泛型T在GenTest类中就是Object类型（java.lang.Object value;）。同样，get方法和set方法也都是将泛型T当作Object来处理的。**如果我们规定泛型是Numeric类或者其子类，那么在这里泛型T就是被当作Numeric类来处理的。

好，既然GenTest类中没有什么乾坤，那么我们继续看使用GenTest的时候又什么新东西：
```java
public class UseGenTest {

    public static void main(String[] args) {
        String value = "value";
        GenTest<String> test = new GenTest<String>();
        test.setValue(value);
        String nv = test.getValue();
    }
}
```
使用javap命令反编译生成的GenTest类的class文件，可以得到下面的输出：
```java
javap -c -p UseGenTest
Compiled from "UseGenTest.java"
public class UseGenTest extends java.lang.Object{
public UseGenTest();
  Code:
   0:   aload_0
   1:   invokespecial   #8; //Method java/lang/Object."<init>":()V
   4:   return

public static void main(java.lang.String[]);
  Code:
   0:   ldc     #16; //String value
   2:   astore_1
   3:   new     #18; //class GenTest
   6:   dup
   7:   invokespecial   #20; //Method GenTest."<init>":()V
   10:  astore_2
   11:  aload_2
   12:  aload_1
   13:  invokevirtual   #21; //Method GenTest.setValue:(Ljava/lang/Object;)V
   16:  aload_2
   17:  invokevirtual   #25; //Method GenTest.getValue:()Ljava/lang/Object;
   20:  checkcast       #29; //class java/lang/String
   23:  astore_3
   24:  return
}
```

重点在17、20和23三处。17就是调用getValue方法。而20则是关键——类型检查。**也就是说，在调用getValue方法之后，并没有直接把返回值赋值给nv，而是先检查了返回值是否是String类型**，换句话说，“String nv = test.getValue();”被编译器变成了“String nv =(String)test.getValue();”。最后，如果检查无误，在23处才会赋值。也就是说，如果没有完成类型检查，则会报出类似ClassCastException，而代码将不会继续向下执行，这就有效的避免了错误的出现。

**也就是说：在类的内部，泛型类型就是被基类型代替的（默认是Object类型），而对外，所有返回值类型为泛型类型的方法，在真正使用返回值之前，都是会经过类型转换的。**

# 为什么不支持泛型的数组？
根据上面的分析可以看出来，泛型其实是挺严谨的，说白了就是在“编译的时候通过增加强制类型转换的代码，来避免用户编写出可能引发ClassCastException的代码”。这其实也算是Java引入泛型的一个目的。

但是，如果允许了泛型数组，那么编译器添加的强制类型转换的代码就会有可能是错误的。

看下面的例子：
```java
//下面的代码使用了泛型的数组，是无法通过编译的
GenTest<String> genArr[] = new GenTest<String>[2];
Object[] test = genArr;
GenTest<StringBuffer> strBuf = new GenTest<StringBuffer>();
strBuf.setValue(new StringBuffer());
test[0] = strBuf;
GenTest<String> ref = genArr[0]; //上面两行相当于使用数组移花接木，让Java编译器把GenTest<StringBuffer>当作了GenTest<String>
String value = ref.getValue();// 这里是重点！
```
上面的代码中，最后一行是重点。根据本文第一部分的介绍，“String value = ref.getValue()”会被替换成“String value =(String)ref.getValue()”。当然我们知道，ref实际上是指向一个存储着StringBuffer对象的GenTest对象。所以，编译器生成出来的代码是隐含着错误的，在运的时候就会抛出ClassCastException。

但是，如果没有“String value = ref.getValue();”这行代码，那么程序可以说没有任何错误。这全都是Java中多态的功劳。我们来分析一下，对于上面代码中创建出来的GenTest对象，其实无论value引用实际指向的是什么对象，对于类中的代码来说都是没有任何影响的——因为在GenTest类中，这个对象仅仅会被当作是基类型的对象（在这里也就是Object的对象）来使用。所以，无论是String的对象，还是StringBuffer的对象，都不可能引发任何问题。举例来说，如果调用valued的hashcode方法，那么，如果value指向的是String的对象，实际执行的就是String类中的hashcode方法，如果是StringBuffer的对象，那么实际执行的就是StringBuffer类中的hashcode方法。

# 解决办法
## 包装类实现泛型数组(编译无错，运行出错)
```java
public class GenericArray<T> {
    private Object[] values;

    public GenericArray(int count){

        values = new Object[count];

    }

    public void setValue(T t,int position){

        values[position] = t;
    }

    public T getValue(int position){

        return (T)values[position];
    }

    //注意此句代码调用会出错
    public T[] getValues(){

        return (T[])values;
    }
}
```
测试代码：
```java
public class Test {

    public static void main(String[] args) {

        GenericArray<String> generic=new GenericArray(10);

        generic.setValue("wenwei1", 0);
        generic.setValue("wenwei2", 1);

        System.out.println(generic.getValue(0));
        System.out.println(generic.getValue(1));
        //
        String[]content = generic.getValues();
        }
}
```
分析：上面代码可以编译通过，而且我们使用GenericArray存储数据可以正常使用，但当我们调用其getValues则会出现强转类型错误（这种错误就是我们在概述中分析的编译器生成强转代码造成的）。 

## 使用反射
```java
public class GenericArray1 <T>{
    private T[] values;

    public GenericArray1(Class<T> type,int length){
        values= (T[])Array.newInstance(type, length);
    }

    public void setValue(T t,int position){

        values[position] = t;
    }

    public T getValue(int position){

        return (T)values[position];
    }

    public T[] getValues(){

        return values;
    }
}
```
测试代码：
```java
public class Test {

    public static void main(String[] args) {

        GenericArray1<String> generic=new GenericArray1<String>(String.class,10);

        generic.setValue("wenwei1", 0);
        generic.setValue("wenwei2", 1);

        System.out.println(generic.getValue(0));
        System.out.println(generic.getValue(1));

        String[]content = generic.getValues();
        System.out.println(content[0]);
        }
}   
```

分析：上面代码可以编译通过，我们使用GenericArray存储数据可以正常使用，而且调用其getValues也不会出现强转类型错误，这种方式比较推荐（因为我们调用Array.newInstance()生成的数组是String[]类型的数组）。

## 使用通配符
The Java™ Tutorials: Generics给出的解决方案如下：
```java
List<?>[] lsa = new List<?>[10];                //1
Object[] oa = lsa;
List<Integer> li = new ArrayList<Integer>();
li.add(new Integer(3));

oa[1] = li;

Integer s = (Integer) lsa[1].get(0)            //2
```
在第1处，用?取代了确定的参数类型。根据通配符的定义以及Java类型擦除的保留上界原则，在2处lsa[1].get(0)取出的将会是Object，所以需要程序员做一次显式的类型转换。假如说程序员要将2处取出的值强转为String，就会出错。所以，这种做法相当于把风险把控交给了程序员。

补充说明：对于泛型数组，最简单的我们可以利用容器List来模拟实现！

# 总结
Java为什么不允许泛型数组？

先看为什么要用泛型。泛型的引入就是为了避免类型的不一致，假如声明一个List<Pair>类型的list，那么在写出语句list.add(5)的时候，编译就会出错，因为类型不匹配，所以**泛型可以将类型不匹配的错误扼杀在编译阶段。**

但是泛型数组就不行了，如果允许创建泛型数组，将能在数组p里存放任何类的对象，并且能够通过编译，因为在编译阶段p被认为是一个Object[ ]，也就是p里面可以放一个int，也可以放一个Pair，这样就**绕过了编译器对泛型的检查**。但是取出的时候就有可能将一个int类型的赋值给Pair类型。为了避免这种错误的出现，Java不支持直接创建泛型数组。

所以最初的问题可以解决如下：
```java
ArrayDeque<Integer>[] stacks = (ArrayDeque<Integer>[]) Array.newInstance(ArrayDeque.class, 2);
```

# 补充
后来无意中发现如果将最初的问题写为：
```java
ArrayDeque<TreeNode>[] stacks = new ArrayDeque[2];
```
即后面的ArrayDeque后面不加泛型限定，可以正常使用。但是好像这种写法不太规范。

# 参考
[分析一下为什么JAVA不支持泛型类型的数组](https://www.cnblogs.com/scutwang/articles/3735219.html)
[java泛型与数组](https://blog.csdn.net/qq_22494029/article/details/79386607)
[Java泛型的实现：“禁止”泛型数组](https://blog.csdn.net/yi_Afly/article/details/52058708)
[Java为什么不能创建泛型数组？](https://blog.csdn.net/aabbwoshishei/article/details/50163261)
[Java 泛型 泛型数组](https://www.cnblogs.com/ixenos/p/5648519.html)
[Java泛型-你可能需要知道这些](https://www.jianshu.com/p/f85b3b205eb4)
[Java解惑之Object、T（泛型）、?区别](https://www.jianshu.com/p/f42f37015c46)