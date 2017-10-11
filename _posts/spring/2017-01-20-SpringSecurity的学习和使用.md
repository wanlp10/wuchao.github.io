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

> 转载自
>
> [Elim 的博客](http://elim.iteye.com/category/182468)  
>
> 参考：  
>
> [Spring Security Reference](http://docs.spring.io/spring-security/site/docs/current/reference/html/) 
>
> [mkyong.com - Spring Security Tutorial](http://www.mkyong.com/spring-security/spring-security-hibernate-annotation-example/) 
>

Spring 是一个非常流行和成功的 Java 应用开发框架。Spring Security 基于 Spring 框架，提供了一套 Web 应用安全性的完整解决方案。一般来说，Web 应用的安全性包括用户认证（Authentication）和用户授权（Authorization）两个部分。用户认证指的是验证某个用户是否为系统中的合法主体，也就是说用户能否访问该系统。用户认证一般要求用户提供用户名和密码。系统通过校验用户名和密码来完成认证过程。用户授权指的是验证某个用户是否有权限执行某个操作。在一个系统中，不同用户所具有的权限是不同的。比如对一个文件来说，有的用户只能进行读取，而有的用户可以进行修改。一般来说，系统会为不同的用户分配不同的角色，而每个角色则对应一系列的权限。 



<!--break-->  



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

上面代码是一个 Spring Security Java Configuration 类，该配置类创建了一个叫做 `springspringSecurityFilterChain` 的 Servlet Filter，它负责管理整个应用的安全。

Spring Security 提供了一个基类  `AbstractSecurityWebApplicationInitializer` 来注册该 `springSecurityFilterChain`。 

### HttpSecurity 

`WebSecurityConfigurerAdapter` 在 `configure(HttpSecurity http)` 方法中提供了一个默认的配置：

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

上面的默认配置：

- 确保对我们的应用程序的任何请求都要求用户进行身份验证。
- 允许用户使用基于表单的登录进行身份验证，formLogin() 方法里面也可以传递一个登录页 URL。
- 允许用户使用HTTP Basic身份验证进行身份验证。 

上面的 Java Config 等价于下面的 XML 命名空间配置：

``` 
<http>
	<intercept-url pattern="/**" access="authenticated"/>
	<form-login />
	<http-basic />
</http> 
```

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

1：按代码声明顺序执行 `http.authorizeRequests()`  下声明的每一个的 matcher 。

2：所有用户可以访问以 "/resources/" 开头 , 等于 "/signup" 或者等于 "/about" 的 URL 而不需要验证 。

3：访问所有以 "/admin/" 开头的 URL 要求用户有 "ROLE_ADMIN" 权限, 因为调用了 hasRole 方法，所以不需要指定 "ROLE_" 前缀 。

4：访问所有以 "/db/" 开头的 URL 要求用户有 "ROLE_ADMIN" 和 "DBA" 权限 。

5：访问所有没有匹配的 URL 要求用户被验证 。

### Handling Logouts 

当使用 `WebSecurityConfigurerAdapter` 时, 就自动拥有退出登录功能, 默认访问 `/logout` 时将退出登录 : 

- 使 HTTP Session 失效 。
- 清除所有配置的 RemeberMe 验证 。 
- 清除 `SecurityContextHolder` 。 
- 重定向到 `/login?logout`  。

类似于配置 login, 这里也有很多选项让你更进一步的配置 logout: 

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

- 提供 logout 支持，当使用 `WebSecurityConfigurerAdapter` 时该功能自动被应用。
- 该 URL  (默认是 /logout) 触发 logout。如果 CSRF 保护被启用了(默认启用)，该请求必须要是一个 POST 请求。
- 该 URL 是 logout 执行后跳转的链接，默认是 /login?logout。
- 让你指定一个自定义的 `LogoutSuccessHandler` , 如果被指定了, logoutSuccessUrl() 就会被忽略 。
- 指定是否在 logout 时使 HttpSession 失效，默认是 true。
- 添加一个 `LogoutHandler` ，默认会在 `LogoutHandler` 最后添加一个 `SecurityContextLogoutHandler` 。
- 允许指定在 logout 成功时被清理的 cookies 的名称。这是添加 `CookieClearingLogoutHandler`的一种快捷方式。

通常，要想自定义 logout 功能，你可以添加 `LogoutHandler` 和/或 `LogoutSuccessHandler` 实现类就可以了。 

#### LogoutHandler 

通常，`LogoutHandler` 实现类指示该类会参与到 logout 的处理过程中，这些实现类会被调用来处理必要的清理工作。Spring Security 提供的实现类：

- [PersistentTokenBasedRememberMeServices](http://docs.spring.io/spring-security/site/docs/current/apidocs/org/springframework/security/web/authentication/rememberme/PersistentTokenBasedRememberMeServices.html)
- [TokenBasedRememberMeServices](http://docs.spring.io/spring-security/site/docs/current/apidocs/org/springframework/security/web/authentication/rememberme/TokenBasedRememberMeServices.html)
- [CookieClearingLogoutHandler](http://docs.spring.io/spring-security/site/docs/current/apidocs/org/springframework/security/web/authentication/logout/CookieClearingLogoutHandler.html)
- [CsrfLogoutHandler](http://docs.spring.io/spring-security/site/docs/current/apidocs/org/springframework/security/web/csrf/CsrfLogoutHandler.html)
- [SecurityContextLogoutHandler](http://docs.spring.io/spring-security/site/docs/current/apidocs/org/springframework/security/web/authentication/logout/SecurityContextLogoutHandler.html) 

我们不需要直接提供 `LogoutHandler` 的实现类，fluent API 为每一个 `LogoutHandler` 实现类各自提供了一个快捷的入口，例如 `deleteCookies()` 方法允许指定一个多多个在退出成功要清理的 cookie 的名称。

#### LogoutSuccessHandler 

`LogoutSuccessHandler` 是在成功退出后由 `LogoutFilter` 调用的，里面处理重定向或转发到相应的目的地址等逻辑。Spring Security 提供了下面两个实现类： 

- [SimpleUrlLogoutSuccessHandler](http://docs.spring.io/spring-security/site/docs/current/apidocs/org/springframework/security/web/authentication/logout/SimpleUrlLogoutSuccessHandler.html)
- HttpStatusReturningLogoutSuccessHandler 

自定义 `LogoutSuccessHandler` 

> [Spring Security 4 基于角色的登录例子（带源码）](http://blog.csdn.net/w605283073/article/details/51322771) 

``` 
@Component
    public class CustomAuthenticationSuccessHandler extends SimpleUrlAuthenticationSuccessHandler {

        private RedirectStrategy redirectStrategy = new DefaultRedirectStrategy();

        private String targetUrl = "/home";

        @Override
        protected void handle(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {
            String targetUrl = determineTargetUrl(request, response);
            if (response.isCommitted()) {
                this.logger.debug("Response has already been committed. Unable to redirect to " + targetUrl);
            } else {
                this.redirectStrategy.sendRedirect(request, response, targetUrl);
            }
        }

        @Override
        protected String determineTargetUrl(HttpServletRequest request, HttpServletResponse response) {
            //:todo
            return targetUrl;
        }
    } 
```


### Authentication Configuration  

#### In-Memory Authentication 

``` 
@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
	@Autowired
	public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
          InMemoryUserDetailsManagerConfigurer<AuthenticationManagerBuilder> configurer = auth.inMemoryAuthentication();
          configurer.withUser("administrator").password("123456").roles("ADMIN").authorities("/admins/**");
          configurer.withUser("user").password("123456").roles("USER");
    }
}
```

#### JDBC Authentication  

> [Spring Security form login using database](http://www.mkyong.com/spring-security/spring-security-form-login-using-database/)  

下面是基于 JDBC 的验证配置, 假设你已经在应用中配置了 `DataSource` 。 

``` 
@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {	
	@Autowired
	private DataSource dataSource;

	@Autowired
	public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
	//        auth
	//                .jdbcAuthentication()
	//                    .dataSource(dataSource)
	//                    .withDefaultSchema()
	//                    .withUser("user").password("password").roles("USER").and()
	//                    .withUser("admin").password("password").roles("USER", "ADMIN"); 
			 auth
              		.jdbcAuthentication()
                    	.dataSource(dataSource)
                    	.usersByUsernameQuery( "select username, password, enabled from users where username = ?")
                    	.authoritiesByUsernameQuery( "select username, authority from authorities where username = ?")
	//                  .groupAuthoritiesByUsername("")
                    	.passwordEncoder(passwordEncoder());       	

	} 

	@Bean
    public PasswordEncoder passwordEncoder() {
        PasswordEncoder passwordEncoder = new BCryptPasswordEncoder();
        return passwordEncoder;
    }
}
```

基于 Jdbc 的自定义验证参考：[Spring Security + Hibernate Annotation Example](http://www.mkyong.com/spring-security/spring-security-hibernate-annotation-example/)  。 

### Multiple HttpSecurity 

我们可以配置多个 HttpSecurity 实例，好比我们有多个 `<http>` 块一样，主要是多次继承 `WebSecurityConfigurationAdapter`。

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

1：创建一个 `WebSecurityConfigurerAdapter` 类， `@Order` 注解指定哪个 `WebSecurityConfigurerAdapter` 类被优先执行。

2： `http.antMatcher`  声明了这个 `HttpSecurity` 只接受以 `/api/` 开头的 URL 请求。

3：创建另一个 `WebSecurityConfigurerAdapter` 类，如果请求的 URL 不是以 `/api/` 开头的就进入这个类，这个类会在 `ApiWebSecurityConfigurationAdapter` 之后被执行，因为它有一个 1 之后的 `@order` 值 (没有 `@Order` 注解默认最后被执行)。 



## Core Components

> 转载自 [Elim 的博客](http://elim.iteye.com/blog/2155786)  

### Authentication 

Authentication 是一个接口，用来表示用户认证信息的，在用户登录认证之前相关信息会封装为一个 Authentication 具体实现类的对象，在登录认证成功之后又会生成一个信息更全面，包含用户权限等信息的 Authentication 对象，然后把它保存在 SecurityContextHolder 所持有的 SecurityContext 中，供后续的程序进行调用，如访问权限的鉴定等。 

### SecurityContext 

`org.springframework.security.core.context.SecurityContext` 接口表示的是当前应用的安全上下文。通过此接口可以获取和设置当前的 Authentication 认证对象。通过认证对象的方法可以判断当前用户是否已经通过认证，以及获取当前认证用户的相关信息，包括用户名、密码和权限等。要使用此认证对象，首先需要获取到 `SecurityContext` 对象。通过 `org.springframework.security.core.context.SecurityContextHolder` 类提供的静态方法 `getContext()` 就可以获取，再通过 `SecurityContext`对象的 `getAuthentication()` 就可以得到认证对象。

### SecurityContextHolder 

SecurityContextHolder 是用来保存 SecurityContext 的。SecurityContext 中含有当前正在访问系统的用户的详细信息。默认情况下，SecurityContextHolder 将使用 ThreadLocal 来保存 SecurityContext，这也就意味着在处于同一线程中的方法中我们可以从 ThreadLocal 中获取到当前的 SecurityContext。因为线程池的原因，如果我们每次在请求完成后都将 ThreadLocal 进行清除的话，那么我们把 SecurityContext 存放在 ThreadLocal 中还是比较安全的。这些工作 Spring Security 已经自动为我们做了，即在每一次 request 结束后都将清除当前线程的 ThreadLocal。   

既然 SecurityContext 是存放在 ThreadLocal 中的，而且在每次权限鉴定的时候都是从 ThreadLocal 中获取 SecurityContext 中对应的 Authentication 所拥有的权限，并且不同的 request 是不同的线程，为什么每次都可以从 ThreadLocal 中获取到当前用户对应的 SecurityContext 呢？在 Web 应用中这是通过 SecurityContextPersistentFilter 实现的，默认情况下其会在每次请求开始的时候从 session 中获取 SecurityContext，然后把它设置给 SecurityContextHolder，在请求结束后又会将 SecurityContextHolder 所持有的 SecurityContext 保存在 session 中，并且清除 SecurityContextHolder 所持有的 SecurityContext。这样当我们第一次访问系统的时候，SecurityContextHolder 所持有的 SecurityContext 肯定是空的，待我们登录成功后，SecurityContextHolder 所持有的 SecurityContext 就不是空的了，且包含有认证成功的 Authentication 对象，待请求结束后我们就会将 SecurityContext 存在 session 中，等到下次请求的时候就可以从 session 中获取到该 SecurityContext 并把它赋予给 SecurityContextHolder 了，由于 SecurityContextHolder 已经持有认证过的 Authentication 对象了，所以下次访问的时候也就不再需要进行登录认证了。 

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



## Web 应用认证过程 

当用户输入了用户名和密码之后，Spring Security 将获取到的用户名和密码封装成一个 ` org.springframework.security.authentication.UsernamePasswordAuthenticationToken` 对象 ( `Authentication`  接口的实现类) ，并将生成的 Token 对象带入 `AuthenticationManager` ( `org.springframework.security.authentication.ProviderManager` ) 验证，AuthenticationManager 实际是 ProviderManager，ProviderManager 实现了 AuthenticationManager 接口。将认证的工作交给 AuthenticationProvider，AuthenticationManager 都会包含多个 AuthenticationProvider 对象，有任何一个 AuthenticationProvider 验证通过，都属于认证通过。  

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



## Filter 

Spring Security 的底层是通过一系列的 Filter 来管理的，每个 Filter 都有其自身的功能，而且各个 Filter 在功能上还有关联关系，所以它们的顺序也是非常重要的。

Spring Security 已经定义了一些 Filter，不管实际应用中你用到了哪些，它们应当保持如下顺序。

- ChannelProcessingFilter，如果你访问的 channel 错了，那首先就会在 channel 之间进行跳转，如 http 变为 https。
- SecurityContextPersistenceFilter，这样的话在一开始进行 request 的时候就可以在 SecurityContextHolder 中建立一个 SecurityContext，然后在请求结束的时候，任何对 SecurityContext 的改变都可以被 copy 到 HttpSession。
- ConcurrentSessionFilter，因为它需要使用 SecurityContextHolder 的功能，而且更新对应 session 的最后更新时间，以及通过 SessionRegistry 获取当前的 SessionInformation 以检查当前的 session 是否已经过期，过期则会调用 LogoutHandler。
- 认证处理机制，如 UsernamePasswordAuthenticationFilter，CasAuthenticationFilter，BasicAuthenticationFilter 等，以至于 SecurityContextHolder 可以被更新为包含一个有效的 Authentication 请求。
- SecurityContextHolderAwareRequestFilter，它将会把 HttpServletRequest 封装成一个继承自 HttpServletRequestWrapper 的 SecurityContextHolderAwareRequestWrapper，同时使用 SecurityContext 实现了 HttpServletRequest 中与安全相关的方法。
- JaasApiIntegrationFilter，如果 SecurityContextHolder 中拥有的 Authentication 是一个 JaasAuthenticationToken，那么该 Filter 将使用包含在 JaasAuthenticationToken 中的 Subject 继续执行 FilterChain。
- RememberMeAuthenticationFilter，如果之前的认证处理机制没有更新 SecurityContextHolder，并且用户请求包含了一个 Remember-Me 对应的 cookie，那么一个对应的 Authentication 将会设给 SecurityContextHolder。
- AnonymousAuthenticationFilter，如果之前的认证机制都没有更新 SecurityContextHolder 拥有的 Authentication，那么一个 AnonymousAuthenticationToken 将会设给 SecurityContextHolder。
- ExceptionTransactionFilter，用于处理在 FilterChain 范围内抛出的 AccessDeniedException 和 AuthenticationException，并把它们转换为对应的 Http 错误码返回或者对应的页面。
- FilterSecurityInterceptor，保护 Web URI，并且在访问被拒绝时抛出异常。



当我们在使用 NameSpace 时，Spring Security 是会自动为我们建立对应的 FilterChain 以及其中的 Filter。但有时我们可能需要添加我们自己的 Filter 到 FilterChain，又或者是因为某些特性需要自己显示的定义 Spring Security 已经为我们提供好的 Filter，然后再把它们添加到 FilterChain。使用 NameSpace 时添加 Filter 到 FilterChain 是通过 http 元素下的 custom-filter 元素来定义的。定义 custom-filter 时需要我们通过 ref 属性指定其对应关联的是哪个 Filter，此外还需要通过 position、before 或者 after 指定该 Filter 放置的位置。如上面提到的那样，Spring Security 对 FilterChain 中 Filter 顺序是有严格的规定的。Spring Security 对那些内置的 Filter 都指定了一个别名，同时指定了它们的位置。我们在定义 custom-filter 的 position、before 和 after 时使用的值就是对应着这些别名所处的位置。如 position=”CAS_FILTER” 就表示将定义的 Filter 放在 CAS_FILTER 对应的那个位置，before=”CAS_FILTER” 就表示将定义的 Filter 放在 CAS_FILTER 之前，after=”CAS_FILTER” 就表示将定义的 Filter 放在 CAS_FILTER 之后。此外还有两个特殊的位置可以指定，FIRST 和 LAST，分别对应第一个和最后一个 Filter，如你想把定义好的 Filter 放在最后，则可以使用 after=”LAST”。

下面按顺序展示 Spring Security 给我们定义好的 FilterChain 中 Filter 对应的位置顺序、它们的别名以及将触发自动添加到 FilterChain 的元素或属性定义 ：  

| **别名**                       | **Filter 类**                             | **对应元素或属性**                              |
| ---------------------------- | ---------------------------------------- | ---------------------------------------- |
| CHANNEL_FILTER               | ChannelProcessingFilter                  | http/intercept-url@requires-channel      |
| SECURITY_CONTEXT_FILTER      | SecurityContextPersistenceFilter         | http                                     |
| CONCURRENT_SESSION_FILTER    | ConcurrentSessionFilter                  | http/session-management/concurrency-control |
| LOGOUT_FILTER                | LogoutFilter                             | http/logout                              |
| X509_FILTER                  | X509AuthenticationFilter                 | http/x509                                |
| PRE_AUTH_FILTER              | AstractPreAuthenticatedProcessingFilter 的子类 | 无                                        |
| CAS_FILTER                   | CasAuthenticationFilter                  | 无                                        |
| FORM_LOGIN_FILTER            | UsernamePasswordAuthenticationFilter     | http/form-login                          |
| BASIC_AUTH_FILTER            | BasicAuthenticationFilter                | http/http-basic                          |
| SERVLET_API_SUPPORT_FILTER   | SecurityContextHolderAwareRequestFilter  | http@servlet-api-provision               |
| JAAS_API_SUPPORT_FILTER      | JaasApiIntegrationFilter                 | http@jaas-api-provision                  |
| REMEMBER_ME_FILTER           | RememberMeAuthenticationFilter           | http/remember-me                         |
| ANONYMOUS_FILTER             | AnonymousAuthenticationFilter            | http/anonymous                           |
| SESSION_MANAGEMENT_FILTER    | SessionManagementFilter                  | http/session-management                  |
| EXCEPTION_TRANSLATION_FILTER | ExceptionTranslationFilter               | http                                     |
| FILTER_SECURITY_INTERCEPTOR  | FilterSecurityInterceptor                | http                                     |
| SWITCH_USER_FILTER           | SwitchUserFilter                         | 无                                        |

### DelegatingFilterProxy 

上面这些 Filter 不需要手动在 web.xml 文件中一个个配置，只需要配置下面这一个 Filter 就行了。

``` 
<filter>
	<filter-name>springSecurityFilterChain</filter-name>
	<filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
</filter>
<filter-mapping>
	<filter-name>springSecurityFilterChain</filter-name>
	<url-pattern>/*</url-pattern>
</filter-mapping>  
```

配置中 DelegatingFilterProxy 是 Spring 中定义的一个 Filter 实现类，其作用是代理真正的 Filter 实现类，也就是说在调用 DelegatingFilterProxy 的 doFilter() 方法时实际上调用的是其代理 Filter 的 doFilter() 方法。其代理 Filter 必须是一个 Spring bean 对象，所以使用 DelegatingFilterProxy 的好处就是其代理 Filter 类可以使用 Spring 的依赖注入机制方便自由的使用 ApplicationContext 中的 bean。那么 DelegatingFilterProxy 如何知道其所代理的 Filter 是哪个呢？这是通过其自身的一个叫 targetBeanName 的属性来确定的，通过该名称，DelegatingFilterProxy 可以从 WebApplicationContext 中获取指定的 bean 作为代理对象。该属性可以通过在 web.xml 中定义 DelegatingFilterProxy 时通过 init-param 来指定，如果未指定的话将默认取其在 web.xml 中声明时定义的名称。在上述配置中，DelegatingFilterProxy 代理的就是名为 SpringSecurityFilterChain 的 Filter。需要注意的是被代理的 Filter 的初始化方法 init() 和销毁方法 destroy() 默认是不会被执行的。通过设置 DelegatingFilterProxy 的 targetFilterLifecycle 属性为 true，可以使被代理 Filter 与 DelegatingFilterProxy 具有同样的生命周期。 

### FilterChainProxy 

Spring Security 底层是通过一系列的 Filter 来工作的，每个 Filter 都有其各自的功能，而且各个 Filter 之间还有关联关系，所以它们的组合顺序也是非常重要的。

使用 Spring Security 时，DelegatingFilterProxy 代理的就是一个 FilterChainProxy。一个 FilterChainProxy 中可以包含有多个 FilterChain，但是某个请求只会对应一个 FilterChain，而一个 FilterChain 中又可以包含有多个 Filter。当我们使用基于 Spring Security 的 NameSpace 进行配置时，系统会自动为我们注册一个名为 springSecurityFilterChain 类型为 FilterChainProxy 的 bean（这也是为什么我们在使用 SpringSecurity 时需要在 web.xml 中声明一个 name 为 springSecurityFilterChain 类型为 DelegatingFilterProxy 的 Filter 了。），而且每一个 http 元素的定义都将拥有自己的 FilterChain，而 FilterChain 中所拥有的 Filter 则会根据定义的服务自动增减。所以我们不需要显示的再定义这些 Filter 对应的 bean 了，除非你想实现自己的逻辑，又或者你想定义的某个属性 NameSpace 没有提供对应支持等。

Spring security 允许我们在配置文件中配置多个 http 元素，以针对不同形式的 URL 使用不同的安全控制。Spring Security 将会为每一个 http 元素创建对应的 FilterChain，同时按照它们的声明顺序加入到 FilterChainProxy。所以当我们同时定义多个 http 元素时要确保将更具有特性的 URL 配置在前。

### Spring Security 定义好的核心 Filter 

通过前面的介绍我们知道 Spring Security 是通过 Filter 来工作的，为保证 Spring Security 的顺利运行，其内部实现了一系列的 Filter。这其中有几个是在使用 Spring Security 的 Web 应用中必定会用到的。接下来我们来简要的介绍一下 FilterSecurityInterceptor、ExceptionTranslationFilter、SecurityContextPersistenceFilter 和 UsernamePasswordAuthenticationFilter。在我们使用 http 元素时前三者会自动添加到对应的 FilterChain 中，当我们使用了 form-login 元素时 UsernamePasswordAuthenticationFilter 也会自动添加到 FilterChain 中。所以我们在利用 custom-filter 往 FilterChain 中添加自己定义的这些 Filter 时需要注意它们的位置。 

#### FilterSecurityInterceptor 

FilterSecurityInterceptor 是用于保护 Http 资源的，它需要一个 AccessDecisionManager 和一个 AuthenticationManager 的引用。它会从 SecurityContextHolder 获取 Authentication，然后通过 SecurityMetadataSource 可以得知当前请求是否在请求受保护的资源。对于请求那些受保护的资源，如果 Authentication.isAuthenticated() 返回 false 或者 FilterSecurityInterceptor 的 alwaysReauthenticate 属性为 true，那么将会使用其引用的 AuthenticationManager 再认证一次，认证之后再使用认证后的 Authentication 替换 SecurityContextHolder 中拥有的那个。然后就是利用 AccessDecisionManager 进行权限的检查。

我们在使用基于 NameSpace 的配置时所配置的 intercept-url 就会跟 FilterChain 内部的 FilterSecurityInterceptor 绑定。如果要自己定义 FilterSecurityInterceptor 对应的 bean，那么该 bean 定义大致如下所示 ： 

``` 
 <bean id="filterSecurityInterceptor"
   class="org.springframework.security.web.access.intercept.FilterSecurityInterceptor">
      <property name="authenticationManager" ref="authenticationManager" />
      <property name="accessDecisionManager" ref="accessDecisionManager" />
      <property name="securityMetadataSource">
         <security:filter-security-metadata-source>
            <security:intercept-url pattern="/admin/**" access="ROLE_ADMIN" />
            <security:intercept-url pattern="/**" access="ROLE_USER,ROLE_ADMIN" />
         </security:filter-security-metadata-source>
      </property>
   </bean> 
```

filter-security-metadata-source 用于配置其 securityMetadataSource 属性。intercept-url 用于配置需要拦截的 URL 与对应的权限关系。 

#### ExceptionTranslationFilter 

通过前面的介绍我们知道在 Spring Security 的 Filter 链表中 ExceptionTranslationFilter 就放在 FilterSecurityInterceptor 的前面。而 ExceptionTranslationFilter 是捕获来自 FilterChain 的异常，并对这些异常做处理。ExceptionTranslationFilter 能够捕获来自 FilterChain 所有的异常，但是它只会处理两类异常，AuthenticationException 和 AccessDeniedException，其它的异常它会继续抛出。如果捕获到的是 AuthenticationException，那么将会使用其对应的 AuthenticationEntryPoint 的 commence() 处理。如果捕获的异常是一个 AccessDeniedException，那么将视当前访问的用户是否已经登录认证做不同的处理，如果未登录，则会使用关联的 AuthenticationEntryPoint 的 commence() 方法进行处理，否则将使用关联的 AccessDeniedHandler 的 handle() 方法进行处理。

**AuthenticationEntryPoint** 是在用户没有登录时用于引导用户进行登录认证的，在实际应用中应根据具体的认证机制选择对应的 AuthenticationEntryPoint。

**AccessDeniedHandler** 用于在用户已经登录了，但是访问了其自身没有权限的资源时做出对应的处理。ExceptionTranslationFilter 拥有的 AccessDeniedHandler 默认是 AccessDeniedHandlerImpl，其会返回一个 403 错误码到客户端。我们可以通过显示的配置 AccessDeniedHandlerImpl，同时给其指定一个 errorPage 使其可以返回对应的错误页面。当然我们也可以实现自己的 AccessDeniedHandler。

``` 
 <bean id="exceptionTranslationFilter"
      class="org.springframework.security.web.access.ExceptionTranslationFilter">
      <property name="authenticationEntryPoint">
         <bean class="org.springframework.security.web.authentication.LoginUrlAuthenticationEntryPoint">
            <property name="loginFormUrl" value="/login.jsp" />
         </bean>
      </property>
      <property name="accessDeniedHandler">
         <bean class="org.springframework.security.web.access.AccessDeniedHandlerImpl">
            <property name="errorPage" value="/access_denied.jsp" />
         </bean>
      </property>
   </bean> 
```

在上述配置中我们指定了 AccessDeniedHandler 为 AccessDeniedHandlerImpl，同时为其指定了 errorPage，这样发生 AccessDeniedException 后将转到对应的 errorPage 上。指定了 AuthenticationEntryPoint 为使用表单登录的 LoginUrlAuthenticationEntryPoint。此外，需要注意的是如果该 filter 是作为自定义 filter 加入到由 NameSpace 自动建立的 FilterChain 中时需把它放在内置的 ExceptionTranslationFilter 后面，否则异常都将被内置的 ExceptionTranslationFilter 所捕获。

``` 
<security:http>
      <security:form-login login-page="/login.jsp"
         username-parameter="username" password-parameter="password"
         login-processing-url="/login.do" />
      <!-- 退出登录时删除 session 对应的 cookie -->
      <security:logout delete-cookies="JSESSIONID" />
      <!-- 登录页面应当是不需要认证的 -->
      <security:intercept-url pattern="/login*.jsp*"
         access="IS_AUTHENTICATED_ANONYMOUSLY" />
      <security:intercept-url pattern="/**" access="ROLE_USER" />
      <security:custom-filter ref="exceptionTranslationFilter" after="EXCEPTION_TRANSLATION_FILTER"/>
   </security:http> 
```

在捕获到 AuthenticationException 之后，调用 AuthenticationEntryPoint 的 commence() 方法引导用户登录之前，ExceptionTranslationFilter 还做了一件事，那就是使用 RequestCache 将当前 HttpServletRequest 的信息保存起来，以至于用户成功登录后需要跳转到之前的页面时可以获取到这些信息，然后继续之前的请求，比如用户可能在未登录的情况下发表评论，待用户提交评论的时候就会将包含评论信息的当前请求保存起来，同时引导用户进行登录认证，待用户成功登录后再利用原来的 request 包含的信息继续之前的请求，即继续提交评论，所以待用户登录成功后我们通常看到的是用户成功提交了评论之后的页面。Spring Security 默认使用的 RequestCache 是 HttpSessionRequestCache，其会将 HttpServletRequest 相关信息封装为一个 SavedRequest 保存在 HttpSession 中。

#### SecurityContextPersistenceFilter 

SecurityContextPersistenceFilter 会在请求开始时从配置好的 SecurityContextRepository 中获取 SecurityContext，然后把它设置给 SecurityContextHolder。在请求完成后将 SecurityContextHolder 持有的 SecurityContext 再保存到配置好的 SecurityContextRepository，同时清除 SecurityContextHolder 所持有的 SecurityContext。在使用 NameSpace 时，Spring  Security 默认会给 SecurityContextPersistenceFilter 的 SecurityContextRepository 设置一个 HttpSessionSecurityContextRepository，其会将 SecurityContext 保存在 HttpSession 中。此外 HttpSessionSecurityContextRepository 有一个很重要的属性 allowSessionCreation，默认为 true。这样需要把 SecurityContext 保存在 session 中时，如果不存在 session，可以自动创建一个。也可以把它设置为 false，这样在请求结束后如果没有可用的 session 就不会保存 SecurityContext 到 session 了。SecurityContextRepository 还有一个空实现，NullSecurityContextRepository，如果在请求完成后不想保存 SecurityContext 也可以使用它。 

这里再补充说明一点为什么 SecurityContextPersistenceFilter 在请求完成后需要清除 SecurityContextHolder 的 SecurityContext。SecurityContextHolder 在设置和保存 SecurityContext 都是使用的静态方法，具体操作是由其所持有的 SecurityContextHolderStrategy 完成的。默认使用的是基于线程变量的实现，即 SecurityContext 是存放在 ThreadLocal 里面的，这样各个独立的请求都将拥有自己的 SecurityContext。在请求完成后清除 SecurityContextHolder 中的 SucurityContext 就是清除 ThreadLocal，Servlet 容器一般都有自己的线程池，这可以避免 Servlet 容器下一次分发线程时线程中还包含 SecurityContext 变量，从而引起不必要的错误。 

下面是一个 SecurityContextPersistenceFilter 的简单配置。

``` 
<bean id="securityContextPersistenceFilter"
   class="org.springframework.security.web.context.SecurityContextPersistenceFilter">
      <property name='securityContextRepository'>
         <bean
         class='org.springframework.security.web.context.HttpSessionSecurityContextRepository'>
            <property name='allowSessionCreation' value='false' />
         </bean>
      </property>
   </bean> 
```

#### UsernamePasswordAuthenticationFilter 

 UsernamePasswordAuthenticationFilter 用于处理来自表单提交的认证。该表单必须提供对应的用户名和密码，对应的参数名默认为 j_username 和 j_password。如果不想使用默认的参数名，可以通过 UsernamePasswordAuthenticationFilter 的 usernameParameter 和 passwordParameter 进行指定。表单的提交路径默认是 “j_spring_security_check”，也可以通过 UsernamePasswordAuthenticationFilter 的 filterProcessesUrl 进行指定。通过属性 postOnly 可以指定只允许登录表单进行 post 请求，默认是 true。其内部还有登录成功或失败后进行处理的 AuthenticationSuccessHandler 和 AuthenticationFailureHandler，这些都可以根据需求做相关改变。此外，它还需要一个 AuthenticationManager 的引用进行认证，这个是没有默认配置的。

``` 
<bean id="usernamePasswordAuthenticationFilter"
   class="org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter">
      <property name="authenticationManager" ref="authenticationManager" />
      <property name="authenticationFailureHandler" ref="failureHandler" />  
      <property name="authenticationSuccessHandler" ref="successHandler" />
      <property name="usernameParameter" value="username" />
      <property name="passwordParameter" value="password" />
      <property name="filterProcessesUrl" value="/login.do" />
   </bean> 
```

如果要在 http 元素定义中使用上述 AuthenticationFilter 定义，那么完整的配置应该类似于如下这样子。

``` 
<beans xmlns="http://www.springframework.org/schema/beans"
   xmlns:security="http://www.springframework.org/schema/security"
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="http://www.springframework.org/schema/beans
          http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
          http://www.springframework.org/schema/security
          http://www.springframework.org/schema/security/spring-security-3.1.xsd">
   <!-- entry-point-ref 指定登录入口 -->
   <security:http entry-point-ref="authEntryPoint">
      <security:logout delete-cookies="JSESSIONID" />
      <security:intercept-url pattern="/login*.jsp*"
         access="IS_AUTHENTICATED_ANONYMOUSLY" />
      <security:intercept-url pattern="/**" access="ROLE_USER" />
      <!-- 添加自己定义的 AuthenticationFilter 到 FilterChain 的 FORM_LOGIN_FILTER 位置 -->
      <security:custom-filter ref="authenticationFilter" position="FORM_LOGIN_FILTER"/>
   </security:http>
   <!-- AuthenticationEntryPoint，引导用户进行登录 -->
   <bean id="authEntryPoint" class="org.springframework.security.web.authentication.LoginUrlAuthenticationEntryPoint">
      <property name="loginFormUrl" value="/login.jsp"/>
   </bean>
   <!-- 认证过滤器 -->
   <bean id="usernamePasswordAuthenticationFilter"
   class="org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter">
      <property name="authenticationManager" ref="authenticationManager" />
      <property name="authenticationFailureHandler" ref="failureHandler" />  
      <property name="authenticationSuccessHandler" ref="successHandler" />
      <property name="usernameParameter" value="username" />
      <property name="passwordParameter" value="password" />
      <property name="filterProcessesUrl" value="/login.do" />
   </bean>
  
   <security:authentication-manager alias="authenticationManager">
      <security:authentication-provider
         user-service-ref="userDetailsService">
         <security:password-encoder hash="md5"
            base64="true">
            <security:salt-source user-property="username" />
         </security:password-encoder>
      </security:authentication-provider>
   </security:authentication-manager>
 
   <bean id="userDetailsService"
      class="org.springframework.security.core.userdetails.jdbc.JdbcDaoImpl">
      <property name="dataSource" ref="dataSource" />
   </bean>
 
</beans> 
```
如果验证成功,进入 `SavedRequestAwareAuthenticationSuccessHandler` 的 `onAuthenticationSuccess` 方法，如果验证是否则进入 `SimpleUrlAuthenticationFailureHandler` 的 `onAuthenticationFailure` 方法。  



## AccessDecisionManager 

所有的 Authentication 实现类都保存了一个 GrantedAuthority 列表，其表示用户所具有的权限。GrantedAuthority 是通过 AuthenticationManager 设置到 Authentication 对象中的，然后 AccessDecisionManager 将从 Authentication 中获取用户所具有的 GrantedAuthority 来鉴定用户是否具有访问对应资源的权限。

GrantedAuthority 是一个接口，其中只定义了一个 getAuthority() 方法，其返回值为 String 类型。该方法允许 AccessDecisionManager 获取一个能够精确代表该权限的字符串。通过返回一个字符串，一个 GrantedAuthority 能够很轻易的被大部分 AccessDecisionManager 读取。如果一个 GrantedAuthority 不能够精确的使用一个 String 来表示，那么其对应的 getAuthority() 方法调用应当返回一个 null，这表示 AccessDecisionManager 必须对该 GrantedAuthority 的实现有特定的支持，从而可以获取该 GrantedAuthority 所代表的权限信息。

Spring Security 内置了一个 GrantedAuthority 的实现，SimpleGrantedAuthority。它直接接收一个表示权限信息的字符串，然后 getAuthority() 方法直接返回该字符串。Spring Security 内置的所有 AuthenticationProvider 都是使用它来封装 Authentication 对象的。

Spring Security 是通过拦截器来控制受保护对象的访问的，如方法调用和 Web 请求。在正式访问受保护对象之前，Spring Security 将使用 AccessDecisionManager 来鉴定当前用户是否有访问对应受保护对象的权限。

AccessDecisionManager 是由 AbstractSecurityInterceptor 调用的，它负责鉴定用户是否有访问对应资源（方法或 URL）的权限。AccessDecisionManager 是一个接口，其中只定义了三个方法，其定义如下。

``` 
public interface AccessDecisionManager {
 
    /**
     * 通过传递的参数来决定用户是否有访问对应受保护对象的权限
     *
     * @param authentication 当前正在请求受包含对象的 Authentication
     * @param object 受保护对象，其可以是一个 MethodInvocation、JoinPoint 或 FilterInvocation。
     * @param configAttributes 与正在请求的受保护对象相关联的配置属性
     *
     */
    void decide(Authentication authentication, Object object, Collection<ConfigAttribute> configAttributes)
        throws AccessDeniedException, InsufficientAuthenticationException;
 
    /**
     * 表示当前 AccessDecisionManager 是否支持对应的 ConfigAttribute
     */
    boolean supports(ConfigAttribute attribute);
 
    /**
     * 表示当前 AccessDecisionManager 是否支持对应的受保护对象类型
     */
    boolean supports(Class<?> clazz);
} 
```

decide() 方法用于决定 authentication 是否符合受保护对象要求的 configAttributes。

supports(ConfigAttribute attribute) 方法是用来判断 AccessDecisionManager 是否能够处理对应的 ConfigAttribute 的。

supports(Class<?> clazz) 方法用于判断配置的 AccessDecisionManager 是否支持对应的受保护对象类型。

### 基于投票的 AccessDecisionManager 实现 

Spring Security 已经内置了几个基于投票的 AccessDecisionManager，当然如果需要你也可以实现自己的 AccessDecisionManager。使用这种方式，一系列的 AccessDecisionVoter 将会被 AccessDecisionManager 用来对 Authentication 是否有权访问受保护对象进行投票，然后再根据投票结果来决定是否要抛出 AccessDeniedException。AccessDecisionVoter 是一个接口，其中定义有三个方法，具体结构如下所示。

``` 
public interface AccessDecisionVoter<S> {
 
    intACCESS_GRANTED = 1;
    intACCESS_ABSTAIN = 0;
    intACCESS_DENIED = -1;
 
    boolean supports(ConfigAttribute attribute);
 
    boolean supports(Class<?> clazz);
 
    int vote(Authentication authentication, S object, Collection<ConfigAttribute> attributes);
} 
```

vote() 方法的返回结果会是 AccessDecisionVoter 中定义的三个常量之一。ACCESS_GRANTED 表示同意，ACCESS_DENIED 表示返回，ACCESS_ABSTAIN 表示弃权。如果一个 AccessDecisionVoter 不能判定当前 Authentication 是否拥有访问对应受保护对象的权限，则其 vote() 方法的返回值应当为弃权 ACCESS_ABSTAIN。

 Spring Security 内置了三个基于投票的 AccessDecisionManager 实现类，它们分别是 AffirmativeBased、ConsensusBased 和 UnanimousBased。

**AffirmativeBased** 的逻辑是这样的：

- 只要有 AccessDecisionVoter 的投票为 ACCESS_GRANTED 则同意用户进行访问；
- 如果全部弃权也表示通过；
- 如果没有一个人投赞成票，但是有人投反对票，则将抛出 AccessDeniedException。

**ConsensusBased** 的逻辑是这样的：

- 如果赞成票多于反对票则表示通过。
- 反过来，如果反对票多于赞成票则将抛出 AccessDeniedException。
- 如果赞成票与反对票相同且不等于 0，并且属性 allowIfEqualGrantedDeniedDecisions 的值为 true，则表示通过，否则将抛出异常 AccessDeniedException。参数 allowIfEqualGrantedDeniedDecisions 的值默认为 true。
- 如果所有的 AccessDecisionVoter 都弃权了，则将视参数 allowIfAllAbstainDecisions 的值而定，如果该值为 true 则表示通过，否则将抛出异常 AccessDeniedException。参数 allowIfAllAbstainDecisions 的值默认为 false。

**UnanimousBased** 的逻辑与另外两种实现有点不一样，另外两种会一次性把受保护对象的配置属性全部传递给 AccessDecisionVoter 进行投票，而 UnanimousBased 会一次只传递一个 ConfigAttribute 给 AccessDecisionVoter 进行投票。这也就意味着如果我们的 AccessDecisionVoter 的逻辑是只要传递进来的 ConfigAttribute 中有一个能够匹配则投赞成票，但是放到 UnanimousBased 中其投票结果就不一定是赞成了。UnanimousBased 的逻辑具体来说是这样的：

- 如果受保护对象配置的某一个 ConfigAttribute 被任意的 AccessDecisionVoter 反对了，则将抛出 AccessDeniedException。
- 如果没有反对票，但是有赞成票，则表示通过。
- 如果全部弃权了，则将视参数 allowIfAllAbstainDecisions 的值而定，true 则通过，false 则抛出 AccessDeniedException。

#### RoleVoter 

RoleVoter 是 Spring Security 内置的一个 AccessDecisionVoter，其会将 ConfigAttribute 简单的看作是一个角色名称，在投票的时如果拥有该角色即投赞成票。如果 ConfigAttribute 是以 “ROLE_” 开头的，则将使用 RoleVoter 进行投票。当用户拥有的权限中有一个或多个能匹配受保护对象配置的以 “ROLE_” 开头的 ConfigAttribute 时其将投赞成票；如果用户拥有的权限中没有一个能匹配受保护对象配置的以 “ROLE_” 开头的 ConfigAttribute，则 RoleVoter 将投反对票；如果受保护对象配置的 ConfigAttribute 中没有以 “ROLE_” 开头的，则 RoleVoter 将弃权。

#### AuthenticatedVoter 

AuthenticatedVoter 也是 Spring Security 内置的一个 AccessDecisionVoter 实现。其主要用来区分匿名用户、通过 Remember-Me 认证的用户和完全认证的用户。完全认证的用户是指由系统提供的登录入口进行成功登录认证的用户。

AuthenticatedVoter 可以处理的 ConfigAttribute 有 IS_AUTHENTICATED_FULLY、IS_AUTHENTICATED_REMEMBERED 和 IS_AUTHENTICATED_ANONYMOUSLY。如果 ConfigAttribute 不在这三者范围之内，则 AuthenticatedVoter 将弃权。否则将视 ConfigAttribute 而定，如果 ConfigAttribute 为 IS_AUTHENTICATED_ANONYMOUSLY，则不管用户是匿名的还是已经认证的都将投赞成票；如果是 IS_AUTHENTICATED_REMEMBERED 则仅当用户是由 Remember-Me 自动登录，或者是通过登录入口进行登录认证时才会投赞成票，否则将投反对票；而当 ConfigAttribute 为 IS_AUTHENTICATED_FULLY 时仅当用户是通过登录入口进行登录的才会投赞成票，否则将投反对票。

AuthenticatedVoter 是通过 AuthenticationTrustResolver 的 isAnonymous() 方法和 isRememberMe() 方法来判断 SecurityContextHolder 持有的 Authentication 是否为 AnonymousAuthenticationToken 或 RememberMeAuthenticationToken 的，即是否为 IS_AUTHENTICATED_ANONYMOUSLY 和 IS_AUTHENTICATED_REMEMBERED。

#### 自定义 Voter 

用户也可以通过实现 AccessDecisionVoter 来实现自己的投票逻辑。



## **基于表达式的权限控制**  

Spring Security 允许我们在定义 URL 访问或方法访问所应有的权限时使用 Spring EL 表达式，在定义所需的访问权限时如果对应的表达式返回结果为 true 则表示拥有对应的权限，反之则无。Spring Security 可用表达式对象的基类是 SecurityExpressionRoot，其为我们提供了如下在使用 Spring EL 表达式对 URL 或方法进行权限控制时通用的内置表达式。

| **表达式**                        | **描述**                                   |
| ------------------------------ | ---------------------------------------- |
| hasRole([role])                | 当前用户是否拥有指定角色。                            |
| hasAnyRole([role1,role2])      | 多个角色是一个以逗号进行分隔的字符串。如果当前用户拥有指定角色中的任意一个则返回 true。 |
| hasAuthority([auth])           | 等同于 hasRole                              |
| hasAnyAuthority([auth1,auth2]) | 等同于 hasAnyRole                           |
| Principle                      | 代表当前用户的 principle 对象                     |
| authentication                 | 直接从 SecurityContext 获取的当前 Authentication 对象 |
| permitAll                      | 总是返回 true，表示允许所有的                        |
| denyAll                        | 总是返回 false，表示拒绝所有的                       |
| isAnonymous()                  | 当前用户是否是一个匿名用户                            |
| isRememberMe()                 | 表示当前用户是否是通过 Remember-Me 自动登录的            |
| isAuthenticated()              | 表示当前用户是否已经登录认证成功了。                       |
| isFullyAuthenticated()         | 如果当前用户既不是一个匿名用户，同时又不是通过 Remember-Me 自动登录的，则返回 true。 |

### 通过表达式控制 URL 权限 

URL 的访问权限是通过 http 元素下的 intercept-url 元素进行定义的，其 access 属性用来定义访问配置属性。默认情况下该属性值只能是以字符串进行分隔的字符串列表，且每一个元素都对应着一个角色，因为默认使用的是 RoleVoter。通过设置 http 元素的 use-expressions=”true” 可以启用 intercept-url 元素的 access 属性对 Spring EL 表达式的支持，use-expressions 的值默认为 false。启用 access 属性对 Spring EL 表达式的支持后每个 access 属性值都应该是一个返回结果为 boolean 类型的表达式，当表达式返回结果为 true 时表示当前用户拥有访问权限。此外 WebExpressionVoter 将加入 AccessDecisionManager 的 AccessDecisionVoter 列表，所以如果不使用 NameSpace 时应当手动添加 WebExpressionVoter 到 AccessDecisionVoter。

``` 
<security:http use-expressions="true">
      <security:form-login/>
      <security:logout/>
      <security:intercept-url pattern="/**" access="hasRole('ROLE_USER')" />
   </security:http> 
```

在上述配置中我们定义了只有拥有 ROLE_USER 角色的用户才能访问系统。

使用表达式控制 URL 权限使用的表达式对象类是继承自 SecurityExpressionRoot 的 WebSecurityExpressionRoot 类。其相比基类而言新增了一个表达式 hasIpAddress。hasIpAddress 可用来限制只有指定 IP 或指定范围内的 IP 才可以访问。

``` 
<security:http use-expressions="true">
      <security:form-login/>
      <security:logout/>
      <security:intercept-url pattern="/**" access="hasRole('ROLE_USER') and hasIpAddress('10.10.10.3')" />
   </security:http> 
```

在上面的配置中我们限制了只有 IP 为”10.10.10.3”，且拥有 ROLE_USER 角色的用户才能访问。hasIpAddress 是通过 Ip 地址或子网掩码来进行匹配的。如果要设置 10.10.10 下所有的子网都可以使用，那么我们对应的 hasIpAddress 的参数应为 “10.10.10.n/24”，其中 n 可以是合法 IP 内的任意值。具体规则可以参照 hasIpAddress() 表达式用于比较的 IpAddressMatcher 的 matches 方法源码。以下是 IpAddressMatcher 的源码：

``` 
package org.springframework.security.web.util;
 
import java.net.InetAddress;
import java.net.UnknownHostException;
import java.util.Arrays;
 
import javax.servlet.http.HttpServletRequest;
 
import org.springframework.util.StringUtils;
 
/**
 * Matches a request based on IP Address or subnet mask matching against the remote address.
 * <p>
 * Both IPv6 and IPv4 addresses are supported, but a matcher which is configured with an IPv4 address will
 * never match a request which returns an IPv6 address, and vice-versa.
 *
 * @author Luke Taylor
 * @since 3.0.2
 */
public final class IpAddressMatcher implements RequestMatcher {
    private final int nMaskBits;
    private final InetAddress requiredAddress;
 
    /**
     * Takes a specific IP address or a range specified using the
     * IP/Netmask (e.g. 192.168.1.0/24 or 202.24.0.0/14).
     *
     * @param ipAddress the address or range of addresses from which the request must come.
     */
    public IpAddressMatcher(String ipAddress) {
 
        if (ipAddress.indexOf('/') > 0) {
            String[] addressAndMask = StringUtils.split(ipAddress, "/");
            ipAddress = addressAndMask[0];
            nMaskBits = Integer.parseInt(addressAndMask[1]);
        } else {
            nMaskBits = -1;
        }
        requiredAddress = parseAddress(ipAddress);
    }
 
    public boolean matches(HttpServletRequest request) {
        return matches(request.getRemoteAddr());
    }
 
    public boolean matches(String address) {
        InetAddress remoteAddress = parseAddress(address);
 
        if (!requiredAddress.getClass().equals(remoteAddress.getClass())) {
            return false;
        }
 
        if (nMaskBits <0) {
            return remoteAddress.equals(requiredAddress);
        }
 
        byte[] remAddr = remoteAddress.getAddress();
        byte[] reqAddr = requiredAddress.getAddress();
 
        int oddBits = nMaskBits % 8;
        int nMaskBytes = nMaskBits/8 + (oddBits == 0 ? 0 : 1);
        byte[] mask = newbyte[nMaskBytes];
 
        Arrays.fill(mask, 0, oddBits == 0 ? mask.length : mask.length - 1, (byte)0xFF);
 
        if (oddBits != 0) {
            int finalByte = (1 << oddBits) - 1;
            finalByte <<= 8-oddBits;
            mask[mask.length - 1] = (byte) finalByte;
        }
 
 //       System.out.println("Mask is" + new sun.misc.HexDumpEncoder().encode(mask));
 
        for (int i=0; i < mask.length; i++) {
            if ((remAddr[i] & mask[i]) != (reqAddr[i] & mask[i])) {
                return alse;
            }
        }
 
        return true;
    }
 
    private InetAddress parseAddress(String address) {
        try {
            return InetAddress.getByName(address);
        } catch (UnknownHostException e) {
            thrownew IllegalArgumentException("Failed to parse address" + address, e);
        }
    }
} 
```

### 通过表达式控制方法权限 

Spring Security 中定义了四个支持使用表达式的注解，分别是 @PreAuthorize、@PostAuthorize、@PreFilter 和 @PostFilter。其中前两者可以用来在方法调用前或者调用后进行权限检查，后两者可以用来对集合类型的参数或者返回值进行过滤。要使它们的定义能够对我们的方法的调用产生影响我们需要设置 global-method-security 元素的 pre-post-annotations=”enabled”，默认为 disabled。

``` 
<security:global-method-security pre-post-annotations="disabled"/> 
```

#### 使用 @PreAuthorize 和 @PostAuthorize 进行访问控制 

@PreAuthorize 可以用来控制一个方法是否能够被调用。

``` 
@Service
public class UserServiceImpl implements UserService {
 
   @PreAuthorize("hasRole('ROLE_ADMIN')")
   public void addUser(User user) {
      System.out.println("addUser................" + user);
   }
 
   @PreAuthorize("hasRole('ROLE_USER') or hasRole('ROLE_ADMIN')")
   public User find(int id) {
      System.out.println("find user by id............." + id);
      return null;
   }
 
} 
```

在上面的代码中我们定义了只有拥有角色 ROLE_ADMIN 的用户才能访问 adduser() 方法，而访问 find() 方法需要有 ROLE_USER 角色或 ROLE_ADMIN 角色。

使用表达式时我们还可以在表达式中使用方法参数。

``` 
public class UserServiceImpl implements UserService {
 
   /**
    * 限制只能查询 Id 小于 10 的用户
    */
   @PreAuthorize("#id<10")
   public User find(int id) {
      System.out.println("find user by id........." + id);
      return null;
   }
  
   /**
    * 限制只能查询自己的信息
    */
   @PreAuthorize("principal.username.equals(#username)")
   public User find(String username) {
      System.out.println("find user by username......" + username);
      return null;
   }
 
   /**
    * 限制只能新增用户名称为 abc 的用户
    */
   @PreAuthorize("#user.name.equals('abc')")
   public void add(User user) {
      System.out.println("addUser............" + user);
   }
 
} 
```

在上面代码中我们定义了调用 find(int id) 方法时，只允许参数 id 小于 10 的调用；调用 find(String username) 时只允许 username 为当前用户的用户名；定义了调用 add() 方法时只有当参数 user 的 name 为 abc 时才可以调用。

有时候可能你会想在方法调用完之后进行权限检查，这种情况比较少，但是如果你有的话，Spring Security 也为我们提供了支持，通过 @PostAuthorize 可以达到这一效果。使用 @PostAuthorize 时我们可以使用内置的表达式 returnObject 表示方法的返回值。我们来看下面这一段示例代码。

``` 
@PostAuthorize("returnObject.id%2==0")
   public User find(int id) {
      User user = new User();
      user.setId(id);
      return user;
   } 
```

上面这一段代码表示将在方法 find() 调用完成后进行权限检查，如果返回值的 id 是偶数则表示校验通过，否则表示校验失败，将抛出 AccessDeniedException。       需要注意的是 @PostAuthorize 是在方法调用完成后进行权限检查，它不能控制方法是否能被调用，只能在方法调用完成后检查权限决定是否要抛出 AccessDeniedException。

#### 使用 @PreFilter 和 @PostFilter 进行过滤 

使用 @PreFilter 和 @PostFilter 可以对集合类型的参数或返回值进行过滤。使用 @PreFilter 和 @PostFilter 时，Spring Security 将移除使对应表达式的结果为 false 的元素。

``` 
@PostFilter("filterObject.id%2==0")
   public List<User> findAll() {
      List<User> userList = new ArrayList<User>();
      User user;
      for (int i=0; i<10; i++) {
         user = new User();
         user.setId(i);
         userList.add(user);
      }
      return userList;
   } 
```

上述代码表示将对返回结果中 id 不为偶数的 user 进行移除。filterObject 是使用 @PreFilter 和 @PostFilter 时的一个内置表达式，表示集合中的当前对象。当 @PreFilter 标注的方法拥有多个集合类型的参数时，需要通过 @PreFilter 的 filterTarget 属性指定当前 @PreFilter 是针对哪个参数进行过滤的。如下面代码就通过 filterTarget 指定了当前 @PreFilter 是用来过滤参数 ids 的。

``` 
@PreFilter(filterTarget="ids", value="filterObject%2==0")
   public void delete(List<Integer> ids, List<String> usernames) {
      ...
   } 
```

### 使用 hasPermission 表达式 

 Spring Security 为我们定义了 hasPermission 的两种使用方式，它们分别对应着 PermissionEvaluator 的两个不同的 hasPermission() 方法。Spring Security 默认处理 Web、方法的表达式处理器分别为 DefaultWebSecurityExpressionHandler 和 DefaultMethodSecurityExpressionHandler，它们都继承自 AbstractSecurityExpressionHandler，其所持有的 PermissionEvaluator 是 DenyAllPermissionEvaluator，其对于所有的 hasPermission 表达式都将返回 false。所以当我们要使用表达式 hasPermission 时，我们需要自已手动定义 SecurityExpressionHandler 对应的 bean 定义，然后指定其 PermissionEvaluator 为我们自己实现的 PermissionEvaluator，然后通过 global-method-security 元素或 http 元素下的 expression-handler 元素指定使用的 SecurityExpressionHandler 为我们自己手动定义的那个 bean。

接下来看一个自己实现 PermissionEvaluator 使用 hasPermission() 表达式的简单示例。

首先实现自己的 PermissionEvaluator，如下所示：

``` 
public class MyPermissionEvaluator implements PermissionEvaluator {
 
   public boolean hasPermission(Authentication authentication,
         Object targetDomainObject, Object permission) {
      if ("user".equals(targetDomainObject)) {
         return this.hasPermission(authentication, permission);
      }
      return false;
   }
 
   /**
    * 总是认为有权限
    */
   public boolean hasPermission(Authentication authentication,
         Serializable targetId, String targetType, Object permission) {
      return true;
   }
  
   /**
    * 简单的字符串比较，相同则认为有权限
    */
   private boolean hasPermission(Authentication authentication, Object permission) {
      Collection<? extends GrantedAuthority> authorities = authentication.getAuthorities();
      for (GrantedAuthority authority : authorities) {
         if (authority.getAuthority().equals(permission)) {
            returntrue;
         }
      }
      return false;
   }
 
} 
```

 接下来在 ApplicationContext 中显示的配置一个将使用 PermissionEvaluator 的 SecurityExpressionHandler 实现类，然后指定其所使用的 PermissionEvaluator 为我们自己实现的那个。这里我们选择配置一个针对于方法调用使用的表达式处理器，DefaultMethodSecurityExpressionHandler，具体如下所示。

``` 
<bean id="expressionHandler"
   class="org.springframework.security.access.expression.method.DefaultMethodSecurityExpressionHandler">
      <property name="permissionEvaluator" ref="myPermissionEvaluator" />
   </bean>
   <!-- 自定义的 PermissionEvaluator 实现 -->
   <bean id="myPermissionEvaluator" class="com.xxx.MyPermissionEvaluator"/>
```

有了 SecurityExpressionHandler 之后，我们还要告诉 Spring Security，在使用 SecurityExpressionHandler 时应该使用我们显示配置的那个，这样我们自定义的 PermissionEvaluator 才能起作用。因为我们上面定义的是针对于方法的 SecurityExpressionHandler，所以我们要指定在进行方法权限控制时应该使用它来进行处理，同时注意设置 pre-post-annotations=”true” 以启用对支持使用表达式的 @PreAuthorize 等注解的支持。 

``` 
<security:global-method-security
      pre-post-annotations="enabled">
      <security:expression-handler ref="expressionHandler" />
   </security:global-method-security> 
```

之后我们就可以在需要进行权限控制的方法上使用 @PreAuthorize 以及 hasPermission() 表达式进行权限控制了。 

``` 
@Service
public class UserServiceImpl implements UserService {
 
   /**
    * 将使用方法 hasPermission(Authentication authentication,
         Object targetDomainObject, Object permission) 进行验证。
    */
   @PreAuthorize("hasPermission('user','ROLE_USER')")
   public User find(int id) {
      return null;
   }
  
   /**
    * 将使用 PermissionEvaluator 的第二个方法，即 hasPermission(Authentication authentication,
         Serializable targetId, String targetType, Object permission) 进行验证。
    */
   @PreAuthorize("hasPermission('targetId','targetType','permission')")
   public User find(String username) {
      return null;
   }
 
   @PreAuthorize("hasPermission('user','ROLE_ADMIN')")
   public void add(User user) {
 
   }
 
} 
```

在上面的配置中，find(int id) 和 add() 方法将使用 PermissionEvaluator 中接收三个参数的 hasPermission() 方法进行验证，而 find(String username) 方法将使用四个参数的 hasPermission() 方法进行验证。因为 hasPermission() 表达式与 PermissionEvaluator 中 hasPermission() 方法的对应关系就是在 hasPermission() 表达式使用的参数基础上加上当前 Authentication 对象调用对应的 hasPermission() 方法进行验证。 



## Thymeleaf 与 Spring Security 

http://elim.iteye.com/blog/2161056 