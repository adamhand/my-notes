# [4] spring cloud服务注册与发现之Eureka(Hoxton版)

## 什么是服务发现和注册
在微服务架构中往往会有一个注册中心，每个微服务都会向注册中心去注册自己的地址及端口信息，注册中心维护着服务名称与服务实例的对应关系。每个微服务都会定时从注册中心获取服务列表，同时汇报自己的运行情况，这样当有的服务需要调用其他服务时，就可以从自己获取到的服务列表中获取实例地址进行调用。

服务发现对于基于云的应用程序来说至关重要，它为应用团队提供了一种可以快速地对在环境中运行的服务实例数量进行水平伸缩的能力，同时有助于提高应用程序的弹性。

代码在[gitee](https://gitee.com/adamhand/spring-cloud-master/tree/master/spring-cloud-eureka)。

## 参考
[Spring Cloud入门-Eureka服务注册与发现(Hoxton版本)](https://blog.csdn.net/ThinkWon/article/details/103726655)</br>
[史上最简单的 SpringCloud 教程 | 第一篇： 服务的注册与发现Eureka(Finchley版本)](https://blog.csdn.net/forezp/article/details/81040925)</br>
[找回Run Dashboard](https://www.cnblogs.com/yangtianle/p/8818255.html)</br>
[显示dashboard](https://jingyan.baidu.com/article/ce4366495a1df73773afd3d3.html)</br>