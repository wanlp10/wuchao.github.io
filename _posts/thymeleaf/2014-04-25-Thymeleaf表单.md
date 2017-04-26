---
layout: post
title: Thymeleaf 表单
category : [JavaEE]
tagline: "Supporting tagline"
tags : [Thymeleaf]
---
{% include JB/setup %}
# Thymeleaf 表单
---

<!--break-->

## 表单提交
aaa

## 表单验证
Thymeleaf 的表单验证。

### thymelaef

### BeanValidation

### Validator


### 示例

#### index.html
```
<form id="addUser" role="form" class="form form-validate" novalidate="novalidate"
      th:method="post" th:action="@{/users}"
      th:object="${userCommand}">
    <div class="modal-body clearfix">
        <div class="col-md-10 col-md-offset-1">
            <div id="addUserNameInput" class="form-group floating-label"
                 th:fragment="username">
                <input type="text" class="form-control" id="username"
                       name="username"
                       th:field="*{username}">
                <label th:for="*{username}"
                       th:text="#{view.users.create.field.username}">用户名</label>
                <p class="help-block"
                   th:text="#{view.users.create.field.username.description}">
                    仅允许输入英文，数字以及下划线，长度为3-20个字符</p>
            </div>
            <div class="form-group floating-label" th:fragment="name">
                <input type="text" class="form-control" id="name"
                       name="name">
                <label th:for="*{name}"
                       th:text="#{view.users.create.field.name}">姓名</label>
                <p class="help-block"
                   th:text="#{view.users.create.field.name.description}">
                    长度为2-20个字符，如：张三</p>
            </div>
            <div class="form-group floating-label" th:fragment="password">
                <input type="password" class="form-control" id="mima"
                       onfocus="this.type='password'"
                       autocomplete="off"
                       th:field="*{password}"/>
                <label for="mima"
                       th:text="#{view.users.create.field.password}">密码</label>
                <p class="help-block"
                   th:text="#{view.users.create.field.password.description}">
                    长度为6-20个字符，由英文，数字以及特殊符号组成，区分大小写</p>
            </div>
            <div class="form-group floating-label" th:fragment="rePassword">
                <!--th:attrappend="class=${#fields.hasErrors('rePassword')?' has-error':''}">-->
                <input type="password" class="form-control" id="querenmima"
                       onfocus="this.type='password'"
                       autocomplete="off"
                       name="rePassword">
                <label for="querenmima"
                       th:text="#{view.users.create.field.rePassword}">确认密码</label>
                <!--<span th:if="${#fields.hasErrors('rePassword')}"
                      th:errors="*{rePassword}"
                      class="help-block"></span>-->
            </div>
            <div class="form-group clearfix" th:fragment="roleNames">
                <div class="col-sm-1 no-padding">
                    <label class="text-lg text-muted">角色</label>
                </div>
                <div class="col-sm-9">
                    <label class="radio-inline radio-styled col-sm-4"
                           th:each="group:${groups}">
                        <input type="radio" name="roleNames"
                               th:value="${group.name}"><span
                            th:text="${group.name}"></span>
                    </label>
                </div>
            </div>
            <div class="form-group clearfix" th:fragment="enabled">
                <div class="col-sm-1 no-padding">
                    <label class="text-lg text-muted">状态</label>
                </div>
                <div class="col-sm-9">
                    <label class="radio-inline radio-styled">
                        <input type="radio" th:field="*{enabled}"
                               value="0"><span
                            th:text="#{view.users.create.field.enabled.false}">禁用</span>
                    </label>
                    <label class="radio-inline radio-styled">
                        <input type="radio" th:field="*{enabled}"
                               value="1"><span
                            th:text="#{view.users.create.field.enabled.true}">启用</span>
                    </label>
                </div>
            </div>
        </div>
    </div>
    <div class="modal-footer">
        <button type="button" data-dismiss="modal"
                class="btn btn-default">&emsp;取消&emsp;</button>
        <button type="submit" class="btn btn-primary"
                th:text="#{view.users.create.button.create}">&emsp;确定&emsp;</button>
    </div>
</form>
```

