---
layout: post
title: Thymeleaf 页面表单验证
category : [学习问题记录]
tagline: "Supporting tagline"
tags : [Spring MVC]
---
{% include JB/setup %}
# Thymeleaf 页面表单验证

> [spring mvc 中使用 spring 的 validator](http://blog.csdn.net/shuwei003/article/details/7213662)    
>
> [springboot 使用校验框架 validation 校验](http://blog.csdn.net/u012373815/article/details/72049796) 
>
> https://stackoverflow.com/questions/3721122/spring-validation-with-valid 

https://docs.spring.io/spring/docs/current/spring-framework-reference/html/validation.html

https://www.mkyong.com/spring-mvc/combine-spring-validator-and-hibernate-validator/ 

<!--break-->

### 前台验证









### 后台验证 



#### Hibernate validator   

定义 command 对象,里面声明校验规则:  

ProductNewCommand.java: 

``` 
public class ProductNewCommand {
  	@Size(min = 1, max = 20, message = "error.product.name.size")
    @Pattern(regexp = "[0-9\\u4e00-\\u9fa5]+$", message = "{error.product.name.pattern}")
    private String name;

    @NotNull
    private ProductType type;
}
```

ProductNewController.java: 

``` 
@PostMapping("/products/new")
public String create(@Valid ProductNewCommand productNewCommand, BindingResult bindingResult, Model model) {
	if (bindingResult.hasErrors()) {
        model.addAttribute("productNewCommand", productNewCommand);
        return "products/new";
    }

    ... 

	return "redirect:/products/" + product.getId();
}
```

在 resources 目录下新建 "ValidationMessages.properties" 文件,文件中的中文要转成 Unicode 编码格式([Unicode 编码转换](http://tool.chinaz.com/tools/unicode.aspx)) . 

> 注意：名字必须为 “ValidationMessages.properties“ 因为 SpringBoot 自动读取 classpath 中的 ValidationMessages.properties 里的错误信息, 通过配置 LocalValidatorFactoryBean 的 messageSource 只能改变 springBoot 国际化访问的文件目录. 

resources/ValidationMessages.properties: 

``` 
error.product.name.size=\u6700\u591a\u53ea\u80fd\u586b\u5199\u0032\u0030\u4e2a\u5b57\u7b26\u0020   # 最多只能填写20个字符   
error.product.name.pattern=\u8bf7\u586b\u5199\u4e2d\u6587\u3001\u6570\u5b57   # 请填写中文、数字
```

另外还可以 Hibernate Validator 的默认错误提示信息. 

如 @NotNull 默认提示 "不能为null", 修改为 "不能为空": 

``` 
javax.validation.constraints.NotNull.message=\u4e0d\u80fd\u4e3a\u7a7a
javax.validation.constraints.NotBlank.message=\u4e0d\u80fd\u4e3a\u7a7a 
```

products/new.html: 

``` 
<form th:action="@{/products/new}" method="post" th:object="${productNewCommand}">
	<div class="form-group">
      <span class="control-label">产品名称：</span>
      <input type="text" class="form-control"
      id="name" name="name" th:value="*{name}" 
      placeholder="请填写产品名称"/>
      <label class="text-danger" 
      th:if="*{#fields.hasErrors('name')}"
      th:text="*{#fields.errors('name')[0]}"></label>
    </div>
</form>
```



### Spring Validator

ProductNewValidator.java: 

    @Component 
    public class ProductNewValidator implements org.springframework.validation.Validator {
     @Override
      public boolean supports(Class<?> clazz) {
          return productNewCommand.class.equals(clazz);
      }
    
      @Override
      public void validate(Object o, Errors errors) {
    
      }
    }
ProductNewController.java: 

```
@Autowired
private ProductNewValidator productNewValidator;

@InitBinder
public void initBinder(WebDataBinder binder)　{
	// binder.setValidator(productNewValidator);
	// 使用 setValidator 方法后只会用 productNewValidator 中的验证,而跳过其他验证
	// 而使用 addValidators 方法后 Hibernate Validator 和 Spring Validator 会同时起作用
	binder.addValidators(productNewValidator);
}

@PostMapping("/products/new")
public String create(@Valid ProductNewCommand productNewCommand, BindingResult bindingResult, Model model) {
	if (bindingResult.hasErrors()) {
        model.addAttribute("productNewCommand", productNewCommand);
        return "products/new";
    }

    ... 

	return "redirect:/products/" + product.getId();
}
```





### FAQ 

1. Hibernate Validator 注解验证时, 在只能填写数字型的表单中填写了中文或英文字符: 

``` 
public class OrderCommand {
  
  @NotNull
  @Digits(integer = 12, fraction = 2)
  private Double price;
}
```


``` 
Failed to convert property value of type java.lang.String to required type java.lang.Double for property price; nested exception is java.lang.NumberFormatException: For input string: "xxx"
```

导入 Hibernate-Search-Engine 依赖: 

``` 
compile "org.hibernate:hibernate-search-engine:5.7.1.Final"
```

在数字型属性上加上以下注释: 

``` 
public class OrderCommand {
  
  @NotNull
  @Field(store = Store.YES, index = Index.YES, analyze = Analyze.NO)
  @NumericField
  @Digits(integer = 12, fraction = 2)
  private Double price;
}
```

这时 @Valid 的 BindingResult 中会报以下错误: 

``` 
Field error in object 'OrderCommand' on field 'price': rejected value [xxx]; codes [typeMismatch.OrderCommand.price,typeMismatch.price,typeMismatch.java.lang.Double,typeMismatch]; arguments [org.springframework.context.support.DefaultMessageSourceResolvable: codes [OrderCommand.price,price]; arguments []; default message [price]]; default message [Failed to convert property value of type 'java.lang.String' to required type 'java.lang.Double' for property 'price'; nested exception is java.lang.NumberFormatException: For input string: "xxx"]
```

在 message.properties 中定义错误信息: 

``` 
typeMismatch.OrderCommand.price=\u53ea\u80fd\u586b\u5199\u6570\u5b57  # 只能填写数字
```





2. 使用自定义 Validator 时报: 

``` 
Resolved exception caused by Handler execution: java.lang.IllegalStateException: Invalid target for Validator ... 
```
> https://stackoverflow.com/questions/14533488/adding-multiple-validators-using-initbinder 

