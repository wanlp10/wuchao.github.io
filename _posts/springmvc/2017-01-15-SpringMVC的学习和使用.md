---
layout: post
title: SpringMVC的学习和使用
category : [JavaEE, Spring]
tagline: "Supporting tagline"
tags : [Spring MVC]
---
{% include JB/setup %}
# SpringMVC的学习和使用
---

> [Spring Web MVC Framework](http://docs.spring.io/spring-framework/docs/4.1.2.RELEASE/spring-framework-reference/html/mvc.html)

<!--break-->

## DispatcherServlet

`DispatcherServlet` 是一个 Servlet，它继承自 `HttpServlet`。下面是一个标准的 Java EE Servlet 配置，它显示了 `DispatcherServlet` 的声明和映射：
```
<web-app>
    <servlet>
        <servlet-name>example</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>example</servlet-name>
        <url-pattern>/example/*</url-pattern>
    </servlet-mapping>

</web-app>
```

上面的配置中，所有以 `/example` 开头的请求都会被名为 `example` 的 `DispatcherServlet` 实例处理。在 Servlet 3.0+ 环境中，你也可以选择以代码的方式来配置 Servlet 容器。
下面是与上面等价的基于代码的配置方式：
```
public class MyWebApplicationInitializer implements WebApplicationInitializer {

    @Override
    public void onStartup(ServletContext container) {
        ServletRegistration.Dynamic registration = container.addServlet("dispatcher", new DispatcherServlet());
        registration.setLoadOnStartup(1);
        registration.addMapping("/example/*");
    }

}
```

`WebApplicationInitializer` 是 Spring MVC 提供的一个接口，它确保基于代码的配置被监测到并且自动初始化 Servlet 3 容器。`AbstractDispatcherServletInitializer` 是一个实现了这个接口的抽象类，
它只要指定一个 Servlet 映射就可以注册 `DispatcherServlet`，详见 [Code-based Servlet container initialization](http://docs.spring.io/spring-framework/docs/4.1.2.RELEASE/spring-framework-reference/html/mvc.html#mvc-container-config)。

The above is only the first step in setting up Spring Web MVC. You now need to configure the various beans used by the Spring Web MVC framework (over and above the DispatcherServlet itself).

As detailed in Section 5.15, “Additional Capabilities of the ApplicationContext”, ApplicationContext instances in Spring can be scoped. In the Web MVC framework, each DispatcherServlet has its own WebApplicationContext, which inherits all the beans already defined in the root WebApplicationContext. These inherited beans can be overridden in the servlet-specific scope, and you can define new scope-specific beans local to a given Servlet instance.

在初始化 `DispatcherServlet` 时，Spring MVC 会在 web 应用的 `WEB-INF` 目录下找到一个名为 *[servlet-name]-servlet.xml* 的文件，并且根据该文件的中 bean 的定义来创建所有 bean 实例，它将覆盖那些在 global scope 中同名的 bean。

参考下面的 `DispatcherServlet` Servlet 的配置（在 `web.xml` 文件中）
```
<web-app>
    <servlet>
        <servlet-name>golfing</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>golfing</servlet-name>
        <url-pattern>/golfing/*</url-pattern>
    </servlet-mapping>
</web-app>
```

如果你的 Servlet 配置如上所示，那么你需要在你的 web 应用中创建一个名为 `/WEB-INF/golfing-servlet.xml` 的文件，该文件将包含你所有的 Spring Web MVC-specific 组件（beans）。
你也可以通过 Servlet 初始化参数修改该配置文件的位置（详细说明见下方代码）。
It is also possible to have just one root context for single DispatcherServlet scenarios by setting an empty contextConfigLocation servlet init parameter, as shown below

```
<web-app>
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/root-context.xml</param-value>
    </context-param>
    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value></param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>/*</url-pattern>
    </servlet-mapping>
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
</web-app>
```

`WebApplicationContext` 是 `ApplicationContext` 的一个拓展，它有一些额外的 web 应用所必须的特性。它与 `ApplicationContext` 不同之处在于它可以解析主题（详见 [Section 17.9, “Using themes”](http://docs.spring.io/spring-framework/docs/4.1.2.RELEASE/spring-framework-reference/html/mvc.html#mvc-themeresolver)），并且它知道它关联的是哪个 Servlet（通过一个指向 `ServletContext` 的链接）。
`WebApplicationContext` 被界定在 `ServletContext` 之中，如果你需要访问它，你可以借助 `RequestContextUtils` 类的 static 方法。

### Special Bean Types In the WebApplicationContext