#### UserCommand.java
```
@Getter
@Setter
public class UserCommand {

    private Long id;

    @Size(min = 3, max = 20)
    private String username;

    @Size(min = 2, max = 20)
    private String name;

    @Size(min = 6, max = 16)
    private String password;

    @Size(min = 6, max = 16)
    private String rePassword;

    @NotEmpty
    private List<String> roleNames;

    private boolean enabled;

}
```

#### UserValidator.java
```
@Component
public class UserValidator implements Validator {

    @Autowired
    private UserRepository userRepository;

    @Override
    public boolean supports(Class<?> clazz) {
        return UserCommand.class.equals(clazz);
    }

    @Override
    public void validate(Object target, Errors errors) {
        UserCommand user = (UserCommand) target;
        User newUser = userRepository.findByUsername(user.getUsername());
        if (newUser != null && !Objects.equals(newUser.getId(), user.getId())) {
            errors.rejectValue("username", "constraints.user.username.exists","用户名已存在");
        }
        if (user.getRePassword() != null && !user.getPassword().equals(user.getRePassword())) {
            errors.rejectValue("rePassword", "constraints.user.rePassword.notEqualsToPassword","两次密码输入不一致");
        }
    }
}
```

#### UserIndexController.java
```
@RequestMapping("/users")
public String index(@ModelAttribute UserCommand userCommand, Model model) {
    List<User> users = userRepository.findAllWithRoles();
    List<Group> groups = groupRepository.findAll();
    model.addAttribute("groups", groups);
    model.addAttribute("users", users);
    return "users/index";
}
```

#### UserCreateController.java
```
@Controller
@RequestMapping(value = "/users")
public class UserCreateController {

    @Autowired
    private UserValidator userValidator;

    @InitBinder
    protected void initBinder(DataBinder binder) {
        binder.addValidators(userValidator);
    }

    @RequestMapping(method = RequestMethod.POST)
    @ResponseBody
    public ResponseEntity create(@Valid UserCommand userCommand, BindingResult bindingResult) {
        if (bindingResult.hasErrors()) {
            return ResponseEntity.status(HttpStatus.BAD_REQUEST)
                    .contentType(MediaType.APPLICATION_JSON)
                    .body(bindingResult.getAllErrors());
        }

        ...

        return ResponseEntity.ok().build();
    }
}
```

#### 验证
![images_2017-4-25_15-13-40](../../../images/images_2017-4-25_15-13-40.png)

### FAQ
```
java.lang.IllegalStateException: Neither BindingResult nor plain target object for bean name 'userCommand' available as request attribute
```

错误发生在 `org.springframework.web.servlet.support.BindStatus` ：

```
...

else {
    // No BindingResult available as request attribute:
    // Probably forwarded directly to a form view.
    // Let's do the best we can: extract a plain target if appropriate.
    Object target = requestContext.getModelObject(beanName);
    if (target == null) {
        throw new IllegalStateException("Neither BindingResult nor plain target object for bean name '" +
                beanName + "' available as request attribute");
    }
    if (this.expression != null && !"*".equals(this.expression) && !this.expression.endsWith("*")) {
        BeanWrapper bw = PropertyAccessorFactory.forBeanPropertyAccess(target);
        this.value = bw.getPropertyValue(this.expression);
        this.valueType = bw.getPropertyType(this.expression);
        this.actualValue = this.value;
    }
    this.errorCodes = new String[0];
    this.errorMessages = new String[0];
}

...

```

页面传入的 BindingResult 对象没有初始化，在这里即找不到名为 userCommand 的 bean 对象。
解决办法是在进入页面前初始化一个名为 userCommand 的对象：

```
@RequestMapping("/users")
public String index(..., @ModelAttribute UserCommand userCommand) {
    ...

    return "users/index";
}
```

或

```
@RequestMapping("/users")
public String index(..., Model model) {
    ...

    model.addAttribute("userCommand", new UserCommand());
    return "users/index";
}
```




