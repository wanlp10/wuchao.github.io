---
layout: post
title: Spring Security 的学习和使用
category : [JavaEE, Spring]
tagline: "Supporting tagline"
tags : [Spring Security]
---
{% include JB/setup %}
# Spring Security 的学习和使用
---

> 参考：  
>
> [Spring Security Reference](http://docs.spring.io/spring-security/site/docs/current/reference/html/) 
>
> [Spring Security 系列博客 - Elim 的博客](http://elim.iteye.com/category/182468) 
>
> [mkyong.com - Spring Security Tutorial](http://www.mkyong.com/spring-security/spring-security-hibernate-annotation-example/) 
>
> [Spring Security 4 基于角色的登录例子（带源码）](http://blog.csdn.net/w605283073/article/details/51322771) 

Spring 是一个非常流行和成功的 Java 应用开发框架。Spring Security 基于 Spring 框架，提供了一套 Web 应用安全性的完整解决方案。一般来说，Web 应用的安全性包括用户认证（Authentication）和用户授权（Authorization）两个部分。用户认证指的是验证某个用户是否为系统中的合法主体，也就是说用户能否访问该系统。用户认证一般要求用户提供用户名和密码。系统通过校验用户名和密码来完成认证过程。用户授权指的是验证某个用户是否有权限执行某个操作。在一个系统中，不同用户所具有的权限是不同的。比如对一个文件来说，有的用户只能进行读取，而有的用户可以进行修改。一般来说，系统会为不同的用户分配不同的角色，而每个角色则对应一系列的权限。 



## Java Configuration 

``` 
import org.springframework.beans.factory.annotation.Autowired;

import org.springframework.context.annotation.*;
import org.springframework.security.config.annotation.authentication.builders.*;
import org.springframework.security.config.annotation.web.configuration.*;

@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
	
}  
```

上面代码是一个 Spring Security Java Configuration 类. 该配置类创建了一个叫做 `springspringSecurityFilterChain` 的 Servlet Filter,它负责管理整个应用的安全. 

Spring Security 提供了一个基类  `AbstractSecurityWebApplicationInitializer` 来注册该 `springSecurityFilterChain` . 注册的方法根据是否使用 Spring 而有所不同: 

#### AbstractSecurityWebApplicationInitializer without Existing Spring

``` 
import org.springframework.security.web.context.*;

public class SecurityWebApplicationInitializer
	extends AbstractSecurityWebApplicationInitializer {

	public SecurityWebApplicationInitializer() {
		super(WebSecurityConfig.class);
	}
} 
```

The `SecurityWebApplicationInitializer` will do the following things:

- Automatically register the springSecurityFilterChain Filter for every URL in your application
- Add a ContextLoaderListener that loads the [WebSecurityConfig](http://docs.spring.io/spring-security/site/docs/current/reference/html/jc.html#jc-hello-wsca). 

#### AbstractSecurityWebApplicationInitializer with Spring MVC

If we were using Spring elsewhere in our application we probably already had a `WebApplicationInitializer` that is loading our Spring Configuration. If we use the previous configuration we would get an error. Instead, we should register Spring Security with the existing `ApplicationContext`. For example, if we were using Spring MVC our `SecurityWebApplicationInitializer` would look something like the following:

```
import org.springframework.security.web.context.*;

public class SecurityWebApplicationInitializer
	extends AbstractSecurityWebApplicationInitializer {

}
```

This would simply only register the springSecurityFilterChain Filter for every URL in your application. After that we would ensure that `WebSecurityConfig` was loaded in our existing ApplicationInitializer. For example, if we were using Spring MVC it would be added in the `getRootConfigClasses()`

```
public class MvcWebApplicationInitializer extends
		AbstractAnnotationConfigDispatcherServletInitializer {

	@Override
	protected Class<?>[] getRootConfigClasses() {
		return new Class[] { WebSecurityConfig.class };
	}

	// ... other overrides ...
}
```



### HttpSecurity 

`WebSecurityConfigurerAdapter` 在 `configure(HttpSecurity http)` 方法中提供了一个默认的配置 : 

``` 
protected void configure(HttpSecurity http) throws Exception {
	http
		.authorizeRequests()
			.anyRequest().authenticated()
			.and()
		.formLogin()
			.and()
		.httpBasic();
} 
```

上面的默认配置: 

- Ensures that any request to our application requires the user to be authenticated
- Allows users to authenticate with form based login
- Allows users to authenticate with HTTP Basic authentication 

### Java Configuration and Form Login 



### Authorize Requests 

```
protected void configure(HttpSecurity http) throws Exception {
	http
		.authorizeRequests()                                                             1
			.antMatchers("/resources/**", "/signup", "/about").permitAll()               2
			.antMatchers("/admin/**").hasRole("ADMIN")                                   3
			.antMatchers("/db/**").access("hasRole('ADMIN') and hasRole('DBA')")         4
			.anyRequest().authenticated()                                                5
			.and()
		// ...
		.formLogin();
}
```

1: 按代码声明顺序执行 `http.authorizeRequests()`  下的 matcher. 

2: 所有用户可以访问以 "/resources/" 开头 , 等于 "/signup" 或者等于 "/about" 的 URL. 

3: 所有以 "/admin/" 开头的 URL 要求用户有 "ROLE_ADMIN" 权限, 因为调用了 hasRole 方法, 所以不需要制定 "ROLE_" 前缀.

4: 所有以 "/db/" 开头的 URL 要求用户有 "ROLE_ADMIN" 和 "DBA" 权限. 

5: 所有没有匹配的 URL 要求用户被验证. 

### Handling Logouts 

当使用 `WebSecurityConfigurerAdapter` 时, 会自动应用退出功能, 默认访问 `/logout` 时将退出登录: 

- 使 HTTP Session 失效 
- 清除所有配置的 RemeberMe 验证 
- 清除 `SecurityContextHolder` 
- 重定向到 `/login?logout`  

类似于配置 login, 这里也有也有很多选项让你更进一步的配置 logout: 

``` 
protected void configure(HttpSecurity http) throws Exception {
	http
		.logout()                                                                    1
			.logoutUrl("/my/logout")                                                 2
			.logoutSuccessUrl("/my/index")                                           3
			.logoutSuccessHandler(logoutSuccessHandler)                              4
			.invalidateHttpSession(true)                                             5
			.addLogoutHandler(logoutHandler)                                         6
			.deleteCookies(cookieNamesToClear)                                       7
			.and()
		...
}
```

- 提供 logout 功能支持, 当使用 `WebSecurityConfigurerAdapter` 时该功能自动被应用 
- 该 URL 触发 logout 功能 (默认是 /logout). 如果 CSRF 保护被启用了(默认启用), 该请求必须要是一个 POST 请求, 详细参考 [JavaDoc](http://docs.spring.io/spring-security/site/docs/current/apidocs/org/springframework/security/config/annotation/web/configurers/LogoutConfigurer.html#logoutUrl-java.lang.String-). 
- 该 URL 是 logout 执行后跳转的链接, 默认是 /login?logout. 
- 让你指定一个自定义的 `LogoutSuccessHandler` , 如果这个被指定了, logoutSuccessUrl() 就会被忽略. 
- 指定是否在 logout 时使 HttpSession 失效, 默认是 true. 
- 添加一个 `LogoutHandler` ,`SecurityContextLogoutHandler` is added as the last `LogoutHandler` by default.
- 允许指定在 logout 成功时将被清理的 cookies 的名称.This is a shortcut for adding a `CookieClearingLogoutHandler`explicitly. 

即自定义 logout 功能,你可以添加 `LogoutHandler` 和/或 `LogoutSuccessHandler` 实现类: 

#### LogoutHandler





#### LogoutSuccessHandler






### Authentication Configuration  

#### In-Memory Authentication 

``` 
@Bean
public UserDetailsService userDetailsService() throws Exception {
	InMemoryUserDetailsManager manager = new InMemoryUserDetailsManager();
	manager.createUser(User.withUsername("user").password("password").roles("USER").build());
	manager.createUser(User.withUsername("admin").password("password").roles("USER","ADMIN").build());
	return manager;
} 
```

#### JDBC Authentication 

下面是基于 JDBC 的验证配置, 假设你已经在应用中配置了 `DataSource` , [jdbc-javaconfig](https://github.com/spring-projects/spring-security/tree/master/samples/javaconfig/jdbc) 提供了一个完整的基于 JDBC 验证的例子. 

``` 
@Autowired
private DataSource dataSource;

@Autowired
public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
	auth
		.jdbcAuthentication()
			.dataSource(dataSource)
			.withDefaultSchema()
			.withUser("user").password("password").roles("USER").and()
			.withUser("admin").password("password").roles("USER", "ADMIN");
} 
```







#### AuthenticationProvider

你可以通过暴露一个自定义 `AuthenticationProvider` 类作为一个 bean 来自定义验证,例如,下面将自定义验证,假设 `SpringAuthenticationProvider` 实现了 `AuthenticationProvider` : 

> This is only used if the `AuthenticationManagerBuilder` has not been populated 

``` 
@Bean
public SpringAuthenticationProvider springAuthenticationProvider() {
	return new SpringAuthenticationProvider();
}
```

#### UserDetailsService 

你可以通过暴露一个自定义 `UserDetailsService` 类作为一个 bean 来自定义验证, 例如, 下面将自定义验证, 假设 `SpringDataUserDetailsService` 实现了 `UserDetailsService` : 

> This is only used if the `AuthenticationManagerBuilder` has not been populated and no `AuthenticationProviderBean` is defined. 

``` 
@Bean
public SpringDataUserDetailsService springDataUserDetailsService() {
	return new SpringDataUserDetailsService();
} 
```



### Multiple HttpSecurity 

我们可以配置多个 HttpSecurity 实例，好比我们有多个 `<http>` 块一样．主要是多次继承 `WebSecurityConfigurationAdapter` . 

```
@EnableWebSecurity
public class MultiHttpSecurityConfig {
	@Bean
	public UserDetailsService userDetailsService() throws Exception {
		InMemoryUserDetailsManager manager = new InMemoryUserDetailsManager();
		manager.createUser(User.withUsername("user").password("password").roles("USER").build());
		manager.createUser(User.withUsername("admin").password("password").roles("USER","ADMIN").build());
		return manager;
	}

	@Configuration
	@Order(1)                                                    	 1
	public static class ApiWebSecurityConfigurationAdapter extends WebSecurityConfigurerAdapter {
		protected void configure(HttpSecurity http) throws Exception {
			http
				.antMatcher("/api/**")                               2
				.authorizeRequests()
					.anyRequest().hasRole("ADMIN")
					.and()
				.httpBasic();
		}
	}

	@Configuration                                                  3  
	public static class FormLoginWebSecurityConfigurerAdapter extends WebSecurityConfigurerAdapter {

		@Override
		protected void configure(HttpSecurity http) throws Exception {
			http
				.authorizeRequests()
					.anyRequest().authenticated()
					.and()
				.formLogin();
		}
	}
} 
```

1: 创建一个 `WebSecurityConfigurerAdapter` 类, `@Order` 指定哪个 `WebSecurityConfigurerAdapter` 类被优先执行.  

2: `http.antMatcher`  声明了这个 `HttpSecurity` 只接受以 `/api/` 开头的 URL 请求. 

3: 创建另一个 `WebSecurityConfigurerAdapter` 类 ,如果请求的 URL 不是以 `/api/` 开头的就进入这个类. 这个类会在 `ApiWebSecurityConfigurationAdapter` 之后被执行, 因为它有一个 1 之后的 `@order` 值 (没有 `@Order` 注解默认最后被执行).  



### Method Security 

#### EnableGlobalMethodSecurity 

我们可以在 `@Configuration` 类上使用 `@EnableGlobalMethodSecurity` 注解启用基于注解的安全. 例如,下面的代码将会启用 Spring Security 的 `@Secured` 注解.  

``` 
@EnableGlobalMethodSecurity(securedEnabled = true)
public class MethodSecurityConfig {
// ...
} 
```

在一个类或接口上的方法上添加一个注解会相应的限制该方法的访问. Spring Security 的 native 注解支持位方法定义一个属性集合. 这些集合将会被传递到 AccessDecisionManager 做实际控制. 

``` 
public interface BankService {

@Secured("IS_AUTHENTICATED_ANONYMOUSLY")
public Account readAccount(Long id);

@Secured("IS_AUTHENTICATED_ANONYMOUSLY")
public Account[] findAccounts();

@Secured("ROLE_TELLER")
public Account post(Account account, double amount);
} 
```

启用支持 JSR-250 的注解: 

``` 
@EnableGlobalMethodSecurity(jsr250Enabled = true)
public class MethodSecurityConfig {
// ...
} 
```

这些标准的语法支持简单的基于角色的约束,但是弱于 Spring Security 的 native 注解.要使用基于表达式的语法,可以使用: 

``` 
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class MethodSecurityConfig {
// ...
} 
```

其等效于下面的 Java 代码: 

``` 
public interface BankService {

@PreAuthorize("isAnonymous()")
public Account readAccount(Long id);

@PreAuthorize("isAnonymous()")
public Account[] findAccounts();

@PreAuthorize("hasAuthority('ROLE_TELLER')")
public Account post(Account account, double amount);
} 
```



#### GlobalMethodSecurityConfiguration 

有时你可能需要执行一些比 `@EnableGlobalMethodSecurity` 注解允许的更复杂的操作. 对于这些类,你可以继承 `GlobalMethodSecurityConfiguration`  确保 `@EnableGlobalMethodSecurity` 注解在你的子类上. 例如, 如果你想提供一个自定义的 `MethodSecurityExpressionHandler` ,你可以使用下面的配置:  

``` 
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class MethodSecurityConfig extends GlobalMethodSecurityConfiguration {
	@Override
	protected MethodSecurityExpressionHandler createExpressionHandler() {
		// ... create and return custom MethodSecurityExpressionHandler ...
		return expressionHandler;
	}
} 
```







## Core Components

> 粘贴自 [](http://elim.iteye.com/blog/2155786) 

### Authentication 

Authentication 是一个接口，用来表示用户认证信息的，在用户登录认证之前相关信息会封装为一个 Authentication 具体实现类的对象，在登录认证成功之后又会生成一个信息更全面，包含用户权限等信息的 Authentication 对象，然后把它保存在 SecurityContextHolder 所持有的 SecurityContext 中，供后续的程序进行调用，如访问权限的鉴定等。 

### SecurityContextHolder 

SecurityContextHolder 是用来保存 SecurityContext 的。SecurityContext 中含有当前正在访问系统的用户的详细信息。默认情况下，SecurityContextHolder 将使用 ThreadLocal 来保存 SecurityContext，这也就意味着在处于同一线程中的方法中我们可以从 ThreadLocal 中获取到当前的 SecurityContext。因为线程池的原因，如果我们每次在请求完成后都将 ThreadLocal 进行清除的话，那么我们把 SecurityContext 存放在 ThreadLocal 中还是比较安全的。这些工作 Spring Security 已经自动为我们做了，即在每一次 request 结束后都将清除当前线程的 ThreadLocal。   

SecurityContextHolder 中定义了一系列的静态方法，而这些静态方法内部逻辑基本上都是通过 SecurityContextHolder 持有的 SecurityContextHolderStrategy 来实现的，如 getContext()、setContext()、clearContext() 等。而默认使用的 strategy 就是基于 ThreadLocal 的 ThreadLocalSecurityContextHolderStrategy。另外，Spring Security 还提供了两种类型的 strategy 实现，GlobalSecurityContextHolderStrategy 和 InheritableThreadLocalSecurityContextHolderStrategy，前者表示全局使用同一个 SecurityContext，如 C/S 结构的客户端；后者使用 InheritableThreadLocal 来存放 SecurityContext，即子线程可以使用父线程中存放的变量。  

一般而言，我们使用默认的 strategy 就可以了，但是如果要改变默认的 strategy，Spring Security 为我们提供了两种方法，这两种方式都是通过改变 strategyName 来实现的。SecurityContextHolder 中为三种不同类型的 strategy 分别命名为 MODE_THREADLOCAL、MODE_INHERITABLETHREADLOCAL 和 MODE_GLOBAL。第一种方式是通过 SecurityContextHolder 的静态方法 setStrategyName() 来指定需要使用的 strategy；第二种方式是通过系统属性进行指定，其中属性名默认为 “spring.security.strategy”，属性值为对应 strategy 的名称。 

使用全局唯一的 SecurityContextHolder:  

```
public void useGlobalSecurityContextHolder() { 
   SecurityContextHolder.setStrategyName(SecurityContextHolder.MODE_GLOBAL); 
} 
```

Spring Security 使用一个 Authentication 对象来描述当前用户的相关信息。SecurityContextHolder 中持有的是当前用户的 SecurityContext，而 SecurityContext 持有的是代表当前用户相关信息的 Authentication 的引用。这个 Authentication 对象不需要我们自己去创建，在与系统交互的过程中，Spring Security 会自动为我们创建相应的 Authentication 对象，然后赋值给当前的 SecurityContext。但是往往我们需要在程序中获取当前用户的相关信息，比如最常见的是获取当前登录用户的用户名。在程序的任何地方，通过如下方式我们可以获取到当前用户的用户名。 

``` 
Object principal = SecurityContextHolder.getContext().getAuthentication().getPrincipal();

if (principal instanceof UserDetails) {
String username = ((UserDetails)principal).getUsername();
} else {
String username = principal.toString();
}  
```

通过 Authentication.getPrincipal() 可以获取到代表当前用户的信息，这个对象通常是 UserDetails 的实例。获取当前用户的用户名是一种比较常见的需求，关于上述代码其实 Spring Security 在 Authentication 中的实现类中已经为我们做了相关实现，所以获取当前用户的用户名最简单的方式应当如下: 

``` 
public String getCurrentUsername() {
	return SecurityContextHolder.getContext().getAuthentication().getName();
}
```

此外，调用 SecurityContextHolder.getContext() 获取 SecurityContext 时，如果对应的 SecurityContext 不存在，则 Spring Security 将为我们建立一个空的 SecurityContext 并进行返回。 

### AuthenticationManager 和 AuthenticationProvider 

AuthenticationManager 是一个用来处理认证（Authentication）请求的接口。在其中只定义了一个方法 authenticate()，该方法只接收一个代表认证请求的 Authentication 对象作为参数，如果认证成功，则会返回一个封装了当前用户权限等信息的 Authentication 对象进行返回。

``` 
Authentication authenticate(Authentication authentication) throws AuthenticationException;  
```

在 Spring Security 中，AuthenticationManager 的默认实现是 ProviderManager，而且它不直接自己处理认证请求，而是委托给其所配置的 AuthenticationProvider 列表，然后会依次使用每一个 AuthenticationProvider 进行认证，如果有一个 AuthenticationProvider 认证后的结果不为 null，则表示该 AuthenticationProvider 已经认证成功，之后的 AuthenticationProvider 将不再继续认证。然后直接以该 AuthenticationProvider 的认证结果作为 ProviderManager 的认证结果。如果所有的 AuthenticationProvider 的认证结果都为 null，则表示认证失败，将抛出一个 ProviderNotFoundException。校验认证请求最常用的方法是根据请求的用户名加载对应的 UserDetails，然后比对 UserDetails 的密码与认证请求的密码是否一致，一致则表示认证通过。Spring Security 内部的 DaoAuthenticationProvider 就是使用的这种方式。其内部使用 UserDetailsService 来负责加载 UserDetails，UserDetailsService 将在下节讲解。在认证成功以后会使用加载的 UserDetails 来封装要返回的 Authentication 对象，加载的 UserDetails 对象是包含用户权限等信息的。认证成功返回的 Authentication 对象将会保存在当前的 SecurityContext 中。

当我们在使用 NameSpace 时， authentication-manager 元素的使用会使 Spring Security 在内部创建一个 ProviderManager，然后可以通过 authentication-provider 元素往其中添加 AuthenticationProvider。当定义 authentication-provider 元素时，如果没有通过 ref 属性指定关联哪个 AuthenticationProvider，Spring Security 默认就会使用 DaoAuthenticationProvider。使用了 NameSpace 后我们就不要再声明 ProviderManager 了。

``` 
 <security:authentication-manager alias="authenticationManager">
      <security:authentication-provider
         user-service-ref="userDetailsService"/>
 </security:authentication-manager> 
```

如果我们没有使用 NameSpace，那么我们就应该在 ApplicationContext 中声明一个 ProviderManager。 

默认情况下，在认证成功后 ProviderManager 将清除返回的 Authentication 中的凭证信息，如密码。所以如果你在无状态的应用中将返回的 Authentication 信息缓存起来了，那么以后你再利用缓存的信息去认证将会失败，因为它已经不存在密码这样的凭证信息了。所以在使用缓存的时候你应该考虑到这个问题。一种解决办法是设置 ProviderManager 的 eraseCredentialsAfterAuthentication 属性为 false，或者想办法在缓存时将凭证信息一起缓存。 

### UserDetailsService 

``` 
package org.springframework.security.core.userdetails;

public interface UserDetailsService { 
	UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
} 
```

通过 Authentication.getPrincipal() 的返回类型是 Object，但很多情况下其返回的其实是一个 UserDetails 的实例。UserDetails 是 Spring Security 中一个核心的接口。其中定义了一些可以获取用户名、密码、权限等与认证相关的信息的方法。Spring Security 内部使用的 UserDetails 实现类大都是内置的 User 类，我们如果要使用 UserDetails 时也可以直接使用该类。在 Spring Security 内部很多地方需要使用用户信息的时候基本上都是使用的 UserDetails，比如在登录认证的时候。登录认证的时候 Spring Security 会通过 UserDetailsService 的 loadUserByUsername() 方法获取对应的 UserDetails 进行认证，认证通过后会将该 UserDetails 赋给认证通过的 Authentication 的 principal，然后再把该 Authentication 存入到 SecurityContext 中。之后如果需要使用用户信息的时候就是通过 SecurityContextHolder 获取存放在 SecurityContext 中的 Authentication 的 principal。

通常我们需要在应用中获取当前用户的其它信息，如 Email、电话等。这时存放在 Authentication 的 principal 中只包含有认证相关信息的 UserDetails 对象可能就不能满足我们的要求了。这时我们可以实现自己的 UserDetails，在该实现类中我们可以定义一些获取用户其它信息的方法，这样将来我们就可以直接从当前 SecurityContext 的 Authentication 的 principal 中获取这些信息了。上文已经提到了 UserDetails 是通过 UserDetailsService 的 loadUserByUsername() 方法进行加载的。UserDetailsService 也是一个接口，我们也需要实现自己的 UserDetailsService 来加载我们自定义的 UserDetails 信息。然后把它指定给 AuthenticationProvider 即可。如下是一个配置 UserDetailsService 的示例。 

``` 
<!-- 用于认证的 AuthenticationManager -->
   <security:authentication-manager alias="authenticationManager">
      <security:authentication-provider
         user-service-ref="userDetailsService" />
   </security:authentication-manager>
 
   <bean id="userDetailsService"
      class="org.springframework.security.core.userdetails.jdbc.JdbcDaoImpl">
      <property name="dataSource" ref="dataSource" />
   </bean> 
```

上述代码中我们使用的 JdbcDaoImpl 是 Spring Security 为我们提供的 UserDetailsService 的实现，另外 Spring Security 还为我们提供了 UserDetailsService 另外一个实现，InMemoryDaoImpl。其作用是从数据库中加载 UserDetails 信息。其中已经定义好了加载相关信息的默认脚本，这些脚本也可以通过 JdbcDaoImpl 的相关属性进行指定。 

#### org.springframework.security.core.userdetails.jdbc.JdbcDaoImpl.java       

JdbcDaoImpl 允许我们从数据库来加载 UserDetails，其底层使用的是 Spring 的 JdbcTemplate 进行操作，所以我们需要给其指定一个数据源。 

使用 NameSpace 时，使用 jdbc-user-service 元素时在底层 Spring Security 默认使用的就是 JdbcDaoImpl: 

``` 
<security:authentication-manager alias="authenticationManager">
      <security:authentication-provider>
         <!-- 基于 Jdbc 的 UserDetailsService 实现，JdbcDaoImpl -->
         <security:jdbc-user-service data-source-ref="dataSource"/>
      </security:authentication-provider>
   </security:authentication-manager> 
```

JdbcDaoImpl.java 中重写了 loadUserByUsername(String username) 方法.

```
public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        List<UserDetails> users = this.loadUsersByUsername(username);
        if(users.size() == 0) {
            this.logger.debug("Query returned no results for user '" + username + "'");
            throw new UsernameNotFoundException(this.messages.getMessage("JdbcDaoImpl.notFound", new Object[]{username}, "Username {0} not found"));
        } else {
            UserDetails user = (UserDetails)users.get(0);
            Set<GrantedAuthority> dbAuthsSet = new HashSet();
            if(this.enableAuthorities) {
                dbAuthsSet.addAll(this.loadUserAuthorities(user.getUsername()));
            }

            if(this.enableGroups) {
                dbAuthsSet.addAll(this.loadGroupAuthorities(user.getUsername()));
            }

            List<GrantedAuthority> dbAuths = new ArrayList(dbAuthsSet);
            this.addCustomAuthorities(user.getUsername(), dbAuths);
            if(dbAuths.size() == 0) {
                this.logger.debug("User '" + username + "' has no authorities and will be treated as 'not found'");
                throw new UsernameNotFoundException(this.messages.getMessage("JdbcDaoImpl.noAuthority", new Object[]{username}, "User {0} has no GrantedAuthority"));
            } else {
                return this.createUserDetails(username, user, dbAuths);
            }
        }
    } 
```

loadUserByUsername() 方法分别调用了下面几个方法：

1. loadUsersByUsername(String username) ： 

```
protected List<UserDetails> loadUsersByUsername(String username) {
        return this.getJdbcTemplate().query(this.usersByUsernameQuery, new String[]{username}, new RowMapper<UserDetails>() {
            public UserDetails mapRow(ResultSet rs, int rowNum) throws SQLException {
                String username = rs.getString(1);
                String password = rs.getString(2);
                boolean enabled = rs.getBoolean(3);
                return new User(username, password, enabled, true, true, true, AuthorityUtils.NO_AUTHORITIES);
            }
        });
    } 
```

通过 usersByUsernameQuery 属性指定通过 username 查询用户信息的 SQL 语句。

2. loadUserAuthorities(String username) ：  

```
 protected List<GrantedAuthority> loadUserAuthorities(String username) {
        return this.getJdbcTemplate().query(this.authoritiesByUsernameQuery, new String[]{username}, new RowMapper<GrantedAuthority>() {
            public GrantedAuthority mapRow(ResultSet rs, int rowNum) throws SQLException {
                String roleName = JdbcDaoImpl.this.rolePrefix + rs.getString(2);
                return new SimpleGrantedAuthority(roleName);
            }
        });
    }
```

通过 authoritiesByUsernameQuery 属性指定通过 username 查询用户所拥有的权限。

3. loadGroupAuthorities(String username) ：  

```
 protected List<GrantedAuthority> loadGroupAuthorities(String username) {
        return this.getJdbcTemplate().query(this.groupAuthoritiesByUsernameQuery, new String[]{username}, new RowMapper<GrantedAuthority>() {
            public GrantedAuthority mapRow(ResultSet rs, int rowNum) throws SQLException {
                String roleName = JdbcDaoImpl.this.getRolePrefix() + rs.getString(3);
                return new SimpleGrantedAuthority(roleName);
            }
        });
    }
```

通过 groupAuthoritiesByUsernameQuery 属性指定根据 username 查询用户组权限（如果通过设置 JdbcDaoImpl 的 enableGroups 为 true 启用了用户组权限的支持）。 

上面的 usersByUsernameQuery 属性, authoritiesByUsernameQuery 属性和 groupAuthoritiesByUsernameQuery 属性分别对应下面三条默认的 SQL 语句, 当这些信息都没有指定时，将使用默认的 SQL 语句。

```
private String usersByUsernameQuery = "select username,password,enabled from users where username = ?"; 

private String authoritiesByUsernameQuery = "select username,authority from authorities where username = ?"; 

private String groupAuthoritiesByUsernameQuery = "select g.id, g.group_name, ga.authority from groups g, group_members gm, group_authorities ga where gm.username = ? and g.id = ga.group_id and g.id = gm.group_id"; 
```

> Spring Security 默认创建的表结构: [Security Database Schema](http://docs.spring.io/spring-security/site/docs/current/reference/html/appendix-schema.html) 。 

上面权限的启用和 sql 查询语句都是可以在 SecurityConfig 类（实现了 WebSecurityConfigurerAdapter 的配置类）中控制和配置的，sql 语句可以根据自己定义的安全实体来查询，但是 select 后面的查询属性的位置顺序要和上面一样，只能改变 where 子句, 例如： 

```
@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

	@Autowired
    private DataSource dataSource;

    @Override
    public void configure(WebSecurity webSecurity) {
        
    }

    @Override
    public void configure(HttpSecurity http) throws Exception {
       
    }

    /**
     * @param auth
     * @throws Exception
     */
    @Autowired
    public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
        JdbcUserDetailsManagerConfigurer<AuthenticationManagerBuilder> configurer = auth.jdbcAuthentication();
        configurer
                .dataSource(dataSource)
                .usersByUsernameQuery(
                        "select username, password, enabled from users where username = ?")
                .authoritiesByUsernameQuery(
                        "select gm.username, g.role_name "
                        + "from groups g, group_members gm, group_authorities ga "
                        + "where gm.username = ? " + "and g.id = ga.group_id "
                        + "and g.id = gm.group_id")
                .groupAuthoritiesByUsername(
                		"select g.id, g.group_name, ga.authority "
                        + "from groups g, group_members gm, group_authorities ga,authorities a "
                        + "where gm.username = ? " + "and g.id = ga.group_id "
                        + "and g.id = gm.group_id and a.authority = ga.authority and 			a.enabled=true");
    }            
  
}
```

#### InMemoryDaoImpl 

InMemoryDaoImpl 主要是测试用的，其只是简单的将用户信息保存在内存中。使用 NameSpace 时，使用 user-service 元素 Spring Security 底层使用的 UserDetailsService 就是 InMemoryDaoImpl。此时，我们可以简单的使用 user 元素来定义一个 UserDetails: 

``` 
<security:user-service>
      <security:user name="user" password="user" authorities="ROLE_USER"/>
   </security:user-service> 
```

如上配置表示我们定义了一个用户 user，其对应的密码为 user，拥有 ROLE_USER 的权限。此外，user-service 还支持通过 properties 文件来指定用户信息，如： 

``` 
 <security:user-service properties="/WEB-INF/config/users.properties"/> 
```

其中属性文件应遵循如下格式： 

``` 
username=password,grantedAuthority[,grantedAuthority][,enabled|disabled] 
```

所以，对应上面的配置文件，我们的 users.properties 文件的内容应该如下所示： 

``` 
#username=password,grantedAuthority[,grantedAuthority][,enabled|disabled]
user=user,ROLE_USER 
```

### GrantedAuthority

Authentication 的 getAuthorities() 可以返回当前 Authentication 对象拥有的权限，即当前用户拥有的权限。其返回值是一个 GrantedAuthority 类型的数组，每一个 GrantedAuthority 对象代表赋予给当前用户的一种权限。GrantedAuthority 是一个接口，其通常是通过 UserDetailsService 进行加载，然后赋予给 UserDetails 的。

GrantedAuthority 中只定义了一个 getAuthority() 方法，该方法返回一个字符串，表示对应权限的字符串表示，如果对应权限不能用字符串表示，则应当返回 null。 

Spring Security 针对 GrantedAuthority 有一个简单实现 SimpleGrantedAuthority。该类只是简单的接收一个表示权限的字符串。Spring Security 内部的所有 AuthenticationProvider 都是使用 SimpleGrantedAuthority 来封装 Authentication 对象。 



## Authentication

http://blog.sina.com.cn/s/blog_5c0522dd0101doey.html 

http://elim.iteye.com/blog/2156765 



当用户输入了用户名和密码之后，`UserDetailsService` 通过用户名找到对应的 `UserDetails` 对象，接着比较密码是否匹配。如果不匹配，则返回出错信息；如果匹配的话，说明用户认证成功，就创建一个` org.springframework.security.authentication.UsernamePasswordAuthenticationToken` 对象 ( `Authentication`  接口的实现类) ，并将生成的 Token 对象带入 `AuthenticationManager` ( `org.springframework.security.authentication.ProviderManager` ) 验证 :  

```
package org.springframework.security.web.authentication; 

import org.springframework.security.authentication.AuthenticationServiceException;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.web.util.matcher.AntPathRequestMatcher;
import org.springframework.util.Assert;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse; 

public class UsernamePasswordAuthenticationFilter extends
		AbstractAuthenticationProcessingFilter {
	
	public static final String SPRING_SECURITY_FORM_USERNAME_KEY = "username";
	public static final String SPRING_SECURITY_FORM_PASSWORD_KEY = "password";

	private String usernameParameter = SPRING_SECURITY_FORM_USERNAME_KEY;
	private String passwordParameter = SPRING_SECURITY_FORM_PASSWORD_KEY;
	private boolean postOnly = true;

	... 

	public Authentication attemptAuthentication(HttpServletRequest request,
			HttpServletResponse response) throws AuthenticationException {
		if (postOnly && !request.getMethod().equals("POST")) {
			throw new AuthenticationServiceException(
					"Authentication method not supported: " + request.getMethod());
		}

		String username = obtainUsername(request);
		String password = obtainPassword(request);

		if (username == null) {
			username = "";
		}

		if (password == null) {
			password = "";
		}

		username = username.trim();

		UsernamePasswordAuthenticationToken authRequest = new UsernamePasswordAuthenticationToken(
				username, password);

		// Allow subclasses to set the "details" property
		setDetails(request, authRequest);

		return this.getAuthenticationManager().authenticate(authRequest);
	} 
} 	
```

验证成功后返回一个封装了用户权限信息的  `Authentication` 对象  , 再通过调用 `SecurityContextHolder.getContext().setAuthentication()` 将该对象保存到当前的  `SecurityContext` :   

```
package org.springframework.security.web.authentication; 

public abstract class AbstractAuthenticationProcessingFilter extends GenericFilterBean implements ApplicationEventPublisherAware, MessageSourceAware {

	... 

    protected void successfulAuthentication(HttpServletRequest request, HttpServletResponse response, FilterChain chain, Authentication authResult) throws IOException, ServletException {
        if(this.logger.isDebugEnabled()) {
            this.logger.debug("Authentication success. Updating SecurityContextHolder to contain: " + authResult);
        }

        SecurityContextHolder.getContext().setAuthentication(authResult);
        this.rememberMeServices.loginSuccess(request, response, authResult);
        if(this.eventPublisher != null) {
            this.eventPublisher.publishEvent(new InteractiveAuthenticationSuccessEvent(authResult, this.getClass()));
        }

        this.successHandler.onAuthenticationSuccess(request, response, authResult);
    } 
    
    ... 
    
}    
```

跳转验证成功后的页面 (可自定义设置) :  

```
package org.springframework.security.web.authentication;

public abstract class AbstractAuthenticationTargetUrlRequestHandler { 

... 

protected void handle(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {
        String targetUrl = this.determineTargetUrl(request, response);
        if(response.isCommitted()) {
            this.logger.debug("Response has already been committed. Unable to redirect to " + targetUrl);
        } else {
            this.redirectStrategy.sendRedirect(request, response, targetUrl);
        }
    } 
    
... 

} 
```









#### 





## Password Encoding 



## Jackson Support 





## Thymeleaf 与 Spring Security 

http://elim.iteye.com/blog/2161056 