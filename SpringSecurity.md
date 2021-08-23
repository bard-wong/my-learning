# WebSecurityConfigurerAdapter

```java
.antMatchers("/index","/myajax.html","login","/js/**","/captcha/**").permitAll()
```

​	根据路径放行

```java
.regexMatchers(".+[.]js").permitAll()
```

根据正则表达式放行

```java
.mvcMatchers("/js/**").servletPath("/v1").permitAll()
```

Servlet Path放行

```java
.anyRequest().authenticated()
```

任何请求验证

```java
.antMatchers("/access/user/**").hasRole("USER")
.antMatchers("/access/user/**").hasAnyRole("USER","ADMIN")  //匹配任一
```

根据角色验证

```java
.antMatchers("/index").hasIpAddress("127.0.0.1")
```

根据IP验证

- permitAll()	允许所有
- anonymous()    匿名
- rememberMe()    登录一次后，后续不用登录
- denyAll()        不允许所有
- authenticated()     认证
- fullyAuthenticated()   必须用户名密码登录



# 实现步骤

- 自定义用户类需实现UserDetails接口
- 创建类实现UserDetailsService接口
- 实现成功失败Handler  实现AuthenticationSuccessHandler和AuthenticationFailureHandler
- 实现拒绝访问Handler   实现AccessDeniedHandler

## config

```java
http.authorizeRequests().successHandler(successHandler).failureHandler(failureHandler)			//登录成功和失败处理的handler
http.exceptionHandling().accessDeniedHandler(deniedHandler)		//拒绝访问处理
```

csrf

```java
.csrf().disable();		//关闭csrf防护
```

CSRF跨站请求伪造，通过伪造用户请求访问受信任站点的非法请求访问

![img](E:\笔记\image\2009040916453171.jpg)

CSRF防护默认开启，默认会拦截请求，进行CSRF处理。CSRF为了保证不是其他第三方网站访问，要求访问时携带参数名为`_csrf`值为`token`的内容，如果token和服务端的token匹配成功，则正常访问

```html
<input type="hidden" th:if="${_csrf}" name="_csrf" th:value="${_csrf.getToken()}">
```

