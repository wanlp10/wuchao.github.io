---
layout: post
title: Spring 验证和国际化
category : [Spring]
tagline: "Supporting tagline"
tags : [Spring, i18n]
---
{% include JB/setup %}
# Spring 验证和国际化
--- 

<!--break-->  

### Validator 国际化

> [SpringMVC表单验证器](http://www.jianshu.com/p/b7fd2279e5cb)

在 ProductCommand 类中添加 name 属性的验证： 
``` 
@Size(min = 1, max = 10)
@Pattern(regexp = "^[\\u4e00-\\u9fa5]*$")
private String name;
``` 
当验证不通过时会产生默认的错误信息，如上面的验证分别会产生下面的两条默认错误信息： 
```  
个数必须在1和10之间 
需要匹配正则表达式"^[\u4e00-\u9fa5]*$" 
``` 
也可以在注解上面加上自定义的默认错误信息，例如在 @Pattern 注解上加上默认错误信息：
``` 
@Size(min = 1, max = 10)
@Pattern(regexp = "^[\\u4e00-\\u9fa5]*$", message = "产品名称格式不正确")
private String name;
``` 
这样如果 name 属性值不匹配正则的话就会显示：
```  
产品名称格式不正确
```
还可以将默认错误信息国际化：
```  
@Size(min = 1, max = 10)
@Pattern(regexp = "^[\\u4e00-\\u9fa5]*$", message = "{product.name.pattern}")
private String name;
``` 
在项目 resources 目录下新建类似`ValidationMessages_xx.properties` 文件名的国际化文件，如`ValidationMessages_zh_CN.properties`表示中文国际化文件。 在`ValidationMessages_zh_CN.properties` 文件中将`product.name.pattern` 配置相对应的错误信息： 
```  
# 产品名称格式不正确
product.name.pattern=\u4ea7\u54c1\u540d\u79f0\u683c\u5f0f\u4e0d\u6b63\u786e
```  
国际化文件里面要将中文转成Unicode（[Unicode编码转换](http://tool.chinaz.com/tools/unicode.aspx)），不然显示时会乱码。 

使用 SpringBoot，spring-boot-starter-web 会自动引入hiberante-validator, validation-api 依赖。

因为 Spring 的国际化默认是以 `messages` 开头的（org.springframework.boot.autoconfigure.context.MessageSourceAutoConfiguration），而上面的修改会将所有的国际化文件都指示以 `ValidationMessages` 开头，所以为了避免冲突，都以 `messages` 开头。

SpringBoot 默认的国际化文件的 `baseName` 是 `i18n/messages`，所以将验证信息的国际化放在 `i18n/messages` 里也可以。 

 

Spring 会根据验证的对象和字段生成规则的错误码（errorCode），然后去国际化文件中查找匹配，如果匹配到了就显示匹配的信息，如果没有匹配到，就显示默认的错误信息。
所以我们只要知道生成错误码的规则，就可以事先在国际化文件中配置好错误码的国际化，而不要在注解上面加默认错误信息。

规则参考： 

错误码的默认生成的错误信息是配置在  `Hibernate Validator` 中的，在国际化文件中可以替换成我们自己想要显示的文字，例如： 
``` 
# Hibernate Validator Default Messages.  
javax.validation.constraints.NotNull.message=\u4e0d\u80fd\u4e3a\u7a7a   

javax.validation.constraints.NotBlank.message=\u4e0d\u80fd\u4e3a\u7a7a  

javax.validation.constraints.NotEmpty.message=\u4e0d\u80fd\u4e3a\u7a7a   
```  

### 消息和文字国际化 
> [Spring Resource bundle with ResourceBundleMessageSource example](https://www.mkyong.com/spring/spring-resource-bundle-with-resourcebundlemessagesource-example/) 
> 
> [Spring Boot国际化（i18n）](http://blog.csdn.net/linxingliang/article/details/52350238) 

