<!-- TOC -->

- [web.xml加载过程](#webxml加载过程)
- [servlet生命周期](#servlet生命周期)
- [servlet完成执行流程](#servlet完成执行流程)
    - [浏览器发起请求](#浏览器发起请求)
    - [创建servlet对象](#创建servlet对象)
    - [调用init()方法](#调用init方法)
    - [调用service方法](#调用service方法)
    - [向浏览器响应](#向浏览器响应)
- [DispatcherServlet的初始化过程](#dispatcherservlet的初始化过程)
    - [spring中上下文的初始化](#spring中上下文的初始化)
    - [DispatcherServlet的初始化](#dispatcherservlet的初始化)
        - [创建`DispatcherServlet`上下文](#创建dispatcherservlet上下文)
        - [刷新`DispatcherServlet`自己的应用上下文](#刷新dispatcherservlet自己的应用上下文)
    - [参考](#参考)
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

## DispatcherServlet的初始化过程
`DispatchServlet`是`spring`中对`servlet`的一个实现，在了解`DispatchServlet`的初始化之前，首先看一下`spring`中`web.xml`的配置。

### spring中上下文的初始化
与上面提到的`web.xml`的加载过程类似，`spring`中`web.xml`的加载过程也是`context-param -> listener -> filter -> servlet`。`context-param`的配置如下：

```xml
<context-param>
  <param-name>contextConfigLocation</param-name>
  <param-value>classpath:applicationContext.xml</param-value>
</context-param>
```
这里的`contextConfigLocation`是整个上下文级别的，是整个`spring`上下文的配置，是在`listner`中加载的，配置如下：

```xml
<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```
`ContextLoaderListener`类型是`springframework`中的原始加载上下文的监听器，`ContextLoaderListener`类如下，继承了`ContextLoader`类，在项目启动时会执行`contextInitialized`方法。

```java
public class ContextLoaderListener extends ContextLoader implements ServletContextListener {
    public void contextInitialized(ServletContextEvent event) {
        this.initWebApplicationContext(event.getServletContext());
    }
}
```
`initWebApplicationContext`方法主要代码如下：

```java
  public WebApplicationContext initWebApplicationContext(ServletContext servletContext) {
      ...
      if (this.context == null) {
          this.context = this.createWebApplicationContext(servletContext);
      }

      if (this.context instanceof ConfigurableWebApplicationContext) {
              ...
              this.configureAndRefreshWebApplicationContext(cwac, servletContext);
          }
      }

      servletContext.setAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, this.context);
      ...
  }
```
上述方法主要作用有三个：

- 创建`WebApplicationContext`
- 加载对应的`spring`配置文件中的`Bean`
- 将`WebApplicationContext`放入`ServletContext`(`Java Web`的全局变量)中

`createWebApplicationContext(servletContext)`方法即是完成创建`WebApplicationContext`工作，这个方法创建了上下文对象，并且支持用户自定义上下文对象，但必须继承`ConfigurableWebApplicationContext`，而`Spring MVC`默认使用`ConfigurableWebApplicationContext`作为`ApplicationContext`(它仅仅是一个接口)的实现。

```java
protected WebApplicationContext createWebApplicationContext(ServletContext sc) {
    Class<?> contextClass = this.determineContextClass(sc);
    if (!ConfigurableWebApplicationContext.class.isAssignableFrom(contextClass)) {
        throw new ApplicationContextException("Custom context class [" + contextClass.getName() + "] is not of type [" + ConfigurableWebApplicationContext.class.getName() + "]");
    } else {
        return (ConfigurableWebApplicationContext)BeanUtils.instantiateClass(contextClass);
    }
}
```
`determineContextClass()`方法如下，该方法找到根上下文的`Class`类型。

```java
protected Class<?> determineContextClass(ServletContext servletContext) {
    String contextClassName = servletContext.getInitParameter("contextClass");
    if (contextClassName != null) {
        try {
            return ClassUtils.forName(contextClassName, ClassUtils.getDefaultClassLoader());
        } catch (ClassNotFoundException var4) {
            throw new ApplicationContextException("Failed to load custom context class [" + contextClassName + "]", var4);
        }
    } else {
        contextClassName = defaultStrategies.getProperty(WebApplicationContext.class.getName());

        try {
            return ClassUtils.forName(contextClassName, ContextLoader.class.getClassLoader());
        } catch (ClassNotFoundException var5) {
            throw new ApplicationContextException("Failed to load default context class [" + contextClassName + "]", var5);
        }
    }
}
```
`Web.xml`中配置了`contextClass`就取其值，但必须是实现`ConfigurableWebApplicationContext`，没有的就从`defaultStrategies`中取默认值。`defaultStrategies`的配置代码如下：

```java
static {
    try {
        ClassPathResource resource = new ClassPathResource("ContextLoader.properties", ContextLoader.class);
        defaultStrategies = PropertiesLoaderUtils.loadProperties(resource);
    } catch (IOException var1) {
        throw new IllegalStateException("Could not load 'ContextLoader.properties': " + var1.getMessage());
    }

    currentContextPerThread = new ConcurrentHashMap(1);
}
```
默认从`ContextLoader.properties`文件中读取配置，改文件的位置在`\org\springframework\web\context\ContextLoader.properties`，内容为：

```java
org.springframework.web.context.WebApplicationContext=org.springframework.web.context.support.XmlWebApplicationContext
```
可以看到，默认是**XmlWebApplicationContext**。

`configureAndRefreshWebApplicationContext()`方法的作用是加载配置文件中的`bean`，这个方法于封装`ApplicationContext`数据并且初始化所有相关`Bean`对象。它会从`web.xml`中读取名为 `contextConfigLocation`的配置，这就是`spring xml`数据源设置，然后放到`ApplicationContext`中，最后调用`refresh`方法执行所有`Java`对象的创建。如下所示。

```java
    protected void configureAndRefreshWebApplicationContext(ConfigurableWebApplicationContext wac, ServletContext sc) {
        String configLocationParam;
        if (ObjectUtils.identityToString(wac).equals(wac.getId())) {
            configLocationParam = sc.getInitParameter("contextId");
            if (configLocationParam != null) {
                wac.setId(configLocationParam);
            } else {
                wac.setId(ConfigurableWebApplicationContext.APPLICATION_CONTEXT_ID_PREFIX + ObjectUtils.getDisplayString(sc.getContextPath()));
            }
        }

        wac.setServletContext(sc);
        configLocationParam = sc.getInitParameter("contextConfigLocation");
        if (configLocationParam != null) {
            wac.setConfigLocation(configLocationParam);
        }

        ConfigurableEnvironment env = wac.getEnvironment();
        if (env instanceof ConfigurableWebEnvironment) {
            ((ConfigurableWebEnvironment)env).initPropertySources(sc, (ServletConfig)null);
        }

        this.customizeContext(sc, wac);
        wac.refresh();
    }
```

最后完成`ApplicationContext`创建之后将其放入`ServletContext`中。到这里，`spring`的上下文就建立了。

### DispatcherServlet的初始化
`DispatcherServlet`的配置如下：

```xml
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
`DispatcherServlet`的配置拦截`url`为`/`，即所有的`web`请求都会经过`DispatcherServlet`，之后由`DispatcherServlet`进行分发。

需要注意的是，需要为`DispatcherServlet`的配置提供一个配置文件，文件命名一般为`[servlet-name]-servlet.xml`，如果没有给`servlet`指定配置文件的位置，并且在默认位置(`/WEB-INF/`)下也没有配置文件，那么系统启动的时候就会报错。

之后，需要使用`DispatcherServlet`类进行初始化，该类的继承关系图如下：

<div align="center">
<img src="https://raw.githubusercontent.com/adamhand/LeetCode-images/master/DispatcherServlet.png">
</div>

初始化`DispatcherServlet`时需要调用`init()`方法，`HttpServletBean`重写了`init`方法，调用时即调用的该方法，从`HttpServletBean`开始即进入了`spring`的范围。`init()`方法如下：

```java
public final void init() throws ServletException {
    if (this.logger.isDebugEnabled()) {
        this.logger.debug("Initializing servlet '" + this.getServletName() + "'");
    }

    try {
        PropertyValues pvs = new HttpServletBean.ServletConfigPropertyValues(this.getServletConfig(), this.requiredProperties);
        BeanWrapper bw = PropertyAccessorFactory.forBeanPropertyAccess(this);
        ResourceLoader resourceLoader = new ServletContextResourceLoader(this.getServletContext());
        bw.registerCustomEditor(Resource.class, new ResourceEditor(resourceLoader, this.getEnvironment()));
        this.initBeanWrapper(bw);
        bw.setPropertyValues(pvs, true);
    } catch (BeansException var4) {
        this.logger.error("Failed to set bean properties on servlet '" + this.getServletName() + "'", var4);
        throw var4;
    }

    this.initServletBean();
    if (this.logger.isDebugEnabled()) {
        this.logger.debug("Servlet '" + this.getServletName() + "' configured successfully");
    }

}
```

主要工作有三个：

- `ServletConfigPropertyValues`用于解析`web.xml`定义中`<servlet>`元素的子元素`<init-param>`中的参数值
- `BeanWrapper`把`DispatcherServlet`当做一个`Bean`去处理，这也是`HttpServletBean`类名的含义。`bw.setPropertyValues(pvs, true)` 将上一步解析的`servlet`初始化参数值绑定到`DispatcherServlet`对应的字段上；
- 调用`initServletBean()`方法

`initServletBean()`方法是在父类`FrameworkServlet`中定义，它调用`initWebApplicationContext`方法初始化`DispatcherServlet`自己的应用上下文。

```java
protected final void initServletBean() throws ServletException {
    ...
    try {
        this.webApplicationContext = this.initWebApplicationContext();
        this.initFrameworkServlet();
    } catch (ServletException var5) {
      ...
    }
    ...
}
```

`initWebApplicationContext`代码如下：

```java
protected WebApplicationContext initWebApplicationContext() {
    WebApplicationContext rootContext = WebApplicationContextUtils.getWebApplicationContext(this.getServletContext());
    WebApplicationContext wac = null;
    if (this.webApplicationContext != null) {
        wac = this.webApplicationContext;
        if (wac instanceof ConfigurableWebApplicationContext) {
            ConfigurableWebApplicationContext cwac = (ConfigurableWebApplicationContext)wac;
            if (!cwac.isActive()) {
                if (cwac.getParent() == null) {
                    cwac.setParent(rootContext);
                }

                this.configureAndRefreshWebApplicationContext(cwac);
            }
        }
    }

    if (wac == null) {
        wac = this.findWebApplicationContext();
    }

    if (wac == null) {
        wac = this.createWebApplicationContext(rootContext);
    }

    if (!this.refreshEventReceived) {
        this.onRefresh(wac);
    }

    if (this.publishContext) {
        String attrName = this.getServletContextAttributeName();
        this.getServletContext().setAttribute(attrName, wac);
        if (this.logger.isDebugEnabled()) {
            this.logger.debug("Published WebApplicationContext of servlet '" + this.getServletName() + "' as ServletContext attribute with name [" + attrName + "]");
        }
    }

    return wac;
}
```

主要功能有三个：

- 利用`WebApplicationContextUtils`类的`getWebApplicationContext`方法获得`ContextLoaderListener`创建的根应用上下文
- 为`DispatcherServlet`创建自己的应用上下文
- 刷新`DispatcherServlet`自己的应用上下文

#### 创建`DispatcherServlet`上下文
若`this.webApplicationContext不`为`null`，则说明`DispatcherServlet`在实例化期间已经被注入了应用上下文，反之说明没有注入上下文，这时首先通过`findWebApplicationContext`方法尝试寻找先前创建的应用上下文。

```java
protected WebApplicationContext findWebApplicationContext() {
    String attrName = this.getContextAttribute();
    if (attrName == null) {
        return null;
    } else {
        WebApplicationContext wac = WebApplicationContextUtils.getWebApplicationContext(this.getServletContext(), attrName);
        if (wac == null) {
            throw new IllegalStateException("No WebApplicationContext found: initializer not registered?");
        } else {
            return wac;
        }
    }
}
```
该方法从`ServletContext`的属性中找到该`servlet`初始化属性(`<init-param>`元素)的`contextAttribute`属性对应的应用上下文，若没有找到则报错。

若没有找到之前初始化的应用上下文那么就需要通过`createWebApplicationContext`方法为`DispatcherServlet`创建一个以根应用上下文为父的应用上下文，默认是**XmlWebApplicationContext**。`createWebApplicationContext`方法主要部分如下。

```java
protected WebApplicationContext createWebApplicationContext(ApplicationContext parent) {
    Class<?> contextClass = this.getContextClass();
    ...
    if (!ConfigurableWebApplicationContext.class.isAssignableFrom(contextClass)) {
        ...
    } else {
        ConfigurableWebApplicationContext wac = (ConfigurableWebApplicationContext)BeanUtils.instantiateClass(contextClass);
        wac.setEnvironment(this.getEnvironment());
        wac.setParent(parent);
        wac.setConfigLocation(this.getContextConfigLocation());
        this.configureAndRefreshWebApplicationContext(wac);
        return wac;
    }
}
```
它的主要作用是实例化`contextClass`指定类型的应用上下文，同时将根应用上下文设置为它的父上下文，并将`<servlet>`的`contextConfigLocation`初始化参数值设置到对应属性。最后调用`configureAndRefreshWebApplicationContext`方法先进一步为`DispatcherServlet`自己的应用上下文设置属性。

#### 刷新`DispatcherServlet`自己的应用上下文
`onRefresh`方法刷新`DispatcherServlet`自己的应用上下文，`DispatcherServlet`类重写了父类`FrameworkServlet`的`onRefresh`方法，该方法调用`initStrategies()`方法实例化九个组件。

```java
@Override
protected void onRefresh(ApplicationContext context) {
    initStrategies(context);
}

protected void initStrategies(ApplicationContext context) {
    initMultipartResolver(context);
    initLocaleResolver(context);
    initThemeResolver(context);
    initHandlerMappings(context);
    initHandlerAdapters(context);
    initHandlerExceptionResolvers(context);
    initRequestToViewNameTranslator(context);
    initViewResolvers(context);
    initFlashMapManager(context);
}
```

最后总结一下`DispatcherServlet`的初始化过程。

|类或接口|初始化入口方法|具体作用|
|-|-|-|
|Servlet|	init(ServletConfig config)|	接口定义，由Web容器调用|
|GenericServlet|	init(ServletConfig config)|	保存ServletConfig，内部调用无参的init方法|
|HttpServlet|	-|-|
|HttpServletBean	|init()	|设置contextConfigLocation的值,内部调用initServletBean()|
|FrameworkServlet	|initServletBean()	|初始化webApplicationContext,内部调用onRefresh(ApplicationContext context)|
|DispatcherServlet	|onRefresh(ApplicationContext context)	|初始化九大组件|

### 参考
[Spring springmvc 的启动流程](https://blog.csdn.net/weixin_43870026/article/details/86621409)</br>
[DispatcherServlet初始化过程](https://www.jianshu.com/p/be981b92f1d2)</br>
[DispatcherServlet的初始化过程](https://www.jianshu.com/p/9865f749e550)</br>
[Spring mvc之DispatcherServlet初始化详解](http://www.360doc.com/content/18/0930/19/13328254_791025670.shtml)</br>


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