---
layout: post
title: Content Negotiation using Spring MVC
category : [JavaEE, Spring]
tagline: "Supporting tagline"
tags : [Spring MVC]
---
{% include JB/setup %}
# Content Negotiation using Spring MVC

> [Content Negotiation using Spring MVC](https://spring.io/blog/2013/05/11/content-negotiation-using-spring-mvc)   
>
> https://www.ibm.com/support/knowledgecenter/en/SSAW57_8.5.5/com.ibm.websphere.nd.doc/ae/twbs_jaxrs_defresource_mediatype.html 



> > https://www.cnblogs.com/li0803/archive/2008/11/03/1324746.html   
>
> Accept 请求报头域用于指定客户端接受哪些类型的信息。eg：
>
> Accept：image/gif，表明客户端希望接受 GIF 图象格式的资源；
>
> Accept：text/html，表明客户端希望接受 html 文本。
>
> Content-Type 实体报头域用于指明发送给接收者的实体正文的媒体类型。eg：
>
> Content-Type:text/html;charset=ISO-8859-1
>
> Content-Type:text/html;charset=GB2312
>
> Spring 判断 response 的 media type 有三种方式: 
>
> - path extension (eg. /image.jpg)
> - URL parameter (eg. ?format=jpg)
> - HTTP Accept header (eg. Accept: image/jpg)  



<!--break-->



There are two ways to generate output using Spring MVC:

- You can use the RESTful `@ResponseBody` approach and HTTP message converters, typically to return data-formats like JSON or XML. Programmatic clients, mobile apps and AJAX enabled browsers are the usual clients.
- Alternatively you may use *view resolution*. Although views are perfectly capable of generating JSON and XML if you wish (more on that in my next post), views are normally used to generate presentation formats like HTML for a traditional web-application.
- Actually there is a third possibility - some applications require both, and Spring MVC supports such combinations easily. We will come back to that right at the end.

In either case you’ll need to deal with multiple representations (or views) of the same data returned by the controller. Working out which data format to return is called *Content Negotiation*. 

There are three situations where we need to know what type of data-format to send in the HTTP response: 

- **HttpMessageConverters:** Determine the right converter to use.
- **Request Mappings:** Map an incoming HTTP request to different methods that return different formats.
- **View Resolution:** Pick the right view to use. 

Determining what format the user has requested relies on a `ContentNegotationStrategy`. There are default implementations available out of the box, but you can also implement your own if you wish.

In this post I want to discuss how to configure and use content negotiation with Spring, mostly in terms of RESTful Controllers using HTTP message converters. In a later [post](http://blog.springsource.org/2013/06/03/content-negotiation-using-views/) I will show how to setup content negotiation specifically for use with views using Spring’s `ContentNegotiatingViewResolver`. 

### How does Content Negotiation Work? 

When making a request via HTTP it is possible to specify what type of response you would like by setting the `Accept` header property. Web browsers have this preset to request HTML (among other things). In fact, if you look, you will see that browsers actually send very confusing Accept headers, which makes relying on them impractical. See [http://www.gethifi.com/blog/browser-rest-http-accept-headers](http://www.gethifi.com/blog/browser-rest-http-accept-headers) for a nice discussion of this problem. Bottom-line: `Accept` headers are messed up and you can’t normally change them either (unless you use JavaScript and AJAX).

So, for those situations where the `Accept` header property is not desirable, Spring offers some conventions to use instead. (This was one of the nice changes in Spring 3.2 making a flexible content selection strategy available across all of Spring MVC not just when using views). You can configure a content negotiation strategy centrally once and it will apply wherever different formats (media types) need to be determined. 

### Enabling Content Negotiation in Spring MVC 

Spring supports a couple of conventions for selecting the format required: URL suffixes and/or a URL parameter. These work alongside the use of `Accept` headers. As a result, the content-type can be requested in any of three ways. By default they are checked in this order:

- Add a path extension (suffix) in the URL. So, if the incoming URL is something like `http://myserver/myapp/accounts/list.html` then HTML is required. For a spreadsheet the URL should be `http://myserver/myapp/accounts/list.xls`. The suffix to media-type mapping is automatically defined via the *JavaBeans Activation Framework* or JAF (so `activation.jar` must be on the class path).
- A URL parameter like this: `http://myserver/myapp/accounts/list?format=xls`. The name of the parameter is `format` by default, but this may be changed. Using a parameter is disabled by default, but when enabled, it is checked second.
- Finally the `Accept` HTTP header property is checked. This is how HTTP is actually defined to work, but, as previously mentioned, it can be problematic to use.

The Java Configuration to set this up, looks like this. Simply customize the predefined content negotiation manager via its configurer. Note the `MediaType` helper class has predefined constants for most well-known media-types. 

``` 
@Configuration
public class WebConfig extends WebMvcConfigurerAdapter {

  /**
    * Setup a simple strategy: use all the defaults and return XML by default when not sure. 
    */
  @Override
  public void configureContentNegotiation(ContentNegotiationConfigurer configurer) {
    configurer.defaultContentType(MediaType.APPLICATION_XML);
  }
} 
```

When using XML configuration, the content negotiation strategy is most easily setup via the `ContentNegotiationManagerFactoryBean`: 

```
<!--
        Setup a simple strategy: 
           1. Take all the defaults.
           2. Return XML by default when not sure. 
       -->
  <bean id="contentNegotiationManager"
             class="org.springframework.web.accept.ContentNegotiationManagerFactoryBean">
       <property name="defaultContentType" value="application/xml" />
  </bean>

 <!-- Make this available across all of Spring MVC -->
 <mvc:annotation-driven content-negotiation-manager="contentNegotiationManager" />
```

The `ContentNegotiationManager` created by either setup is an implementation of `ContentNegotationStrategy` that implements the *PPA Strategy* (path extension, then parameter, then Accept header) described above.  

### Additional Configuration Options 

In Java configuration, the strategy can be fully customized using methods on the configurer: 

```
@Configuration
public class WebConfig extends WebMvcConfigurerAdapter {

  /**
    *  Total customization - see below for explanation.
    */
  @Override
  public void configureContentNegotiation(ContentNegotiationConfigurer configurer) {
    configurer.favorPathExtension(false).
            favorParameter(true).
            parameterName("mediaType").
            ignoreAcceptHeader(true).
            useJaf(false).
            defaultContentType(MediaType.APPLICATION_JSON).
            mediaType("xml", MediaType.APPLICATION_XML).
            mediaType("json", MediaType.APPLICATION_JSON);
  }
} 
```

In XML, the strategy can be configured using methods on the factory bean: 

```
 <!-- Total customization - see below for explanation. -->
  <bean id="contentNegotiationManager"
             class="org.springframework.web.accept.ContentNegotiationManagerFactoryBean">
    <property name="favorPathExtension" value="false" />
    <property name="favorParameter" value="true" />
    <property name="parameterName" value="mediaType" />
    <property name="ignoreAcceptHeader" value="true"/>
    <property name="useJaf" value="false"/>
    <property name="defaultContentType" value="application/json" />
 
    <property name="mediaTypes">
        <map>
            <entry key="json" value="application/json" />
            <entry key="xml" value="application/xml" />
       </map>
    </property>
</bean> 
```

What we did, in both cases:

- Disabled path extension. Note that favor does not mean use one approach in preference to another, it just enables or disables it. The order of checking is always path extension, parameter, Accept header.
- Enable the use of the URL parameter but instead of using the default parameter, `format`, we will use `mediaType` instead.
- Ignore the `Accept` header completely. This is often the best approach if most of your clients are actually web-browsers (typically making REST calls via AJAX).
- Don't use the JAF, instead specify the media type mappings manually - we only wish to support JSON and XML. 

### Listing User Accounts Example 

To return a list of accounts in JSON or XML, I need a Controller like this. We will ignore the HTML generating methods for now. 

```
@Controller
class AccountController {
    @RequestMapping(value="/accounts", method=RequestMethod.GET)
    @ResponseStatus(HttpStatus.OK)
    public @ResponseBody List<Account> list(Model model, Principal principal) {
        return accountManager.getAccounts(principal) );
    }

    // Other methods ...
} 
```

Here is the content-negotiation strategy setup: 

```
<!-- Simple strategy: only path extension is taken into account -->
	<bean id="cnManager"
		class="org.springframework.web.accept.ContentNegotiationManagerFactoryBean">
		<property name="favorPathExtension" value="true"/>
		<property name="ignoreAcceptHeader" value="true" />
		<property name="defaultContentType" value="text/html" />
		<property name="useJaf" value="false"/>

		<property name="mediaTypes">
			<map>
				<entry key="html" value="text/html" />
				<entry key="json" value="application/json" />
				<entry key="xml" value="application/xml" />
			</map>
		</property>
	</bean> 
```

Or, using Java Configuration, the code looks like this: 

```
	@Override
	public void configureContentNegotiation(
			ContentNegotiationConfigurer configurer) {
		// Simple strategy: only path extension is taken into account
		configurer.favorPathExtension(true).
			ignoreAcceptHeader(true).
			useJaf(false).
			defaultContentType(MediaType.TEXT_HTML).
			mediaType("html", MediaType.TEXT_HTML).
			mediaType("xml", MediaType.APPLICATION_XML).
			mediaType("json", MediaType.APPLICATION_JSON);
	} 
```

Provided I have JAXB2 and Jackson on my classpath, Spring MVC will automatically setup the necessary `HttpMessageConverters`. My domain classes must also be marked up with JAXB2 and Jackson annotations to enable conversion (otherwise the message converters don’t know what to do). In response to comments (below).

How does the system know whether to convert to XML or JSON? Because of content negotiation - any one of the three (*PPA Strategy*) options discussed above will be used depending on how the `ContentNegotiationManager` is configured. In this case the URL ends in `accounts.json` because the path-extension is the only strategy enabled.

In the sample code you can switch between XML or Java Configuration of MVC by setting an active profile in the `web.xml`. The profiles are “xml” and “javaconfig” respectively. 

 ### Combining Data and Presentation Formats 

Spring MVC’s REST support builds on the existing MVC Controller framework. So it is possible to have the same web-applications return information both as raw data (like JSON) and using a presentation format (like HTML).

Both techniques can easily be used side by side in the same controller, like this: 

```
@Controller
class AccountController {
    // RESTful method
    @RequestMapping(value="/accounts", produces={"application/xml", "application/json"})
    @ResponseStatus(HttpStatus.OK)
    public @ResponseBody List<Account> listWithMarshalling(Principal principal) {
        return accountManager.getAccounts(principal);
    }

    // View-based method
    @RequestMapping("/accounts")
    public String listWithView(Model model, Principal principal) {
        // Call RESTful method to avoid repeating account lookup logic
        model.addAttribute( listWithMarshalling(principal) );

        // Return the view to use for rendering the response
        return ¨accounts/list¨;
    }
} 
```

There is a simple Pattern here: the `@ResponseBody` method handles all data access and integration with the underlying service layer (the `AccountManager`). The second method calls the first and sets up the response in the Model for use by a View. This avoids duplicated logic.

To determine which of the two `@RequestMapping` methods to pick, we are again using our PPA content negotiation strategy. It allows the `produces` option to work. URLs ending with `accounts.xml` or `accounts.json` map to the first method, any other URLs ending in `accounts.anything` map to the second. 

### Another Approach 

Alternatively we could do the whole thing with just one method if we used views to generate all possible content-types. This is where the `ContentNegotiatingViewResolver` comes in and that will be the subject of my next [post](2017-08-02-share-Content-Negotiation-using-Views). 