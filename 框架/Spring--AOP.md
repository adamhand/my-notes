# Spring--AOP
---
# 概念
## 一个例子

首先需要说明的是，AOP是一种思想，它不是Spring特有的，Spring的AOP只是AOP的一种实现方式。

先举个例子，用通俗的语言了解一下什么是AOP。假如我们要实现一个简单计算器，它的功能包括加法、减法、乘法和除法四种。并且要求在实现核心功能代码的同时进行参数验证和日志处理。如下图所示：
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/aop.jpg" width="500">
</center>

可以看到，add()、sub()、mul()和div()四个方法中有很多相同的流程，而AOP的思想就是将这些相同的流程提取出来，让主流程中只包含纯净的业务逻辑代码。**所以AOP的本质是在一系列纵向的控制流程中，把那些相同的子流程提取成一个横向的面**(图中很形象地表示出了**“横向切面”**这个概念)。

## AOP
AOP（Aspect Oriented Programming），即面向切面编程，可以说是OOP（Object Oriented Programming，面向对象编程）的补充和完善。OOP引入封装、继承、多态等概念来建立一种对象层次结构，用于模拟公共行为的一个集合。不过OOP允许开发者定义纵向的关系，但并不适合定义横向的关系，例如日志功能。日志代码往往横向地散布在所有对象层次中，而与它对应的对象的核心功能毫无关系对于其他类型的代码，如安全性、异常处理和透明的持续性也都是如此，这种散布在各处的无关的代码被称为横切（cross cutting），在OOP设计中，它导致了大量代码的重复，而不利于各个模块的重用。

AOP技术恰恰相反，它利用一种称为"横切"的技术，剖解开封装的对象内部，并将那些影响了多个类的公共行为封装到一个可重用模块，并将其命名为"Aspect"，即切面。所谓"切面"，简单说就是那些与业务无关，却为业务模块所共同调用的逻辑或责任封装起来，便于减少系统的重复代码，降低模块之间的耦合度，并有利于未来的可操作性和可维护性。

使用"横切"技术，AOP把软件系统分为两个部分：**核心关注点**和**横切关注点**。业务处理的主要流程是核心关注点，与之关系不大的部分是横切关注点。横切关注点的一个特点是，他们经常发生在核心关注点的多处，而各处基本相似，比如权限认证、日志、事物。AOP的作用在于分离系统中的各种关注点，将核心关注点和横切关注点分离开来。

## AOP中的关键概念
>- **连接点(join point)：**程序运行中的一些时间点, 例如一个方法的执行, 或者是一个异常的处理。在 Spring AOP 中, join point 总是方法的执行点, 即只有方法连接点。连接点其实就是一种标志，标志着要在哪里切入。
>- **切入点（pointcut）：**对连接点进行拦截的定义。
>- **横切关注点：**对哪些方法进行拦截，拦截后怎么处理，这些关注点称之为横切关注点。
>- **切面（aspect）：**类是对物体特征的抽象，切面就是对横切关注点的抽象。
>- **通知（advice）：**所谓通知指的就是指拦截到连接点之后要执行的代码，通知分为前置、后置、异常、最终、环绕通知五类。
>- **Target(目标对象)：**代理的目标对象。
>- **织入（weave）：**将切面应用到目标对象并导致代理对象创建的过程。
>- **Proxy（代理）:**一个类被AOP织入增强后，就产生一个结果代理类。

下面以一个简单的例子来比喻一下 AOP 中 aspect, jointpoint, pointcut 与 advice 之间的关系.

让我们来假设一下, 从前有一个叫爪哇的小县城, 在一个月黑风高的晚上, 这个县城中发生了命案. 作案的凶手十分狡猾, 现场没有留下什么有价值的线索. 不过万幸的是, 刚从隔壁回来的老王恰好在这时候无意中发现了凶手行凶的过程, 但是由于天色已晚, 加上凶手蒙着面, 老王并没有看清凶手的面目, 只知道凶手是个男性, 身高约七尺五寸. 爪哇县的县令根据老王的描述, 对守门的士兵下命令说: 凡是发现有身高七尺五寸的男性, 都要抓过来审问. 士兵当然不敢违背县令的命令, 只好把进出城的所有符合条件的人都抓了起来.

来让我们看一下上面的一个小故事和 AOP 到底有什么对应关系. 首先我们知道, 在 Spring AOP 中 join point 指代的是所有方法的执行点, 而 point cut 是一个描述信息, 它修饰的是 join point, 通过 point cut, 我们就可以确定哪些 join point 可以被织入 Advice. 对应到我们在上面举的例子, 我们可以做一个简单的类比, join point 就相当于 **爪哇的小县城里的百姓**, point cut 就相当于 **老王所做的指控, 即凶手是个男性, 身高约七尺五寸**, 而 advice 则是施加在符合老王所描述的嫌疑人的动作: **抓过来审问**. 为什么可以这样类比呢?

