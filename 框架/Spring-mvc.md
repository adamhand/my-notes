<!-- TOC -->

- [web.xml加载过程](#webxml加载过程)
- [servlet生命周期](#servlet生命周期)
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