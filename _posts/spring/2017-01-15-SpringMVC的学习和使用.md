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

> [Spring Web MVC Framework](http://docs.spring.io/spring-framework/docs/4.1.2.RELEASE/spring-framework-reference/html/mvc.html) 

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

#### 支持的方法参数类型： 

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

#### 支持的返回值类型 

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



