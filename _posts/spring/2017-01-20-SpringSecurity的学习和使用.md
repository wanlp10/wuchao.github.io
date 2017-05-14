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
> [Spring Security 中文文档](http://www.mossle.com/docs/springsecurity3/html/springsecurity.html) 
>
> [使用 Spring Security 保护 Web 应用的安全](https://www.ibm.com/developerworks/cn/java/j-lo-springsecurity/) 
>
> [mkyong.com - Spring Security Tutorial](http://www.mkyong.com/spring-security/spring-security-hibernate-annotation-example/) 
>
> [Spring Security 4 基于角色的登录例子（带源码）](http://blog.csdn.net/w605283073/article/details/51322771) 

Spring 是一个非常流行和成功的 Java 应用开发框架。Spring Security 基于 Spring 框架，提供了一套 Web 应用安全性的完整解决方案。一般来说，Web 应用的安全性包括用户认证（Authentication）和用户授权（Authorization）两个部分。用户认证指的是验证某个用户是否为系统中的合法主体，也就是说用户能否访问该系统。用户认证一般要求用户提供用户名和密码。系统通过校验用户名和密码来完成认证过程。用户授权指的是验证某个用户是否有权限执行某个操作。在一个系统中，不同用户所具有的权限是不同的。比如对一个文件来说，有的用户只能进行读取，而有的用户可以进行修改。一般来说，系统会为不同的用户分配不同的角色，而每个角色则对应一系列的权限。 



## UserDetailsService 

### org.springframework.security.core.userdetails.UserDetailsService.java

``` 
public interface UserDetailsService { 
	UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
} 
```

UserDetailsService 接口的实现类如下: 

#### org.springframework.security.provisioning.InMemoryUserDetailsManager.java (InMemoryDaoImpl.java) 

 ``` 
public class InMemoryUserDetailsManager implements UserDetailsManager { 
	
	... 
	
	public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        UserDetails user = (UserDetails)this.users.get(username.toLowerCase());
        if(user == null) {
            throw new UsernameNotFoundException(username);
        } else {
            return new User(user.getUsername(), user.getPassword(), user.isEnabled(), user.isAccountNonExpired(), user.isCredentialsNonExpired(), user.isAccountNonLocked(), user.getAuthorities());
        }
    } 
    
    ... 
    
}
 ```

#### org.springframework.security.core.userdetails.jdbc.JdbcDaoImpl.java 

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

根据用户名查询所有匹配用户。 



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

根据用户名查询用户权限（如果启用了用户权限）。



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

根据用户名查询组权限（如果启用了组权限）。 



这三个方法分别调用了下面三条查询语句：

``` 
private String usersByUsernameQuery = "select username,password,enabled from users where username = ?"; 

private String authoritiesByUsernameQuery = "select username,authority from authorities where username = ?"; 

private String groupAuthoritiesByUsernameQuery = "select g.id, g.group_name, ga.authority from groups g, group_members gm, group_authorities ga where gm.username = ? and g.id = ga.group_id and g.id = gm.group_id"; 
```

上面权限的启用和 sql 查询语句都是可以（在实现了 WebSecurityConfigurerAdapter 的类中）控制和配置的，sql 语句可以根据自己定义的安全实体来查询，但是 select 后面的查询属性的位置顺序要和上面一样，只能改变 where 子句： 

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



## 验证过程（以 JDBC 验证为例）

> [](http://docs.spring.io/spring-security/site/docs/current/reference/html/technical-overview.html)  
>
> [](http://blog.sina.com.cn/s/blog_5c0522dd0101doey.html) 

根据提交的用户名和密码构造 Token (org.springframework.security.authentication.UsernamePasswordAuthenticationToken) , 并将该 Token 对象带入 `AuthenticationManager` (org.springframework.security.authentication.ProviderManager) 验证, 验证成功后返回一个 `Authentication` 对象 .  

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

验证成功: 

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

跳转页面: 

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



