# Java--代理
---

# 一、概念
给某个对象提供一个代理对象，并由代理对象控制对于原对象的访问。

代理类的应用场景很多，最容易想到的情况就是权限过滤，我有一个类做某项业务，但是由于安全原因只有某些用户才可以调用这个类，此时我们就可以做一个该类的代理类，要求所有请求必须通过该代理类，由该代理类做权限判断，如果安全则调用实际类的业务开始处理。

还有就是Spring的AOP，利用代理实现切面编程，将不同业务逻辑中相同功能的代码提取出来，减少了耦合。

Java里实现代理有两种方式：**静态代理**和**动态代理**。动态代理又可以使用**JDKProxy**或者**CGLib**的方式实现。

# 二、静态代理
静态代理的特点就是，代理和被代理对象在代理之前就是已经确定好的，它们都实现共同的接口或者继承相同的抽象类。

静态代理通常用于对原有业务逻辑的扩充。比如要在原有类的基础上加上日志功能，但又不能修改袁石磊，就可以使用代理模式，通过让代理类持有真实对象，然后在原代码中调用代理类方法，来达到添加所需要业务逻辑的目的。有点类似于**装饰模式**。

一个典型的代理模式通常有三个角色，这里称之为**代理三要素**：**共同接口**、**真实对象**和**代理对象**。

## 共同接口
一个Hello接口。
```java
public interface Hello {
    void say(String name);
}
```
## 真实对象
实现了Hello接口。
```java
public class HelloImp implements Hello {
    @Override
    public void say(String name) {
        System.out.println("Hello" + name);
    }
}
```
## 代理对象
写一个 HelloProxy 类，让它去调用 HelloImpl 的 say() 方法，在调用的前后分别进行逻辑处理
```java
public class HelloProxy implements Hello{
    private HelloImp helloImp;

    public HelloProxy(){
        helloImp = new HelloImp();
    }

    @Override
    public void say(String name) {
        before();
        helloImp.say(name);
        after();
    }

    private void before(){
        System.out.println("Before");
    }

    private void after(){
        System.out.println("After");
    }
}
```

测试类和结果如下：
```java
public static void main(String[] args) {
    HelloProxy helloProxy = new HelloProxy();
    helloProxy.say("Jack");
}

Before
HelloJack
After
```

# 三、动态代理
Java实现动态代理的方式有两种：使用**JDKProxy**或使用**CGLib**类库。**使用JDKProxy代理的类必须实现一个接口，CGLIB即使代理类没有实现任何接口也可以实现动态代理功能。**

JDKProxy使用反射机制实现，CGLib使用ASM实现。

## JDKProxy
使用JDKProxy实现的动态代理主要包含以下三个元素：
- 业务接口（Interface）
业务的抽象表示
- 业务具体实现类（concreteManager）
实现业务接口，执行具体的业务操作
- 业务代理类（proxyHandler，实现了InvocationHandler接口的类）
代理方法的直接调用者，通过InvocationHandler中的invoke方法直接发起代理
- 客户端调用对象（client）
发起业务

实现步骤如下：
- 创建被代理的接口和类；
- 创建一个类实现InvocationHandler接口，实现其中的invoke()方法；
- 调用Proxy的静态方法，创建一个代理类：newProxyInstance(ClassLoader loader, Class[] interfaces, InvocationHander h)；
- 通过代理调用方法；

还是基于上面的Hello接口和HelloImp类。下面创建一个实现InvocationHandler接口的方法DynamicProxy，在 DynamicProxy 类中，定义了一个 Object 类型的 target 变量，它就是被代理的目标对象，通过构造函数来初始化。

DynamicProxy 实现了 InvocationHandler 接口，那么必须实现该接口的 invoke 方法，参数见注释。在该方法中，直接通过反射去 invoke method，在调用前后分别处理 before 与 after，最后将 result 返回。

接着，在DynamicProxy 里添加了一个 getProxy() 方法，在里面调用 JDK 给我们提供的 Proxy 类的工厂方法 newProxyInstance() 去动态地创建一个 Hello 接口的代理类，最后调用这个代理类的 say() 方法。这样可以简化主函数中的代码。

