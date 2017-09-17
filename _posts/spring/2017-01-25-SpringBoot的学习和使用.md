---
layout: post
title: Spring Boot 的学习和使用
category : [JavaEE, Spring]
tagline: "Supporting tagline"
tags : [Spring Boot]
---
{% include JB/setup %}
# Spring Boot 的学习和使用
---


## Developing web applications 

### The ‘Spring Web MVC framework’ 



#### Static Content 

Spring Boot 默认使用 classpath 目录或 `ServletContext` 的根目录下的 `/static` , `/public` ,`/resources` 或 `/META-INF/resources` 存放静态资源. Spring MVC 使用 `ResourceHttpRequestHandler` 处理静态资源请求(默认是: /**), 所以可以通过继承 `WebMvcConfigurer` 类并覆盖 `addResourceHandler` 方法来修改默认配置: 

``` 
@Configuration
public class WebConfig extends WebMvcConfigurerAdapter {
  @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/resources/**").addResourceLocations("file:" + imgUploadPath);
    }
}
```

或者在 application.properties 配置文件中配置: 

``` 
spring.mvc.static-path-pattern=/resources/**
spring.resources.static-locations=/,file:${imgUploadPath} 
```





## 初始化数据库 

> [初始化数据库和导入数据](http://www.jianshu.com/p/468a8fa752a7) 

### 使用 Spring JPA with Hibernate 初始化数据库

这种方法中，由* Hibernate* 库完成大部分工作，我们只需要配置合适的配置项，并创建对应的实体类的定义。在这个方案中我们主要使用以下配置项：

- *spring.jpa.hibernate.ddl-auto=create-drop* 配置项告诉 Hibernate 通过* @Entity* 模型的定义自动推断数据库定义并创建合适的表。在程序启动时，经由 Hibernate 计算出的 schema 会用来创建表结构，在程序结束时这些表也被删除。即使程序强制退出或者奔溃，在重新启动的时候也会先把之前的表删除，并重新创建——因此 "create-drop" 这种配置不适合生产环境。PS: 如果程序没有显式配置* spring.jpa.hibernate.ddl-auto* 属性，Spring Boot 会给 H2 这类的嵌入式数据库配置 create-drop，因此需要仔细斟酌这个配置项。
- 在 classpath 下创建* import.sql* 文件供* Hibernate* 使用，该文件中的内容是一些 SQL 语句，将会在应用程序启动时运行。尽管该文件中可以写任何有效的 SQL 语句，不过建议只写数据操作语句，例如 INSERT、UPDATE 等等。

### 使用 Spring JDBC 初始化数据库

如果项目中没有用 JPA 或者你不想依赖 Hibernate 库，Spring 提供另外一种方法来设置数据库，当然，首先需要提供* spring-boot-starter-jdbc* 依赖。

- *spring.jpa.hibernate.ddl-auto=none* 表示 Hibernate 不会自动创建数据库表结构。在生产环境中最好用这个设置，能够避免你不小心将数据库全部删除。
- *schema.sql* 文件包含创建数据库表结构的 SQL 语句，在应用程序启动过程中，需要创建数据库表结构时，执行该文件中的 DDL 语句。Hibernate 会自动删除已经存在的表，如果我们希望只有某个表不存在的时候才创建它，可以在这个文件开头最好先使用`DROP TABLE IF EXISTS`删除可能存在的表，再使用`CREATE TABLE IF NOT EXISTS`创建表。这种用法可以灵活得定义数据库中的表结构，因此在生产环境中用更安全。
- *data.sql* 的作用跟上一个方法的* import.sql* 一样，用于存放数据导入的 SQL 语句。

考虑到这是 Spring 的特性，我们可以不只是全局定义数据库定义文件，还可以针对不同的数据库定义不同的文件。例如，可以定义给 Oracle 数据库使用的 schema-oracle.sql，给 MySQL 数据库用的 schema-mysql.sql 文件；对于 data.sql 文件，则可以由不同数据库共用。如果你希望覆盖 Spring Boot 的自动推断，可以配置* spring.datasource.platform* 属性。 

> Tip：如果你希望使用别的名字代替 schema.sql 或者 data.sql，Spring Boot 也提供了对应的配置属性，即* spring.datasource.schema* 和* spring.datasource.data*。