>- **join point** --> 爪哇的小县城里的百姓: 因为根据定义, join point 是所有可能被织入 advice 的候选的点, 在 Spring AOP 中, 则可以认为所有方法执行点都是 join point. 而在我们上面的例子中, 命案发生在小县城中, 按理说在此县城中的所有人都有可能是嫌疑人.
>- **point cut** --> 男性, 身高约七尺五寸: 我们知道, 所有的方法(joint point) 都可以织入 advice, 但是我们并不希望在所有方法上都织入 advice, 而 pointcut 的作用就是提供一组规则来匹配 joinpoint, 给满足规则的 joinpoint 添加 advice. 同理, 对于县令来说, 他再昏庸, 也知道不能把县城中的所有百姓都抓起来审问, 而是根据凶手是个男性, 身高约七尺五寸, 把符合条件的人抓起来. 在这里凶手是个男性, 身高约七尺五寸 就是一个修饰谓语, 它限定了凶手的范围, 满足此修饰规则的百姓都是嫌疑人, 都需要抓起来审问.
>- **advice** --> 抓过来审问, advice 是一个动作, 即一段 Java 代码, 这段 Java 代码是作用于 point cut 所限定的那些 join point 上的. 同理, 对比到我们的例子中, 抓过来审问 这个动作就是对作用于那些满足 男性, 身高约七尺五寸 的爪哇的小县城里的百姓.
>- **aspect**: aspect 是 point cut 与 advice 的组合, 因此在这里我们就可以类比: "根据老王的线索, 凡是发现有身高七尺五寸的男性, 都要抓过来审问" 这一整个动作可以被认为是一个 aspect。