@SuppressWarnings("unchecked") 注解表示忽略编译时的警告（因为 Proxy.newProxyInstance() 方法返回的是一个 Object，这里我强制转换为 T 了，这是向下转型，IDE 中就会有警告，编译时也会出现提示。
```java
public class DynamicProxy implements InvocationHandler {
    //被代理的对象
    private Object target;

    public DynamicProxy(Object target){
        this.target = target;
    }

    /**
     * @param proxy：被代理的对象
     * @param method：被代理对象的方法
     * @param args：方法的参数
     * @return
     * @throws InvocationTargetException
     * @throws IllegalAccessException
     */
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws InvocationTargetException, IllegalAccessException {
        before();
        Object result = method.invoke(target, args);
        after();
        return result;
    }

    /**
     * Proxy.newProxyInstance的三个参数：
     * 1. 被代理类的类加载器
     * 2. 该实现类的所有接口
     * 3. 动态代理对象
     * @param <T>
     * @return
     */
    @SuppressWarnings("unchecked")
    public <T> T getProxy(){
        return (T) Proxy.newProxyInstance(
                target.getClass().getClassLoader(),
                target.getClass().getInterfaces(),
                this
        );
    }

    private void before(){
        System.out.println("Before");
    }

    private void after(){
        System.out.println("After");
    }
}
```
测试方法和结果如下：
```java
public static void main(String[] args) {
    DynamicProxy dynamicProxy = new DynamicProxy(new HelloImp());
    Hello hello = dynamicProxy.getProxy();

    hello.say("Jack");
}

Before
HelloJack
After
```

## CGLib
Cglib是一个优秀的动态代理框架，它的底层使用ASM在内存中动态的生成被代理类的子类，使用CGLIB即使代理类没有实现任何接口也可以实现动态代理功能。CGLIB具有简单易用，它的运行速度要远远快于JDK的Proxy动态代理：

cglib有两种可选方式，**继承**和**引用**。第一种是基于继承实现的动态代理，所以可以直接通过super调用target方法，但是这种方式在spring中是不支持的，因为这样的话，这个target对象就不能被spring所管理，所以cglib还是才用类似jdk的方式，通过持有target对象来达到拦截方法的效果。

使用CGLib实现代理的步骤如下：
- 创建被代理的接口和类；
- 实现 CGLib 提供的 MethodInterceptor 实现类，并填充 intercept() 方法；
- 通过Enhancer类产生一个代理类；
- 通过代理调用方法；

```java
public class CGLibProxy implements MethodInterceptor {
    private static CGLibProxy instance = new CGLibProxy();

    private CGLibProxy(){}

    public <T> T getProxy(Class<T> cls){
        return (T) Enhancer.create(cls, this);
    }

    /**
     * @param o：目标类的实例
     * @param method：目标方法的反射对象
     * @param objects：目标方法的参数
     * @param methodProxy：代理类的实例
     * @return
     * @throws Throwable
     */
    @Override
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        before();
        Object restult = methodProxy.invokeSuper(o, objects);
        after();
        return restult;
    }

    public static CGLibProxy getInstance(){
        return instance;
    }

    private void before(){
        System.out.println("Before");
    }

    private void after(){
        System.out.printf("After");
    }
}
```
在上述代码中，使用了单例模式，简化了主函数中的代码；

主函数和结果如下：
```java
public static void main(String[] args) {
    HelloImp helloImp = CGLibProxy.getInstance().getProxy(HelloImp.class);

    helloImp.say("Jack");
}

Before
HelloJack
After
```

# 参考
[Proxy那点事](https://my.oschina.net/huangyong/blog/159788)</br>
[Java动态代理实现及原理分析](https://www.jianshu.com/p/23d3f1a2b3c7)</br>
[CGLIB介绍与原理](https://blog.csdn.net/zghwaicsdn/article/details/50957474)</br>
[静态代理和动态代理的理解](https://blog.csdn.net/wangqyoho/article/details/77584832)</br>
[java动态代理原理及解析](https://blog.csdn.net/Scplove/article/details/52451899)</br>
[详解java动态代理机制以及使用场景(一)](https://blog.csdn.net/u011784767/article/details/78281384)</br>
[Java动态代理的两种实现方法](https://blog.csdn.net/heyutao007/article/details/49738887)</br>
[Java设计模式之动态代理](https://www.cnblogs.com/lfdingye/p/7717063.html)</br>
[CGLib之Enhancer](https://blog.csdn.net/jiaotuwoaini/article/details/51675684)</br>
[代理8 cglib demo以及Enhancer源码解析](https://www.jianshu.com/p/20203286ccd9)</br>
[基于 JDK 的动态代理-佟刚老师《Spring4视频教程》学习笔记（16）](https://blog.csdn.net/lw_power/article/details/44278419)</br>
[java的动态代理机制详解](http://www.importnew.com/20732.html)</br>
[JAVA动态代理](http://www.importnew.com/26166.html)</br>
