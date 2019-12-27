<!-- TOC -->

- [1. Spring概念](#1-spring概念)
- [2. IOC(DI)](#2-iocdi)
    - [2.1. IOC和DI的概念和理解](#21-ioc和di的概念和理解)
    - [2.2. IOC原理](#22-ioc原理)
        - [2.2.1. 工厂模式和反射](#221-工厂模式和反射)
        - [2.2.2. IOC容器的技术剖析](#222-ioc容器的技术剖析)
        - [2.2.3. spring运行原理](#223-spring运行原理)
- [3. IOC配置(配置bean)](#3-ioc配置配置bean)
    - [3.1. xml方式](#31-xml方式)
        - [3.1.1. 通过全类名（反射）](#311-通过全类名反射)
        - [3.1.2. 通过工厂方法配置bean](#312-通过工厂方法配置bean)
            - [3.1.2.1. 静态工厂方法](#3121-静态工厂方法)
            - [3.1.2.2. 实例工厂方法](#3122-实例工厂方法)
        - [3.1.3. 通过FactoryBean配置bean（需要继承FactoryBean接口）](#313-通过factorybean配置bean需要继承factorybean接口)
    - [3.2. 注解方式](#32-注解方式)
        - [3.2.1. 组件扫描](#321-组件扫描)
        - [3.2.2. 常用注解](#322-常用注解)
            - [3.2.2.1. @Bean](#3221-bean)
            - [3.2.2.2. @Autowired](#3222-autowired)
            - [3.2.2.3. @Required](#3223-required)
            - [3.2.2.4. @Qualifier](#3224-qualifier)
        - [3.2.3. JSR(Java Specification Requests，Java规范提案)注解](#323-jsrjava-specification-requestsjava规范提案注解)
            - [3.2.3.1. @Resource](#3231-resource)
        - [几个注解之间的区别](#几个注解之间的区别)
            - [@Component和@Configuration](#component和configuration)
            - [什么时候使用@Bean](#什么时候使用bean)
    - [3.3. 注解方式和xml文件方式的关系](#33-注解方式和xml文件方式的关系)
- [4. DI(依赖注入方式)](#4-di依赖注入方式)
    - [4.1. setter注入](#41-setter注入)
    - [4.2. 构造器注入](#42-构造器注入)
    - [4.3. 工厂方法注入（比较少用，先放置）](#43-工厂方法注入比较少用先放置)
    - [4.4. setter注入的三种写法](#44-setter注入的三种写法)
        - [4.4.1. 正常方式](#441-正常方式)
        - [4.4.2. 快捷方式](#442-快捷方式)
        - [4.4.3. p模式](#443-p模式)
- [5. 在一个bean里引用另一个bean](#5-在一个bean里引用另一个bean)
    - [5.1. ref=”beanName”](#51-refbeanname)
        - [5.1.1. setter](#511-setter)
        - [5.1.2. 构造器](#512-构造器)
    - [5.2. ref bean=”beanName”](#52-ref-beanbeanname)
        - [5.2.1. setter](#521-setter)
        - [5.2.2. 构造器](#522-构造器)
    - [5.3. 内部bean](#53-内部bean)
- [6. 级联属性赋值](#6-级联属性赋值)
- [7. 集合属性](#7-集合属性)
    - [7.1. List示例](#71-list示例)
    - [7.2. Set示例](#72-set示例)
    - [7.3. Map示例](#73-map示例)
    - [7.4. Properties示例](#74-properties示例)
    - [7.5. 单例的集合bean](#75-单例的集合bean)
- [8. 参考](#8-参考)
- [9. spring、pring boot和spring mvc的区别](#9-springpring-boot和spring-mvc的区别)

<!-- /TOC -->

# 1. Spring概念
spring是开源的轻量级框架

spring核心主要两部分：
-  aop：面向切面编程，扩展功能不是修改源代码实现
-  ioc：控制反转：对象的创建不是通过new方式实现，而是交给spring配置创建类对象

spring是一站式框架，spring在javaee三层结构中，每一层都提供不同的解决技术
-  web层：springMVC
-  service层：spring的ioc
-  dao层：spring的jdbcTemplate

# 2. IOC(DI)
## 2.1. IOC和DI的概念和理解
IOC（DI）：通常，每个对象在使用他的合作对象时，自己均要使用像new object（） 这样的语法来完成合作对象的申请工作，比如在A类的对象中要使用B类的对象，就要通过new方法创建一个B类对象。这样，对象间的耦合度高了。而IOC的思想是：Spring容器来实现这些相互依赖对象的创建、协调工作。对象只需要关系业务逻辑本身就可以了。从这方面来说，对象如何得到他的协作对象的责任被反转了（IOC、DI）。

DI其实就是IOC的另外一种说法。DI是由Martin Fowler 在2004年初的一篇论文中首次提出的。他总结：控制的什么被反转了？就是：获得**依赖对象**的方式反转了。

下面通俗一点理解IOC和DI：

- **IoC（Inversion of Control，控制倒转）**。这是spring的核心，贯穿始终。所谓IoC，对于spring框架来说，就是由spring来负责控制对象的生命周期和对象间的关系。打个找对象的比方，按照一般做法，找对象的每个环节都得自己设计和实现，包括相亲、吃饭、看电影、结婚等等。但是oru如果使用IOC，情况就简单多了，IOC相当于一个婚姻介绍所，双方都会将自己的情况告诉婚介所，并告诉它自己理想的另一半的条件，婚介所会按照提供的要求提供一个异性；如果婚介的人选不符合要求，就会抛出异常。整个过程不再由自己控制，而是有婚介这样一个类似容器的机构来控制。
Spring所倡导的开发方式就是如此，所有的类都会在spring容器中登记，告诉spring你是个什么东西，你需要什么东西，然后spring会在系统运行到适当的时候，把你要的东西主动给你，同时也把你交给其他需要你的东西。所有的类的创建、销毁都由 spring来控制，也就是说控制对象生存周期的不再是引用它的对象，而是spring。对于某个具体的对象而言，以前是它控制其他对象，现在是所有对象都被spring控制，所以这叫控制反转。
- **DI（Dependency Injection，依赖注入）**。IoC的一个重点是在系统运行中，动态的向某个对象提供它所需要的其他对象。这一点是通过DI（Dependency Injection，依赖注入）来实现的。比如对象A需要操作数据库，以前我们总是要在A中自己编写代码来获得一个Connection对象，有了 spring我们就只需要告诉spring，A中需要一个Connection，至于这个Connection怎么构造，何时构造，A不需要知道。在系统运行时，spring会在适当的时候制造一个Connection，然后像打针一样，注射到A当中，这样就完成了对各个对象之间关系的控制。A需要依赖 Connection才能正常运行，而这个Connection是由spring注入到A中的，依赖注入的名字就这么来的。依赖注入是通过反射来实现的。

## 2.2. IOC原理
**Spring中的IoC的实现原理就是工厂模式加反射机制**

### 2.2.1. 工厂模式和反射
先来看一下工厂模式。下面是不使用反射的工厂模式。
```java
interface fruit{
    public abstract void eat();
} 
class Apple implements fruit{
     public void eat(){
         System.out.println("Apple");
     }
} 
class Orange implements fruit{
     public void eat(){
         System.out.println("Orange");
     }
}
//构造工厂类
//也就是说以后如果我们在添加其他的实例的时候只需要修改工厂类就行了
class Factory{
     public static fruit getInstance(String fruitName){
         fruit f=null;
         if("Apple".equals(fruitName)){
             f=new Apple();
         }
         if("Orange".equals(fruitName)){
             f=new Orange();
         }
         return f;
     }
}
class hello{
     public static void main(String[] a){
         fruit f=Factory.getInstance("Orange");
         f.eat();
     }
}
```
上面写法的缺点是当我们再添加一个子类的时候，就需要修改工厂类了。如果我们添加太多的子类的时候，改动就会很多。下面用反射机制实现工厂模式：
```java
interface fruit{
     public abstract void eat();
}
class Apple implements fruit{
    public void eat(){
         System.out.println("Apple");
     }
}
class Orange implements fruit{
    public void eat(){
        System.out.println("Orange");
    }
}
class Factory{
    public static fruit getInstance(String ClassName){
        fruit f=null;
        try{
            f=(fruit)Class.forName(ClassName).newInstance();
        }catch (Exception e) {
            e.printStackTrace();
        }
        return f;
    }
}
class hello{
    public static void main(String[] a){
        fruit f=Factory.getInstance("Reflect.Apple");
        if(f!=null){
            f.eat();
        }
    }
}
```
 现在就算我们添加任意多个子类的时候，工厂类都不需要修改。使用反射机制实现的工厂模式可以通过反射取得接口的实例，但是需要传入完整的包和类名。而且用户也无法知道一个接口有多少个可以使用的子类，所以我们通过属性文件的形式配置所需要的子类。

下面编写使用反射机制并结合属性文件的工厂模式（即IoC）。首先创建一个fruit.properties的资源文件：
```java
apple=Reflect.Apple
orange=Reflect.Orange
```
然后编写主类代码：
```java
interface fruit{
    public abstract void eat();
}
class Apple implements fruit{
    public void eat(){
        System.out.println("Apple");
    }
}
class Orange implements fruit{
    public void eat(){
        System.out.println("Orange");
    }
}
//操作属性文件类
class init{
    public static Properties getPro() throws FileNotFoundException, IOException{
        Properties pro=new Properties();
        File f=new File("fruit.properties");
        if(f.exists()){
            pro.load(new FileInputStream(f));
        }else{
            pro.setProperty("apple", "Reflect.Apple");
            pro.setProperty("orange", "Reflect.Orange");
            pro.store(new FileOutputStream(f), "FRUIT CLASS");
        }
        return pro;
    }
}
class Factory{
    public static fruit getInstance(String ClassName){
        fruit f=null;
        try{
            f=(fruit)Class.forName(ClassName).newInstance();
        }catch (Exception e) {
            e.printStackTrace();
        }
        return f;
    }
}
class hello{
    public static void main(String[] a) throws FileNotFoundException, IOException{
        Properties pro=init.getPro();
        fruit f=Factory.getInstance(pro.getProperty("apple"));
        if(f!=null){
            f.eat();
        }
    }
}
```

### 2.2.2. IOC容器的技术剖析
IOC中最基本的技术就是“反射(Reflection)”编程，通俗来讲就是**根据给出的类名（字符串方式）来动态地生成对象，这种编程方式可以让对象在生成时才被决定到底是哪一种对象。**只是在Spring中要生产的对象都在配置文件中给出定义，目的就是提高灵活性和可维护性。

我们可以**把IOC容器的工作模式看做是工厂模式的升华，可以把IOC容器看作是一个工厂，这个工厂里要生产的对象都在配置文件中给出定义，然后利用编程语言提供的反射机制，根据配置文件中给出的类名生成相应的对象。**从实现来看，IOC是把以前在工厂方法里写死的对象生成代码，改变为由配置文件来定义，也就是把工厂和对象生成这两者独立分隔开来，目的就是提高灵活性和可维护性。

### 2.2.3. spring运行原理
main函数如下：
```java
public static void main(String[] args) {
	ApplicationContext context = new FileSystemXmlApplicationContext("applicationContext.xml");
	Animal animal = (Animal) context.getBean("animal");
	animal.say();
}
```
下面是applicationContext.xml：
```java
<bean id="animal" class="phz.springframework.test.Cat">
	<property name="name" value="kitty" />
</bean>
```
它其中定义了一个bean：phz.springframework.test.Cat
```java
public class Cat implements Animal {
	private String name;
	public void say() {
		System.out.println("I am " + name + "!");
	}
	public void setName(String name) {
		this.name = name;
	}
}
```
Cat类实现了phz.springframework.test.Animal接口
```java
public interface Animal {
	public void say();
}
```
很明显上面的代码输出I am kitty!那么到底Spring是如何做到的呢？接下来写个简单的Spring 来看看Spring 到底是怎么运行的。
首先，我们定义一个Bean类，这个类用来存放一个Bean拥有的属性：
```java
/* Bean Id */
private String id;
/* Bean Class */
private String type;
/* Bean Property */
private Map<String, Object> properties = new HashMap<String, Object>();
```
一个Bean包括id,type,和Properties。接下来Spring 就开始加载我们的配置文件了，将配置的信息保存在一个HashMap中，HashMap的key就是Bean 的 Id ，HasMap 的value是这个Bean，只有这样才能通过context.getBean("animal")这个方法获得Animal这个类。我们都知道Spirng可以注入基本类型，而且可以注入像List，Map这样的类型，接下来就让我们以Map为例看看Spring是怎么保存的吧 。

Map配置可以像下面的
```java
<bean id="test" class="Test">
	<property name="testMap">
		<map>
			<entry key="a">
				<value>1</value>
			</entry>
			<entry key="b">
				<value>2</value>
			</entry>
		</map>
	</property>
</bean>
```
Spring是怎样保存上面的配置呢？代码如下：
```java
if (beanProperty.element("map") != null) {
	Map<String, Object> propertiesMap = new HashMap<String, Object>();
	Element propertiesListMap = (Element) beanProperty.elements().get(0);
	Iterator<?> propertiesIterator = propertiesListMap.elements().iterator();
	
	while (propertiesIterator.hasNext()) {
		Element vet = (Element) propertiesIterator.next();
		if (vet.getName().equals("entry")) {
			String key = vet.attributeValue("key");
			Iterator<?> valuesIterator = vet.elements().iterator();
			while (valuesIterator.hasNext()) {
				Element value = (Element) valuesIterator.next();
				if (value.getName().equals("value")) {
					propertiesMap.put(key, value.getText());
				}
				if (value.getName().equals("ref")) {
					propertiesMap.put(key, new String[] { value.attributeValue("bean") });
				}
			}
		}
	}
	bean.getProperties().put(name, propertiesMap);
}
```
接下来就进入最核心部分了，让我们看看Spring 到底是怎么依赖注入的吧，其实依赖注入的思想也很简单，它是通过反射机制实现的，在实例化一个类时，它通过反射调用类中set方法将事先保存在HashMap中的类属性注入到类中。让我们看看具体它是怎么做的吧。 
首先实例化一个类，像这样
```java
public static Object newInstance(String className) {
	Class<?> cls = null;
	Object obj = null;
	try {
		cls = Class.forName(className);
		obj = cls.newInstance();
	} catch (ClassNotFoundException e) {
		throw new RuntimeException(e);
	} catch (InstantiationException e) {
		throw new RuntimeException(e);
	} catch (IllegalAccessException e) {
		throw new RuntimeException(e);
	}
	return obj;
}
```
接着它将这个类的依赖注入进去，像这样
```java
public static void setProperty(Object obj, String name, String value) {
	Class<? extends Object> clazz = obj.getClass();
	try {
		String methodName = returnSetMthodName(name);
		Method[] ms = clazz.getMethods();
		for (Method m : ms) {
			if (m.getName().equals(methodName)) {
				if (m.getParameterTypes().length == 1) {
					Class<?> clazzParameterType = m.getParameterTypes()[0];
					setFieldValue(clazzParameterType.getName(), value, m,obj);
					break;
				}
			}
		}
	} catch (SecurityException e) {
		throw new RuntimeException(e);
	} catch (IllegalArgumentException e) {
		throw new RuntimeException(e);
	} catch (IllegalAccessException e) {
		throw new RuntimeException(e);
	} catch (InvocationTargetException e) {
		throw new RuntimeException(e);
	}
}
```
最后它将这个类的实例返回给我们，我们就可以用了。我们还是以Map为例看看它是怎么做的，我写的代码里面是创建一个HashMap并把该HashMap注入到需要注入的类中，像这样，
```java
if (value instanceof Map) {
	Iterator<?> entryIterator = ((Map<?, ?>) value).entrySet().iterator();
	Map<String, Object> map = new HashMap<String, Object>();
	while (entryIterator.hasNext()) {
		Entry<?, ?> entryMap = (Entry<?, ?>) entryIterator.next();
		if (entryMap.getValue() instanceof String[]) {
			map.put((String) entryMap.getKey(),
				getBean(((String[]) entryMap.getValue())[0]));
		}
	}
	BeanProcesser.setProperty(obj, property, map);
}
```

# 3. IOC配置(配置bean)
什么是bean。首先，spring一个很重要的贡献就是解耦，它将 类(class) 从源文件中抽离出来，放到xml文件中，使得我们可以用xml文件对源文件中的类进行配置和修改，这样就不用每次跑去乱找源文件。但如果我们需要在xml文件中对源文件进行修改，必须在xml文件中建立一种映射关系，也就是说在xml文件中为源文件中的类起一个“别名”，并形成关联，这样我们修改xml文件的时候才会调用源文件的中的对应类和属性。而bean就是源文件中的类在xml文件中的“别名”。

bean的配置形式有两种：**通过xml的方式**和**通过注解的方式**。其中，通过xml的方式又可以分为三种
-  通过全类名（反射）方式
-  通过工厂方法（静态工厂方法&实例工厂方法）
-  FactoryBean的方式。

## 3.1. xml方式
### 3.1.1. 通过全类名（反射）
要求，源文件中要有一个无参的构造函数。
```java
<bean id = “helloWorld” class = “com.cn.HelloWorld”> 
	<property name=”haha” value=”spring”></property>
</bean>
```

### 3.1.2. 通过工厂方法配置bean
#### 3.1.2.1. 静态工厂方法
直接调用某一个类的静态方法就可以返回bean的实例，不需要创建对象。
```java
package com.yl.factory;

import java.util.HashMap;
import java.util.Map;

/**
 * 静态工厂方法：直接调用某一个类的静态方法就可与返回bean的实例
 * @author yul
 *
 */
public class StaticCarFactory {
    
    private static Map<String, Car> cars = new HashMap<String, Car>();
    
    static {
        cars.put("audi", new Car("audi", 300000));
        cars.put("ford", new Car("ford", 300000));
    }
    /**
     * 静态工厂方法
     * @param name
     * @return
     */
    public static Car getCar(String name) {
        
        return cars.get(name);
    }
}
```
```jsp
<!-- 通过静态工厂方法来配置bean，注意不是配置静态工厂方法实例，而是配置bean实例 -->
<!-- 
    class属性：指向静态工厂方法的全类名
    factory-method：指向静态工厂方法的名字
    constructor-arg：如果静态工厂方法需要传入参数，则使用constructor-arg来配置参数
 -->
<bean id="car1" 
    class="com.yl.factory.StaticCarFactory" factory-method="getCar">
    <constructor-arg value="audi"></constructor-arg>
</bean>
```

#### 3.1.2.2. 实例工厂方法
```java
package com.yl.factory;

import java.util.HashMap;
import java.util.Map;
/***
 * 实例工厂方法：实例工厂的方法，即现需要创建工厂本身，在调用工厂的实例方法来返回bean的实例
 * @author yul
 *
 */
public class InstanceCarFactory {
    
    private Map<String, Car> cars = new HashMap<String, Car>();
    
    public InstanceCarFactory() {
        cars = new HashMap<String, Car>();
        cars.put("audi", new Car("audi", 300000));
        cars.put("ford", new Car("ford", 400000));
    }
    
    public Car getCar(String brand) {
        return cars.get(brand);
    }
}
```
```jsp
<!-- 配置工厂的实例 -->
<bean id="carFactory" class="com.yl.factory.InstanceCarFactory"></bean>

<!-- 
    factory-bean：指向实例工厂方法的bean
    factory-method：指向实例工厂方法的名字
    constructor-arg：如果实例工厂方法需要传入参数，则使用constructor-arg来配置参数
 -->
<!-- 通过实例工厂方法来配置bean -->
<bean id="car2" factory-bean="carFactory" factory-method="getCar">
    <constructor-arg value="ford"></constructor-arg>
</bean>
```
### 3.1.3. 通过FactoryBean配置bean（需要继承FactoryBean接口）
当配置的bean里需要引用其他的bean，通过FactoryBean配置是最好的方式。
过程：定义类实现FactoryBean接口，并实现三个方法：
-  getObject()：返回对象
-  getObjectType()：返回对象的类型
-  isSingleton()：对象是否为单例。

编写配置文件，class指向FactoryBean的全类名。
```java
package com.yl.factorybean;

import org.springframework.beans.factory.FactoryBean;
//自定义的Factorybean需要实现FactoryBean接口
public class CarFactoryBean implements FactoryBean<Car> {
    
    private String brand;
    
    public void setBrand(String brand) {
        this.brand = brand;
    }
    
    /**
     * 返回bean的对象
     */
    @Override
    public Car getObject() throws Exception {
        // TODO Auto-generated method stub
        return new Car("BMW", 600000);
    }
    /**
     * 返回bean的类型
     */
    @Override
    public Class<?> getObjectType() {
        // TODO Auto-generated method stub
        return Car.class;
    }
    
    @Override
    public boolean isSingleton() {
        // TODO Auto-generated method stub
        return true;
    }
    
}
```
```jsp
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!-- 
        通过Factorybean来配置bean的实例
        class：指向Factorybean的全类名
        property：配置Factorybean的属性
        
        但实际返回的实例却是Factorybean的getObject()方法返回的实例
     -->
    <bean id="car" class="com.yl.factorybean.CarFactoryBean">
        <property name="brand" value="BMW"></property>
    </bean>
</beans>
```

## 3.2. 注解方式
### 3.2.1. 组件扫描
spring能够从classpath下自动扫描，侦测和实例化具有特定注解的组件。特定的组件包括：

-  @Component：基本组件，标识了一个受spring管理的组件
-  @Repository：标识持久层组件
-  @Service：标识服务层（业务层）组件
-  @Controller：标识表现层组件
-  @Configuration：配置注解，作用和上述四个相同
-  @Bean：上述五个注解均为类注解，@Bean是方法层面的注解，需要在上述五个注解的类中才能生效

对于扫描的的组件，spring有默认的命名策略：使用非限定类名，第一个字母小写。也可以在注解中通过value属性标识组件的名称。

比如，某个类名为UserService，那么默认的组件名为userService。如果类名为UserServiceImp，则可以用value指明其组件名为userService。也可以用userServiceImp，但是不符合习惯。

当在组件类上使用了特定的注解之后，还需要在spring的配置文件中声明<context:component-scan>来标识需要扫描的包。<context:component-scan>可以拥有若干个<context:include-filter>（需要use-default-filters="false"配合使用）或<context:exclude-filter>子节点过滤器，前者表示要包含的目标类，后者表示要排除在外的目标类。

或者不使用`xml`，使用`java`配置类，在配置类上声明`@ComponentScan`。该注解默认扫描该类下组件，扫描范围可以设定，比如使用`@ComponentScan(basePackages="包名")`或者`@ComponentScan(basePackagesClasses={类名1.class, 类名2.class)`。

spring内建支持如下四种过滤器：

-  annotation：该过滤器要指定一个annotation名，如lee.AnnotationTest。
-  assignable：类名过滤器，该过滤器直接指定一个java类。
-  regex:正则表达式过滤器，该过滤器指定一个正则表达式，匹配该正则表达式的java类将满足该过滤规则，如org\.example\.default.*。
-  aspectj:如org.example..*service+。

当需要扫描多个包时，可以使用逗号隔开。
```jsp
<!--指定扫描的包-->
<context:component-scan
    base-package="com.atguigu.spring.beans,annotation"
    use-default-filters="false"><!--不再使用默认的filter，只使用指定的filter-->
    <!--resource-pattern="repository/*.class">仅扫描repository下的包-->
   
    <!--
    <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Service"/>
    -->
    <context:include-filter type="annotation" expression="org.springframework.stereotype.Service"/>
</context:component-scan>
```

### 3.2.2. 常用注解
常用注解详细解释：

#### 3.2.2.1. @Bean 
@Bean是一个方法级别上的注解，主要用在@Configuration注解的类里，也可以用在@Component注解的类里。使用@Bean注解和使用xml配置bean效果一样。

#### 3.2.2.2. @Autowired
@Autowired注解可用于修饰属性、setter 方法、构造方法。其作用是为了消除Java代码里面的getter/setter与bean属性中的property。@Autowired注解的意思就是，当Spring发现@Autowired注解时，将自动在代码上下文中找到和其匹配（默认是类型匹配）的Bean，并自动注入到相应的地方去。

@Autowired注解自动装配具有兼容类型的单个bean属性。默认情况下，所有使用@Autowired注解的属性都需要设置，当Spring找不到匹配的bean装配属性时，会抛出异常。若某一属性允许不被设置，可以设置@Autowired注解的required属性为false。 

默认情况下，当IOC容器里存在多个类型兼容的bean时（例如现在UserDao接口有两个实现类，UserDaoImpl只是其一），通过类型的自动装配将无法工作。有两种解决方式，一种是通过@Repository(“userDao”)设置UserDaoImpl实例的名称为userDao，然后在UserService中声明UserDao组件名为userDao即可。第二种方式是在装配属性上使用@Qualifier注解进行设置。
```java
public class Zoo {
    //getter和setter可以去掉
    @Autowired
    private Tiger tiger;
    
    @Autowired
    private Monkey monkey;
    
    public String toString(){
        return tiger + "\n" + monkey;
    }
}
```
```jsp
<beans
    <context:component-scan base-package="com.spring" />
    <!--属性注入可以去掉-->
    <bean id="zoo" class="com.spring.model.Zoo" />
    <bean id="tiger" class="com.spring.model.Tiger" />
    <bean id="monkey" class="com.spring.model.Monkey" />

</beans>
```

#### 3.2.2.3. @Required
@Required 注解只能用于修饰 bean 属性的 setter 方法。受影响的 bean 属性必须在配置时被填充在 xml 配置文件中，否则容器将抛出BeanInitializationException。

例如，下述程序会出错，因为age的setter方法没有在bean中注入，而age的setter方法标记了@Required，也就是必须要输入，抛出的异常为：BeanInitializationException。
```java
package com.jsoft.testspring.testannotationrequired;

import org.springframework.beans.factory.annotation.Required;

public class Student {
    private Integer age;
    private String name;
    
    @Required
    public void setAge(Integer age){
        this.age = age;
    }
    
    public Integer getAge(){
        return this.age;
    }
    
    public void setName(String name){
        this.name = name;
    }
    
    public String getName(){
        return this.name;
    }
}
```
```jsp
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
                        http://www.springframework.org/schema/beans/spring-beans.xsd
                        http://www.springframework.org/schema/context
                        http://www.springframework.org/schema/context/spring-context.xsd">

   <context:annotation-config/>

   <bean id="student" class="com.jsoft.testspring.testannotationrequired.Student">
        <property name="name" value="Jim"/>
        <!--这样补充之后，就不会报错了。
        <property name="age" value="27"/>
        -->
   </bean>

</beans>
```

#### 3.2.2.4. @Qualifier
在@Autowired注解中，提到了如果发现有多个候选的 bean 都符合修饰类型，Spring 就会抓瞎了。

那么，如何解决这个问题。

可以通过@Qualifier指定 bean 名称来锁定真正需要的那个 bean。
```java
public class AnnotationQualifier {
    private static final Logger log = LoggerFactory.getLogger(AnnotationQualifier.class);

    @Autowired
    @Qualifier("dog") /** 去除这行，会报异常 */
    Animal dog;

    Animal cat;

    public Animal getDog() {
        return dog;
    }

    public void setDog(Animal dog) {
        this.dog = dog;
    }

    public Animal getCat() {
        return cat;
    }

    @Autowired
    public void setCat(@Qualifier("cat") Animal cat) {
        this.cat = cat;
    }

    public static void main(String[] args) throws Exception {
        AbstractApplicationContext ctx =
                new ClassPathXmlApplicationContext("spring/spring-annotation.xml");

        AnnotationQualifier annotationQualifier =
                (AnnotationQualifier) ctx.getBean("annotationQualifier");

        log.debug("Dog name: {}", annotationQualifier.getDog().getName());
        log.debug("Cat name: {}", annotationQualifier.getCat().getName());
        ctx.close();
    }
}

abstract class Animal {
    public String getName() {
        return null;
    }
}

class Dog extends Animal {
    public String getName() {
        return "狗";
    }
}

class Cat extends Animal {
    public String getName() {
        return "猫";
    }
}
```
```jsp
<!-- 测试@Qualifier -->
<bean id="dog" class="org.zp.notes.spring.beans.annotation.sample.Dog"/>
<bean id="cat" class="org.zp.notes.spring.beans.annotation.sample.Cat"/>
<bean id="annotationQualifier" class="org.zp.notes.spring.beans.annotation.sample.AnnotationQualifier"/>
```

### 3.2.3. JSR(Java Specification Requests，Java规范提案)注解
#### 3.2.3.1. @Resource
@Resource注解与@Autowired注解作用非常相似
```java
public class AnnotationResource {
    private static final Logger log = LoggerFactory.getLogger(AnnotationResource.class);

    @Resource(name = "flower")
    Plant flower;

    @Resource(name = "tree")
    Plant tree;

    public Plant getFlower() {
        return flower;
    }

    public void setFlower(Plant flower) {
        this.flower = flower;
    }

    public Plant getTree() {
        return tree;
    }

    public void setTree(Plant tree) {
        this.tree = tree;
    }

    public static void main(String[] args) throws Exception {
        AbstractApplicationContext ctx =
                        new ClassPathXmlApplicationContext("spring/spring-annotation.xml");

        AnnotationResource annotationResource =
                        (AnnotationResource) ctx.getBean("annotationResource");
        log.debug("type: {}, name: {}", annotationResource.getFlower().getClass(), annotationResource.getFlower().getName());
        log.debug("type: {}, name: {}", annotationResource.getTree().getClass(), annotationResource.getTree().getName());
        ctx.close();
    }
}
```
```jsp
<!-- 测试@Resource -->
<bean id="flower" class="org.zp.notes.spring.beans.annotation.sample.Flower"/>
<bean id="tree" class="org.zp.notes.spring.beans.annotation.sample.Tree"/>
<bean id="annotationResource" class="org.zp.notes.spring.beans.annotation.sample.AnnotationResource"/>
```
@Resource的装配顺序：
-  @Resource后面没有任何内容，默认通过name属性去匹配bean，找不到再按type去匹配
-  指定了name或者type则根据指定的类型去匹配bean
-  指定了name和type则根据指定的name和type去匹配bean，任何一个不匹配都将报错

@Autowired和@Resource两个注解的区别：
-  @Autowired默认按照byType方式进行bean匹配，@Resource默认按照byName方式进行bean匹配
-  @Autowired是Spring的注解，@Resource是J2EE的注解

Spring属于第三方的，J2EE是Java自己的东西，因此，建议使用@Resource注解，以减少代码和Spring之间的耦合。

### 几个注解之间的区别
#### @Component和@Configuration

- 用法不太相同。`@Component`可以用在所有类，`@Configuration`一般用在有`@Bean`等注解的类上，作为配置类
- `@Component`产生的`bean`为多例，`@Configuration`中所有使用`@Bean`的方法都会使用动态代理，产生的是单例。测试代码如下。

```java
public interface Dog {
}

public class LittleDog implements Dog {
}

public class Master {
    private Dog dog;

    public Master(Dog dog) {
        this.dog = dog;
    }

    public Dog getDog() {
        return this.dog;
    }
}

@Configuration
public class MyConfig0 {
    @Bean
    public Dog dog() {
        return new LittleDog();
    }

    @Bean
    public Master master() {
        return new Master(dog());
    }
}

@Component
public class MyConfig1 {
    @Bean
    public Dog dog() {
        return new LittleDog();
    }

    @Bean
    public Master master() {
        return new Master(dog());
    }
}

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = MyConfig0.class) // 修改这里的class来测试两个注解是否为单例模式，当为@Configuration时，dog和master0.getDog()是一个对象，当为@Component时不是一个对象
public class ConfigTest {
    @Autowired
    private Master master0;

    @Autowired
    private Master master1;

    @Autowired
    private Dog dog;

    @Test
    public void test() {
        System.out.println(master0.getDog() == master1.getDog());
        System.out.println(dog == master0.getDog());
    }
}
```
要想让`MyConfig1`中的`dog`也为单例，则可以做如下更改：

```java
@Component
public class MyConfig1 {
    @Autowired
    private Dog dog;

    @Bean
    public Dog dog() {
        return new LittleDog();
    }

    @Bean
    public Master master() {
        return new Master(dog);
    }
}
```

#### 什么时候使用@Bean
当需要使用第三方的库中的组件时，第三方组件可能以`jar`包的形式存在，这时没办法在它的类上添加`@Component`和`@Autowired`注解，这时可以考虑使用`@Bean`手动进行装配，如下。

```java
@Configuration
public class MyClass {
  // classA和classB就是jar包里写好的
  @Bean
  public ClassA getClassA() {
    return new ClassA();
  }
  @Bean
  public ClassB getClassB() {
    return new ClassB();
  }
  .....
}
```

[参考](https://github.com/dunwu/spring-notes/blob/master/docs/spring/core/ioc.md#xml-%E9%85%8D%E7%BD%AE)

## 3.3. 注解方式和xml文件方式的关系
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/component1.jpg" width="500">

<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/component2.jpg" width="500">

<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/component3.jpg" width="500">
</center>

# 4. DI(依赖注入方式)
依赖注入也可以叫做属性注入，主要有三种方式：setter方法注入、构造器注入和工厂方法注入(不常用)。

## 4.1. setter注入
使用这种方法时，bean要有一个setter方法，在下例中，类中必须有一个setName()方法。
```jsp
<bean id = "helloWorld" class="com.cn.HelloWorld">
    <property name="name" value="World"></property>
</bean>
```

## 4.2. 构造器注入
```jsp
<bean id = "car" class = "com.cn.Car">
    <constructor-arg vaule="audi" index="0" />
    <constructor-arg vaule="shangahi" index="1" />
    <constructor-arg vaule="3000" type="double" />
    
    <constructor-arg vaule="audi" type="java.lang.String" />
    <constructor-arg vaule="shangahi" type="java.long.String" />
    <constructor-arg vaule="3000" type="int" />
</bean>
```
其中，index表示参数的顺序，type表示参数的类型，可以混用。type可以用来解决构造器注入歧义的问题。

## 4.3. 工厂方法注入（比较少用，先放置）

## 4.4. setter注入的三种写法
setter注入又有是那种不同的写法：
### 4.4.1. 正常方式
```jsp
<bean id="FileNameGenerator" class="com.yiibai.common.FileNameGenerator">
	<property name="name">
		<value>yiibai</value>
	</property>
	<property name="type">
		<value>txt</value>
	</property>
</bean>
```
注：使用构造器注入的时候也可以按照这种方法写。比如上述构造器注入的最后一个属性可以这样写：
```jsp
<constructor-arg type="int" >
	<value>3000</value>
<constructor-arg />
```
注：若value中的值有特殊字符，比如<>，可以用<![CDATA[] ]>进行转义，例如;
```jsp
<constructor-arg type="java.long.String" />
	<value><![CDATA[<Shanghai>]]></value>
<constructor-arg />
```

### 4.4.2. 快捷方式
```jsp
<bean id="FileNameGenerator" class="com.yiibai.common.FileNameGenerator">
	<property name="name" value="yiibai" />
	<property name="type" value="txt" />
</bean>
```

### 4.4.3. p模式
```jsp
<bean id="FileNameGenerator" class="com.yiibai.common.FileNameGenerator" 
             p:name="yiibai" p:type="txt" />
```

# 5. 在一个bean里引用另一个bean
## 5.1. ref=”beanName”
### 5.1.1. setter
```jsp
<bean id=”person” class=”com.cn.Person”>
	<property name=”car” ref=”Car”>
</bean>
```
### 5.1.2. 构造器
```jsp
<bean id = “person” class = “com.cn.Personr”>
    <constructor-arg index=”0” ref=”Car” />
</bean>
```
## 5.2. ref bean=”beanName”
### 5.2.1. setter
```jsp
<bean id=”person” class=”com.cn.Person”>
	<property name=”car” >
    	<ref bean=”Car”/>
    </property>
</bean>
```

### 5.2.2. 构造器
```jsp
<bean id = “person” class = “com.cn.Personr”>
    <constructor-arg index=”0” />
    	<ref bean=”Car”/>
    <constructor-arg/>
</bean>
```

## 5.3. 内部bean
```jsp
<bean id=”person” class=”com.cn.Person”>
	<property name=”name” value=”Tom”></property>
	<property name=”age” value=”30”></property>
	<property name=”car”>
    	<bean class=”com.cn.Car”>
    		<constructor-arg value=”Ford”></constructor-arg>
    		<constructor-arg value=”Changan”></constructor-arg>
    		<constructor-arg value=”20000” type=”double”></constructor-arg>
    	</bean>
    </property>
</bean>
```
注：内部bean不能被外部引用；内部bean不用写id。

# 6. 级联属性赋值
属性首先需要初始化才可以为级联属性赋值。
```jsp
<!--初始化属性-->
<property name=”car”>
	<bean……
</property>
<!--为car中的属性赋值-->
<property name=”car.maxSpeed” value=”260”></properry>
```

# 7. 集合属性
## 7.1. List示例 
```jsp
<property name="lists">
	<list>
		<value>1</value>
		<ref bean="PersonBean" />
		<bean class="com.yiibai.common.Person">
			<property name="name" value="yiibaiList" />
			<property name="address" value="Hainan" />
			<property name="age" value="28" />
		</bean>
	</list>
</property>
```
## 7.2. Set示例 
```jsp
<property name="sets">
	<set>
		<value>1</value>
		<ref bean="PersonBean" />
		<bean class="com.yiibai.common.Person">
			<property name="name" value="yiibaiSet" />
			<property name="address" value="Hainan" />
			<property name="age" value="28" />
		</bean>
	</set>
</property>
```
## 7.3. Map示例 
```jsp
<property name="maps">
	<map>
		<entry key="Key 1" value="1" />
		<entry key="Key 2" value-ref="PersonBean" />
		<entry key="Key 3">
			<bean class="com.yiibai.common.Person">
				<property name="name" value="yiibaiMap" />
				<property name="address" value="Hainan" />
				<property name="age" value="28" />
			</bean>
		</entry>
	</map>
</property>
```

## 7.4. Properties示例 
```jsp
<property name="pros">
	<props>
		<prop key="admin">admin@yiibai.com</prop>
		<prop key="support">support@yiibai.com</prop>
	</props>
</property>
```

## 7.5. 单例的集合bean
配置单例的集合bean，以供多个bean进行调用。
```jsp
<util:list id="cars">
    <ref bean="car"/>
    <ref bean="car2"/>
</util:list>
<!--调用上面配置的bean-->
<bean id=”person” class=”com.cn.Person”>
	<property name=”name” value=”Jack”></property>
	<property name=”age” value=”29”></property>
	<property name=”cars” ref=”cars”></property>
</bean>
```


# 8. 参考
-  https://blog.csdn.net/fuzhongmin05/article/details/61614873
-  https://blog.csdn.net/fuzhongmin05/article/details/61614873
-  https://blog.csdn.net/m13666368773/article/details/7802126
-  https://blog.csdn.net/qq_22654611/article/details/52606960
-  https://blog.csdn.net/it_man/article/details/4402245

# 9. spring、pring boot和spring mvc的区别
spring boot就是一个大框架里面包含了许许多多的东西，其中spring就是最核心的内容之一，当然就包含spring mvc。

spring mvc 是只是spring 处理web层请求的一个模块。


因此他们的关系大概就是这样：

spring mvc < spring <springboot。
 

spring boot 我理解就是把 spring spring mvc spring data jpa 等等的一些常用的常用的基础框架组合起来，提供默认的配置，然后提供可插拔的设计，就是各种 starter ，来方便开发者使用这一系列的技术，套用官方的一句话， spring 家族发展到今天，已经很庞大了，作为一个开发者，如果想要使用 spring 家族一系列的技术，需要一个一个的搞配置，然后还有个版本兼容性问题，其实挺麻烦的，偶尔也会有小坑出现，其实蛮影响开发进度， spring boot 就是来解决这个问题，提供了一个解决方案吧，可以先不关心如何配置，可以快速的启动开发，进行业务逻辑编写，各种需要的技术，加入 starter 就配置好了，直接使用，可以说追求开箱即用的效果吧.