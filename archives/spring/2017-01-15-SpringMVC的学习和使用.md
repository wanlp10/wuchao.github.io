---
layout: post
title: Spring MVC 的学习和使用
category : [JavaEE, Spring]
tagline: "Supporting tagline"
tags : [Spring MVC]
---
{% include JB/setup %}
# Spring MVC 的学习和使用
---

> [Spring Web MVC Framework](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/)  

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

<!--break-->

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

### Default DispatcherServlet Configuration

Spring `DispatcherServlet` 使用一些特殊的 bean 来处理请求并渲染页面。这些特殊的 bean 如下表所示： 

| Bean type                                | Explanation                              |
| ---------------------------------------- | ---------------------------------------- |
| [HandlerMapping](http://docs.spring.io/spring-framework/docs/4.1.2.RELEASE/spring-framework-reference/html/mvc.html#mvc-handlermapping) | Maps incoming requests to handlers and a list of pre- and post-processors (handler interceptors) based on some criteria the details of which vary by `HandlerMapping` implementation. The most popular implementation supports annotated controllers but other implementations exists as well. |
| HandlerAdapter                           | Helps the `DispatcherServlet` to invoke a handler mapped to a request regardless of the handler is actually invoked. For example, invoking an annotated controller requires resolving various annotations. Thus the main purpose of a `HandlerAdapter` is to shield the `DispatcherServlet` from such details. |
| [HandlerExceptionResolver](http://docs.spring.io/spring-framework/docs/4.1.2.RELEASE/spring-framework-reference/html/mvc.html#mvc-exceptionhandlers) | Maps exceptions to views also allowing for more complex exception handling code. |
| [ViewResolver](http://docs.spring.io/spring-framework/docs/4.1.2.RELEASE/spring-framework-reference/html/mvc.html#mvc-viewresolver) | Resolves logical String-based view names to actual `View` types. |
| [LocaleResolver](http://docs.spring.io/spring-framework/docs/4.1.2.RELEASE/spring-framework-reference/html/mvc.html#mvc-localeresolver) & [LocaleContextResolver](http://docs.spring.io/spring-framework/docs/4.1.2.RELEASE/spring-framework-reference/html/mvc.html#mvc-timezone) | Resolves the locale a client is using and possibly their time zone, in order to be able to offer internationalized views |
| [ThemeResolver](http://docs.spring.io/spring-framework/docs/4.1.2.RELEASE/spring-framework-reference/html/mvc.html#mvc-themeresolver) | Resolves themes your web application can use, for example, to offer personalized layouts |
| [MultipartResolver](http://docs.spring.io/spring-framework/docs/4.1.2.RELEASE/spring-framework-reference/html/mvc.html#mvc-multipart) | Parses multi-part requests for example to support processing file uploads from HTML forms. |
| [FlashMapManager](http://docs.spring.io/spring-framework/docs/4.1.2.RELEASE/spring-framework-reference/html/mvc.html#mvc-flash-attributes) | Stores and retrieves the "input" and the "output" `FlashMap` that can be used to pass attributes from one request to another, usually across a redirect. |

在 `org.springframework.web.servlet.DispatcherServlet.properties` 中配置了这些 bean 在 Spring MVC 默认使用的实现类： 

``` 
# Default implementation classes for DispatcherServlet's strategy interfaces.
# Used as fallback when no matching beans are found in the DispatcherServlet context.
# Not meant to be customized by application developers.

org.springframework.web.servlet.LocaleResolver=org.springframework.web.servlet.i18n.AcceptHeaderLocaleResolver

org.springframework.web.servlet.ThemeResolver=org.springframework.web.servlet.theme.FixedThemeResolver

org.springframework.web.servlet.HandlerMapping=org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping,\
	org.springframework.web.servlet.mvc.annotation.DefaultAnnotationHandlerMapping

org.springframework.web.servlet.HandlerAdapter=org.springframework.web.servlet.mvc.HttpRequestHandlerAdapter,\
	org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter,\
	org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter

org.springframework.web.servlet.HandlerExceptionResolver=org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerExceptionResolver,\
	org.springframework.web.servlet.mvc.annotation.ResponseStatusExceptionResolver,\
	org.springframework.web.servlet.mvc.support.DefaultHandlerExceptionResolver

org.springframework.web.servlet.RequestToViewNameTranslator=org.springframework.web.servlet.view.DefaultRequestToViewNameTranslator

org.springframework.web.servlet.ViewResolver=org.springframework.web.servlet.view.InternalResourceViewResolver

org.springframework.web.servlet.FlashMapManager=org.springframework.web.servlet.support.SessionFlashMapManager 
```

如果你手动配置了其中某一个实现类，Spring MVC 将会忽略在 `DispatcherServlet.properties` 中配置的实现类。 

具体配置详见 [Section 17.16, “Configuring Spring MVC](http://docs.spring.io/spring-framework/docs/4.1.2.RELEASE/spring-framework-reference/html/mvc.html#mvc-config) . 



### DispatcherServlet Processing Sequence 

`DispatcherServlet` 按照下面的顺序处理请求： 

- 查找 `WebApplicationContext` 并且将其作为一个属性（默认为：`DispatcherServlet.WEB_APPLICATION_CONTEXT_ATTRIBUTE` ）绑定在 request 中，这样在 controller 和进程中的其他元素处理请求时就可以使用它。
- The locale resolver is bound to the request to enable elements in the process to resolve the locale to use when processing the request (rendering the view, preparing data, and so on). If you do not need locale resolving, you do not need it. 
- The theme resolver is bound to the request to let elements such as views determine which theme to use. If you do not use themes, you can ignore it. 
- If you specify a multipart file resolver, the request is inspected for multiparts; if multiparts are found, the request is wrapped in a `MultipartHttpServletRequest` for further processing by other elements in the process. See [Section 17.10, “Spring’s multipart (file upload) support”](http://docs.spring.io/spring-framework/docs/4.1.2.RELEASE/spring-framework-reference/html/mvc.html#mvc-multipart) for further information about multipart handling. 
- An appropriate handler is searched for. If a handler is found, the execution chain associated with the handler (preprocessors, postprocessors, and controllers) is executed in order to prepare a model or rendering.
- If a model is returned, the view is rendered. If no model is returned, (may be due to a preprocessor or postprocessor intercepting the request, perhaps for security reasons), no view is rendered, because the request could already have been fulfilled. 


定义在 `WebApplicationContext` 中的 Handler 异常解析器可以获取到请求处理中抛出的异常。使用这些异常解析器允许你自定义行为去定位异常的位置。 



## Implementing Controllers 

### Defining a controller with @Controller 

`@Controller` 注解表明这是一个 Controller 类，Spring 不要求 Controller 类继承任何 Controller 基类，也不要求引用任何 Servlet API，但如果需要，你仍然可以引用 Servlet-specific 特性。 



### Mapping Requests With @RequestMapping 

你可以在一个类上或者某一个处理器方法上使用 `@RequestMapping` 注解去映射像 `/appointments` 这样的 URL 。通常类级别的注解在一个 form controller 上映射一个特定的请求路径（或路径 pattern），此外，方法级别的注解用指定一个 HTTP 请求方法（"GET", "POST" 等）或者一个请求参数条件来缩小主映射范围。

``` 
@Controller
@RequestMapping("/appointments")
public class AppointmentsController {

    private final AppointmentBook appointmentBook;

    @Autowired
    public AppointmentsController(AppointmentBook appointmentBook) {
        this.appointmentBook = appointmentBook;
    }

    @RequestMapping(method = RequestMethod.GET)
    public Map<String, Appointment> get() {
        return appointmentBook.getAppointmentsForToday();
    }

    @RequestMapping(value="/{day}", method = RequestMethod.GET)
    public Map<String, Appointment> getForDay(@PathVariable @DateTimeFormat(iso=ISO.DATE) Date day, Model model) {
        return appointmentBook.getAppointmentsForDay(day);
    }

    @RequestMapping(value="/new", method = RequestMethod.GET)
    public AppointmentForm getNewForm() {
        return new AppointmentForm();
    }

    @RequestMapping(method = RequestMethod.POST)
    public String add(@Valid AppointmentForm appointment, BindingResult result) {
        if (result.hasErrors()) {
            return "appointments/new";
        }
        appointmentBook.addAppointment(appointment);
        return "redirect:/appointments";
    }
} 
```

上面的例子中，`@RequestMapping` 在多个位置使用了。第一次是在类级别上使用 ，它表示该 controller 中的所有处理器方法的映射路径都相对于`/appointments` 路径。`get()` 方法更进一步细化了 `@RequestMapping` 注解：他仅仅接受 GET 请求，即 `/appointments` 路径的 GET 请求会进入该方法。`add()` 方法与 `get()`方法相似。`getNewForm()`方法既指定了 HTTP 请求方法也指定了映射路径，它表示 `/appointments/new` 路径的 GET 请求会进入该方法。 

`getForDay()` 方法展示了另一种 `@RequestMapping` 注解的使用方法：URI template（详见 [the next section](http://docs.spring.io/spring-framework/docs/4.1.2.RELEASE/spring-framework-reference/html/mvc.html#mvc-ann-requestmapping-uri-templates)  ）。

类级别上的 `@RequestMapping` 注解不是必需的，没有类级别上的 `@RequestMapping` 注解，controller 中所有的方法映射路径都变成了绝对路径。 

``` 
@Controller
public class ClinicController {

    private final Clinic clinic;

    @Autowired
    public ClinicController(Clinic clinic) {
        this.clinic = clinic;
    }

    @RequestMapping("/")
    public void welcomeHandler() {
    }

    @RequestMapping("/vets")
    public ModelMap vetsHandler() {
        return new ModelMap(this.clinic.getVets());
    }

} 
```

上面例子中没有指定 GET，PUT 和 POST 等 HTTP 请求方法，因为 `@RequestMapping` 注解默认使用 GET 请求方法。 



#### New Support Classes for @RequestMapping methods in Spring MVC 3.1 

Spring 3.1 为 `@RequestMapping` 方法引入了一组新的支持类，分别叫做 `RequestMappingHandlerMapping` 和 `RequestMappingHandlerAdapter` 。新的支持类被 MVC 命名空间和 MVC 的 Java config 默认启用，如果两者都不使用必须要手动配置。这里介绍旧的和新的支持类之间一些重要的区别。 

在 Spring 3.1 之前，类型和方法级别的请求映射分别在两个阶段被检查——首先 `DefaultAnnotationHandlerMapping` 会选择一个 controller， 然后 `AnnotationMethodHandlerAdapter` 会调用某个确切的方法。 

而在 Spring 3.1 中，`RequestMappingHandlerMapping` 是唯一一个判断哪一个方法应该被调用来处理请求的地方。 

现在不用再做以下事情了： 

- Select a controller first with a `SimpleUrlHandlerMapping` or `BeanNameUrlHandlerMapping` and then narrow the method based on `@RequestMapping`annotations.
- Rely on method names as a fall-back mechanism to disambiguate between two `@RequestMapping` methods that don’t have an explicit path mapping URL path but otherwise match equally, e.g. by HTTP method. In the new support classes `@RequestMapping` methods have to be mapped uniquely.
- Have a single default method (without an explicit path mapping) with which requests are processed if no other controller method matches more concretely. In the new support classes if a matching method is not found a 404 error is raised.   



#### URI Template Patterns 

一个 URI template 就是一个类似 URL 的字符串，它包含了一个或多个变量名称，当你将这些变量替换成变量值时，这个 URI templates 就变成了一个 URI。[proposed RFC](http://bitworking.org/projects/URI-Templates/) 为 URI Templates 定义了一个 URI 是如何被参数化的。例如， URI Template `http://www.example.com/users/{userId}`  包含了一个 userId 变量名，为这个变量赋一个 fred 值 ：`http://www.example.com/users/fred`。  

在 Spring MVC 中，你可以在方法形参中使用 `PathVariable` 注解将该形参与 URI template 中某个变量绑定： 

``` 
@RequestMapping(value="/owners/{ownerId}", method=RequestMethod.GET)
public String findOwner(@PathVariable String ownerId, Model model) {
    Owner owner = ownerService.findOwner(ownerId);
    model.addAttribute("owner", owner);
    return "displayOwner";
} 
```

该段代码的 URI Template `/owners/{ownerId}` 指定了一个名为  `ownerId` 的变量，当请求调用该方法时，URI Template 中的变量 `ownerId` 的值就被赋值给了使用 `@PathVariable` 注解 与之绑定的变量。例如当请求 `/owners/fred` ，findOwner() 方法中的形参 `ownerId` 就被赋值成 fred。 

使用 `@PathVariable` 注解，Spring MVC 需要去 URI Template 根据变量名称匹配，你可以在注解中指定变量名： 

``` 
@RequestMapping(value="/owners/{ownerId}", method=RequestMethod.GET)
public String findOwner(@PathVariable("ownerId") String theOwner, Model model) {
    // implementation omitted
}  

```

如果你的 URI Template 中的变量名与方法中的变量名匹配，则可以不用特定。 

一个方法可以有多个 `@PathVariable` 注解： 

``` 
@RequestMapping(value="/owners/{ownerId}/pets/{petId}", method=RequestMethod.GET)
public String findPet(@PathVariable String ownerId, @PathVariable String petId, Model model) {
    Owner owner = ownerService.findOwner(ownerId);
    Pet pet = owner.getPet(petId);
    model.addAttribute("pet", pet);
    return "displayPet";
} 
```

当一个 `@PathVariable` 注解使用了一个 `Map<String, String>` 变量，那么这个 map 变量将会包含所有的 URI Template 变量。 

一个 URI Template 可以是类型级别和方法级别的 `@RequestMapping` 注解拼接起来的，只要 `findPet()` 方法是通过 `/owners/42/pets/21` 请求进入的： 

``` 
@Controller
@RequestMapping("/owners/{ownerId}")
public class RelativePathUriTemplateController {

    @RequestMapping("/pets/{petId}")
    public void findPet(@PathVariable String ownerId, @PathVariable String petId, Model model) {
        // implementation omitted
    }

} 
```

一个 `@PathVariable` 变量可以是如何简单类型，如 int, long, Date 等，Spring 会自动将这些简单类型转换成一个合适的类型，如果转换失败将抛出一个 `TypeMismatchException`  异常。  

#### Path Pattern Matching By Suffix 



#### Matrix Variables 



#### Consumable Media Types 

你可以在请求里指定一组 consumable media type 集合，只有 `Content-Teye` 请求头匹配指定的 media 类型，该请求才会被匹配：

``` 
@Controller
@RequestMapping(value = "/pets", method = RequestMethod.POST, consumes="application/json")
public void addPet(@RequestBody Pet pet, Model model) {
    // implementation omitted
} 
```

#### Producible Media Types 

你可以在请求里指定一组 producible media type 集合，只有 `Accept` 请求头匹配了该集合中的任一个，该请求才会被匹配。Furthermore, use of the *produces* condition ensures the actual content type used to generate the response respects the media types specified in the *produces*condition。例如：

``` 
@Controller
@RequestMapping(value = "/pets/{petId}", method = RequestMethod.GET, produces="application/json")
@ResponseBody
public Pet getPet(@PathVariable String petId, Model model) {
    // implementation omitted
} 
```

> @RequestMapping(value = "/produces", produces = "application/json")：表示将功能处理方法将生产 json 格式的数据，此时根据请求头中的 Accept 进行匹配，如请求头 “Accept:application/json” 时即可匹配;
>
> @RequestMapping(value = "/produces", produces = "application/xml")：表示将功能处理方法将生产 [xml 格式](https://www.baidu.com/s?wd=xml%E6%A0%BC%E5%BC%8F&tn=44039180_cpr&fenlei=mv6quAkxTZn0IZRqIHckPjm4nH00T1dBnHfvnyD3mWbznj6suAR30ZwV5Hcvrjm3rH6sPfKWUMw85HfYnjn4nH6sgvPsT6KdThsqpZwYTjCEQLGCpyw9Uz4Bmy-bIi4WUvYETgN-TLwGUv3EnHm1rHb4nWcYP1Rvnj0drHDsn0)的数据，此时根据请求头中的 Accept 进行匹配，如请求头 “Accept:application/xml” 时即可匹配。
>
> 此种方式相对使用 @RequestMapping 的 “headers = "Accept=application/json"” 更能表明你的目的。
>
> 一、当你有如下 Accept 头：
> ①Accept：text/html,application/xml,application/json
> 将按照如下顺序进行 produces 的匹配 ①text/html ②application/xml ③application/json
> ②Accept：application/xml;q=0.5,application/json;q=0.9,text/html
> 将按照如下顺序进行 produces 的匹配 ①text/html ②application/json ③application/xml
> q 参数为媒体类型的质量因子，越大则优先权越高 (从 0 到 1)
> ③Accept：*/*,text/*,text/html
> 将按照如下顺序进行 produces 的匹配 ①text/html ②text/* ③*/*
>
> 即匹配规则为：最明确的优先匹配。
>
> 二、窄化时是覆盖 而 非继承
> 如类级别的映射为 @RequestMapping(value="/narrow", produces="text/html")，方法级别的为 @RequestMapping(produces="application/xml")，此时方法级别的映射将覆盖类级别的，因此请求头 “Accept:application/xml” 是成功的，而 “text/html” 将报 406 错误码，表示不支持的请求媒体类型。
>
> 只有生产者 / 消费者 模式 是 覆盖，其他的使用方法是继承，如 headers、params 等都是继承。
>
> 三、组合使用是 “或” 的关系
> @RequestMapping(produces={"text/html", "application/json"}) ：将匹配 “Accept:text/html” 或 “Accept:application/json”。

#### Request Parameters and Header Values 

你也可以通过请求参数条件如 `"myParam"`， `"!myParam"`， `"myParam=myValue"` 来缩小请求匹配范围。

``` 
@Controller
@RequestMapping("/owners/{ownerId}")
public class RelativePathUriTemplateController {

    @RequestMapping(value = "/pets/{petId}", method = RequestMethod.GET, params="myParam=myValue")
    public void findPet(@PathVariable String ownerId, @PathVariable String petId, Model model) {
        // implementation omitted
    }

} 
```



### Defining @RequestMapping handler methods 

一个 `@RequestMapping` 处理器方法可以有一个非常灵活的方法签名。除了 `BindingResult` 类型的参数位置是固定的外，其他参数在形参表中的顺序可以是任意的。

#### Supported method argument types：

- Request or response objects (Servlet API). Choose any specific request or response type, for example `ServletRequest` or `HttpServletRequest`.
- Session object (Servlet API): of type `HttpSession`. An argument of this type enforces the presence of a corresponding session. As a consequence, such an argument is never `null`. 
  > 访问 Session 可能不是线程安全的，尤其是在一个 Servlet 环境中。如果允许大量请求并发访问一个 session，可以考虑设置 ``RequestMappingHandlerAdapter` 的 "synchronizeOnSession" 的值为 "true"。
- `org.springframework.web.context.request.WebRequest` or `org.springframework.web.context.request.NativeWebRequest`. Allows for generic request parameter access as well as request/session attribute access, without ties to the native Servlet/Portlet API.
- `java.util.Locale` for the current request locale, determined by the most specific locale resolver available, in effect, the configured `LocaleResolver` in a Servlet environment.
- `java.io.InputStream` / `java.io.Reader` for access to the request’s content. This value is the raw InputStream/Reader as exposed by the Servlet API.
- `java.io.OutputStream` / `java.io.Writer` for generating the response’s content. This value is the raw OutputStream/Writer as exposed by the Servlet API.
- `org.springframework.http.HttpMethod` for the HTTP request method.
- `java.security.Principal` containing the currently authenticated user.
- `@PathVariable` annotated parameters for access to URI template variables. See [the section called “URI Template Patterns”](http://docs.spring.io/spring-framework/docs/4.1.2.RELEASE/spring-framework-reference/html/mvc.html#mvc-ann-requestmapping-uri-templates).
- `@MatrixVariable` annotated parameters for access to name-value pairs located in URI path segments. See [the section called “Matrix Variables”](http://docs.spring.io/spring-framework/docs/4.1.2.RELEASE/spring-framework-reference/html/mvc.html#mvc-ann-matrix-variables).
- `@RequestParam` annotated parameters for access to specific Servlet request parameters. Parameter values are converted to the declared method argument type. See [the section called “Binding request parameters to method parameters with @RequestParam”](http://docs.spring.io/spring-framework/docs/4.1.2.RELEASE/spring-framework-reference/html/mvc.html#mvc-ann-requestparam).
- `@RequestHeader` annotated parameters for access to specific Servlet request HTTP headers. Parameter values are converted to the declared method argument type. See [the section called “Mapping request header attributes with the @RequestHeader annotation”](http://docs.spring.io/spring-framework/docs/4.1.2.RELEASE/spring-framework-reference/html/mvc.html#mvc-ann-requestheader).
- `@RequestBody` annotated parameters for access to the HTTP request body. Parameter values are converted to the declared method argument type using`HttpMessageConverter`s. See [the section called “Mapping the request body with the @RequestBody annotation”](http://docs.spring.io/spring-framework/docs/4.1.2.RELEASE/spring-framework-reference/html/mvc.html#mvc-ann-requestbody).
- `@RequestPart` annotated parameters for access to the content of a "multipart/form-data" request part. See [Section 17.10.5, “Handling a file upload request from programmatic clients”](http://docs.spring.io/spring-framework/docs/4.1.2.RELEASE/spring-framework-reference/html/mvc.html#mvc-multipart-forms-non-browsers) and [Section 17.10, “Spring’s multipart (file upload) support”](http://docs.spring.io/spring-framework/docs/4.1.2.RELEASE/spring-framework-reference/html/mvc.html#mvc-multipart).
- `HttpEntity<?>` parameters for access to the Servlet request HTTP headers and contents. The request stream will be converted to the entity body using`HttpMessageConverter`s. See [the section called “Using HttpEntity”](http://docs.spring.io/spring-framework/docs/4.1.2.RELEASE/spring-framework-reference/html/mvc.html#mvc-ann-httpentity).
- `java.util.Map` / `org.springframework.ui.Model` / `org.springframework.ui.ModelMap` for enriching the implicit model that is exposed to the web view.
- `org.springframework.web.servlet.mvc.support.RedirectAttributes` to specify the exact set of attributes to use in case of a redirect and also to add flash attributes (attributes stored temporarily on the server-side to make them available to the request after the redirect). `RedirectAttributes` is used instead of the implicit model if the method returns a "redirect:" prefixed view name or `RedirectView`.
- Command or form objects to bind request parameters to bean properties (via setters) or directly to fields, with customizable type conversion, depending on `@InitBinder` methods and/or the HandlerAdapter configuration. See the `webBindingInitializer` property on `RequestMappingHandlerAdapter`. Such command objects along with their validation results will be exposed as model attributes by default, using the command class class name - e.g. model attribute "orderAddress" for a command object of type "some.package.OrderAddress". The `ModelAttribute` annotation can be used on a method argument to customize the model attribute name used.
- `org.springframework.validation.Errors` / `org.springframework.validation.BindingResult` validation results for a preceding command or form object (the immediately preceding method argument).
- `org.springframework.web.bind.support.SessionStatus` status handle for marking form processing as complete, which triggers the cleanup of session attributes that have been indicated by the `@SessionAttributes` annotation at the handler type level.
- `org.springframework.web.util.UriComponentsBuilder` a builder for preparing a URL relative to the current request’s host, port, scheme, context path, and the literal part of the servlet mapping.

The `Errors` or `BindingResult` parameters have to follow the model object that is being bound immediately as the method signature might have more that one model object and Spring will create a separate `BindingResult` instance for each of them so the following sample won’t work:

``` 
@RequestMapping(method = RequestMethod.POST)
public String processSubmit(@ModelAttribute("pet") Pet pet, Model model, BindingResult result) { ... 
} 
```

注意，这里有一个 `Model` 参数在 `Pet` 和 `BindingResult` 之间，要使该方法正常运行必须按下面这样重新调整形参的顺序： 

``` 
@RequestMapping(method = RequestMethod.POST)
public String processSubmit(@ModelAttribute("pet") Pet pet, BindingResult result, Model model) { ... 
}  
```

#### Supported method return types: 

- A `ModelAndView` object, with the model implicitly enriched with command objects and the results of `@ModelAttribute` annotated reference data accessor methods.
- A `Model` object, with the view name implicitly determined through a `RequestToViewNameTranslator` and the model implicitly enriched with command objects and the results of `@ModelAttribute` annotated reference data accessor methods.
- A `Map` object for exposing a model, with the view name implicitly determined through a `RequestToViewNameTranslator` and the model implicitly enriched with command objects and the results of `@ModelAttribute` annotated reference data accessor methods.
- A `View` object, with the model implicitly determined through command objects and `@ModelAttribute` annotated reference data accessor methods. The handler method may also programmatically enrich the model by declaring a `Model` argument (see above).
- A `String` value that is interpreted as the logical view name, with the model implicitly determined through command objects and `@ModelAttribute` annotated reference data accessor methods. The handler method may also programmatically enrich the model by declaring a `Model` argument (see above).
- `void` if the method handles the response itself (by writing the response content directly, declaring an argument of type `ServletResponse` / `HttpServletResponse` for that purpose) or if the view name is supposed to be implicitly determined through a `RequestToViewNameTranslator` (not declaring a response argument in the handler method signature).
- If the method is annotated with `@ResponseBody`, the return type is written to the response HTTP body. The return value will be converted to the declared method argument type using `HttpMessageConverter`s. See [the section called “Mapping the response body with the @ResponseBody annotation”](http://docs.spring.io/spring-framework/docs/4.1.2.RELEASE/spring-framework-reference/html/mvc.html#mvc-ann-responsebody).
- An `HttpEntity<?>` or `ResponseEntity<?>` object to provide access to the Servlet response HTTP headers and contents. The entity body will be converted to the response stream using `HttpMessageConverter`s. See [the section called “Using HttpEntity”](http://docs.spring.io/spring-framework/docs/4.1.2.RELEASE/spring-framework-reference/html/mvc.html#mvc-ann-httpentity).
- An `HttpHeaders` object to return a response with no body.
- A `Callable<?>` can be returned when the application wants to produce the return value asynchronously in a thread managed by Spring MVC.
- A `DeferredResult<?>` can be returned when the application wants to produce the return value from a thread of its own choosing.
- A `ListenableFuture<?>` can be returned when the application wants to produce the return value from a thread of its own choosing.
- Any other return type is considered to be a single model attribute to be exposed to the view, using the attribute name specified through `@ModelAttribute` at the method level (or the default attribute name based on the return type class name). The model is implicitly enriched with command objects and the results of `@ModelAttribute` annotated reference data accessor methods.  

#### Binding request parameters to method parameters with @RequestParam 

使用 `@RequestParam` 注解将请求参数和 controller 方法的参数绑定： 

``` 
@Controller
@RequestMapping("/pets")
@SessionAttributes("pet")
public class EditPetForm {

    // ...

    @RequestMapping(method = RequestMethod.GET)
    public String setupForm(@RequestParam("petId") int petId, ModelMap model) {
        Pet pet = this.clinic.loadPet(petId);
        model.addAttribute("pet", pet);
        return "petForm";
    }

    // ...

} 
```

使用了该注解的参数是必须要传值，如果要指定该参数为可选的参数，可以设置 `@RequestParam` 的 `required` 属性值为 "false"，例如：@RequestParam(value="id", required=false)，这样该参数就可以为空了。 

当一个 `Map<String, String>` 或 `MultiValue<String, String>` 参数使用了  `@RequestParam` 注解，所有请求参数都将被填充到 map 对象中。 

#### Mapping the request body with the @RequestBody annotation 

`@ResponseBody` 注解与 `@RequestBody` 注解类型，这个注解可以被放在一个方法声明的上面，表明该方法的返回值类型应该被直接写到 HTTP 响应体中，例如：

``` 
@RequestMapping(value = "/something", method = RequestMethod.PUT)
@ResponseBody
public String helloWorld() {
    return "Hello World";
} 
```

使用了 `@ResponseBody` ，Spring  使用 `HttpMessageConverter` 转换返回对象到一个响应体中。 

#### Creating REST Controllers with the @RestController annotation 

`@RestController` 注解相当于 `@RequestMapping` 和 `@ResponseBody` 两者的组合。

#### Using HttpEntity 

`HttpEntity` 与`@RequestBody` 和 `@ResponseBody` 相类似，除了可以访问 request 和 response 的 body 内容外，`HttpEntity`（和 `ResponseEntity`） 也允许访问 request 和 response 的 header 内容： 

``` 
@RequestMapping("/something")
public ResponseEntity<String> handle(HttpEntity<byte[]> requestEntity) throws UnsupportedEncodingException {
    String requestHeader = requestEntity.getHeaders().getFirst("MyRequestHeader"));
    byte[] requestBody = requestEntity.getBody();

    // do something with request header and body

    HttpHeaders responseHeaders = new HttpHeaders();
    responseHeaders.set("MyResponseHeader", "MyValue");
    return new ResponseEntity<String>("Hello World", responseHeaders, HttpStatus.CREATED);
} 
```

上面的例子获取了 request header 的 `MyRequestHeader` 的值，并且获取 body 内容，返回一个 byte 数组。它把 `MyResponseHeader` 添加到 response 中，把 `Hello World` 写到响应流中，并且设置相应状态码的值为 201。

As with `@RequestBody` and `@ResponseBody`, Spring uses `HttpMessageConverter` to convert from and to the request and response streams. For more information on these converters, see the previous section and [Message Converters](http://docs.spring.io/spring-framework/docs/4.1.2.RELEASE/spring-framework-reference/html/remoting.html#rest-message-conversion).　

#### Using @ModelAttribute on a method 

一个 `@ModelAttribute` 注解在一个方法上表明这个方法是用来添加一个或多个 model attribute 的。 `@ModelAttribute` 注解所在方法支持的参数类型 `@RequestMapping` 注解方法所支持的参数类型相同，但是不能直接被映射到 request 请求中。在 Controller 中， `@ModelAttribute` 注解所在方法会在所有 `@RequestMapping  ` 所在方法前被执行。

``` 
// Add one attribute
// The return value of the method is added to the model under the name "account"
// You can customize the name via @ModelAttribute("myAccount")

@ModelAttribute
public Account addAccount(@RequestParam String number) {
    return accountManager.findAccount(number);
}

// Add multiple attributes

@ModelAttribute
public void populateModel(@RequestParam String number, Model model) {
    model.addAttribute(accountManager.findAccount(number));
    // add more ...
} 
```

`@ModelAttribute` 方法被用来把所需的公共属性填充到 model 对象中，例如要获取一个 command 对象

在 HTML 的 form 表单中使用。

注意两种样式的 `@ModelAttribute` 方法。第一种是隐式的添加一个属性并返回它，第二种是方法形参中接收一个 `Model` 对象并添加任意数量的属性到该对象中。按需选择使用方式。 

一个 `Controller` 可以有任意数量的 `@ModelAttribute` 方法，同一个 controller 中的所有的这些方法会在 `@RequestMapping` 方法之前被执行。 

`@ModelAttribute` 方法也可以被定义在一个 `@ControllerAdvice` 注解的类里面，`@ControllerAdvice` 类里面的 `@ModelAttribute`  方法会被应用到所有 controller 上。See the [the section called “Advising controllers with the `@ControllerAdvice` annotation”](http://docs.spring.io/spring-framework/docs/4.1.2.RELEASE/spring-framework-reference/html/mvc.html#mvc-ann-controller-advice) section for more details. 

> What happens when a model attribute name is not explicitly specified? In such cases a default name is assigned to the model attribute based on its type. For example if the method returns an object of type `Account`, the default name used is "account". You can change that through the value of the `@ModelAttribute` annotation. If adding attributes directly to the `Model`, use the appropriate overloaded `addAttribute(..)` method - i.e., with or without an attribute name. 

`@ModelAttribute` 注解也可以被在 `@RequestMapping` 方法中使用，这时 `@RequestMapping` 方法的返回值将被作为一个 model attribute 而不是一个 view name。The view name is derived from view name conventions instead much like for methods returning void — see [Section 17.13.3, “The View - RequestToViewNameTranslator”](http://docs.spring.io/spring-framework/docs/4.1.2.RELEASE/spring-framework-reference/html/mvc.html#mvc-coc-r2vnt). 



#### Using @ModelAttribute on a method argument 

`@ModelAttribute` 注解在一个方法形参上表示该形参会从 model 对象中获取相应的值，如果该参数在 model 中不存在，则该参数首先会被实例化，然后添加到 model 中；如果该参数在 model 中，则该参数中的所有字段会被用 request parameters 中与之 name 相匹配的参数填充值。这就是著名的 Spring MVC 数据绑定，一个非常有用的机制，这使你避免手动地解析每一个 form 属性。 

``` 
@RequestMapping(value="/owners/{ownerId}/pets/{petId}/edit", method = RequestMethod.POST)
public String processSubmit(@ModelAttribute Pet pet) { } 
```

以上例子中，Pet 对象会如何得到？下面是几种观点： 

- 它可能已经在 model 中，由于使用了 `@SessionAttributes` 注解 — see [the section called “Using @SessionAttributes to store model attributes in the HTTP session between requests”](http://docs.spring.io/spring-framework/docs/4.1.2.RELEASE/spring-framework-reference/html/mvc.html#mvc-ann-sessionattrib). 
- 它可能已经在 model 中，由于该 controller 中的 `@ModelAttribute` 方法。 
- 它可能被获取基于一个 URI template 变量和 type converter 。 
- 它可能被使用它自己的默认构造方法实例化。 

一个 `@ModelAttribute` 方法是一个从数据库中获取数据的通用方式，这可以通过使用 `@SessionAttributes` 注解使数据被存储在 request 中。 某些情况下，他可能很方便通过 URI template 和一个 type converter 获取属性值： 

``` 
@RequestMapping(value="/accounts/{account}", method = RequestMethod.PUT)
public String save(@ModelAttribute("account") Account account) {

} 
```

该例子中 model 属性的名称（如："account"）匹配一个 URI template 变量的名称，如果你注册了 `Converter<String, Account>`  它就可以返回一个 `String` 型的 account 值到一个 `Account` 实例中，然后上面的例子就可以正常使用而不需要一个 `@ModelAttribute` 方法了。 

下一步是数据绑定（data binding）。`WebDataBinder` 类会根据 request 的参数名称（包括 query string parameters 和 form fields）和 model attribute 的字段名称进行匹配，在类型转换（从 String  类型转换成目标字段类型）之后，匹配的字段将会被填充到 Model 中匹配对象的相应字段。Data binding and validation are covered in [Chapter 7, *Validation, Data Binding, and Type Conversion*](http://docs.spring.io/spring-framework/docs/4.1.2.RELEASE/spring-framework-reference/html/validation.html). Customizing the data binding process for a controller level is covered in [the section called “Customizing WebDataBinder initialization”](http://docs.spring.io/spring-framework/docs/4.1.2.RELEASE/spring-framework-reference/html/mvc.html#mvc-ann-webdatabinder). 

数据绑定可能会出现错误，例如缺少类必要的字段或者类型转换错误，这事可以紧接着 `@ModelAttribute` 参数后面添加一个 `BindingResult` 类型的参数： 

``` 
@RequestMapping(value="/owners/{ownerId}/pets/{petId}/edit", method = RequestMethod.POST)
public String processSubmit(@ModelAttribute("pet") Pet pet, BindingResult result) {

    if (result.hasErrors()) {
        return "petForm";
    }

    // ...

} 
```

With a `BindingResult` you can check if errors were found in which case it’s common to render the same form where the errors can be shown with the help of Spring’s `<errors>` form tag. 

此外，数据绑定时你也可以使用自定义的 validator，`BindingResult` 对象也会记录数据绑定中出现的错误。这使数据绑定和验证错误集中在一个地方，然后再报告结果给用户： 

``` 
@RequestMapping(value="/owners/{ownerId}/pets/{petId}/edit", method = RequestMethod.POST)
public String processSubmit(@ModelAttribute("pet") Pet pet, BindingResult result) {

    new PetValidator().validate(pet, result);
    if (result.hasErrors()) {
        return "petForm";
    }

    // ...

} 
```

或者你可以添加 JSR-303 的 `@Valid` 注解来自动调用验证： 

``` 
@RequestMapping(value="/owners/{ownerId}/pets/{petId}/edit", method = RequestMethod.POST)
public String processSubmit(@Valid @ModelAttribute("pet") Pet pet, BindingResult result) {

    if (result.hasErrors()) {
        return "petForm";
    }

    // ...

} 
```

See [Section 7.8, “Spring Validation”](http://docs.spring.io/spring-framework/docs/4.1.2.RELEASE/spring-framework-reference/html/validation.html#validation-beanvalidation) and [Chapter 7, *Validation, Data Binding, and Type Conversion*](http://docs.spring.io/spring-framework/docs/4.1.2.RELEASE/spring-framework-reference/html/validation.html) for details on how to configure and use validation. 

#### Using @SessionAttributes to store model attributes in the HTTP session between requests

类级别的 `@SessionAttribute` 注解为一个特定的 handler 声明 session attributes, 这通常罗列出 将要被显示存储在 session 或者一些会话存储的 model attributes 的名称或类型,作为后续请求之间的表单bean. 

下面的代码片段展示类这个注解的使用:

``` 
@Controller
@RequestMapping("/editPet.do")
@SessionAttributes("pet")
public class EditPetForm {
    // ...
}
```

#### Using @SessionAttribute to access pre-existing global session attributes 

####  

#### Using @RequestAttribute to access request attributes 



#### Working with "application/x-www-form-urlencoded" data 

`@ModelAttribute` 注解支持从浏览器客户端和从非浏览器客户端表单提交时使用,但是当请求是 HTTP PUT 请求时就不一样了,浏览器可以通过 HTTP GET 请求或者 HTTP POST 请求来提交表单,非浏览器客户端除了 HTTP GET 和 HTTP POST 请求外还可以通过 HTTP PUT 请求来提交表单. Servlet 标准要求只有 HTTP POST 请求支持使用 `ServletRequest.getParameter*()` 系列方法来访问表单字段. 

为了支持 HTTP PUT 请求 和 PATCH 请求,`spring-web` 模块提供了 `HttpPutFormContentFilter`  拦截器, 这个可以在  `web.xml` 中配置: 

``` 
<filter>
    <filter-name>httpPutFormFilter</filter-name>
    <filter-class>org.springframework.web.filter.HttpPutFormContentFilter</filter-class>
</filter>

<filter-mapping>
    <filter-name>httpPutFormFilter</filter-name>
    <servlet-name>dispatcherServlet</servlet-name>
</filter-mapping>

<servlet>
    <servlet-name>dispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
</servlet> 
```

上面配置的拦截器会拦截 content type 为 `application/x-www-form-urlencoded` 的 HTTP PUT 和 PATCH 请求,然后从请求体中读取数据,再包装 `ServletRequest` ,以此来让 `ServletRequest.getParameter*()` 系列方法在读取表单数据时可用. 

> As `HttpPutFormContentFilter` consumes the body of the request, it should not be configured for PUT or PATCH URLs that rely on other converters for `application/x-www-form-urlencoded`. This includes `@RequestBody MultiValueMap<String, String>` and `HttpEntity<MultiValueMap<String, String>>`. 



#### Mapping cookie values with the @CookieValue annotation 



#### Mapping request header attributes with the @RequestHeader annotation 

`@RequestHeader` 注解允许一个方法参数绑定一个请求头. 

请求头示例: 

```
Host                    localhost:8080
Accept                  text/html,application/xhtml+xml,application/xml;q=0.9
Accept-Language         fr,en-gb;q=0.7,en;q=0.3
Accept-Encoding         gzip,deflate
Accept-Charset          ISO-8859-1,utf-8;q=0.7,*;q=0.7
Keep-Alive              300 
```

下面的代码演示如果获取 `Accept-Encoding` 和 `Keep-Alive` 请求头: 

``` 
@RequestMapping("/displayHeaderInfo.do")
public void displayHeaderInfo(@RequestHeader("Accept-Encoding") String encoding,
        @RequestHeader("Keep-Alive") long keepAlive) {
    //...
} 
```

当该注解放在了  `Map<String, String>`, `MultiValueMap<String, String>`, 或者 `HttpHeaders` 参数上时, map 对象会被填充请求头值. 

#### Method Parameters And Type Conversion 

当从请求中获取 String 类型的值时不需要做任何转换,但当目标参数不是 String 类型时, Spring 会自动将从请求中获取的值转换成适当的类型.所有的简单类型例如 int, long, Date 等都支持自动转换.你也可以通过 `WebDataBinder` 自定义数据转换过程 (see [the section called “Customizing WebDataBinder initialization”](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#mvc-ann-webdatabinder)) ,或者  by registering `Formatters` with the `FormattingConversionService` (see [Section 9.6, “Spring Field Formatting”](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#format)). 

#### Customizing WebDataBinder initialization 

想要通过 Spring 的 `WebDataBinder` 来自定义 request 参数和 PropertyEditors 的绑定,你可以在一个 `@ControllerAdvice` 注解的  controller 中定义一个 `@InitBinder` 注解的方法,或者提供一个自定义的 `WebBindingInitializer` . See the [the section called “Advising controllers with @ControllerAdvice and @RestControllerAdvice”](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#mvc-ann-controller-advice) section for more details. 

#### Customizing data binding with @InitBinder 

在 controller 的方法中使用 `@InitBinder` 注解允许你在 controller 类中直接配置 web 数据绑定, `@InitBinder` 定义的方法会初始化 `WebDataBinder`,它会在 handler 方法被填充 command 和表单对象参数时使用. 

这样的 init-binder 方法支持所有 `@RequestMapping` 注解的方法支持的参数, 除了 command,form对象和相应的验证结果对象, init-binder 方法一定不能有返回值,因此他们通常被声明位 `void`, 通常方法参数包含 `WebDataBinder` ,加上 `WebRequest` 或者 `java.util.Locale`, 这允许代码注册上下文特定的编辑器. 

下面的示例演示了使用 `@InitBinder ` 注解为所有 `java.util.Date` 类型的表单属性配置一个 `CustomDateEditor`. 

``` 
@Controller
public class MyFormController {

    @InitBinder
    protected void initBinder(WebDataBinder binder) {
        SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
        dateFormat.setLenient(false);
        binder.registerCustomEditor(Date.class, new CustomDateEditor(dateFormat, false));
    }

    // ...
} 
```

Alternatively, as of Spring 4.2, consider using `addCustomFormatter` to specify `Formatter` implementations instead of `PropertyEditor` instances. This is particularly useful if you happen to have a `Formatter`-based setup in a shared `FormattingConversionService` as well, with the same approach to be reused for controller-specific tweaking of the binding rules. 

``` 
@Controller
public class MyFormController {

    @InitBinder
    protected void initBinder(WebDataBinder binder) {
        binder.addCustomFormatter(new DateFormatter("yyyy-MM-dd"));
    }

    // ...
}
```



#### Configuring a custom WebBindingInitializer 



#### Advising controllers with @ControllerAdvice and @RestControllerAdvice 

The `@ControllerAdvice` annotation is a component annotation allowing implementation classes to be auto-detected through classpath scanning. It is automatically enabled when using the MVC namespace or the MVC Java config.

Classes annotated with `@ControllerAdvice` can contain `@ExceptionHandler`, `@InitBinder`, and `@ModelAttribute` annotated methods, and these methods will apply to `@RequestMapping` methods across all controller hierarchies as opposed to the controller hierarchy within which they are declared.

`@RestControllerAdvice` is an alternative where `@ExceptionHandler` methods assume `@ResponseBody` semantics by default. 

Both `@ControllerAdvice` and `@RestControllerAdvice` can target a subset of controllers:

```
// Target all Controllers annotated with @RestController
@ControllerAdvice(annotations = RestController.class)
public class AnnotationAdvice {}

// Target all Controllers within specific packages
@ControllerAdvice("org.example.controllers")
public class BasePackageAdvice {}

// Target all Controllers assignable to specific classes
@ControllerAdvice(assignableTypes = {ControllerInterface.class, AbstractController.class})
public class AssignableTypesAdvice {} 
```

#### Jackson Serialization View Support 

It can sometimes be useful to filter contextually the object that will be serialized to the HTTP response body. In order to provide such capability, Spring MVC has built-in support for rendering with [Jackson’s Serialization Views](http://wiki.fasterxml.com/JacksonJsonViews). 

To use it with an `@ResponseBody` controller method or controller methods that return `ResponseEntity`, simply add the `@JsonView` annotation with a class argument specifying the view class or interface to be used: 

```
@RestController
public class UserController {

    @GetMapping("/user")
    @JsonView(User.WithoutPasswordView.class)
    public User getUser() {
        return new User("eric", "7!jd#h23");
    }
}

public class User {

    public interface WithoutPasswordView {};
    public interface WithPasswordView extends WithoutPasswordView {};

    private String username;
    private String password;

    public User() {
    }

    public User(String username, String password) {
        this.username = username;
        this.password = password;
    }

    @JsonView(WithoutPasswordView.class)
    public String getUsername() {
        return this.username;
    }

    @JsonView(WithPasswordView.class)
    public String getPassword() {
        return this.password;
    }
}
```

> Note that despite `@JsonView` allowing for more than one class to be specified, the use on a controller method is only supported with exactly one class argument. Consider the use of a composite interface if you need to enable multiple views. 

For controllers relying on view resolution, simply add the serialization view class to the model:

```
@Controller
public class UserController extends AbstractController {

    @GetMapping("/user")
    public String getUser(Model model) {
        model.addAttribute("user", new User("eric", "7!jd#h23"));
        model.addAttribute(JsonView.class.getName(), User.WithoutPasswordView.class);
        return "userView";
    }
}
```

#### Jackson JSONP Support 

In order to enable [JSONP](https://en.wikipedia.org/wiki/JSONP) support for `@ResponseBody` and `ResponseEntity` methods, declare an `@ControllerAdvice` bean that extends`AbstractJsonpResponseBodyAdvice` as shown below where the constructor argument indicates the JSONP query parameter name(s):

```
@ControllerAdvice
public class JsonpAdvice extends AbstractJsonpResponseBodyAdvice {

    public JsonpAdvice() {
        super("callback");
    }
}
```

For controllers relying on view resolution, JSONP is automatically enabled when the request has a query parameter named `jsonp` or `callback`. Those names can be customized through `jsonpParameterNames` property. 



### Asynchronous Request Processing 





## Handler mappings  

In previous versions of Spring, users were required to define one or more `HandlerMapping` beans in the web application context to map incoming web requests to appropriate handlers. With the introduction of annotated controllers, you generally don’t need to do that because the `RequestMappingHandlerMapping` automatically looks for `@RequestMapping` annotations on all `@Controller` beans. However, do keep in mind that all `HandlerMapping` classes extending from `AbstractHandlerMapping` have the following properties that you can use to customize their behavior: 

- `interceptors` List of interceptors to use. `HandlerInterceptor`s are discussed in [Section 22.4.1, “Intercepting requests with a HandlerInterceptor”](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#mvc-handlermapping-interceptor).
- `defaultHandler` Default handler to use, when this handler mapping does not result in a matching handler.
- `order` Based on the value of the order property (see the `org.springframework.core.Ordered` interface), Spring sorts all handler mappings available in the context and applies the first matching handler.
- `alwaysUseFullPath` If `true` , Spring uses the full path within the current Servlet context to find an appropriate handler. If `false` (the default), the path within the current Servlet mapping is used. For example, if a Servlet is mapped using `/testing/*` and the `alwaysUseFullPath` property is set to true,`/testing/viewPage.html` is used, whereas if the property is set to false, `/viewPage.html` is used.
- `urlDecode` Defaults to `true`, as of Spring 2.5. If you prefer to compare encoded paths, set this flag to `false`. However, the `HttpServletRequest` always exposes the Servlet path in decoded form. Be aware that the Servlet path will not match when compared with encoded paths.

The following example shows how to configure an interceptor:

```
<beans>
    <bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping">
    
    
    // RequestMappingHandlerMapping.java 没有该属性 ??? 
        <property name="interceptors">
            <bean class="example.MyInterceptor"/>
        </property>
    </bean>
<beans> 
```

### Intercepting requests with a HandlerInterceptor 

Spring’s handler mapping mechanism includes handler interceptors, which are useful when you want to apply specific functionality to certain requests, for example, checking for a principal.

Interceptors located in the handler mapping must implement `HandlerInterceptor` from the `org.springframework.web.servlet` package. This interface defines three methods: `preHandle(..)` is called *before* the actual handler is executed; `postHandle(..)` is called *after* the handler is executed; and `afterCompletion(..)` is called *after the complete request has finished*. These three methods should provide enough flexibility to do all kinds of preprocessing and postprocessing.

The `preHandle(..)` method returns a boolean value. You can use this method to break or continue the processing of the execution chain. When this method returns `true`, the handler execution chain will continue; when it returns false, the `DispatcherServlet` assumes the interceptor itself has taken care of requests (and, for example, rendered an appropriate view) and does not continue executing the other interceptors and the actual handler in the execution chain.

Interceptors can be configured using the `interceptors` property, which is present on all `HandlerMapping` classes extending from `AbstractHandlerMapping`. This is shown in the example below: 

```
<beans>
    <bean id="handlerMapping"
            class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping">
        <property name="interceptors">
            <list>
                <ref bean="officeHoursInterceptor"/>
            </list>
        </property>
    </bean>

    <bean id="officeHoursInterceptor"
            class="samples.TimeBasedAccessInterceptor">
        <property name="openingTime" value="9"/>
        <property name="closingTime" value="18"/>
    </bean>
</beans> 
```

```
package samples;

public class TimeBasedAccessInterceptor extends HandlerInterceptorAdapter {

    private int openingTime;
    private int closingTime;

    public void setOpeningTime(int openingTime) {
        this.openingTime = openingTime;
    }

    public void setClosingTime(int closingTime) {
        this.closingTime = closingTime;
    }

    public boolean preHandle(HttpServletRequest request, HttpServletResponse response,
            Object handler) throws Exception {
        Calendar cal = Calendar.getInstance();
        int hour = cal.get(HOUR_OF_DAY);
        if (openingTime <= hour && hour < closingTime) {
            return true;
        }
        response.sendRedirect("http://host.com/outsideOfficeHours.html");
        return false;
    }
}
```

Any request handled by this mapping is intercepted by the `TimeBasedAccessInterceptor`. If the current time is outside office hours, the user is redirected to a static HTML file that says, for example, you can only access the website during office hours.

> When using the `RequestMappingHandlerMapping` the actual handler is an instance of `HandlerMethod` which identifies the specific controller method that will be invoked.

As you can see, the Spring adapter class `HandlerInterceptorAdapter` makes it easier to extend the `HandlerInterceptor` interface.

> In the example above, the configured interceptor will apply to all requests handled with annotated controller methods. If you want to narrow down the URL paths to which an interceptor applies, you can use the MVC namespace or the MVC Java config, or declare bean instances of type `MappedInterceptor`to do that. See [Section 22.16.1, “Enabling the MVC Java Config or the MVC XML Namespace”](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#mvc-config-enable). 

Note that the `postHandle` method of `HandlerInterceptor` is not always ideally suited for use with `@ResponseBody` and `ResponseEntity` methods. In such cases an `HttpMessageConverter` writes to and commits the response before `postHandle` is called which makes it impossible to change the response, for example to add a header. Instead an application can implement `ResponseBodyAdvice` and either declare it as an `@ControllerAdvice` bean or configure it directly on `RequestMappingHandlerAdapter`. 





## Resolving views 





## Handling exceptions 

### HandlerExceptionResolver 

Spring `HandlerExceptionResolver` implementations deal with unexpected exceptions that occur during controller execution. A `HandlerExceptionResolver`somewhat resembles the exception mappings you can define in the web application descriptor `web.xml`. However, they provide a more flexible way to do so. For example they provide information about which handler was executing when the exception was thrown. Furthermore, a programmatic way of handling exceptions gives you more options for responding appropriately before the request is forwarded to another URL (the same end result as when you use the Servlet specific exception mappings).

Besides implementing the `HandlerExceptionResolver` interface, which is only a matter of implementing the `resolveException(Exception, Handler)`method and returning a `ModelAndView`, you may also use the provided `SimpleMappingExceptionResolver` or create `@ExceptionHandler` methods. The `SimpleMappingExceptionResolver` enables you to take the class name of any exception that might be thrown and map it to a view name. This is functionally equivalent to the exception mapping feature from the Servlet API, but it is also possible to implement more finely grained mappings of exceptions from different handlers. The `@ExceptionHandler` annotation on the other hand can be used on methods that should be invoked to handle an exception. Such methods may be defined locally within an `@Controller` or may apply to many `@Controller` classes when defined within an `@ControllerAdvice` class. The following sections explain this in more detail. 

### @ExceptionHandler 

The `HandlerExceptionResolver` interface and the `SimpleMappingExceptionResolver` implementations allow you to map Exceptions to specific views declaratively along with some optional Java logic before forwarding to those views. However, in some cases, especially when relying on `@ResponseBody` methods rather than on view resolution, it may be more convenient to directly set the status of the response and optionally write error content to the body of the response.

You can do that with `@ExceptionHandler` methods. When declared within a controller such methods apply to exceptions raised by `@RequestMapping` methods of that controller (or any of its sub-classes). You can also declare an `@ExceptionHandler` method within an `@ControllerAdvice` class in which case it handles exceptions from `@RequestMapping` methods from many controllers. Below is an example of a controller-local `@ExceptionHandler` method: 

```
@Controller
public class SimpleController {

    // @RequestMapping methods omitted ...

    @ExceptionHandler(IOException.class)
    public ResponseEntity<String> handleIOException(IOException ex) {
        // prepare responseEntity
        return responseEntity;
    }

}
```

The `@ExceptionHandler` value can be set to an array of Exception types. If an exception is thrown that matches one of the types in the list, then the method annotated with the matching `@ExceptionHandler` will be invoked. If the annotation value is not set then the exception types listed as method arguments are used.

Much like standard controller methods annotated with a `@RequestMapping` annotation, the method arguments and return values of `@ExceptionHandler` methods can be flexible. For example, the `HttpServletRequest` can be accessed in Servlet environments and the `PortletRequest` in Portlet environments. The return type can be a `String`, which is interpreted as a view name, a `ModelAndView` object, a `ResponseEntity`, or you can also add the `@ResponseBody` to have the method return value converted with message converters and written to the response stream. 

### Handling Standard Spring MVC Exceptions 

Spring MVC may raise a number of exceptions while processing a request. The `SimpleMappingExceptionResolver` can easily map any exception to a default error view as needed. However, when working with clients that interpret responses in an automated way you will want to set specific status code on the response. Depending on the exception raised the status code may indicate a client error (4xx) or a server error (5xx).

The `DefaultHandlerExceptionResolver` translates Spring MVC exceptions to specific error status codes. It is registered by default with the MVC namespace, the MVC Java config, and also by the `DispatcherServlet` (i.e. when not using the MVC namespace or Java config). Listed below are some of the exceptions handled by this resolver and the corresponding status codes:

| Exception                                | HTTP Status Code             |
| ---------------------------------------- | ---------------------------- |
| `BindException`                          | 400 (Bad Request)            |
| `ConversionNotSupportedException`        | 500 (Internal Server Error)  |
| `HttpMediaTypeNotAcceptableException`    | 406 (Not Acceptable)         |
| `HttpMediaTypeNotSupportedException`     | 415 (Unsupported Media Type) |
| `HttpMessageNotReadableException`        | 400 (Bad Request)            |
| `HttpMessageNotWritableException`        | 500 (Internal Server Error)  |
| `HttpRequestMethodNotSupportedException` | 405 (Method Not Allowed)     |
| `MethodArgumentNotValidException`        | 400 (Bad Request)            |
| `MissingPathVariableException`           | 500 (Internal Server Error)  |
| `MissingServletRequestParameterException` | 400 (Bad Request)            |
| `MissingServletRequestPartException`     | 400 (Bad Request)            |
| `NoHandlerFoundException`                | 404 (Not Found)              |
| `NoSuchRequestHandlingMethodException`   | 404 (Not Found)              |
| `TypeMismatchException`                  | 400 (Bad Request)            |

The `DefaultHandlerExceptionResolver` works transparently by setting the status of the response. However, it stops short of writing any error content to the body of the response while your application may need to add developer-friendly content to every error response for example when providing a REST API. You can prepare a `ModelAndView` and render error content through view resolution — i.e. by configuring a `ContentNegotiatingViewResolver`, `MappingJackson2JsonView`, and so on. However, you may prefer to use `@ExceptionHandler` methods instead.

If you prefer to write error content via `@ExceptionHandler` methods you can extend `ResponseEntityExceptionHandler` instead. This is a convenient base for`@ControllerAdvice` classes providing an `@ExceptionHandler` method to handle standard Spring MVC exceptions and return `ResponseEntity`. That allows you to customize the response and write error content with message converters. See the `ResponseEntityExceptionHandler` javadocs for more details.

### Annotating Business Exceptions With @ResponseStatus 

A business exception can be annotated with `@ResponseStatus`. When the exception is raised, the `ResponseStatusExceptionResolver` handles it by setting the status of the response accordingly. By default the `DispatcherServlet` registers the `ResponseStatusExceptionResolver` and it is available for use.

### Customizing the Default Servlet Container Error Page 

When the status of the response is set to an error status code and the body of the response is empty, Servlet containers commonly render an HTML formatted error page. To customize the default error page of the container, you can declare an `<error-page>` element in `web.xml`. Up until Servlet 3, that element had to be mapped to a specific status code or exception type. Starting with Servlet 3 an error page does not need to be mapped, which effectively means the specified location customizes the default Servlet container error page.

```
<error-page>
    <location>/error</location>
</error-page>
```

Note that the actual location for the error page can be a JSP page or some other URL within the container including one handled through an `@Controller` method:

When writing error information, the status code and the error message set on the `HttpServletResponse` can be accessed through request attributes in a controller:

```
@Controller
public class ErrorController {

    @RequestMapping(path = "/error", produces = MediaType.APPLICATION_JSON_UTF8_VALUE)
    @ResponseBody
    public Map<String, Object> handle(HttpServletRequest request) {

        Map<String, Object> map = new HashMap<String, Object>();
        map.put("status", request.getAttribute("javax.servlet.error.status_code"));
        map.put("reason", request.getAttribute("javax.servlet.error.message"));

        return map;
    }

}
```

or in a JSP:

```
<%@ page contentType="application/json" pageEncoding="UTF-8"%>
{
    status:<%=request.getAttribute("javax.servlet.error.status_code") %>,
    reason:<%=request.getAttribute("javax.servlet.error.message") %>
}
```

#### 总结

> https://www.cnblogs.com/xinzhao/p/4902295.html

在 Spring MVC 中，所有用于处理在请求映射和请求处理过程中抛出的异常的类，都要实现 HandlerExceptionResolver 接口。AbstractHandlerExceptionResolver 实现该接口和 Orderd 接口，是 HandlerExceptionResolver 类的实现的基类。ResponseStatusExceptionResolver 等具体的异常处理类均在 AbstractHandlerExceptionResolver 之上，实现了具体的异常处理方式。一个基于 Spring MVC 的 Web 应用程序中，可以存在多个实现了 HandlerExceptionResolver 的异常处理类，他们的执行顺序，由其 order 属性决定, order 值越小，越是优先执行, 在执行到第一个返回不是 null 的 ModelAndView 的 Resolver 时，不再执行后续的尚未执行的 Resolver 的异常处理方法。。

下面我逐个介绍一下 SpringMVC 提供的这些异常处理类的功能。

##### DefaultHandlerExceptionResolver

HandlerExceptionResolver 接口的默认实现，基本上是 Spring MVC 内部使用，用来处理 Spring 定义的各种标准异常，将其转化为相对应的 HTTP Status Code。其处理的异常类型有：

```
handleNoSuchRequestHandlingMethod
handleHttpRequestMethodNotSupported
handleHttpMediaTypeNotSupported
handleMissingServletRequestParameter
handleServletRequestBindingException
handleTypeMismatch
handleHttpMessageNotReadable
handleHttpMessageNotWritable
handleMethodArgumentNotValidException
handleMissingServletRequestParameter
handleMissingServletRequestPartException
handleBindException
```

##### ResponseStatusExceptionResolver

用来支持 `@ResponseStatus` 注解的使用，处理使用了 ResponseStatus 注解的异常，根据注解的内容，返回相应的 HTTP Status Code 和内容给客户端。如果 Web 应用程序中配置了 ResponseStatusExceptionResolver，那么我们就可以使用 ResponseStatus 注解来注解我们自己编写的异常类，并在 Controller 中抛出该异常类，之后 ResponseStatusExceptionResolver 就会自动帮我们处理剩下的工作。

这是一个自己编写的异常，用来表示订单不存在：

```
 @ResponseStatus(value=HttpStatus.NOT_FOUND, reason="No such Order")  // 404
    public class OrderNotFoundException extends RuntimeException {
        // ...
    }
```

这是一个使用该异常的 Controller 方法：

```
@RequestMapping(value="/orders/{id}", method=GET)
    public String showOrder(@PathVariable("id") long id, Model model) {
        Order order = orderRepository.findOrderById(id);
        if (order == null) throw new OrderNotFoundException(id);
        model.addAttribute(order);
        return "orderDetail";
    }
```

这样，当 OrderNotFoundException 被抛出时，ResponseStatusExceptionResolver 会返回给客户端一个 HTTP Status Code 为 404 的响应。

##### AnnotationMethodHandlerExceptionResolver 和 ExceptionHandlerExceptionResolver

用来支持 ExceptionHandler 注解，使用被 ExceptionHandler 注解所标记的方法来处理异常。其中 AnnotationMethodHandlerExceptionResolver 在 3.0 版本中开始提供，ExceptionHandlerExceptionResolver 在 3.1 版本中开始提供，从 3.2 版本开始，Spring 推荐使用 ExceptionHandlerExceptionResolver。
如果配置了 AnnotationMethodHandlerExceptionResolver 和 ExceptionHandlerExceptionResolver 这两个异常处理 bean 之一，那么我们就可以使用 ExceptionHandler 注解来处理异常。

下面是几个 ExceptionHandler 注解的使用例子：

```
@Controller
public class ExceptionHandlingController {

  // @RequestHandler methods
  ...
  
  // 以下是异常处理方法
  
  // 将DataIntegrityViolationException转化为Http Status Code为409的响应
  @ResponseStatus(value=HttpStatus.CONFLICT, reason="Data integrity violation")  // 409
  @ExceptionHandler(DataIntegrityViolationException.class)
  public void conflict() {
    // Nothing to do
  }
  
  // 针对SQLException和DataAccessException返回视图databaseError
  @ExceptionHandler({SQLException.class,DataAccessException.class})
  public String databaseError() {
    // Nothing to do.  Returns the logical view name of an error page, passed to
    // the view-resolver(s) in usual way.
    // Note that the exception is _not_ available to this view (it is not added to
    // the model) but see "Extending ExceptionHandlerExceptionResolver" below.
    return "databaseError";
  }

  // 创建ModleAndView，将异常和请求的信息放入到Model中，指定视图名字，并返回该ModleAndView
  @ExceptionHandler(Exception.class)
  public ModelAndView handleError(HttpServletRequest req, Exception exception) {
    logger.error("Request: " + req.getRequestURL() + " raised " + exception);

    ModelAndView mav = new ModelAndView();
    mav.addObject("exception", exception);
    mav.addObject("url", req.getRequestURL());
    mav.setViewName("error");
    return mav;
  }
}
```

需要注意的是，上面例子中的 ExceptionHandler 方法的作用域，只是在本 Controller 类中。如果需要使用 ExceptionHandler 来处理全局的 Exception，则需要使用 ControllerAdvice 注解。

```
@ControllerAdvice
class GlobalDefaultExceptionHandler {
    public static final String DEFAULT_ERROR_VIEW = "error";

    @ExceptionHandler(value = Exception.class)
    public ModelAndView defaultErrorHandler(HttpServletRequest req, Exception e) throws Exception {
        // 如果异常使用了ResponseStatus注解，那么重新抛出该异常，Spring框架会处理该异常。 
        if (AnnotationUtils.findAnnotation(e.getClass(), ResponseStatus.class) != null)
            throw e;

        // 否则创建ModleAndView，处理该异常。
        ModelAndView mav = new ModelAndView();
        mav.addObject("exception", e);
        mav.addObject("url", req.getRequestURL());
        mav.setViewName(DEFAULT_ERROR_VIEW);
        return mav;
    }
}
```

##### SimpleMappingExceptionResolver

提供了将异常映射为视图的能力，高度可定制化。其提供的能力有：

1. 根据异常的类型，将异常映射到视图；
2. 可以为不符合处理条件没有被处理的异常，指定一个默认的错误返回；
3. 处理异常时，记录 log 信息；
4. 指定需要添加到 Modle 中的 Exception 属性，从而在视图中展示该属性。

```
@Configuration
@EnableWebMvc 
public class MvcConfiguration extends WebMvcConfigurerAdapter {
    @Bean(name="simpleMappingExceptionResolver")
    public SimpleMappingExceptionResolver createSimpleMappingExceptionResolver() {
        SimpleMappingExceptionResolver r = new SimpleMappingExceptionResolver();

        Properties mappings = new Properties();
        mappings.setProperty("DatabaseException", "databaseError");
        mappings.setProperty("InvalidCreditCardException", "creditCardError");

        r.setExceptionMappings(mappings);  // 默认为空
        r.setDefaultErrorView("error");    // 默认没有
        r.setExceptionAttribute("ex"); 
        r.setWarnLogCategory("example.MvcLogger"); 
        return r;
    }
    ...
}
```

##### 自定义 ExceptionResolver

Spring MVC 的异常处理非常的灵活，如果提供的 ExceptionResolver 类不能满足使用，我们可以实现自己的异常处理类。可以通过继承 SimpleMappingExceptionResolver 来定制 Mapping 的方式和能力，也可以直接继承 AbstractHandlerExceptionResolver 来实现其它类型的异常处理类。

#### Spring MVC 是如何创建和使用这些 Resolver 的？

首先看 Spring MVC 是怎么加载异常处理 bean 的。

1. Spring MVC 有两种加载异常处理类的方式，一种是根据类型，这种情况下，会加载 ApplicationContext 下所有实现了 ExceptionResolver 接口的 bean，并根据其 order 属性排序，依次调用；一种是根据名字，这种情况下会加载 ApplicationContext 下，名字为 handlerExceptionResolver 的 bean。
2. 不管使用那种加载方式，如果在 ApplicationContext 中没有找到异常处理 bean，那么 Spring MVC 会加载默认的异常处理 bean。
3. 默认的异常处理 bean 定义在 DispatcherServlet.properties 中。

```
org.springframework.web.servlet.HandlerExceptionResolver=org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerExceptionResolver,\
    org.springframework.web.servlet.mvc.annotation.ResponseStatusExceptionResolver,\
    org.springframework.web.servlet.mvc.support.DefaultHandlerExceptionResolver
```

以下代码摘自 DispatcherServlet，描述了异常处理类的加载过程：

```
/**
 * Initialize the HandlerMappings used by this class.
 * <p>If no HandlerMapping beans are defined in the BeanFactory for this namespace,
 * we default to BeanNameUrlHandlerMapping.
 */
private void initHandlerMappings(ApplicationContext context) {
    this.handlerMappings = null;

    if (this.detectAllHandlerMappings) {
        // Find all HandlerMappings in the ApplicationContext, including ancestor contexts.
        Map<String, HandlerMapping> matchingBeans =
                BeanFactoryUtils.beansOfTypeIncludingAncestors(context, HandlerMapping.class, true, false);
        if (!matchingBeans.isEmpty()) {
            this.handlerMappings = new ArrayList<HandlerMapping>(matchingBeans.values());
            // We keep HandlerMappings in sorted order.
            OrderComparator.sort(this.handlerMappings);
        }
    }
    else {
        try {
            HandlerMapping hm = context.getBean(HANDLER_MAPPING_BEAN_NAME, HandlerMapping.class);
            this.handlerMappings = Collections.singletonList(hm);
        }
        catch (NoSuchBeanDefinitionException ex) {
            // Ignore, we'll add a default HandlerMapping later.
        }
    }

    // Ensure we have at least one HandlerMapping, by registering
    // a default HandlerMapping if no other mappings are found.
    if (this.handlerMappings == null) {
        this.handlerMappings = getDefaultStrategies(context, HandlerMapping.class);
        if (logger.isDebugEnabled()) {
            logger.debug("No HandlerMappings found in servlet '" + getServletName() + "': using default");
        }
    }
}
```

Spring MVC 是怎么使用异常处理 bean 的。

1. Spring MVC 把请求映射和处理过程放到 try catch 中，捕获到异常后，使用异常处理 bean 进行处理。
2. 所有异常处理 bean 按照 order 属性排序，在处理过程中，遇到第一个成功处理异常的异常处理 bean 之后，不再调用后续的异常处理 bean。

以下代码摘自 DispatcherServlet，描述了处理异常的过程。

```
/**
 * Process the actual dispatching to the handler.
 * <p>The handler will be obtained by applying the servlet's HandlerMappings in order.
 * The HandlerAdapter will be obtained by querying the servlet's installed HandlerAdapters
 * to find the first that supports the handler class.
 * <p>All HTTP methods are handled by this method. It's up to HandlerAdapters or handlers
 * themselves to decide which methods are acceptable.
 * @param request current HTTP request
 * @param response current HTTP response
 * @throws Exception in case of any kind of processing failure
 */
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
    HttpServletRequest processedRequest = request;
    HandlerExecutionChain mappedHandler = null;
    boolean multipartRequestParsed = false;

    WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

    try {
        ModelAndView mv = null;
        Exception dispatchException = null;

        try {
            processedRequest = checkMultipart(request);
            multipartRequestParsed = (processedRequest != request);

            // Determine handler for the current request.
            mappedHandler = getHandler(processedRequest);
            if (mappedHandler == null || mappedHandler.getHandler() == null) {
                noHandlerFound(processedRequest, response);
                return;
            }

            // Determine handler adapter for the current request.
            HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

            // Process last-modified header, if supported by the handler.
            String method = request.getMethod();
            boolean isGet = "GET".equals(method);
            if (isGet || "HEAD".equals(method)) {
                long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
                if (logger.isDebugEnabled()) {
                    logger.debug("Last-Modified value for [" + getRequestUri(request) + "] is: " + lastModified);
                }
                if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
                    return;
                }
            }

            if (!mappedHandler.applyPreHandle(processedRequest, response)) {
                return;
            }

            // Actually invoke the handler.
            mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

            if (asyncManager.isConcurrentHandlingStarted()) {
                return;
            }

            applyDefaultViewName(request, mv);
            mappedHandler.applyPostHandle(processedRequest, response, mv);
        }
        catch (Exception ex) {
            dispatchException = ex;
        }
        processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
    }
    catch (Exception ex) {
        triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
    }
    catch (Error err) {
        triggerAfterCompletionWithError(processedRequest, response, mappedHandler, err);
    }
    finally {
        if (asyncManager.isConcurrentHandlingStarted()) {
            // Instead of postHandle and afterCompletion
            if (mappedHandler != null) {
                mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
            }
        }
        else {
            // Clean up any resources used by a multipart request.
            if (multipartRequestParsed) {
                cleanupMultipart(processedRequest);
            }
        }
    }
}


/**
 * Determine an error ModelAndView via the registered HandlerExceptionResolvers.
 * @param request current HTTP request
 * @param response current HTTP response
 * @param handler the executed handler, or {@code null} if none chosen at the time of the exception
 * (for example, if multipart resolution failed)
 * @param ex the exception that got thrown during handler execution
 * @return a corresponding ModelAndView to forward to
 * @throws Exception if no error ModelAndView found
 */
protected ModelAndView processHandlerException(HttpServletRequest request, HttpServletResponse response,
        Object handler, Exception ex) throws Exception {

    // Check registered HandlerExceptionResolvers...
    ModelAndView exMv = null;
    for (HandlerExceptionResolver handlerExceptionResolver : this.handlerExceptionResolvers) {
        exMv = handlerExceptionResolver.resolveException(request, response, handler, ex);
        if (exMv != null) {
            break;
        }
    }
    if (exMv != null) {
        if (exMv.isEmpty()) {
            request.setAttribute(EXCEPTION_ATTRIBUTE, ex);
            return null;
        }
        // We might still need view name translation for a plain error model...
        if (!exMv.hasView()) {
            exMv.setViewName(getDefaultViewName(request));
        }
        if (logger.isDebugEnabled()) {
            logger.debug("Handler execution resulted in exception - forwarding to resolved error view: " + exMv, ex);
        }
        WebUtils.exposeErrorRequestAttributes(request, ex, getServletName());
        return exMv;
    }

    throw ex;
}
```

#### 何时该使用何种 ExceptionResolver？

Spring 提供了很多选择和非常灵活的使用方式，下面是一些使用建议：

1. 如果自定义异常类，考虑加上 ResponseStatus 注解；
2. 对于没有 ResponseStatus 注解的异常，可以通过使用 ExceptionHandler+ControllerAdvice 注解，或者通过配置 SimpleMappingExceptionResolver，来为整个 Web 应用提供统一的异常处理。
3. 如果应用中有些异常处理方式，只针对特定的 Controller 使用，那么在这个 Controller 中使用 ExceptionHandler 注解。
4. 不要使用过多的异常处理方式，不然的话，维护起来会很苦恼，因为异常的处理分散在很多不同的地方。





## HTTP caching support 

A good HTTP caching strategy can significantly improve the performance of a web application and the experience of its clients. The `'Cache-Control'` HTTP response header is mostly responsible for this, along with conditional headers such as `'Last-Modified'` and `'ETag'`.

The `'Cache-Control'` HTTP response header advises private caches (e.g. browsers) and public caches (e.g. proxies) on how they can cache HTTP responses for further reuse.

An [ETag](https://en.wikipedia.org/wiki/HTTP_ETag) (entity tag) is an HTTP response header returned by an HTTP/1.1 compliant web server used to determine change in content at a given URL. It can be considered to be the more sophisticated successor to the `Last-Modified` header. When a server returns a representation with an ETag header, the client can use this header in subsequent GETs, in an `If-None-Match` header. If the content has not changed, the server returns `304: Not Modified`.

This section describes the different choices available to configure HTTP caching in a Spring Web MVC application. 

### Cache-Control HTTP header 

Spring Web MVC supports many use cases and ways to configure "Cache-Control" headers for an application. While [RFC 7234 Section 5.2.2](https://tools.ietf.org/html/rfc7234#section-5.2.2) completely describes that header and its possible directives, there are several ways to address the most common cases.

Spring Web MVC uses a configuration convention in several of its APIs: `setCachePeriod(int seconds)`:

- A `-1` value won’t generate a `'Cache-Control'` response header.
- A `0` value will prevent caching using the `'Cache-Control: no-store'` directive.
- An `n > 0` value will cache the given response for `n` seconds using the `'Cache-Control: max-age=n'` directive.

The [`CacheControl`](http://docs.spring.io/spring-framework/docs/4.3.10.RELEASE/javadoc-api/org/springframework/http/CacheControl.html) builder class simply describes the available "Cache-Control" directives and makes it easier to build your own HTTP caching strategy. Once built, a `CacheControl` instance can then be accepted as an argument in several Spring Web MVC APIs.

```
// Cache for an hour - "Cache-Control: max-age=3600"
CacheControl ccCacheOneHour = CacheControl.maxAge(1, TimeUnit.HOURS);

// Prevent caching - "Cache-Control: no-store"
CacheControl ccNoStore = CacheControl.noStore();

// Cache for ten days in public and private caches,
// public caches should not transform the response
// "Cache-Control: max-age=864000, public, no-transform"
CacheControl ccCustom = CacheControl.maxAge(10, TimeUnit.DAYS)
                                    .noTransform().cachePublic();
```

### HTTP caching support for static resources 

Static resources should be served with appropriate `'Cache-Control'` and conditional headers for optimal performance. [Configuring a `ResourceHttpRequestHandler`](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#mvc-config-static-resources) for serving static resources not only natively writes `'Last-Modified'` headers by reading a file’s metadata, but also `'Cache-Control'` headers if properly configured.

You can set the `cachePeriod` attribute on a `ResourceHttpRequestHandler` or use a `CacheControl` instance, which supports more specific directives:

```
@Configuration
@EnableWebMvc
public class WebConfig extends WebMvcConfigurerAdapter {

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/resources/**")
                .addResourceLocations("/public-resources/")
                .setCacheControl(CacheControl.maxAge(1, TimeUnit.HOURS).cachePublic());
    }

}
```

And in XML:

```
<mvc:resources mapping="/resources/**" location="/public-resources/">
    <mvc:cache-control max-age="3600" cache-public="true"/>
</mvc:resources>
```

### Support for the Cache-Control, ETag and Last-Modified response headers in Controllers



### Shallow ETag support





## Code-based Servlet container initialization 

In a Servlet 3.0+ environment, you have the option of configuring the Servlet container programmatically as an alternative or in combination with a `web.xml` file. Below is an example of registering a `DispatcherServlet`:

```
import org.springframework.web.WebApplicationInitializer;

public class MyWebApplicationInitializer implements WebApplicationInitializer {

    @Override
    public void onStartup(ServletContext container) {
        XmlWebApplicationContext appContext = new XmlWebApplicationContext();
        appContext.setConfigLocation("/WEB-INF/spring/dispatcher-config.xml");

        ServletRegistration.Dynamic registration = container.addServlet("dispatcher", new DispatcherServlet(appContext));
        registration.setLoadOnStartup(1);
        registration.addMapping("/");
    }

}
```

`WebApplicationInitializer` is an interface provided by Spring MVC that ensures your implementation is detected and automatically used to initialize any Servlet 3 container. An abstract base class implementation of `WebApplicationInitializer` named `AbstractDispatcherServletInitializer` makes it even easier to register the `DispatcherServlet` by simply overriding methods to specify the servlet mapping and the location of the `DispatcherServlet` configuration.

This is recommended for applications that use Java-based Spring configuration:

```
public class MyWebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

    @Override
    protected Class<?>[] getRootConfigClasses() {
        return null;
    }

    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class[] { MyWebConfig.class };
    }

    @Override
    protected String[] getServletMappings() {
        return new String[] { "/" };
    }

}
```

If using XML-based Spring configuration, you should extend directly from `AbstractDispatcherServletInitializer`:

```
public class MyWebAppInitializer extends AbstractDispatcherServletInitializer {

    @Override
    protected WebApplicationContext createRootApplicationContext() {
        return null;
    }

    @Override
    protected WebApplicationContext createServletApplicationContext() {
        XmlWebApplicationContext cxt = new XmlWebApplicationContext();
        cxt.setConfigLocation("/WEB-INF/spring/dispatcher-config.xml");
        return cxt;
    }

    @Override
    protected String[] getServletMappings() {
        return new String[] { "/" };
    }

}
```

`AbstractDispatcherServletInitializer` also provides a convenient way to add `Filter` instances and have them automatically mapped to the `DispatcherServlet`:

```
public class MyWebAppInitializer extends AbstractDispatcherServletInitializer {

    // ...

    @Override
    protected Filter[] getServletFilters() {
        return new Filter[] { new HiddenHttpMethodFilter(), new CharacterEncodingFilter() };
    }

}
```

Each filter is added with a default name based on its concrete type and automatically mapped to the `DispatcherServlet`.

The `isAsyncSupported` protected method of `AbstractDispatcherServletInitializer` provides a single place to enable async support on the `DispatcherServlet` and all filters mapped to it. By default this flag is set to `true`.

Finally, if you need to further customize the `DispatcherServlet` itself, you can override the `createDispatcherServlet` method.





## Configuring Spring MVC 

[Section 22.2.1, “Special Bean Types In the WebApplicationContext”](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#mvc-servlet-special-bean-types) and [Section 22.2.2, “Default DispatcherServlet Configuration”](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#mvc-servlet-config) explained about Spring MVC’s special beans and the default implementations used by the `DispatcherServlet`. In this section you’ll learn about two additional ways of configuring Spring MVC. Namely the MVC Java config and the MVC XML namespace.

The MVC Java config and the MVC namespace provide similar default configuration that overrides the `DispatcherServlet` defaults. The goal is to spare most applications from having to create the same configuration and also to provide higher-level constructs for configuring Spring MVC that serve as a simple starting point and require little or no prior knowledge of the underlying configuration.

You can choose either the MVC Java config or the MVC namespace depending on your preference. Also as you will see further below, with the MVC Java config it is easier to see the underlying configuration as well as to make fine-grained customizations directly to the created Spring MVC beans. But let’s start from the beginning.

### Enabling the MVC Java Config or the MVC XML Namespace 

To enable MVC Java config add the annotation `@EnableWebMvc` to one of your `@Configuration` classes:

```
@Configuration
@EnableWebMvc
public class WebConfig {

}
```

To achieve the same in XML use the `mvc:annotation-driven` element in your DispatcherServlet context (or in your root context if you have no DispatcherServlet context defined):

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:mvc="http://www.springframework.org/schema/mvc"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <mvc:annotation-driven/>

</beans>
```

The above registers a `RequestMappingHandlerMapping`, a `RequestMappingHandlerAdapter`, and an `ExceptionHandlerExceptionResolver` (among others) in support of processing requests with annotated controller methods using annotations such as `@RequestMapping`, `@ExceptionHandler`, and others.

It also enables the following:

1. Spring 3 style type conversion through a [ConversionService](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#core-convert) instance in addition to the JavaBeans PropertyEditors used for Data Binding.

2. Support for [formatting](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#format) Number fields using the `@NumberFormat` annotation through the `ConversionService`.

3. Support for [formatting](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#format) `Date`, `Calendar`, `Long`, and Joda Time fields using the `@DateTimeFormat` annotation.

4. Support for [validating](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#mvc-config-validation) `@Controller` inputs with `@Valid`, if a JSR-303 Provider is present on the classpath.

5. `HttpMessageConverter` support for `@RequestBody` method parameters and `@ResponseBody` method return values from `@RequestMapping` or `@ExceptionHandler` methods.

   This is the complete list of HttpMessageConverters set up by mvc:annotation-driven:

   1. `ByteArrayHttpMessageConverter` converts byte arrays.
   2. `StringHttpMessageConverter` converts strings.
   3. `ResourceHttpMessageConverter` converts to/from `org.springframework.core.io.Resource` for all media types.
   4. `SourceHttpMessageConverter` converts to/from a `javax.xml.transform.Source`.
   5. `FormHttpMessageConverter` converts form data to/from a `MultiValueMap<String, String>`.
   6. `Jaxb2RootElementHttpMessageConverter` converts Java objects to/from XML — added if JAXB2 is present and Jackson 2 XML extension is not present on the classpath.
   7. `MappingJackson2HttpMessageConverter` converts to/from JSON — added if Jackson 2 is present on the classpath.
   8. `MappingJackson2XmlHttpMessageConverter` converts to/from XML — added if [Jackson 2 XML extension](https://github.com/FasterXML/jackson-dataformat-xml) is present on the classpath.
   9. `AtomFeedHttpMessageConverter` converts Atom feeds — added if Rome is present on the classpath.
   10. `RssChannelHttpMessageConverter` converts RSS feeds — added if Rome is present on the classpath.

See [Section 22.16.12, “Message Converters”](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#mvc-config-message-converters) for more information about how to customize these default converters.

> Jackson JSON and XML converters are created using `ObjectMapper` instances created by [`Jackson2ObjectMapperBuilder`](http://docs.spring.io/spring-framework/docs/4.3.10.RELEASE/javadoc-api/org/springframework/http/converter/json/Jackson2ObjectMapperBuilder.html) in order to provide a better default configuration.
>
> This builder customizes Jackson’s default properties with the following ones:
>
> 1. [`DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES`](https://fasterxml.github.io/jackson-databind/javadoc/2.6/com/fasterxml/jackson/databind/DeserializationFeature.html#FAIL_ON_UNKNOWN_PROPERTIES) is disabled.
> 2. [`MapperFeature.DEFAULT_VIEW_INCLUSION`](https://fasterxml.github.io/jackson-databind/javadoc/2.6/com/fasterxml/jackson/databind/MapperFeature.html#DEFAULT_VIEW_INCLUSION) is disabled.
>
> It also automatically registers the following well-known modules if they are detected on the classpath:
>
> 1. [jackson-datatype-jdk7](https://github.com/FasterXML/jackson-datatype-jdk7): support for Java 7 types like `java.nio.file.Path`.
> 2. [jackson-datatype-joda](https://github.com/FasterXML/jackson-datatype-joda): support for Joda-Time types.
> 3. [jackson-datatype-jsr310](https://github.com/FasterXML/jackson-datatype-jsr310): support for Java 8 Date & Time API types.
> 4. [jackson-datatype-jdk8](https://github.com/FasterXML/jackson-datatype-jdk8): support for other Java 8 types like `Optional`. 

### Customizing the Provided Configuration 

To customize the default configuration in Java you simply implement the `WebMvcConfigurer` interface or more likely extend the class `WebMvcConfigurerAdapter`and override the methods you need:

```
@Configuration
@EnableWebMvc
public class WebConfig extends WebMvcConfigurerAdapter {

    // Override configuration methods...

}
```

To customize the default configuration of `<mvc:annotation-driven/>` check what attributes and sub-elements it supports. You can view the [Spring MVC XML schema](http://schema.spring.io/mvc/spring-mvc.xsd) or use the code completion feature of your IDE to discover what attributes and sub-elements are available.

### Conversion and Formatting 

By default formatters for `Number` and `Date` types are installed, including support for the `@NumberFormat` and `@DateTimeFormat` annotations. Full support for the Joda Time formatting library is also installed if Joda Time is present on the classpath. To register custom formatters and converters, override the `addFormatters`method:

```
@Configuration
@EnableWebMvc
public class WebConfig extends WebMvcConfigurerAdapter {

    @Override
    public void addFormatters(FormatterRegistry registry) {
        // Add formatters and/or converters
    }

}
```

In the MVC namespace the same defaults apply when `<mvc:annotation-driven>` is added. To register custom formatters and converters simply supply a `ConversionService`:

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:mvc="http://www.springframework.org/schema/mvc"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <mvc:annotation-driven conversion-service="conversionService"/>

    <bean id="conversionService"
            class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
        <property name="converters">
            <set>
                <bean class="org.example.MyConverter"/>
            </set>
        </property>
        <property name="formatters">
            <set>
                <bean class="org.example.MyFormatter"/>
                <bean class="org.example.MyAnnotationFormatterFactory"/>
            </set>
        </property>
        <property name="formatterRegistrars">
            <set>
                <bean class="org.example.MyFormatterRegistrar"/>
            </set>
        </property>
    </bean>

</beans>
```

> See [Section 9.6.4, “FormatterRegistrar SPI”](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#format-FormatterRegistrar-SPI) and the `FormattingConversionServiceFactoryBean` for more information on when to use FormatterRegistrars. 

### Validation   

Spring provides a [Validator interface](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#validator) that can be used for validation in all layers of an application. In Spring MVC you can configure it for use as a global `Validator`instance, to be used whenever an `@Valid` or `@Validated` controller method argument is encountered, and/or as a local `Validator` within a controller through an `@InitBinder` method. Global and local validator instances can be combined to provide composite validation.

Spring also [supports JSR-303/JSR-349](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#validation-beanvalidation-overview) Bean Validation via `LocalValidatorFactoryBean` which adapts the Spring `org.springframework.validation.Validator` interface to the Bean Validation `javax.validation.Validator` contract. This class can be plugged into Spring MVC as a global validator as described next.

By default use of `@EnableWebMvc` or `<mvc:annotation-driven>` automatically registers Bean Validation support in Spring MVC through the `LocalValidatorFactoryBean` when a Bean Validation provider such as Hibernate Validator is detected on the classpath. 

> Sometimes it’s convenient to have a LocalValidatorFactoryBean injected into a controller or another class. The easiest way to do that is to declare your own @Bean and also mark it with @Primary in order to avoid a conflict with the one provided with the MVC Java config.
> If you prefer to use the one from the MVC Java config, you’ll need to override the mvcValidator method from WebMvcConfigurationSupport and declare the method to explicitly return LocalValidatorFactory rather than Validator. See Section 22.16.13, “Advanced Customizations with MVC Java Config” for information on how to switch to extend the provided configuration. 

Alternatively you can configure your own global `Validator` instance:

```
@Configuration
@EnableWebMvc
public class WebConfig extends WebMvcConfigurerAdapter {

    @Override
    public Validator getValidator(); {
        // return "global" validator
    }

}
```

and in XML:

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:mvc="http://www.springframework.org/schema/mvc"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <mvc:annotation-driven validator="globalValidator"/>

</beans>
```

To combine global with local validation, simply add one or more local validator(s):

```
@Controller
public class MyController {

    @InitBinder
    protected void initBinder(WebDataBinder binder) {
        binder.addValidators(new FooValidator());
    }

}
```

With this minimal configuration any time an `@Valid` or `@Validated` method argument is encountered, it will be validated by the configured validators. Any validation violations will automatically be exposed as errors in the `BindingResult` accessible as a method argument and also renderable in Spring MVC HTML views.

### Interceptors 

You can configure `HandlerInterceptors` or `WebRequestInterceptors` to be applied to all incoming requests or restricted to specific URL path patterns.

An example of registering interceptors in Java:

```
@Configuration
@EnableWebMvc
public class WebConfig extends WebMvcConfigurerAdapter {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LocaleInterceptor());
        registry.addInterceptor(new ThemeInterceptor()).addPathPatterns("/**").excludePathPatterns("/admin/**");
        registry.addInterceptor(new SecurityInterceptor()).addPathPatterns("/secure/*");
    }

}
```

And in XML use the `<mvc:interceptors>` element:

```
<mvc:interceptors>
    <bean class="org.springframework.web.servlet.i18n.LocaleChangeInterceptor"/>
    <mvc:interceptor>
        <mvc:mapping path="/**"/>
        <mvc:exclude-mapping path="/admin/**"/>
        <bean class="org.springframework.web.servlet.theme.ThemeChangeInterceptor"/>
    </mvc:interceptor>
    <mvc:interceptor>
        <mvc:mapping path="/secure/*"/>
        <bean class="org.example.SecurityInterceptor"/>
    </mvc:interceptor>
</mvc:interceptors>
```

### Content Negotiation 

You can configure how Spring MVC determines the requested media types from the request. The available options are to check the URL path for a file extension, check the "Accept" header, a specific query parameter, or to fall back on a default content type when nothing is requested. By default the path extension in the request URI is checked first and the "Accept" header is checked second.

The MVC Java config and the MVC namespace register `json`, `xml`, `rss`, `atom` by default if corresponding dependencies are on the classpath. Additional path extension-to-media type mappings may also be registered explicitly and that also has the effect of whitelisting them as safe extensions for the purpose of RFD attack detection (see [the section called “Suffix Pattern Matching and RFD”](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#mvc-ann-requestmapping-rfd) for more detail).

Below is an example of customizing content negotiation options through the MVC Java config:

```
@Configuration
@EnableWebMvc
public class WebConfig extends WebMvcConfigurerAdapter {

    @Override
    public void configureContentNegotiation(ContentNegotiationConfigurer configurer) {
        configurer.mediaType("json", MediaType.APPLICATION_JSON);
    }
}
```

In the MVC namespace, the `<mvc:annotation-driven>` element has a `content-negotiation-manager` attribute, which expects a `ContentNegotiationManager` that in turn can be created with a `ContentNegotiationManagerFactoryBean`:

```
<mvc:annotation-driven content-negotiation-manager="contentNegotiationManager"/>

<bean id="contentNegotiationManager" class="org.springframework.web.accept.ContentNegotiationManagerFactoryBean">
    <property name="mediaTypes">
        <value>
            json=application/json
            xml=application/xml
        </value>
    </property>
</bean>
```

If not using the MVC Java config or the MVC namespace, you’ll need to create an instance of `ContentNegotiationManager` and use it to configure `RequestMappingHandlerMapping` for request mapping purposes, and `RequestMappingHandlerAdapter` and `ExceptionHandlerExceptionResolver` for content negotiation purposes.

Note that `ContentNegotiatingViewResolver` now can also be configured with a `ContentNegotiationManager`, so you can use one shared instance throughout Spring MVC.

In more advanced cases, it may be useful to configure multiple `ContentNegotiationManager` instances that in turn may contain custom`ContentNegotiationStrategy` implementations. For example you could configure `ExceptionHandlerExceptionResolver` with a `ContentNegotiationManager` that always resolves the requested media type to `"application/json"`. Or you may want to plug a custom strategy that has some logic to select a default content type (e.g. either XML or JSON) if no content types were requested.

### View Controllers 

This is a shortcut for defining a `ParameterizableViewController` that immediately forwards to a view when invoked. Use it in static cases when there is no Java controller logic to execute before the view generates the response.

An example of forwarding a request for `"/"` to a view called `"home"` in Java:

```
@Configuration
@EnableWebMvc
public class WebConfig extends WebMvcConfigurerAdapter {

    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/").setViewName("home");
    }

}
```

And the same in XML use the `<mvc:view-controller>` element:

```
<mvc:view-controller path="/" view-name="home"/>
```

### View Resolvers 

The MVC config simplifies the registration of view resolvers.

The following is a Java config example that configures content negotiation view resolution using FreeMarker HTML templates and Jackson as a default `View` for JSON rendering:

```
@Configuration
@EnableWebMvc
public class WebConfig extends WebMvcConfigurerAdapter {

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        registry.enableContentNegotiation(new MappingJackson2JsonView());
        registry.jsp();
    }

}
```

And the same in XML:

```
<mvc:view-resolvers>
    <mvc:content-negotiation>
        <mvc:default-views>
            <bean class="org.springframework.web.servlet.view.json.MappingJackson2JsonView"/>
        </mvc:default-views>
    </mvc:content-negotiation>
    <mvc:jsp/>
</mvc:view-resolvers>
```

Note however that FreeMarker, Velocity, Tiles, Groovy Markup and script templates also require configuration of the underlying view technology.

The MVC namespace provides dedicated elements. For example with FreeMarker:

```
<mvc:view-resolvers>
    <mvc:content-negotiation>
        <mvc:default-views>
            <bean class="org.springframework.web.servlet.view.json.MappingJackson2JsonView"/>
        </mvc:default-views>
    </mvc:content-negotiation>
    <mvc:freemarker cache="false"/>
</mvc:view-resolvers>

<mvc:freemarker-configurer>
    <mvc:template-loader-path location="/freemarker"/>
</mvc:freemarker-configurer>
```

In Java config simply add the respective "Configurer" bean:

```
@Configuration
@EnableWebMvc
public class WebConfig extends WebMvcConfigurerAdapter {

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        registry.enableContentNegotiation(new MappingJackson2JsonView());
        registry.freeMarker().cache(false);
    }

    @Bean
    public FreeMarkerConfigurer freeMarkerConfigurer() {
        FreeMarkerConfigurer configurer = new FreeMarkerConfigurer();
        configurer.setTemplateLoaderPath("/WEB-INF/");
        return configurer;
    }

}
```

### Serving of Resources 

This option allows static resource requests following a particular URL pattern to be served by a `ResourceHttpRequestHandler` from any of a list of `Resource`locations. This provides a convenient way to serve static resources from locations other than the web application root, including locations on the classpath. The `cache-period` property may be used to set far future expiration headers (1 year is the recommendation of optimization tools such as Page Speed and YSlow) so that they will be more efficiently utilized by the client. The handler also properly evaluates the `Last-Modified` header (if present) so that a `304` status code will be returned as appropriate, avoiding unnecessary overhead for resources that are already cached by the client. For example, to serve resource requests with a URL pattern of `/resources/**` from a `public-resources` directory within the web application root you would use:

```
@Configuration
@EnableWebMvc
public class WebConfig extends WebMvcConfigurerAdapter {

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/resources/**").addResourceLocations("/public-resources/");
    }

}
```

And the same in XML:

```
<mvc:resources mapping="/resources/**" location="/public-resources/"/>
```

To serve these resources with a 1-year future expiration to ensure maximum use of the browser cache and a reduction in HTTP requests made by the browser:

```
@Configuration
@EnableWebMvc
public class WebConfig extends WebMvcConfigurerAdapter {

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/resources/**").addResourceLocations("/public-resources/").setCachePeriod(31556926);
    }

}
```

And in XML:

```
<mvc:resources mapping="/resources/**" location="/public-resources/" cache-period="31556926"/>
```

For more details, see [HTTP caching support for static resources](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#mvc-caching-static-resources).

The `mapping` attribute must be an Ant pattern that can be used by `SimpleUrlHandlerMapping`, and the `location` attribute must specify one or more valid resource directory locations. Multiple resource locations may be specified using a comma-separated list of values. The locations specified will be checked in the specified order for the presence of the resource for any given request. For example, to enable the serving of resources from both the web application root and from a known path of `/META-INF/public-web-resources/` in any jar on the classpath use:

```
@EnableWebMvc
@Configuration
public class WebConfig extends WebMvcConfigurerAdapter {

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/resources/**")
                .addResourceLocations("/", "classpath:/META-INF/public-web-resources/");
    }

}
```

And in XML:

```
<mvc:resources mapping="/resources/**" location="/, classpath:/META-INF/public-web-resources/"/>
```

When serving resources that may change when a new version of the application is deployed it is recommended that you incorporate a version string into the mapping pattern used to request the resources so that you may force clients to request the newly deployed version of your application’s resources. Support for versioned URLs is built into the framework and can be enabled by configuring a resource chain on the resource handler. The chain consists of one more `ResourceResolver` instances followed by one or more `ResourceTransformer` instances. Together they can provide arbitrary resolution and transformation of resources.

The built-in `VersionResourceResolver` can be configured with different strategies. For example a `FixedVersionStrategy` can use a property, a date, or other as the version. A `ContentVersionStrategy` uses an MD5 hash computed from the content of the resource (known as "fingerprinting" URLs). Note that the `VersionResourceResolver` will automatically use the resolved version strings as HTTP ETag header values when serving resources.

`ContentVersionStrategy` is a good default choice to use except in cases where it cannot be used (e.g. with JavaScript module loaders). You can configure different version strategies against different patterns as shown below. Keep in mind also that computing content-based versions is expensive and therefore resource chain caching should be enabled in production.

Java config example;

```
@Configuration
@EnableWebMvc
public class WebConfig extends WebMvcConfigurerAdapter {

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/resources/**")
                .addResourceLocations("/public-resources/")
                .resourceChain(true).addResolver(
                    new VersionResourceResolver().addContentVersionStrategy("/**"));
    }

}
```

XML example:

```
<mvc:resources mapping="/resources/**" location="/public-resources/">
	<mvc:resource-chain>
		<mvc:resource-cache/>
		<mvc:resolvers>
			<mvc:version-resolver>
				<mvc:content-version-strategy patterns="/**"/>
			</mvc:version-resolver>
		</mvc:resolvers>
	</mvc:resource-chain>
</mvc:resources>
```

In order for the above to work the application must also render URLs with versions. The easiest way to do that is to configure the `ResourceUrlEncodingFilter`which wraps the response and overrides its `encodeURL` method. This will work in JSPs, FreeMarker, Velocity, and any other view technology that calls the response `encodeURL` method. Alternatively, an application can also inject and use directly the `ResourceUrlProvider` bean, which is automatically declared with the MVC Java config and the MVC namespace.

Webjars are also supported with `WebJarsResourceResolver`, which is automatically registered when the `"org.webjars:webjars-locator"` library is on classpath. This resolver allows the resource chain to resolve version agnostic libraries from HTTP GET requests `"GET /jquery/jquery.min.js"` will return resource `"/jquery/1.2.0/jquery.min.js"`. It also works by rewriting resource URLs in templates`<script src="/jquery/jquery.min.js"/> → <script src="/jquery/1.2.0/jquery.min.js"/>`.

### Falling Back On the "Default" Servlet To Serve Resources 



### Path Matching 

This allows customizing various settings related to URL mapping and path matching. For details on the individual options check out the [PathMatchConfigurer](http://docs.spring.io/spring-framework/docs/4.3.10.RELEASE/javadoc-api/org/springframework/web/servlet/config/annotation/PathMatchConfigurer.html) API.

Below is an example in Java config:

```
@Configuration
@EnableWebMvc
public class WebConfig extends WebMvcConfigurerAdapter {

    @Override
    public void configurePathMatch(PathMatchConfigurer configurer) {
        configurer
            .setUseSuffixPatternMatch(true)
            .setUseTrailingSlashMatch(false)
            .setUseRegisteredSuffixPatternMatch(true)
            .setPathMatcher(antPathMatcher())
            .setUrlPathHelper(urlPathHelper());
    }

    @Bean
    public UrlPathHelper urlPathHelper() {
        //...
    }

    @Bean
    public PathMatcher antPathMatcher() {
        //...
    }

}
```

And the same in XML, use the `<mvc:path-matching>` element:

```
<mvc:annotation-driven>
    <mvc:path-matching
        suffix-pattern="true"
        trailing-slash="false"
        registered-suffixes-only="true"
        path-helper="pathHelper"
        path-matcher="pathMatcher"/>
</mvc:annotation-driven>

<bean id="pathHelper" class="org.example.app.MyPathHelper"/>
<bean id="pathMatcher" class="org.example.app.MyPathMatcher"/>
```

### Message Converters 

Customization of `HttpMessageConverter` can be achieved in Java config by overriding [`configureMessageConverters()`](http://docs.spring.io/spring-framework/docs/4.3.10.RELEASE/javadoc-api/org/springframework/web/servlet/config/annotation/WebMvcConfigurerAdapter.html#configureMessageConverters-java.util.List-) if you want to replace the default converters created by Spring MVC, or by overriding [`extendMessageConverters()`](http://docs.spring.io/spring-framework/docs/4.3.10.RELEASE/javadoc-api/org/springframework/web/servlet/config/annotation/WebMvcConfigurerAdapter.html#extendMessageConverters-java.util.List-) if you just want to customize them or add additional converters to the default ones.

Below is an example that adds Jackson JSON and XML converters with a customized `ObjectMapper` instead of default ones:

```
@Configuration
@EnableWebMvc
public class WebConfiguration extends WebMvcConfigurerAdapter {

    @Override
    public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
        Jackson2ObjectMapperBuilder builder = new Jackson2ObjectMapperBuilder()
                .indentOutput(true)
                .dateFormat(new SimpleDateFormat("yyyy-MM-dd"))
                .modulesToInstall(new ParameterNamesModule());
        converters.add(new MappingJackson2HttpMessageConverter(builder.build()));
        converters.add(new MappingJackson2XmlHttpMessageConverter(builder.xml().build()));
    }

}
```

In this example, `Jackson2ObjectMapperBuilder` is used to create a common configuration for both `MappingJackson2HttpMessageConverter` and `MappingJackson2XmlHttpMessageConverter` with indentation enabled, a customized date format and the registration of [jackson-module-parameter-names](https://github.com/FasterXML/jackson-module-parameter-names) that adds support for accessing parameter names (feature added in Java 8).

Other interesting Jackson modules are available:

1. [jackson-datatype-money](https://github.com/zalando/jackson-datatype-money): support for `javax.money` types (unofficial module)
2. [jackson-datatype-hibernate](https://github.com/FasterXML/jackson-datatype-hibernate): support for Hibernate specific types and properties (including lazy-loading aspects)

It is also possible to do the same in XML:

```
<mvc:annotation-driven>
    <mvc:message-converters>
        <bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter">
            <property name="objectMapper" ref="objectMapper"/>
        </bean>
        <bean class="org.springframework.http.converter.xml.MappingJackson2XmlHttpMessageConverter">
            <property name="objectMapper" ref="xmlMapper"/>
        </bean>
    </mvc:message-converters>
</mvc:annotation-driven>

<bean id="objectMapper" class="org.springframework.http.converter.json.Jackson2ObjectMapperFactoryBean"
      p:indentOutput="true"
      p:simpleDateFormat="yyyy-MM-dd"
      p:modulesToInstall="com.fasterxml.jackson.module.paramnames.ParameterNamesModule"/>

<bean id="xmlMapper" parent="objectMapper" p:createXmlMapper="true"/>
```

### Advanced Customizations with MVC Java Config



### Advanced Customizations with the MVC Namespace

