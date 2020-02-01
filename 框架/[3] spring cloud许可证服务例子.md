<!-- TOC -->

- [spring cloud许可证服务例子](#spring-cloud许可证服务例子)
    - [搭建配置中心](#搭建配置中心)
    - [搭建许可证服务web客户端](#搭建许可证服务web客户端)
        - [新建表](#新建表)
        - [配置文件](#配置文件)
        - [增删改查服务](#增删改查服务)
    - [测试](#测试)

<!-- /TOC -->

# 1. spring cloud许可证服务例子

学习了`spring cloud`服务配置的两种方法，现在用一个例子来进一步熟悉配置中心的相关知识。例子来源于《`spring`微服务实战》的第三章的许可证服务，在本书的第`65`页。

许可证服务作为`web`客户端，读取配置中心中关于数据库的配置信息，并实现在数据库中的增删改查操作。原书中使用的是数据库为`postgresql`，本例使用的是`mysql`。

本文代码在[gitee](https://gitee.com/adamhand/spring-cloud-master/tree/master/spring-coud-license)。

## 1.1. 搭建配置中心
本例使用`git`存放配置文件，地址为`https://gitee.com/adamhand/cloud-server-config.git`，在`licensingservice`文件夹下。其中`licensingservice.yml`文件中存储了有关数据库的配置，如下：

```yaml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/db_license?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8&useSSL=false
    username: root
    password: root
  jpa:
    datasource-platform: org.hibernate.dialect.MySQL5InnoDBDialect
    show-sql: true
    hibernate:
      ddl-auto: update

example:
  property: example-property
```

配置中心的配置文件`application.yml`文件如下：

```yaml
server:
  port: 8080

spring:
  cloud:
    config:
      server:
        git:
          uri: https://gitee.com/adamhand/cloud-server-config.git
          search-paths: licensingservice
          username: adamhand
          password: shuchong@dai123
```
同时需要在引导类上加上`@EnableConfigServer`注解。这样，配置中心就搭建好了。

## 1.2. 搭建许可证服务web客户端
许可证服务作为`web`客户端，读取配置中心的关于数据库的配置，并根据该配置对数据库进行配置，之后完成简单的“增删改查”的操作。

### 1.2.1. 新建表
本例使用`mysql`数据库，新建表以及添加字段的`sql`语句如下：

```sql
create table licenses(
    license_id varchar(100) primary key not null,
    organization_id text not null,
    product_name text not null,
    comment varchar(100)
)Engine=InnoDB, charset=utf8;

INSERT INTO licenses (license_id,  organization_id, product_name)
VALUES ('f3831f8c-c338-4ebe-a82a-e2fc1d1ff78a', 'e254f8c-c442-4ebe-a82a-e2fc1d1ff78a', 'customer-crm-co');
```

### 1.2.2. 配置文件
许可证服务的`bootstrap.yml`配置文件的内容如下：

```yaml
server:
  port: 8081
spring:
  application:
    name: licensingservice  # 对应配置中心的licensingservice文件夹
  profiles:
    active: default         # 对应licensingservice.yml文件
  cloud:
    config:
      uri: http://localhost:8080

```

### 1.2.3. 增删改查服务
单个许可证记录的`JPA`模型代码如下：

```java
package cn.licenseclient.model;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.Id;
import javax.persistence.Table;

@Entity
@Table(name = "licenses")
public class License {
    @Id
    @Column(name = "license_id", nullable = false)
    private String licenseId;

    @Column(name = "organization_id", nullable = false)
    private String organizationId;

    @Column(name = "product_name", nullable = false)
    private String productName;

    @Column(name = "comment", nullable = false)
    private String comment;

    public License withComment(String c) {
        this.comment = c;
        return this;
    }

    public void withId(String id) {
        this.licenseId = id;
    }

    public String getLicenseId() {
        return this.licenseId;
    }
}
```

在`LicenseRepostory`接口中定义查询方法：

```java
package cn.licenseclient.repository;

import cn.licenseclient.model.License;
import org.springframework.data.repository.CrudRepository;
import org.springframework.stereotype.Repository;

import java.util.List;

@Repository
public interface LicenseRepository extends CrudRepository<License, String> {
    public List<License> findByOrOrganizationId(String organizationId);

    public License findByOrOrganizationIdAndLicenseId(String organizationId, String licenseId);
}
```

用于执行数据库命令的`LicenseService`类：

```java
package cn.licenseclient.services;

import cn.licenseclient.config.ServiceConfig;
import cn.licenseclient.model.License;
import cn.licenseclient.repository.LicenseRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.UUID;

@Service
public class LicenseService {
    @Autowired
    private LicenseRepository licenseRepository;

    @Autowired
    private ServiceConfig serviceConfig;

    public License getLicense(String organizationId, String licensId) {
        License license = licenseRepository.findByOrOrganizationIdAndLicenseId(
                organizationId, licensId
        );
        return license.withComment(serviceConfig.getExampleProperty());
    }

    public List<License> getLicensesByOrgId(String organizationId) {
        return licenseRepository.findByOrOrganizationId(organizationId);
    }

    public void saveLicense(License license) {
        license.withId(UUID.randomUUID().toString());
        licenseRepository.save(license);
    }
}
```

用于从配置中心获取属性的`ServiceConfig`类：

```java
package cn.licenseclient.config;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

@Component
public class ServiceConfig {
    @Value("${example.property}")
    private String exampleProperty;

    public String getExampleProperty() {
        return exampleProperty;
    }
}
```

控制类：

```java
package cn.licenseclient.controller;

import cn.licenseclient.config.ServiceConfig;
import cn.licenseclient.model.License;
import cn.licenseclient.services.LicenseService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping(value="v1/organizations/{organizationId}/licenses")
public class LicenseServiceController {
    @Autowired
    private LicenseService licenseService;

    @Autowired
    private ServiceConfig serviceConfig;

    @RequestMapping(value="/",method = RequestMethod.GET)
    public List<License> getLicenses(@PathVariable("organizationId") String organizationId) {

        return licenseService.getLicensesByOrgId(organizationId);
    }

    @RequestMapping(value="/{licenseId}",method = RequestMethod.GET)
    public String getLicenses( @PathVariable("organizationId") String organizationId,
                                @PathVariable("licenseId") String licenseId) {

        return licenseService.getLicense(organizationId,licenseId).getLicenseId();
    }

    @RequestMapping(value="{licenseId}",method = RequestMethod.PUT)
    public String updateLicenses( @PathVariable("licenseId") String licenseId) {
        return String.format("This is the put");
    }

    @RequestMapping(value="/",method = RequestMethod.POST)
    public void saveLicenses(@RequestBody License license) {
        licenseService.saveLicense(license);
    }

    @RequestMapping(value="{licenseId}",method = RequestMethod.DELETE)
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public String deleteLicenses( @PathVariable("licenseId") String licenseId) {
        return String.format("This is the Delete");
    }
}
```

## 1.3. 测试
先后启动配置中心和许可证服务，通过如下命令访问`ServiceController`的方法：

```curl
curl -X GET http://localhost:8081/v1/organizations/e254f8c-c442-4ebe-a82a-e2fc1d1ff78a/licenses/f3831f8c-c338-4ebe-a82a-e2fc1d1ff78a
```
会返回`f3831f8c-c338-4ebe-a82a-e2fc1d1ff78a`。