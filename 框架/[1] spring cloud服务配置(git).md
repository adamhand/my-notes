<!-- TOC -->

- [spring cloud服务配置(git)](#spring-cloud服务配置git)
    - [什么是服务配置](#什么是服务配置)
    - [建立git仓库](#建立git仓库)
    - [搭建配置服务器](#搭建配置服务器)
    - [测试配置服务器](#测试配置服务器)
    - [环境仓库](#环境仓库)
        - [多环境配置](#多环境配置)
    - [搭建客户端](#搭建客户端)
        - [动态更新配置](#动态更新配置)
    - [参考](#参考)
    - [问题](#问题)
    - [补充：yml和properties配置文件的位置](#补充yml和properties配置文件的位置)
    - [补充： bootstrap.yml、application.yml、application.properties和bootstrap.properties的优先级](#补充-bootstrapymlapplicationymlapplicationproperties和bootstrapproperties的优先级)
        - [参考](#参考)

<!-- /TOC -->

# 1. spring cloud服务配置(git)

## 1.1. 什么是服务配置
在软件开发中，将代码和配置信息分开是一种解耦的思想，比如使用`yaml`文件、`xml`文件、`json`文件或`properties`文件来单独存储配置信息，在项目发布时与项目打包并形成产品，如果配置中有任何更改，则需要编译并重新打包项目并将其重新部署到服务器上。这种方式适用于少量应用程序的情况下，但是在基于云的引用程序中可能包含数百个微服务，每个微服务可能会运行多个实例，这种方式将会十分繁琐。

解决上述问题的想法是使用一个服务(应用程序)来管理其他服务的配置，它在服务器上独立运行。`Spring Cloud`配置服务器提供**分布式、动态化集中管理**应用配置信息的能力。

`spring cloud`提供了两种方式来存储服务配置：

- 基于本地文件系统存储
- 基于`git`版本控制存储

下面先看第二种方式。

本文配置中心代码在[gitee](https://gitee.com/adamhand/spring-cloud-master/tree/master/spring-cloud-config-server-git)，客户端代码在[gitee](https://gitee.com/adamhand/spring-cloud-master/tree/master/spring-cloud-client)。

## 1.2. 建立git仓库
在搭建配置服务器之前，需要建立`git`仓库用于存放配置文件。这里使用`gitee`，仓库路径为`https://gitee.com/adamhand/cloud-server-config.git`，如下图所示。

<div align="center">
<img src="https://gitee.com/adamhand/leetcodeimages/raw/master/config-server-git1.PNG">
</div>

并在仓库中建立两个配置文件`default-config.yml`和`dev-config.yml`，文件内容分别为：

```yaml
name: default
```

```yaml
name: dev
```

## 1.3. 搭建配置服务器
首先，在`https://start.spring.io`上建立一个`spring boot`项目，添加`web`、`actuator`和`config server`依赖，如下图所示。

<div align="center">
<img src="https://gitee.com/adamhand/leetcodeimages/raw/master/config-server-git0.PNG">
</div>

在启动类中添加 `@EnableConfigServer` 注解，激活应用配置服务器。

```java
@SpringBootApplication
@EnableConfigServer
public class SpringCloudConfigServerGitApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringCloudConfigServerGitApplication.class, args);
	}

}
```

在`resources`文件夹下建立`application.yml`文件，内容为：

```yaml
server:
  port: 8080

spring:
  cloud:
    config:
      server:
        git:
          uri: https://gitee.com/adamhand/cloud-server-config.git
          username: adamhand
          password: shuchong@dai123
          force-pull: true
          default-label: master
```

## 1.4. 测试配置服务器
启动项目，打开浏览器，输入`http://localhost:8080/default/config`，会显示：

```json
{
	"name": "default",
	"profiles": ["config"],
	"label": null,
	"version": "fe61b99e78d0e7f0216e218fe41ccead0537a423",
	"state": null,
	"propertySources": [{
		"name": "https://gitee.com/adamhand/cloud-server-config.git/default-config.yml",
		"source": {
			"name": "default"
		}
	}]
}
```

同理，输入`http://localhost:8080/dev/config`会显示：

```json
{
	"name": "dev",
	"profiles": ["config"],
	"label": null,
	"version": "fe61b99e78d0e7f0216e218fe41ccead0537a423",
	"state": null,
	"propertySources": [{
		"name": "https://gitee.com/adamhand/cloud-server-config.git/dev-config.yml",
		"source": {
			"name": "dev"
		}
	}]
}
```

输入`http://localhost:8080/dev-config.yml`则会显示：

```json
name: dev
```

此时，证明配置服务器已经搭建成功。

## 1.5. 环境仓库
为什么按照上面的`url`就能访问到配置服务其中的配置呢？

`Spring Cloud`配置服务器管理多个客户端应用的配置信息，然而这些配置信息需要通过一定的规则获取。`Spring Cloud Config Server`提供 `EnvironmentRepository` 接口供客户端应用获取，其中获取维度有三：

- `{application}`：配置客户端应用名称，即配置项：`spring.application.name`
- `{profile}`：配置客户端应用当前激活的 `Profile`，即配置项：`spring.profiles.active`
- `{label}`：配置服务端标记的版本信息，如 `Git `中的分支名，默认是`master`

官方文档原文如下：

- `{application}`, which maps to `spring.applicaton.name` on the client side.
- `{profile}`, which maps to `spring.profiles.active` on the client(comma-separated list).
- `{label}`, which is a server side feature labelling a "versioned" set of config files.

仓库中的配置文件会被转换成web接口，访问可以参照以下的规则：

```
/{application}/{profile}[/{label}]
/{application}-{profile}.yml
/{label}/{application}-{profile}.yml
/{application}-{profile}.properties
/{label}/{application}-{profile}.properties
```
比如上面的`default-config.yml`文件的`application`就是`default`，`profile`就是`config`。

注意：

- 第一个规则的`label`是可以省略的，默认是`master`分支
- 无论的配置文件是`properties`还是`yml`，只要是应用名`+`环境名能匹配到这个配置文件，那么就能取到
- 如果是想直接定位到没有写环境名的默认配置，那么就可以使用`default`去匹配没有环境名的配置文件

### 1.5.1. 多环境配置
在`Spring Boot`中多环境配置文件名需要满足`application-{profile}.yaml`的格式，其中`{profile}`对应环境标识，比如：

```
application-dev.yml：开发环境
application-test.yml：测试环境
application-prod.yml：生产环境
```

至于哪个具体的配置文件会被加载，需要在`application.yml`文件中通过`spring.profiles.active`属性来设置，其值对应`{profile}`值。
如：

```
spring:
  profiles:
    active: test
```

就会加载`application-test.yml`配置文件内容。

## 1.6. 搭建客户端
配置服务器搭建好之后，进行了简单的测试，现在搭建一个客户端，用于读取配置服务器上的配置。

首先还是打开`https://spring.start/io`，新建`spring boot`项目，并添加`actuator`、`config-client`和`web`依赖。

之后打开项目，新建一个`controller`，如下：

```java
@RestController
@RefreshScope
public class NameController {
    @Value("${name: 本地名称}")
    private String name;

    @RequestMapping("/message")
    public String message() {
        return this.name;
    }
}
```

该控制器匹配`/message`路径，并返回`name`变量的值。`@Value`会取来自配置中心服务的配置项或本地环境变量等，此处取了配置中心的`name`的值，并给了它一个默认值"本地名称"，即如果远程配置中心不可用时，此变量将会用默认值初始化。

因为需要在`web`客户端项目完全启动之前去加载配置中心的配置项，所以需要在`src/main/resources`下创建`bootstrap.yml`文件，然后指定此客户端的名字和远程配置中心的`uri`：

```
server:
  port: 8081

spring:
  application:
    name: dev-config
  cloud:
    config:
      uri: http://localhost:8080
```

`server.port`制定了本`client`项目的服务端口，不能和配置中心的重复。`spring.application.name`指定了此项目的名字，用来取配置中心相同文件名的配置文件，即配置中心应有一文件名为`dev-config.yml`的配置文件。`spring.cloud.config.uri`指定了远程配置中心服务的`uri`地址，为`http://localhost/8080`。

之后先启动配置中心，再启动该`client`项目，则打开浏览器访问`http://localhost/8081/message`会返回`dev`，为配置文件`dev-config.yml`中的`name`的值。而如果不打开配置中心，则会返回本地默认的值，即`本地名称`。

### 1.6.1. 动态更新配置
为了避免每次配置更新后都要重启配置中心，`Spring boot`提供了`spring-boot-starter-actuator`组件，用来进行生产环境的维护，如检查健康信息等。上面`NameController`的`@RefreshScope`的作用就是动态地加载配置中心修改后的配置。还需要在配置中心的`dev-config.yml`添加如下内容用以暴露`acurator的/refresh`终端`api`。

```yaml
name: dev

management:
  endpoints:
    web:
      exposure:
        include: "*"
```

重启配置中心服务和`web`客户端。修改`name`为：

```
name: 更新:dev
```

将修改提交到`git`仓库，之后发送一个空的`post`请求到客户端的`/refresh`：

```curl
$ curl http://localhost:8081/actuator/refresh -d {} -H "Content-Type: application/json"
```

之后刷新页面，即可看到更新后的消息。

## 1.7. 参考
[Spring Cloud配置服务](https://www.yiibai.com/spring_cloud/understanding-spring-cloud-config-server.html)</br>
[Spring Cloud 入门教程 - 搭建配置中心服务](https://segmentfault.com/a/1190000014688483)<br>
[Spring Cloud 之配置服务器（上）](https://www.jianshu.com/p/2832cc612e04)</br>
[springCloud微服务系列——配置中心第一篇——配置管理策略](https://blog.csdn.net/guduyishuai/article/details/81703732)</br>
[Git仓库配置详解——Environment Repository](https://blog.csdn.net/Sicily_winner/article/details/88894141)</br>
[SpringCloud学习系列之四-----配置中心(Config)使用详解](https://blog.csdn.net/qazwsxpcm/article/details/88578076)</br>
[Spring Cloud（十四）Config 配置中心与客户端的使用与详细](https://www.cnblogs.com/hellxz/p/9306507.html)</br>
[为Spring Cloud Config Server配置远程git仓库](https://www.jianshu.com/p/ed2a6542fd2f)</br>
[SpringBoot的配置文件](https://www.jianshu.com/p/8bb40afa2e8c)</br>
[springcloud：配置中心git示例](https://yq.aliyun.com/articles/158718)</br>
[Spring Cloud Config（二）：基于Git搭建配置中心](https://blog.csdn.net/u010647035/article/details/83692188)</br>
[springcloud 使用git作为配置中心](https://blog.csdn.net/abreaking2012/article/details/81392837)</br>
[SpringCloud配置中心Config的搭建](https://liuyanzhao.com/9625.html)</br>


## 1.8. 问题

- 配置中心的spring.application.name和uri中{application}有什么区别？
- 多配制文件时，{application}的名称有限制吗？
- applicatoin.yml和bootstrap.yml的区别

## 1.9. 补充：yml和properties配置文件的位置

- 启动时，`spring`会从`classpath`下的`/config`目录或者`classpath`的根目录查找`application.properties`或`application.yml`
- `/config`优先于`classpath`根目录
- `@PropertySource`这个注解可以指定具体的属性配置文件，优先级比较低
- 相同优先级位置同时有`application.properties`和`application.yml`，那么`application.properties`里面的属性就会覆盖`application.yml`里的属性

## 1.10. 补充： bootstrap.yml、application.yml、application.properties和bootstrap.properties的优先级
使用上面的`web`客户端进行测试，最后的结果是，四种配置文件的优先级为：

```
application.properties > application.yml > bootstrap.properties > bootstrap.yml
```

存在相同配置时，**高优先级的配置文件会覆盖低优先级的配置文件**。造成这种结果的原因是不同配置文件的加载顺序不同，`bootstrap`类的配置文件会优于`application`类的配置文件加载，而后加载的配置文件会覆盖先加载的配置文件中的相同配置，所以直观上看起来后加载的配置文件的优先级更更高。

所以，四种配置文件的加载顺序和它们的优先级正好相反，加载顺序为：

```
bootstrap.yml > bootstrap.properties > application.yml > application.properties
```

即：

- `bootstrap`优于`application`加载
- `yml`优于`properties`加载

测试用的四种配置文件的内容分别为：

```yml
# bootstrap.yml

server:
  port: 8081

spring:
  application:
    name: dev-config
  cloud:
    config:
      uri: http://localhost:8080
```

```yml
# bootstrap.properties

server.port=8889

spring.application.name=dev-config
spring.cloud.config.uri=http://localhost:8080
```

```yml
# application.yml

server:
  port: 8088

spring:
  application:
    name: dev-config
  cloud:
    config:
      uri: http://localhost:8080
```

```yml
# application.properties

server.port=8888

spring.application.name=dev-config
spring.cloud.config.uri=http://localhost:8080
```

### 1.10.1. 参考
[bootstrap.yml & application.yml](https://www.jianshu.com/p/713c75d663f9)</br>
[bootstrap.yml、application.yml、spring.application.name-profile.yml属性覆盖](https://www.jianshu.com/p/44243a5f7af4)

