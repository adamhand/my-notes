# spring boot
---

# spring boot启动
`spring boot`中总是会有一个启动类：
```
@SpringBootApplication
public class GooseyApplication {

    public static void main(String[] args) {
        SpringApplication.run(GooseyApplication.class, args);
    }
}
```
其中`@SpringBootApplication`注解和`SpringApplication.run()`与`spring boot`的启动关系很大。

## @SpringBootApplication注解
`ctrl`+左键点开该注解，可以看到该注解又被包括几个注解：
```
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {
```
`spring boot`官方文档说`@SpringBootApplication`等同于`@Configuration`+ `@EnableAutoConfiguration`+`@ComponentScan`。所以，上面几个比较重要的注解有三个：

- `@SpringBootConfiguration`（`@SpringBootConfiguration`用`@Configuration`）
- `@EnableAutoConfiguration`
- `@ComponentScan`

### @Configuration
`@Configuration`用于定义配置类，可替换`xml`配置文件，被注解的类内部包含有一个或多个被`@Bean`注解的方法，可以作为`bean`被注册到`IoC`容器中。`spring boot`推荐使用注解的方式。

XML跟config配置方式的区别：

- 表达形式层面
基于XML配置的方式是这样：

<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/springboot1.jpg">
</center>

而基于JavaConfig的配置方式是这样：

<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/springboot2.jpg">
</center>

- 注册bean定义层面
基于XML的配置形式是这样：

<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/springboot3.jpg">
</center>

而基于JavaConfig的配置形式是这样的：

<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/springboot4.jpg">
</center>

任何一个标注了@Bean的方法，其返回值将作为一个bean定义注册到Spring的IoC容器，方法名将默认成该bean定义的id。

- 表达依赖注入关系层面
为了表达bean与bean之间的依赖关系，在XML形式中一般是这样：

<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/springboot5.jpg">
</center>

而基于JavaConfig的配置形式是这样的：
<center>
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/springboot6.jpg">
</center>

如果一个bean的定义依赖其他bean,则直接调用对应的JavaConfig类中依赖bean的创建方法就可以了。

## @ComponentScan
`@ComponentScan`的功能其实就是自动扫描并加载符合条件的组件（比如`@Component`和`@Repository`等）或者`bean`定义，最终将这些`bean`定义加载到`IoC`容器中。

可以通过`basePackages`等属性来细粒度的定制`@ComponentScan`自动扫描的范围，如果不指定，则默认`Spring`框架实现会从声明`@ComponentScan`所在类的`package`进行扫描。**所以`SpringBoot`的启动类最好是放在`root package`下，因为默认不指定`basePackages`。**

