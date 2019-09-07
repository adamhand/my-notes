# Spring事务

## Spring事务管理的两种方式
spring支持**编程式事务管理**和**声明式事务管理**两种方式。

- 编程式事务使用TransactionTemplate或者直接使用底层的PlatformTransactionManager。对于编程式事务管理，spring推荐使用TransactionTemplate。
- 声明式事务是建立在AOP之上的。其本质是对方法前后进行拦截，然后在目标方法开始之前创建或者加入一个事务，在执行完目标方法之后根据执行情况提交或者回滚事务。声明式事务最大的优点就是不需要通过编程的方式管理事务，这样就不需要在业务逻辑代码中掺杂事务管理的代码，只需在配置文件中做相关的事务规则声明(或通过基于@Transactional注解的方式)，便可以将事务规则应用到业务逻辑中。

显然声明式事务管理要优于编程式事务管理，这正是spring倡导的非侵入式的开发方式。声明式事务管理使业务代码不受污染。和编程式事务相比，声明式事务唯一不足地方是，它的最细粒度只能作用到方法级别，无法做到像编程式事务那样可以作用到代码块级别。但是即便有这样的需求，也存在很多变通的方法，比如，可以将需要进行事务管理的代码块独立为方法等等。

声明式事务管理也有两种常用的方式：

- 基于`<tx>`和`<aop>`名字空间的xml配置文件。目前推荐的方式，其最大特点是与 Spring AOP 结合紧密，可以充分利用切点表达式的强大支持，使得管理事务更加灵活。
- 基于@Transactional注解。 将声明式事务管理简化到了极致。开发人员只需在配置文件中加上一行启用相关后处理 Bean 的配置，然后在需要实施事务管理的方法或者类上使用 @Transactional 指定事务规则即可实现事务管理，而且功能也不比其他方式逊色。

## spring事务特性
spring所有的事务管理策略类都继承自org.springframework.transaction.PlatformTransactionManager接口。

### 隔离级别
TransactionDefinition接口定义隔离级别：

|隔离级别|解释|
|-|-|
|ISOLATION_DEFAULT|使用数据库默认的事务隔离级别|
|ISOLATION_READ_UNCOMMITTED|对应数据库的read uncommited级别。它允许别外一个事务可以看到这个事务未提交的数据，会产生脏读，不可重复读和幻读。|
|ISOLATION_READ_COMMITTED|对应数据库的read commited级别。另外一个事务不能读取该事务未提交的数据。这种事务隔离级别可以避免脏读出现，但是可能会出现不可重复读和幻读。|
|ISOLATION_REPEATABLE_READ|对应数据库的repeatable read级别。这种事务隔离级别可以防止脏读，不可重复读。但是可能出现幻读。|
|ISOLATION_SERIALIZABLE|对应数据库的serializable级别。事务被处理为顺序执行。除了防止脏读，不可重复读外，还避免了幻读。|

### 事务传播行为
事务传播行为用来描述由某一个事务传播行为修饰的方法被嵌套进另一个方法的时事务如何传播。Spring中有七种传播行为。

|传播行为|说明|
|-|-|
|PROPAGATION_REQUIRED(required)|如果当前没有事务，就新建一个事务，如果已经存在一个事务中，加入到这个事务中。这是最常见的选择。|
|PROPAGATION_SUPPORTS(supports)|支持当前事务，如果当前没有事务，就以非事务方式执行。|
|PROPAGATION_MANDATORY(mandatory)|使用当前的事务，如果当前没有事务，就抛出异常。|
|PROPAGATION_REQUIRES_NEW(requires_new)|新建事务，如果当前存在事务，把当前事务挂起。|
|PROPAGATION_NOT_SUPPORTED(not_supported)|以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。|
|PROPAGATION_NEVER(never)|以非事务方式执行，如果当前存在事务，则抛出异常。|
|PROPAGATION_NESTED(nested)|如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则执行与PROPAGATION_REQUIRED类似的操作。|

## 参考
[Spring事务详细解释，满满的都是干货！](https://blog.csdn.net/u010963948/article/details/82761383)</br>
[Spring 事务机制详解](https://www.jianshu.com/p/8ddc01f23540)</br>
[Spring的编程式事务和声明式事务](https://www.cnblogs.com/nnngu/p/8627662.html)</br>
[Spring编程式和声明式事务实例讲解](https://blog.csdn.net/qq_34337272/article/details/80419316)</br>
[Spring 事务实现分析](https://www.jianshu.com/p/2449cd914e3c)</br>
[一文带你看懂Spring事务！](http://blog.itpub.net/69900354/viewspace-2565243/)</br>