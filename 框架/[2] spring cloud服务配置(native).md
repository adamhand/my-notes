# spring cloud服务配置(native)

下面看一下使用本地文件系统存储配置的服务配置方式。使用本地文件存储配置的方式和使用`git`存储配置的方式实现起来区别不大，在建立项目、需要的依赖等方面都相同，最大的区别是编写配置文件的时候。

本文代码在[gitee](https://gitee.com/adamhand/spring-cloud-master/tree/master/spring-cloud-config-server-native)。

首先，还是建立`spring boot`项目，并添加`web`、`actuator`和`config server`依赖。之后在启动类中添加 `@EnableConfigServer` 注解，激活应用配置服务器。

在`src/main/resources`目录下新建`configs`目录，用于存放配置文件。编写`native-dev.yml`和`native-prod.yml`两个配置文件，内容分别为：

```yaml
# native-dev.yml
name: dev

# native-prod.yml
name: prod
```

项目目录如下图所示：

<div align="center">
<img src="https://gitee.com/adamhand/leetcodeimages/raw/master/config-server-native0.PNG">
</div>

之后编写配置文件，在`resources`文件夹下新建`application.yml`文件，内容为：

```yml
server:
  port: 8080

spring:
  profiles:
    active: native
  cloud:
    config:
      server:
        native:
          search-locations: classpath:/configs
```

之后便可以启动项目了，启动后访问`http://localhost:8080/native/dev`和`http://localhost:8080/native/prod`，分别返回

```json
{
	"name": "native",
	"profiles": ["dev"],
	"label": null,
	"version": null,
	"state": null,
	"propertySources": [{
		"name": "classpath:/configs/native-dev.yml",
		"source": {
			"name": "dev"
		}
	}]
}
```

```json
{
	"name": "native",
	"profiles": ["prod"],
	"label": null,
	"version": null,
	"state": null,
	"propertySources": [{
		"name": "classpath:/configs/native-prod.yml",
		"source": {
			"name": "prod"
		}
	}]
}
```

证明配置服务搭建成功。之后使用`web`客户端来获取配置的方式就不再赘述了。