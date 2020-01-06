<!-- TOC -->

- [web.xml加载过程](#webxml加载过程)
- [servlet生命周期](#servlet生命周期)
- [servlet完成执行流程](#servlet完成执行流程)
    - [浏览器发起请求](#浏览器发起请求)
    - [创建servlet对象](#创建servlet对象)
    - [调用init()方法](#调用init方法)
    - [调用service方法](#调用service方法)
    - [向浏览器响应](#向浏览器响应)
- [DispatcherServlet配置方式](#dispatcherservlet配置方式)
    - [使用xml配置](#使用xml配置)
    - [使用Java Config配置](#使用java-config配置)

<!-- /TOC -->
## web.xml加载过程
`web.xml`的加载过程为：`context-param -> listener -> filter -> servlet`。具体过程为：

- 启动`web`项目时，容器会在`web.xml`文件中读取两个节点：`<context-param></context-param>`和`<listener></listener>`。
- 容器创建一个`ServletContext(application)`，整个`web`项目会共享这个上下文。
- 容器以`<context-param></context-param>`的`name`作为键，以`value`作为值，以键值对的方式存放到`ServletContext`中。
- 容器创建`<listener></listener>`中的类实例，根据配置的class类路径`<listener-class>`来创建监听。
- 容器会读取`<filter></filter>`，根据指定的类路径来实例化过滤器。
- 初始化`servlet`。

## servlet生命周期
加载完`web.xml`后，需要初始化`servlet`。`servlet`的生命周期包括三部分：初始化、调用和销毁。

- 初始化阶段。调用`init()`方法完成对`servlet`的初始化。
- 响应客户请求阶段，调用`service()`方法。由`service()`方法根据提交方式选择执行`doGet()`或者`doPost()`方法。
- 终止阶段，调用`destory()`方法。

过程如下图所示：

<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/servlet%20life.jpg">
</div>

## servlet完成执行流程
了解了`servlet`的生命周期之后，再了解一下从浏览器请求到返回的过程中`servlet`的完整执行流程。

### 浏览器发起请求
浏览器向服务器发起请求，服务器首先会到`web.xml`里寻找路径名。具体步骤为。

- 浏览器输入访问路径后，携带了请求行，请求头和请求体
- 根据访问路径找到已注册的`servlet`名称，既下图中的`demo`
- 根据映射找到对应的`servlet`名 
- 根据根据`servlet`名找到我们定类名，这里便是`DemoServlet`类

<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/servlet1.jpg">
</div>

### 创建servlet对象
服务器找到全限定类名后，通过反射创建对象，同时也创建了servletConfig，里面存放了一些初始化信息（服务器只会创建一次servlet对象，所以servletConfig也只有一个）。

<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/servlet2.jpg">
</div>

### 调用init()方法
对象创建好之后，首先要执行`init`方法。这时由于自定义的`sevlet`程序里没有定义`init()`方法，需要沿着继承链寻找父类中的`init()`方法：`DemoServlet -> HttpServlet -> GenericServlet`，最终调用`GenericServlet`中的`init()`方法。

<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/servlet3.jpg">
</div>

### 调用service方法
接着，服务器会创建两个对象：`ServletRequest`请求对象和`ServletResponse`响应对象，用来封装浏览器的请求数据和封装向浏览器的响应数据。

接着服务器会默认在`DemoServlet`寻找`service(ServletRequest req, ServletResponse res)`方法，但是该类中没有`service`，于是会沿着调用链在父类中寻找，在父类`HttpServlet`中发现有此方法，则直接调用此方法，并将之前创建好的两个对象传入，将传入的两个参数强转，并调用`HttpServlet`下的另外一个`service`方法。

接着执行`service(HttpServletRequest req, HttpServletResponseresp)`方法，在此方法内部进行了判断请求方式，并执行`doGet`和`doPost`，但是这两个方法已经在`DemoServlet`中被重写了，所以会执行重写的方法。

<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/servlet4.jpg">
</div>

### 向浏览器响应
将`doGet()`或`doPost()`方法执行的结果封装成响应消息，返回给浏览器。

<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/servlet5.jpg">
</div>

## DispatcherServlet配置方式
### 使用xml配置
需要在`web.xml`文件中进行如下配置：

```xml
  <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:applicationContext.xml</param-value>
  </context-param>

  <filter>
    <filter-name>encodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <async-supported>true</async-supported>
    <init-param>
      <param-name>encoding</param-name>
      <param-value>UTF-8</param-value>
    </init-param>
  </filter>
  <filter-mapping>
    <filter-name>encodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>

  <listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>

  <servlet>
    <servlet-name>dispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:applicationContext.xml</param-value>
    </init-param>
  </servlet>

  <servlet-mapping>
    <servlet-name>dispatcherServlet</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>
```

### 使用Java Config配置
不再使用`xml`的配置方式，需要使用`Java`语言来进行配置。几个文件的目录如下。

```
mvc
 --RootConfig
 --WebConfig
 --SpittrWebAppInitializer
 --HelloWorldController
```

```java
@Configuration
@ComponentScan(basePackages = "mvc", excludeFilters = {
        @ComponentScan.Filter(type = FilterType.ANNOTATION, value = EnableWebMvc.class)
})
public class RootConfig {
}

@Configuration
@EnableWebMvc
@ComponentScan("mvc")
public class WebConfig extends WebMvcConfigurerAdapter {
    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }

    @Bean
    public ViewResolver viewResolver() {
        InternalResourceViewResolver resolver = new InternalResourceViewResolver();
        resolver.setPrefix("/WEB-INF/views/");
        resolver.setSuffix(".jsp");
        resolver.setExposeContextBeansAsAttributes(true);
        return resolver;
    }
}

public class SpittrWebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {
    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class<?>[] { RootConfig.class };
    }

    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class<?>[] { WebConfig.class };
    }

    @Override
    protected String[] getServletMappings() {
        return new String[] { "/" };
    }
}
```
编写`Controller`类。

```java
@Controller
public class HelloWorldController {
    @RequestMapping("/")
    public String helloWorld() {
        return "hello world";
    }
}
```