---
参考：
[轻松理解AOP思想(面向切面编程)]( https://www.cnblogs.com/Wolfmanlq/p/6036019.html)
[Spring3：AOP](http://www.cnblogs.com/xrq730/p/4919025.html)

---

# Spring实现AOP
在Spring中实现AOP可以有两种途径：使用Spring自带的机制实现；使用AspectJ辅助实现。

Spring本身提供了两种方式来生成代理对象: **JDKProxy**和**Cglib**，具体使用哪种方式生成由AopProxyFactory根据AdvisedSupport对象的配置来决定。**默认的策略是如果目标类是接口，则使用JDK动态代理技术，否则使用Cglib来生成代理**。使用Spring自身的机制实现AOP稍微复杂，下面先演示用AspectJ来实现。

## AspectJ实现Spring AOP
使用AspectJ实现AOP也可以分为两种方式：**使用注解**和**使用xml文件配置**。

### 使用注解
常见AspectJ的注解：
>- @Before – 方法执行前运行
>- @After – 运行在方法返回结果后
>- @AfterReturning – 运行在方法返回一个结果后，在拦截器返回结果。
>- @AfterThrowing – 运行方法在抛出异常后，
>- @Around – 围绕方法执行运行，结合以上这三个通知。

下面的例子使用IDEA建立maven + Java Project项目，比导包方便。

**1. 目录结构**
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/aop-demo-project.PNG">
</center>

**2. Spring Beans**
CustomerBo和CustomerBoImp中的代码：
```java
package com.aop.CustomerBo;

public interface CustomerBo {

    void addCustomer();

    String addCustomerReturnValue();

    void addCustomerThrowException() throws Exception;

    void addCustomerAround(String name);
}
```
```java
//CustomerBoImp
package com.aop.CustomerBoImp;

import com.aop.CustomerBo.CustomerBo;

public class CustomerBoImp implements CustomerBo {

    public void addCustomer(){
        System.out.println("addCustomer() is running ");
    }

    public String addCustomerReturnValue(){
        System.out.println("addCustomerReturnValue() is running ");
        return "abc";
    }

    public void addCustomerThrowException() throws Exception {
        System.out.println("addCustomerThrowException() is running ");
        throw new Exception("Generic Error");
    }

    public void addCustomerAround(String name){
        System.out.println("addCustomerAround() is running, args : " + name);
    }
}
```

**3. 启用AspectJ**
在 Spring 配置文件，把“`<aop:aspectj-autoproxy />`”，并定义Aspect(拦截)和普通的bean。
```xml
<!--applicationContext.xml-->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">

    <aop:aspectj-autoproxy/>

    <bean id="customerBo" class="com.aop.CustomerBoImp.CustomerBoImp"/>

    <!--Aspect-->
    <bean id = "logAspect" class="com.aop.Aspect.LoggingAspect"/>

</beans>
```

**4. Aspect类**
```java
//LoggingAspect
package com.aop.Aspect;
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;

import java.util.Arrays;

@Aspect
public class LoggingAspect {

    /**
     * 在方法执行前的通知。
     * @param joinPoint
     */
    @Before("execution(* com.aop.CustomerBo.CustomerBo.addCustomer(..))")
    public void logBefore(JoinPoint joinPoint) {

        System.out.println("logBefore() is running!");
        System.out.println("hijacked : " + joinPoint.getSignature().getName());
        System.out.println("******");
    }

    /**
     * 在方法执行后的通知，无论方法有没有发生异常。
     * @param joinPoint
     */
    @After("execution(* com.aop.CustomerBo.CustomerBo.addCustomer(..))")
    public void logAfter(JoinPoint joinPoint) {

        System.out.println("logAfter() is running!");
        System.out.println("hijacked : " + joinPoint.getSignature().getName());
        System.out.println("******");

    }

    /**
     * 在方法正常执行后的通知。能够访问到方法的返回值。
     * @param joinPoint
     * @param result
     */
    @AfterReturning(
            pointcut = "execution(* com.aop.CustomerBo.CustomerBo.addCustomerReturnValue(..))",
            returning= "result")
    public void logAfterReturning(JoinPoint joinPoint, Object result) {

        System.out.println("logAfterReturning() is running!");
        System.out.println("hijacked : " + joinPoint.getSignature().getName());
        System.out.println("Method returned value is : " + result);
        System.out.println("******");
    }

    /**
     * 在目标方法出现异常时会执行的代码。
     * @param joinPoint
     * @param error
     */
    @AfterThrowing(
            pointcut = "execution(* com.aop.CustomerBo.CustomerBo.addCustomerThrowException(..))",
            throwing= "error")
    public void logAfterThrowing(JoinPoint joinPoint, Throwable error) {

        System.out.println("logAfterThrowing() is running!");
        System.out.println("hijacked : " + joinPoint.getSignature().getName());
        System.out.println("Exception : " + error);
        System.out.println("******");

    }

    /**
     * 环绕通知需要携带ProceedingJoinPoint类型的参数，
     * 环绕通知类似于动态代理的全过程：ProceedingJoinPoint类型的参数可以决定是否执行目标方法。
     * @param joinPoint
     * @throws Throwable
     */
    @Around("execution(* com.aop.CustomerBo.CustomerBo.addCustomerAround(..))")
    public void logAround(ProceedingJoinPoint joinPoint) throws Throwable {

        System.out.println("logAround() is running!");
        System.out.println("hijacked method : " + joinPoint.getSignature().getName());
        System.out.println("hijacked arguments : " + Arrays.toString(joinPoint.getArgs()));

        System.out.println("Around before is running!");
        joinPoint.proceed(); //continue on the intercepted method
        System.out.println("Around after is running!");

        System.out.println("******");

    }
}
```

**5. App代码**
```java

package com.aop;
import com.aop.CustomerBo.CustomerBo;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class App {
    public static void main( String[] args ) throws Exception {
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");

        CustomerBo customerBo = (CustomerBo) applicationContext.getBean("customerBo");

        //customerBo.addCustomer();

        //customerBo.addCustomerReturnValue();

        //customerBo.addCustomerThrowException();

        customerBo.addCustomerAround("adamhand");
    }
}
```

**6. 执行结果**
```java
//语句：前置通知和后置通知
customerBo.addCustomer();

//结果
logBefore() is running!
hijacked : addCustomer
******
addCustomer() is running 
logAfter() is running!
hijacked : addCustomer
******
```
```java
//语句：方法返回后的通知
customerBo.addCustomerReturnValue();

//结果
addCustomerReturnValue() is running 
logAfterReturning() is running!
hijacked : addCustomerReturnValue
Method returned value is : abc
******
```
```java
//语句：异常通知
customerBo.addCustomerThrowException();

//结果
addCustomerThrowException() is running 
	at com.aop.CustomerBoImp.CustomerBoImp.addCustomerThrowException(CustomerBoImp.java:18)
logAfterThrowing() is running!
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
hijacked : addCustomerThrowException
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
Exception : java.lang.Exception: Generic Error
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
******
	at java.lang.reflect.Method.invoke(Method.java:498)
	...
******
```
```java
//语句：环绕通知
customerBo.addCustomerAround("adamhand")

//结果
logAround() is running!
hijacked method : addCustomerAround
hijacked arguments : [adamhand]
Around before is running!
addCustomerAround() is running, args : adamhand
Around after is running!
******
```

### 使用xml文件
AspectJ中注解方式和xml文件方式的对应关系如下：
>- `<aop:before> = @Before`
>- `<aop:after> = @After`
>- `<aop:after-returning> = @AfterReturning`
>- `<aop:after-throwing> = @AfterReturning`
>- `<aop:after-around> = @Around`

还是上面的例子，转成xml文件配置方式之后，只需要将LoggerAspect文件中的注解去掉，然后applicationContext.xml文件改为：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">

  <aop:aspectj-autoproxy/>

  <bean id="customerBo" class="com.aop.xml.CustomerBoImp.CustomerBoImp"/>

  <!--Aspect-->
  <bean id = "logAspect" class="com.aop.xml.Aspect.LoggingAspect"/>

  <aop:config>
    <aop:aspect id="aspectLoggging" ref="logAspect">
      <!-- @Before -->
      <aop:pointcut id="pointCutBefore"
                    expression="execution(* com.aop.xml.CustomerBo.CustomerBo.addCustomer(..))" />
      <aop:before method="logBefore" pointcut-ref="pointCutBefore" />

      <!-- @After -->
      <aop:pointcut id="pointCutAfter"
                    expression="execution(* com.aop.xml.CustomerBo.CustomerBo.addCustomer(..))" />
      <aop:after method="logAfter" pointcut-ref="pointCutAfter" />
      

      <!-- @AfterReturning -->
      <aop:pointcut id="pointCutAfterReturning"
                    expression="execution(* com.aop.xml.CustomerBo.CustomerBo.addCustomerReturnValue(..))" />
      <aop:after-returning method="logAfterReturning"
                           returning="result" pointcut-ref="pointCutAfterReturning" />

      <!-- @AfterThrowing -->
      <aop:pointcut id="pointCutAfterThrowing"
                    expression="execution(* com.aop.xml.CustomerBo.CustomerBo.addCustomerThrowException(..))" />
      <aop:after-throwing method="logAfterThrowing"
                          throwing="error" pointcut-ref="pointCutAfterThrowing" />

      <!-- @Around -->
      <aop:pointcut id="pointCutAround"
                    expression="execution(* com.aop.xml.CustomerBo.CustomerBo.addCustomerAround(..))" />

      <aop:around method="logAround" pointcut-ref="pointCutAround" />
    </aop:aspect>
  </aop:config>

</beans>
```

## 使用Spring自带的工具实现AOP
在Spring AOP中，有 4 种类型通知(advices)的支持：
> - 通知(Advice)之前 - 该方法执行前运行
> - 通知(Advice)返回之后 – 运行后，该方法返回一个结果 
> - 通知(Advice)抛出之后 – 运行方法抛出异常后
> - 环绕通知 – 环绕方法执行运行，结合以上这三个通知

使用Spring实现AOP的方式也有两种：经典的基于代理的AOP实现(即不使用`<aop:fonfig>`等标签)；另一种是用纯粹的POJO切面（就是纯粹通过`<aop:fonfig>`标签配置，也是一种比较简单的方式）。

### 经典方式
**例子在eclipse下面会执行成功，但是在idea下面会报Exception in thread "main" java.lang.ClassCastException: com.sun.proxy.$Proxy2 cannot be cast to com.aop.self.CustomerService.CustomerService错误，不知道是什么原因。以下程序是使用eclipse编写的**

**1. 创建被代理类法**
```java
package com.yiibai.customer.services;

public class CustomerService
{
	private String name;
	private String url;

	public void setName(String name) {
		this.name = name;
	}

	public void setUrl(String url) {
		this.url = url;
	}

	public void printName(){
		System.out.println("Customer name : " + this.name);
	}
	
	public void printURL(){
		System.out.println("Customer website : " + this.url);
	}
	
	public void printThrowException(){
		throw new IllegalArgumentException();
	}
}
```

**2. 前置通知**
> - 创建一个实现 MethodBeforeAdvice 接口的类。
> - 在 bean 配置文件(applicationContext.xml)，创建一个 bean 的 HijackBeforeMethod 类，并命名为“customerServiceProxy” 作为一个新的代理对象。
    - ‘target’ – 定义你想拦截的bean。 
    - ‘interceptorNames’ – 定义要应用这个代理/目标对象的类(通知)。
    
```java
package com.yiibai.aop;

import java.lang.reflect.Method;
import org.springframework.aop.MethodBeforeAdvice;

//before a method is invoke
public class HijackBeforeMethod implements MethodBeforeAdvice
{
	@Override
	public void before(Method method, Object[] args, Object target)
			throws Throwable {
		System.out.println("HijackBeforeMethod : Before method hijacked!");
	}
}
```
```xml
<bean id="customerService" class="com.yiibai.customer.services.CustomerService" >
   	<property name="name" value="Yiibai" />
   	<property name="url" value="http://www.yiibai.com" />
</bean>

<bean id="HijackBeforeMethodBeanAdvice" class="com.yiibai.aop.HijackBeforeMethod" />

<bean id="customerServiceProxy"
	class="org.springframework.aop.framework.ProxyFactoryBean">

	<property name="target" ref="customerService" />
	
	<property name="interceptorNames">
		<list>
			<value>customerAdvisor</value>
		</list>
	</property>
</bean>

 <bean id="customerAdvisor"
	class="org.springframework.aop.support.NameMatchMethodPointcutAdvisor">
	<property name="mappedName" value="printName" />
	<property name="advice" ref="HijackBeforeMethodBeanAdvice" />
</bean> 
```

**3. 后置通知**
创建一个实现AfterReturningAdvice接口的类。配置文件的写法和上面差不多，只是将HijackBeforeMethod 改为HijackAfterMethod 。
```java
package com.yiibai.aop;

import java.lang.reflect.Method;
import org.springframework.aop.AfterReturningAdvice;

//after a method is invoke
public class HijackAfterMethod implements AfterReturningAdvice
{
	@Override
	public void afterReturning(Object returnValue, Method method,
			Object[] args, Object target) throws Throwable {
		System.out.println("HijackAfterMethod : After method hijacked!");
	}
}
```

**4. 抛出后通知**
它将在执行方法抛出一个异常后。创建一个实现ThrowsAdvice接口的类，并创建一个afterThrowing方法拦截抛出：IllegalArgumentException异常。
```java
package com.yiibai.aop;

import org.springframework.aop.ThrowsAdvice;

public class HijackThrowException implements ThrowsAdvice
{
	public void afterThrowing(IllegalArgumentException e) throws Throwable {
		System.out.println("HijackThrowException : Throw exception hijacked!");
	}
}
```

**5. 环绕通知**
它结合了上面的三个通知，在方法执行过程中执行。创建一个实现了MethodInterceptor接口的类。必须调用“methodInvocation.proceed();” 继续在原来的方法执行，否则原来的方法将不会执行。
```java
package com.yiibai.aop;

import java.util.Arrays;
import org.aopalliance.intercept.MethodInterceptor;
import org.aopalliance.intercept.MethodInvocation;

public class HijackAroundMethod implements MethodInterceptor
{
	@Override
	public Object invoke(MethodInvocation methodInvocation) throws Throwable {
		
		System.out.println("Method name : " 
				+ methodInvocation.getMethod().getName());
		System.out.println("Method arguments : " 
				+ Arrays.toString(methodInvocation.getArguments()));

		//same with MethodBeforeAdvice
		System.out.println("HijackAroundMethod : Before method hijacked!");
		
		try{
			//proceed to original method call
			Object result = methodInvocation.proceed();
		
			//same with AfterReturningAdvice
			System.out.println("HijackAroundMethod : Before after hijacked!");
		
			return result;
		
		}catch(IllegalArgumentException e){
			//same with ThrowsAdvice
			System.out.println("HijackAroundMethod : Throw exception hijacked!");
			throw e;
		}	
	}	
}
```
配置文件为：
```xml
<bean id="customerService" class="com.yiibai.customer.services.CustomerService" >
   	<property name="name" value="Yiibai" />
   	<property name="url" value="http://www.yiibai.com" />
</bean>

<bean id="hijackAroundMethodBeanAdvice" class="com.yiibai.aop.HijackAroundMethod" />

<bean id="customerServiceProxy"
	class="org.springframework.aop.framework.ProxyFactoryBean">

	<property name="target" ref="customerService" />
	
	<property name="interceptorNames">
		<list>
			<value>customerAdvisor</value>
		</list>
	</property>
</bean>

<!--两种写法-->
<!--  	<bean id="customerAdvisor"
	class="org.springframework.aop.support.NameMatchMethodPointcutAdvisor">
	<property name="mappedName" value="printName" />
	<property name="advice" ref="hijackAroundMethodBeanAdvice" />
</bean>  -->

 <bean id="customerAdvisor"
	class="org.springframework.aop.support.RegexpMethodPointcutAdvisor">
	<property name="patterns">
		<list>
			<value>.*URL.*</value>
		</list>
	</property>

	<property name="advice" ref="hijackAroundMethodBeanAdvice" />
</bean> 
```

### 使用`<aop:fonfig>`标签
**1. 定义接口和借口实现类**
```java
package com.aop.pojo.Sleepable;

public interface Sleepable {
    public void sleep();
}
```
```java
package com.aop.pojo.SleepImp;

import com.aop.pojo.Sleepable.Sleepable;

public class SleepImp implements Sleepable {
    @Override
    public void sleep() {
        System.out.println("睡觉");
    }
}
```

**2. 定义切面类**
```java
package com.aop.pojo.SleepHelper;

public class SleepHelper {
    public void beforeSleep(){
        System.out.println("睡觉前要脱衣服");
    }

    public void afterSleep(){
        System.out.println("起床后要穿衣服");
    }
}
```

**3. 配置文件**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">

    <!--定义被代理类-->
    <bean id="sleepImp" class="com.aop.pojo.SleepImp.SleepImp"/>
    <!--定义通知内容，也就是切入点执行前后需要做的事情-->
    <bean id="sleepHelper" class="com.aop.pojo.SleepHelper.SleepHelper"/>
    
    <!--另一种写法-->
<!--    <aop:config>
        <aop:aspect ref="sleepHelper">
            <aop:before method="beforeSleep" pointcut="execution(* *.sleep(..))"/>
            <aop:after method="afterSleep" pointcut="execution(* *.sleep(..))"/>
        </aop:aspect>
    </aop:config>-->

    <aop:config>
        <aop:aspect ref="sleepHelper">
            <aop:pointcut id="sleepHelpers" expression="execution(* *.sleep(..))" />
            <aop:before pointcut-ref="sleepHelpers" method="beforeSleep" />
            <aop:after pointcut-ref="sleepHelpers" method="afterSleep" />
        </aop:aspect>
    </aop:config>

</beans>
```

**4. 主类**
```java
public class App 
{
    public static void main( String[] args )
    {
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        Sleepable sleepImp = (Sleepable) context.getBean("sleepImp");

        sleepImp.sleep();
    }
}
```
---
**注意：**
Spring AOP部分使用JDK动态代理或者CGLIB来为目标对象创建代理。如果被代理的目标实现了至少一个接口，则会使用JDK动态代理。所有该目标类型实现的接口都将被代理。若该目标对象没有实现任何接口，则使用CGLIB代理。

如果用JDK动态代理，就必须为被代理的目标实现一个接口（要注意的地方是：需要将ctx.getBean()方法的返回值用接口类型接收）；如果使用CGLIB强制代理，就必选事先将CGLIB包导入项目，并在配置文件中设置`proxy-target-class="true"`，这个属性决定是基于接口的还是基于类的代理被创建：
```xml
<aop:config proxy-target-class="true">
    <aop:aspect ref="sleepHelper">
        <aop:pointcut id="sleepHelpers" expression="execution(* *.sleep(..))" />
        <aop:before pointcut-ref="sleepHelpers" method="beforeSleep" />
        <aop:after pointcut-ref="sleepHelpers" method="afterSleep" />
    </aop:aspect>
</aop:config>
```

---

## 总结：Spring AOP的实现方式
使用Spring实现AOP方式分为两种：
>- 使用Spring原生的方式：JDKProxy或者CGLib
>- 使用AspectJ：使用注解或者xml

---

# Spring AOP实现原理
下面来研究一下Spring如何使用JDK来生成代理对象，具体的生成代码放在JdkDynamicAopProxy这个类中，直接上相关代码：
```java
/**
    * <ol>
    * <li>获取代理类要实现的接口,除了Advised对象中配置的,还会加上SpringProxy, Advised(opaque=false)
    * <li>检查上面得到的接口中有没有定义 equals或者hashcode的接口
    * <li>调用Proxy.newProxyInstance创建代理对象
    * </ol>
    */
   public Object getProxy(ClassLoader classLoader) {
       if (logger.isDebugEnabled()) {
           logger.debug("Creating JDK dynamic proxy: target source is " +this.advised.getTargetSource());
       }
       Class[] proxiedInterfaces =AopProxyUtils.completeProxiedInterfaces(this.advised);
       findDefinedEqualsAndHashCodeMethods(proxiedInterfaces);
       return Proxy.newProxyInstance(classLoader, proxiedInterfaces, this);
}
```
下面的问题是，代理对象生成了，那切面是如何织入的？

我们知道InvocationHandler是JDK动态代理的核心，生成的代理对象的方法调用都会委托到InvocationHandler.invoke()方法。而通过JdkDynamicAopProxy的签名我们可以看到这个类其实也实现了InvocationHandler，下面我们就通过分析这个类中实现的invoke()方法来具体看下Spring AOP是如何织入切面的。
```java
publicObject invoke(Object proxy, Method method, Object[] args) throwsThrowable {
       MethodInvocation invocation = null;
       Object oldProxy = null;
       boolean setProxyContext = false;
 
       TargetSource targetSource = this.advised.targetSource;
       Class targetClass = null;
       Object target = null;
 
       try {
           //eqauls()方法，具目标对象未实现此方法
           if (!this.equalsDefined && AopUtils.isEqualsMethod(method)){
                return (equals(args[0])? Boolean.TRUE : Boolean.FALSE);
           }
 
           //hashCode()方法，具目标对象未实现此方法
           if (!this.hashCodeDefined && AopUtils.isHashCodeMethod(method)){
                return newInteger(hashCode());
           }
 
           //Advised接口或者其父接口中定义的方法,直接反射调用,不应用通知
           if (!this.advised.opaque &&method.getDeclaringClass().isInterface()
                    &&method.getDeclaringClass().isAssignableFrom(Advised.class)) {
                // Service invocations onProxyConfig with the proxy config...
                return AopUtils.invokeJoinpointUsingReflection(this.advised,method, args);
           }
 
           Object retVal = null;
 
           if (this.advised.exposeProxy) {
                // Make invocation available ifnecessary.
                oldProxy = AopContext.setCurrentProxy(proxy);
                setProxyContext = true;
           }
 
           //获得目标对象的类
           target = targetSource.getTarget();
           if (target != null) {
                targetClass = target.getClass();
           }
 
           //获取可以应用到此方法上的Interceptor列表
           List chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method,targetClass);
 
           //如果没有可以应用到此方法的通知(Interceptor)，此直接反射调用 method.invoke(target, args)
           if (chain.isEmpty()) {
                retVal = AopUtils.invokeJoinpointUsingReflection(target,method, args);
           } else {
                //创建MethodInvocation
                invocation = newReflectiveMethodInvocation(proxy, target, method, args, targetClass, chain);
                retVal = invocation.proceed();
           }
 
           // Massage return value if necessary.
           if (retVal != null && retVal == target &&method.getReturnType().isInstance(proxy)
                    &&!RawTargetAccess.class.isAssignableFrom(method.getDeclaringClass())) {
                // Special case: it returned"this" and the return type of the method
                // is type-compatible. Notethat we can't help if the target sets
                // a reference to itself inanother returned object.
                retVal = proxy;
           }
           return retVal;
       } finally {
           if (target != null && !targetSource.isStatic()) {
                // Must have come fromTargetSource.
               targetSource.releaseTarget(target);
           }
           if (setProxyContext) {
                // Restore old proxy.
                AopContext.setCurrentProxy(oldProxy);
           }
       }
    }
```
主流程可以简述为：获取可以应用到此方法上的通知链（Interceptor Chain）,如果有,则应用通知,并执行joinpoint; 如果没有,则直接反射执行joinpoint。而这里的关键是通知链是如何获取的以及它又是如何执行的，下面逐一分析下。

首先，从上面的代码可以看到，通知链是通过Advised.getInterceptorsAndDynamicInterceptionAdvice()这个方法来获取的,我们来看下这个方法的实现:
```java
public List<Object>getInterceptorsAndDynamicInterceptionAdvice(Method method, Class targetClass) {
   MethodCacheKeycacheKey = new MethodCacheKey(method);
   List<Object>cached = this.methodCache.get(cacheKey);
   if(cached == null) {
            cached= this.advisorChainFactory.getInterceptorsAndDynamicInterceptionAdvice(
                               this,method, targetClass);
            this.methodCache.put(cacheKey,cached);
   }
   returncached;
}
```
可以看到实际的获取工作其实是由AdvisorChainFactory. getInterceptorsAndDynamicInterceptionAdvice()这个方法来完成的，获取到的结果会被缓存。

下面来分析下这个方法的实现：
```java
/**
    * 从提供的配置实例config中获取advisor列表,遍历处理这些advisor.如果是IntroductionAdvisor,
    * 则判断此Advisor能否应用到目标类targetClass上.如果是PointcutAdvisor,则判断
    * 此Advisor能否应用到目标方法method上.将满足条件的Advisor通过AdvisorAdaptor转化成Interceptor列表返回.
    */
    publicList getInterceptorsAndDynamicInterceptionAdvice(Advised config, Methodmethod, Class targetClass) {
       // This is somewhat tricky... we have to process introductions first,
       // but we need to preserve order in the ultimate list.
       List interceptorList = new ArrayList(config.getAdvisors().length);
 
       //查看是否包含IntroductionAdvisor
       boolean hasIntroductions = hasMatchingIntroductions(config,targetClass);
 
       //这里实际上注册一系列AdvisorAdapter,用于将Advisor转化成MethodInterceptor
       AdvisorAdapterRegistry registry = GlobalAdvisorAdapterRegistry.getInstance();
 
       Advisor[] advisors = config.getAdvisors();
        for (int i = 0; i <advisors.length; i++) {
           Advisor advisor = advisors[i];
           if (advisor instanceof PointcutAdvisor) {
                // Add it conditionally.
                PointcutAdvisor pointcutAdvisor= (PointcutAdvisor) advisor;
                if(config.isPreFiltered() ||pointcutAdvisor.getPointcut().getClassFilter().matches(targetClass)) {
                    //TODO: 这个地方这两个方法的位置可以互换下
                    //将Advisor转化成Interceptor
                    MethodInterceptor[]interceptors = registry.getInterceptors(advisor);
 
                    //检查当前advisor的pointcut是否可以匹配当前方法
                    MethodMatcher mm =pointcutAdvisor.getPointcut().getMethodMatcher();
 
                    if (MethodMatchers.matches(mm,method, targetClass, hasIntroductions)) {
                        if(mm.isRuntime()) {
                            // Creating a newobject instance in the getInterceptors() method
                            // isn't a problemas we normally cache created chains.
                            for (intj = 0; j < interceptors.length; j++) {
                               interceptorList.add(new InterceptorAndDynamicMethodMatcher(interceptors[j],mm));
                            }
                        } else {
                            interceptorList.addAll(Arrays.asList(interceptors));
                        }
                    }
                }
           } else if (advisor instanceof IntroductionAdvisor){
                IntroductionAdvisor ia =(IntroductionAdvisor) advisor;
                if(config.isPreFiltered() || ia.getClassFilter().matches(targetClass)) {
                    Interceptor[] interceptors= registry.getInterceptors(advisor);
                    interceptorList.addAll(Arrays.asList(interceptors));
                }
           } else {
                Interceptor[] interceptors =registry.getInterceptors(advisor);
                interceptorList.addAll(Arrays.asList(interceptors));
           }
       }
       return interceptorList;
}
```
这个方法执行完成后，Advised中配置能够应用到连接点或者目标类的Advisor全部被转化成了MethodInterceptor。

接下来我们再看下得到的拦截器链是怎么起作用的。
```java
if (chain.isEmpty()) {
    retVal = AopUtils.invokeJoinpointUsingReflection(target,method, args);
} else {
    //创建MethodInvocation
    invocation = newReflectiveMethodInvocation(proxy, target, method, args, targetClass, chain);
    retVal = invocation.proceed();
}
```
从这段代码可以看出，如果得到的拦截器链为空，则直接反射调用目标方法，否则创建MethodInvocation，调用其proceed方法，触发拦截器链的执行，来看下具体代码
```java
public Object proceed() throws Throwable {
   //  We start with an index of -1and increment early.
   if (this.currentInterceptorIndex == this.interceptorsAndDynamicMethodMatchers.size()- 1) {
       //如果Interceptor执行完了，则执行joinPoint
       return invokeJoinpoint();
   }

   Object interceptorOrInterceptionAdvice =
       this.interceptorsAndDynamicMethodMatchers.get(++this.currentInterceptorIndex);
   
   //如果要动态匹配joinPoint
   if (interceptorOrInterceptionAdvice instanceof InterceptorAndDynamicMethodMatcher){
       // Evaluate dynamic method matcher here: static part will already have
       // been evaluated and found to match.
       InterceptorAndDynamicMethodMatcher dm =
            (InterceptorAndDynamicMethodMatcher)interceptorOrInterceptionAdvice;
       //动态匹配：运行时参数是否满足匹配条件
       if (dm.methodMatcher.matches(this.method, this.targetClass,this.arguments)) {
            //执行当前Intercetpor
            returndm.interceptor.invoke(this);
       }
       else {
            //动态匹配失败时,略过当前Intercetpor,调用下一个Interceptor
            return proceed();
       }
   }
   else {
       // It's an interceptor, so we just invoke it: The pointcutwill have
       // been evaluated statically before this object was constructed.
       //执行当前Intercetpor
       return ((MethodInterceptor) interceptorOrInterceptionAdvice).invoke(this);
   }
}
```

# 参考
[Spring AOP四种实现方式Demo详解与相关知识探究](https://blog.csdn.net/zhangliangzi/article/details/52334964)
[Spring AOP 的实现机制](http://www.importnew.com/28342.html)
[Spring3：AOP](http://www.cnblogs.com/xrq730/p/4919025.html)
[Spring AOP 实现原理](https://blog.csdn.net/MoreeVan/article/details/11977115)
[java.lang.ClassCastException: com.sun.proxy.$Proxy27 cannot be cast to com.bbk.n002.service.QuestionService](https://www.cnblogs.com/liuling/p/2014-8-23-001.html)
[Spring—AOP两种代理机制对比（JDK和CGLib动态代理）](https://blog.csdn.net/qq1723205668/article/details/56481476)
[Spring AOP通知实例 – Advice](https://www.yiibai.com/spring/spring-aop-examples-advice.html#)