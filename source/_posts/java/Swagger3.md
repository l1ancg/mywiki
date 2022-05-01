---
title: openapi（Swagger3）的应用
date: 2022-01-23 12:52:23
tags:
  - java
  - document
---



> 本文使用的是基于文档注释的模式提供的，如果使用注解模式，测试不太好用，官方给的样例是基于`1.6.5-SNAPSHOT`的代码，而发布的最新版本是`1.6.4`，有些功能貌似还不好用，反而使用基于注释的方法，契合正常的开发规范；使用`javadoc`的模式，`openapi`会读取接口注释中的内容生成api手册，如果接口的`RequestBody`和`ResponseBody`是对象的形式，`openapi`也会读取对象内的字段注释；所以在写注释时，要按照`javadoc`标准，而且书写规范的注释也是程序开发过程当中必不可少的，尤其是具有一定服务规模或团队人数的情况下。

## 1 Maven依赖

添加依赖`maven`，使用到的UI和javadoc工具

```xml
<dependency>
  <groupId>org.springdoc</groupId>
  <artifactId>springdoc-openapi-ui</artifactId>
  <version>1.6.4</version>
</dependency>
<dependency>
  <groupId>org.springdoc</groupId>
  <artifactId>springdoc-openapi-javadoc</artifactId>
  <version>1.6.4</version>
</dependency>
```

添加使用到的`maven`插件

```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-compiler-plugin</artifactId>
  <configuration>
    <parameters>true</parameters>
    <annotationProcessorPaths>
      <path>
        <groupId>com.github.therapi</groupId>
        <artifactId>therapi-runtime-javadoc-scribe</artifactId>
        <version>0.13.0</version>
      </path>
    </annotationProcessorPaths>
  </configuration>
</plugin>
```

## 2 配置文件

在`application.properties`（或`application.yml`）文件中增加一下配置内容，下面所有的配置是可选的，因为通常在程序中都会对接口进行身份认证的拦截，所以此处提供的是一套自定义的路径地址，并且都是以`/open`开头的。

```properties
springdoc.swagger-ui.path=/open/swagger-ui
springdoc.swagger-ui.url=/open/api-docs
springdoc.api-docs.path=/open/api-docs
springdoc.swagger-ui.config-url=/open/api-docs/swagger-config
springdoc.swagger-ui.use-root-path=true
springdoc.swagger-ui.disable-swagger-default-url=true
```



## 3 查看文档

http://localhost:8080/open/swagger-ui