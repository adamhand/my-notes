# maven札记

## maven常用命令

- `mvn clean`：清除编译后目录，默认是`target`目录
- `mvn compile`：编译源代码
- `mvn install`：在本地`repository`中安装`jar`，以供其他程序根据依赖调用（包含`mvn compile`，`mvn package`，然后上传到本地仓库）
- `mvn deploy`：上传到远程服务器(包含`mvn install`)
- `mvn package`：打包
- `mvn test`：运行测试

本文所用的代码在[这里](https://github.com/adamhand/maven-master)。

### `mvn compile`、`mvn test`和`mvn clean`
新建如下目录：

```
maven-test0
  - src
    - main
      - java
       - cn
        - model
    - test
      - java
        - cn
          - model
```
之后在源码目录(`src/mian/java/cn/model`)下新建`HelloWorld.java`，代码如下：

```java
package cn.model;

class HelloWorld {
	public String hello() {
		return "hello world";
	}
}
```
在测试目录(`src/test/java/cn/model`)下建立`HelloWorldTest.java`，代码如下：

```java
package cn.model;

import org.junit.*;
import org.junit.Assert.*;
import cn.model.HelloWorld;
public class HelloWorldTest {
    @Test
	public void testHello() {
        Assert.assertEquals("hello world", new HelloWorld().hello());	
	}
}
```
在项目根目录(`maven-test0`)下新建`pom.xml`文件，添加使用到的`junit`依赖，内容如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>cn.model</groupId>
  <artifactId>maven-test0</artifactId>
  <version>1.0-SNAPSHOT</version>

  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
    </dependency>
  </dependencies>
</project>
```
之后，打开`cmd`，进入**项目根目录(maven-test0)**，执行：

```
mvn compile
```
编译项目，成功之后，结果如下：

```
...
[INFO] Compiling 1 source file to D:\Prom\maven-test-master\maven-test0\target\classes
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  1.238 s
...
```
同时在项目根目录下会生成`target`文件夹。之后执行

```
mvn test
```
来执行测试代码，执行成功之后结果如下：

```
...
-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running cn.model.HelloWorldTest
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.062 sec

Results :

Tests run: 1, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
...
```
使用`mvn clean`可以将`target`目录全部删除。

### `mvn install`命令
新建如下目录：

```
maven-test1
  - src
    - main
      - java
       - cn
        - model
    - test
      - java
        - cn
          - model
```
在源代码目录里新建`SayHelloWorld.java`，在该类中调用`maven-test0`中的`HelloWorld`类的`hello()`方法。代码如下：

```java
package cn.model;

import cn.model.HelloWorld;

class SayHelloWorld {
	public String sayHello() {
		return new HelloWorld().hello();
	}
}
```
在测试代码目录新建`SayHelloWorldTest.java`，用于测试，代码如下：

```java
package cn.model;

import org.junit.*;
import org.junit.Assert.*;
import cn.model.SayHelloWorld;
public class SayHelloWorldTest {
    @Test
	public void testHello() {
        Assert.assertEquals("hello world", new SayHelloWorld().sayHello());	
	}
}
```
此时的`pom.xml`文件如下(省略了不重要的部分)：

```xml
...
  <modelVersion>4.0.0</modelVersion>

  <groupId>cn.model</groupId>
  <artifactId>maven-test1</artifactId>
  <version>1.0-SNAPSHOT</version>

  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
    </dependency>
  </dependencies>
...
```
这时，进入`maven-test1`目录下执行`mvn compile`命令会报错，如下：

```
[ERROR] COMPILATION ERROR :
[INFO] -------------------------------------------------------------
[ERROR] /D:/Prom/maven-test-master/maven-test1/src/main/java/cn/model/SayHelloWorld.java:[3,16] 找不到符号
  符号:   类 HelloWorld
  位置: 程序包 cn.model
[ERROR] /D:/Prom/maven-test-master/maven-test1/src/main/java/cn/model/SayHelloWorld.java:[7,28] 找不到符号
  符号:   类 HelloWorld
  位置: 类 cn.model.SayHelloWorld
```
原因是`maven-test0`中的`HelloWorld`类没有发布，不能被`import`。

回到`maven-test0`目录下，执行`mvn install`命令将`jar`包发布到本地仓库中，修改`maven-test1`的`pom.xml`文件，添加依赖，如下：

```xml
  <modelVersion>4.0.0</modelVersion>

  <groupId>cn.model</groupId>
  <artifactId>maven-test1</artifactId>
  <version>1.0-SNAPSHOT</version>

  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
    </dependency>
	<dependency>
      <groupId>cn.model</groupId>
      <artifactId>maven-test0</artifactId>
      <version>1.0-SNAPSHOT</version>
    </dependency>
  </dependencies>
```
这时，再次在`maven-test1`目录下执行`mvn compile`和`mvn test`就可以成功了。

### `mvn archetype`
`maven`为开发者提供了项目骨架，可以使用`mvn archetype`命令来使用这些框架。

新建空目录`maven-test2`，从`cmd`中进入目录，执行

```
mvn archetype:generate
```
之后`maven`会下载一些依赖，然后提示选择骨架的类型，如下：

```
Choose a number or apply filter (format: [groupId:]artifactId, case sensitive contains): 7:
```
这里输入`7`，编号为`7`的骨架为：

```
7: internal -> org.apache.maven.archetypes:maven-archetype-quickstart (An archetype which contains a sample Maven project.)
```
之后会提示定义`groupid`、`artifactId`、版本号和包名：

```
Define value for property 'groupId': cn.model
Define value for property 'artifactId': maven-test2
Define value for property 'version' 1.0-SNAPSHOT: : 1.0-SNAPSHOT
Define value for property 'package' cn.model: : cn.model
```
回车之后按照提示输入`y`即可生成骨架。如下：

```
maven-test2
  - maven-test2
    - src
      - main
        - java
          - cn
            - model
      - test
        - java
          - cn
            - model
```
为了一次性输入`groupid`、`artifactId`、版本号和包名，可以给`archetype`命令加参数，如下：

```
mvn archetype:generate -DgroupId=cn.model -DartifactId=maven-test3 -Dversion=1.0.0-SNAPSHOT -Dpackage=cn.model.service
```
生成的目录为：

```
maven-test3
  - maven-test3
    - src
      - main
        - java
          - cn
            - model
              - service
      - test
        - java
          - cn
            - model
              - service
```

## `maven`中的坐标和仓库
在`maven`世界中任何一个依赖、插件或者项目构建的输出都可以称为构件，任何一个构件都有一个**坐标**作为唯一的标识。
这个坐标就是：`groupId、artifactId、version`；

- `groupId`：公司或组织的域名倒序+当前项目名称
- `artifactId`：当前项目的模块名称
- `version`：当前模块的版本

比如，良好的命名示例为：

```xml
<groupId>com.gossip.sahara</groupId>
<artifactId>sahara-model</artifactId>
<version>1.0.0-SNAPSHOT</version>
```
其中`com.gossip`为公司域名的倒写，`sahara`为项目名，`sahara-model`为项目中的模块名。

`maven`中的**仓库**就是存放依赖或插件的地方，分为**本地仓库**和**远程仓库**。会优先从本地仓库中查找依赖，若找不到再去远程仓库。仓库是基于简单文件系统存储的。远程仓库又包括中央仓库、私服(自己搭建的`maven`依赖仓库)。

`maven`中央仓库服务器大都位于国外，若没有翻墙，在国内访问起开可能会很慢，于是有了**镜像仓库**，可以自己进行配置。在`maven`安装包中`conf`文件夹下的`settings.xml`文件中`mirrors`标签填写镜像仓库的地址即可。如下：

```xml
<mirrors>
    <mirror>
        <id>nexus-aliyun</id>
        <mirrorOf>*</mirrorOf>
        <name>Nexus aliyun</name>
        <url>http://maven.aliyun.com/nexus/content/groups/public</url>
    </mirror>
</mirrors>
```
在`github`上创建自己的仓库的方式可以参见[GitHub上创建自己的Maven仓库并引用](https://blog.csdn.net/sunxiaoju/article/details/85331265)。

## `maven`生命周期
一个项目的构建过成通常包括清理、编译、测试、打包、集成测试、验证、部署等。`maven`从中抽取了一套完善的、易扩展的生命周期。`maven`的生命周期是抽象的，其中的具体任务都交由插件来完成。当没有在`pom.xml`中定义插件时，`maven`会使用默认定义的插件来完成相关工作。

`maven`定义了三套生命周期：`clean、default、site`，每个生命周期都包含了一些阶段。三套生命周期相互独立，但各个生命周期中的阶段却是有顺序的。

### `clean`生命周期
包括三个子过程：

- `pre-clean`：执行清理前的工作
- `clean`：清理上一次构建生成的文件
- `post-clean`：执行清理后的工作

### `default`生命周期
该周期是最关键的生命周期，包括许多步骤，前面的`compile`、`test`、`package`和`install`过程都属于该生命周期。

### `site`生命周期
主要包括：

- `pre-site`：生成站点前要完成的工作
- `site`：生成项目的站点文档
- `post-site`：生成站点后要完成的工作
- `site-deploy`：将生成的站点发布到指定的服务器

## 参考
[项目管理利器——maven](https://www.imooc.com/learn/443)</br>
[maven中的概念](https://blog.csdn.net/baidu_35761016/article/details/80564518)</br>
[maven生命周期](https://www.jianshu.com/p/4ae682e679ad)