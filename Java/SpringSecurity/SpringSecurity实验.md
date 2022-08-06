





# 实验一：放行首页和静态资源

在配置类中重写父类的 configure(HttpSecurity security)方法。



```java
// 将当前类标记为配置类
@Configuration
// 启用Web环境下权限控制功能
@EnableWebSecurity
public class WebAppSecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity security) throws Exception {
        security
                .authorizeRequests()			// 对请求进行授权
                .antMatchers("/index.jsp")		// 针对/index.jsp路径进行授权
                .permitAll()					// 可以无条件访问
                .antMatchers("/layui/**")		// 针对/layui目录下所有资源进行授权
                .permitAll()					// 可以无条件访问
                .and()
                .authorizeRequests()			// 对请求进行授权
                .anyRequest()					// 任意请求
                .authenticated()				// 需要登录以后才可以访问
                .and()
                .formLogin();					// 使用表单形式登录
    }
}
```





# 实验二：未认证请求跳转到自定义登录页



```java
// 将当前类标记为配置类
@Configuration
// 启用Web环境下权限控制功能
@EnableWebSecurity
public class WebAppSecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity security) throws Exception {
        security
                .authorizeRequests()			// 对请求进行授权
                .antMatchers("/index.jsp")		// 针对/index.jsp路径进行授权
                .permitAll()					// 可以无条件访问
                .antMatchers("/layui/**")		// 针对/layui目录下所有资源进行授权
                .permitAll()					// 可以无条件访问
                .and()
                .authorizeRequests()			// 对请求进行授权
                .anyRequest()					// 任意请求
                .authenticated()				// 需要登录以后才可以访问
                .and()
                .formLogin()					// 使用表单形式登录

                // 关于loginPage()方法的特殊说明
                // 指定登录页的同时会影响到：“提交登录表单的地址”、“退出登录地址”、“登录失败地址”
                // /index.jsp GET - the login form 去登录页面
                // /index.jsp POST - process the credentials and if valid authenticate the user 提交登录表单
                // /index.jsp?error GET - redirect here for failed authentication attempts 登录失败
                // /index.jsp?logout GET - redirect here after successfully logging out 退出登录
                .loginPage("/index.jsp")		// 指定登录页面（如果没有指定会访问SpringSecurity自带的登录页）

                // loginProcessingUrl()方法指定了登录地址，就会覆盖loginPage()方法中设置的默认值/index.jsp POST
                .loginProcessingUrl("/do/login.html");	// 指定提交登录表单的地址
        
    }
}
```

说明：

- formLogin()开启表单登录功能
- loginPage()指定登录页地址
- loginProcessingUrl()处理登录请求的 URL 地址
- usernameParameter()设置用户名的请求参数名
- passwordParameter()设置密码的请求参数名
- defaultSuccessUrl()登录成功后前往的地址



指定登录页前后 SpringSecurity 登录地址变化：

![image-20210813152157881](https://gitee.com/jobim/blogimage/raw/master/img/20210813152157.png)



提示：如果希望在指定登录页面地址后指定登录操作本身地址，可以调用 **loginProcessingUrl("登录地址").permitAll()**方法。这里之所以要调用permitAll()方法是为了允许访问登录地址，不然这个登录地址也需要登录后才能访问。



# 实验三：设置登录系统的账号、密码



1. 给 index.jsp 设置表单

   ```jsp
   <p>${SPRING_SECURITY_LAST_EXCEPTION.message}</p><%--登录失败的提示信息--%>
   <form action="${pageContext.request.contextPath }/do/login.html" method="post">
       <input type="hidden" name="${_csrf.parameterName}" value="${_csrf.token}"/>
       <input type="text" name="loginAcct"  class="layui-input">
       <input type="text" name="userPswd" placeholder="密码" >
       <button type="submit" >登 入</button>
   </form>
   ```



2. 后端配置

   ```java
   // 将当前类标记为配置类
   @Configuration
   // 启用Web环境下权限控制功能
   @EnableWebSecurity
   public class WebAppSecurityConfig extends WebSecurityConfigurerAdapter {
   
       //重写另外一个父类的方法，来设置登录系统的账号密码
       @Override
       protected void configure(AuthenticationManagerBuilder builder) throws Exception {
           builder
                   .inMemoryAuthentication()	// 在内存中完成账号、密码的检查
                   .withUser("tom")			// 指定账号
                   .password("123123")			// 指定密码
                   .roles("ADMIN")				// 指定当前用户的角色
                   .and()
                   .withUser("jerry")			// 指定账号
                   .password("123123")			// 指定密码
                   .authorities("UPDATE")		// 指定当前用户的权限
           ;
       }
   
       @Override
       protected void configure(HttpSecurity security) throws Exception {
           security.authorizeRequests()// 对请求进行授权
                   .antMatchers("/index.jsp")		// 针对/index.jsp路径进行授权
                   .permitAll()					// 可以无条件访问
                   .antMatchers("/layui/**")		// 针对/layui目录下所有资源进行授权
                   .permitAll()					// 可以无条件访问
                   .and()
                   .authorizeRequests()			// 对请求进行授权
                   .anyRequest()					// 任意请求
                   .authenticated()				// 需要登录以后才可以访问
                   .and()
                   .formLogin()					// 使用表单形式登录
   
                   // 关于loginPage()方法的特殊说明
                   // 指定登录页的同时会影响到：“提交登录表单的地址”、“退出登录地址”、“登录失败地址”
                   // /index.jsp GET - the login form 去登录页面
                   // /index.jsp POST - process the credentials and if valid authenticate the user 提交登录表单
                   // /index.jsp?error GET - redirect here for failed authentication attempts 登录失败
                   // /index.jsp?logout GET - redirect here after successfully logging out 退出登录
                   .loginPage("/index.jsp")		// 指定登录页面（如果没有指定会访问SpringSecurity自带的登录页）
   
                   // loginProcessingUrl()方法指定了登录地址，就会覆盖loginPage()方法中设置的默认值/index.jsp POST
                   .loginProcessingUrl("/do/login.html")  // 指定提交登录表单的地址
                   .usernameParameter("loginAcct")			// 定制登录账号的请求参数名
                   .passwordParameter("userPswd")			// 定制登录密码的请求参数名
                   .defaultSuccessUrl("/main.html")		// 登录成功后前往的地址
           ;
       }
   }
   ```



注意：如果没有提供角色或权限（ roles() 或 authorities()方法），那么会抛出异常，异常消息是 Cannot pass a null GrantedAuthority collection。



## csrf防止跨站请求伪造



csrf要求每次请求**必须带_csrf 值**。

![image-20210813160449755](https://gitee.com/jobim/blogimage/raw/master/img/20210813160449.png)



发送登录请求时没有携带_csrf 值，则返回下面错误：

![image-20210813160417201](https://gitee.com/jobim/blogimage/raw/master/img/20210813160417.png)







# 实验四：用户注销



logout()方法：开启注销功能
logoutUrl()方法：自定义注销功能的 URL 地址



## 禁用CSRF功能的前提下的退出操作

配置中禁用CSRF功能

```java
@Override
protected void configure(HttpSecurity security) throws Exception {
    security.authorizeRequests()// 对请求进行授权
            .antMatchers("/index.jsp")		// 针对/index.jsp路径进行授权
            .permitAll()					// 可以无条件访问
            .antMatchers("/layui/**")		// 针对/layui目录下所有资源进行授权
            .permitAll()					// 可以无条件访问
            .and()
            .authorizeRequests()			// 对请求进行授权
            .anyRequest()					// 任意请求
            .authenticated()				// 需要登录以后才可以访问
            .and()
            .formLogin()					// 使用表单形式登录

            // 关于loginPage()方法的特殊说明
            // 指定登录页的同时会影响到：“提交登录表单的地址”、“退出登录地址”、“登录失败地址”
            // /index.jsp GET - the login form 去登录页面
            // /index.jsp POST - process the credentials and if valid authenticate the user 提交登录表单
            // /index.jsp?error GET - redirect here for failed authentication attempts 登录失败
            // /index.jsp?logout GET - redirect here after successfully logging out 退出登录
            .loginPage("/index.jsp")		// 指定登录页面（如果没有指定会访问SpringSecurity自带的登录页）

            // loginProcessingUrl()方法指定了登录地址，就会覆盖loginPage()方法中设置的默认值/index.jsp POST
            .loginProcessingUrl("/do/login.html")  // 指定提交登录表单的地址
            .usernameParameter("loginAcct")			// 定制登录账号的请求参数名
            .passwordParameter("userPswd")			// 定制登录密码的请求参数名
            .defaultSuccessUrl("/main.html")		// 登录成功后前往的地址
            .and()
            .csrf()
            .disable()					// 禁用CSRF功能
            .logout()								// 开启退出功能
            .logoutUrl("/do/logout.html")			// 指定处理退出请求的URL地址
            .logoutSuccessUrl("/index.jsp")			// 退出成功后前往的地址
            ;
}
```



前端退出登录操作：

```html
<a href="${pageContext.request.contextPath }/do/logout.html">退出</a>
```



## 启用CSRF时的退出操作

如果 CSRF 功能没有禁用，那么**退出请求必须携带 CSRF 的 token 值**。如果禁用了 CSRF功能则任何请求方式都可以。
logoutSuccessUrl()方法：退出成功后前往的 URL 地址
addLogoutHandler()方法：添加退出处理器
logoutSuccessHandler()方法：退出成功处理器



```jsp
<form id="logoutForm" action="${pageContext.request.contextPath }/do/logout.html" method="post">
	<input type="hidden" name="${_csrf.parameterName}" value="${_csrf.token}"/>
</form>
<a id="logoutAnchor" href="">退出</a>
<script type="text/javascript">
	window.onload = function() {
		// 给超链接的DOM对象绑定单击响应函数
		document.getElementById("logoutAnchor").onclick = function() {
			// 提交包含csrf参数的表单
			document.getElementById("logoutForm").submit();
			// 取消超链接的默认行为
			return false;
		};
	};
</script>
```





# 实验五：基于角色或权限进行访问控制



```java
// 将当前类标记为配置类
@Configuration
// 启用Web环境下权限控制功能
@EnableWebSecurity
public class WebAppSecurityConfig extends WebSecurityConfigurerAdapter {

    //重写另外一个父类的方法，来设置登录系统的账号密码
    //通过 AuthenticationManagerBuilder 对象设置用户登录时具备的角色
    @Override
    protected void configure(AuthenticationManagerBuilder builder) throws Exception {
        builder
                .inMemoryAuthentication()	// 在内存中完成账号、密码的检查
                .withUser("tom")			// 指定账号
                .password("123123")			// 指定密码
                .roles("ADMIN","学徒")			// 指定当前用户的角色
                .and()
                .withUser("jerry")			// 指定账号
                .password("123123")			// 指定密码
                .authorities("UPDATE","内门弟子")		// 指定当前用户的权限
        ;
    }

    //通过 HttpSecurity 对象设置资源的角色要求
    @Override
    protected void configure(HttpSecurity security) throws Exception {
        security
                .authorizeRequests()			// 对请求进行授权
                .antMatchers("/index.jsp")		// 针对/index.jsp路径进行授权
                .permitAll()					// 可以无条件访问
                .antMatchers("/layui/**")		// 针对/layui目录下所有资源进行授权
                .permitAll()					// 可以无条件访问
                .antMatchers("/level1/**")		// 针对/level1/**路径设置访问要求
                .hasRole("学徒")					// 要求用户具备“学徒”角色才可以访问
                .antMatchers("/level2/**")		// 针对/level2/**路径设置访问要求
                .hasAuthority("内门弟子")			// 要求用户具备“内门弟子”权限才可以访问
                .and()
                .authorizeRequests()			// 对请求进行授权
                .anyRequest()					// 任意请求
                .authenticated()				// 需要登录以后才可以访问
                .and()
                .formLogin()					// 使用表单形式登录

                // 关于loginPage()方法的特殊说明
                // 指定登录页的同时会影响到：“提交登录表单的地址”、“退出登录地址”、“登录失败地址”
                // /index.jsp GET - the login form 去登录页面
                // /index.jsp POST - process the credentials and if valid authenticate the user 提交登录表单
                // /index.jsp?error GET - redirect here for failed authentication attempts 登录失败
                // /index.jsp?logout GET - redirect here after successfully logging out 退出登录
                .loginPage("/index.jsp")		// 指定登录页面（如果没有指定会访问SpringSecurity自带的登录页）

                // loginProcessingUrl()方法指定了登录地址，就会覆盖loginPage()方法中设置的默认值/index.jsp POST
                .loginProcessingUrl("/do/login.html")  // 指定提交登录表单的地址
                .usernameParameter("loginAcct")			// 定制登录账号的请求参数名
                .passwordParameter("userPswd")			// 定制登录密码的请求参数名
                .defaultSuccessUrl("/main.html")		// 登录成功后前往的地址
                .and()
//                .csrf()
//                .disable()					// 禁用CSRF功能
                .logout()								// 开启退出功能
                .logoutUrl("/do/logout.html")			// 指定处理退出请求的URL地址
                .logoutSuccessUrl("/index.jsp")			// 退出成功后前往的地址
                ;
    }
}
```



注意：调整顺序

```java
.antMatchers("/level1/**") 	//设置匹配/level1/**的地址
.hasRole("学徒") 				//要求具备“学徒”角色
.antMatchers("/level2/**")
.hasRole("大师")
.antMatchers("/level3/**")
.hasRole("宗师")
.anyRequest() 				//其实未设置的所有请求
.authenticated() 			//需要认证才可以访问
```



**anyRequest()** 设置范围更大
**antMatchers()**设置范围相对小
如果**anyRequest()** 先调用，会把后面**antMatchers()**的设置覆盖，导致**antMatchers()**无效。所以要先做具体小范围设置，再做大范围模糊设置。



注意：SpringSecurity 会在角色字符串前面加**“ROLE_”前缀**，所以以后从数据库查询得到的角色信息时，时需要我们自己给角色字符串前面加“ROLE_”前缀。



# 实验六：自定义403错误页面



设置错误页面：no_auth.jsp



**前往自定义页面方式一：**

配置文件中设置HttpSecurity 对象

```java
.and()
.exceptionHandling()
.accessDeniedPage("/to/no/auth/page");
```

controller方法：

```java
@RequestMapping("/to/no/auth/page")
public String toNoAuthPage() {
return "no_auth";
}
```



**前往自定义页面方式二：**

```java
.and()
.exceptionHandling()               // 指定异常处理器
.accessDeniedHandler(new AccessDeniedHandler() {

    @Override
    public void handle(HttpServletRequest request, HttpServletResponse response,
                       AccessDeniedException accessDeniedException) throws IOException, ServletException {
        request.setAttribute("message", "抱歉！您无法访问这个资源！☆☆☆"); //往请求域中放消息
        request.getRequestDispatcher("/WEB-INF/views/no_auth.jsp").forward(request, response);
    }
})
```





# 实验七：记住我功能（内存版）

## 





HttpSecurity 对象调用 rememberMe()方法。
登录表单携带名为 remember-me 的请求参数。具体做法是将登录表单中的checkbox 的 name 设置为 remember-me

```html
<input type="checkbox" name="remember-me"lay-skin="primary"title="记住码">
```

如 果 不 能 使 用 “ remember-me ” 作 为 请 求 参 数 名 称 ， 可 以 使 用rememberMeParameter()方法定制。



通过开发者工具看到浏览器端存储了名为remember-me的Cookie。根据这个 Cookie的 value 在服务器端找到以前登录的 User。而且这个 Cookie 被设置为存储 2 个星期的时间

![image-20210813171036439](https://gitee.com/jobim/blogimage/raw/master/img/20210813171036.png)



# 实验八：查询数据库完成认证



## SpringSecurity 默认实现（了解）

builder.jdbcAuthentication().usersByUsernameQuery("tom");

在usersByUsernameQuery("tom")等方法中最终调用JdbcDaoImpl类的方法查询数据库。

![image-20210813173002614](https://gitee.com/jobim/blogimage/raw/master/img/20210813173002.png)



SpringSecurity 的默认实现已经将 SQL 语句硬编码在了 JdbcDaoImpl 类中。这种
情况下，我们有下面三种选择：

* 按照 JdbcDaoImpl 类中 SQL 语句设计表结构。
* 修改 JdbcDaoImpl 类的源码。
* 不使用 jdbcAuthentication()。



## 自定义数据库查询方式



自定义实现**UserDetailsService接口**的类并自动装配

```java
@Component
public class MyUserDetailsService implements UserDetailsService {
	
	@Autowired
	private JdbcTemplate jdbcTemplate;

	// 总目标：根据表单提交的用户名查询User对象，并装配角色、权限等信息
	@Override
	public UserDetails loadUserByUsername(
			String username  // 表单提交的用户名
	) throws UsernameNotFoundException {
		
		// 1.从数据库查询Admin对象
		String sql = "SELECT id,login_acct,user_pswd,user_name,email FROM t_admin WHERE login_acct=?";
		List<Admin> list = jdbcTemplate.query(sql, new BeanPropertyRowMapper<>(Admin.class), username);
		Admin admin = list.get(0);
		
		// 2.给Admin设置角色权限信息
		List<GrantedAuthority> authorities = new ArrayList<>();
		
		authorities.add(new SimpleGrantedAuthority("ROLE_ADMIN"));
		authorities.add(new SimpleGrantedAuthority("UPDATE"));
		
		// 3.把admin对象和authorities封装到UserDetails中
		String userpswd = admin.getUserPswd();
		
		return new User(username, userpswd, authorities);
	}

}
```



设置使用自定义 UserDetailsService 完成登录

```java
    @Autowired
    private MyUserDetailsService userDetailsService;

    //重写另外一个父类的方法，来设置登录系统的账号密码
    @Override
    protected void configure(AuthenticationManagerBuilder builder) throws Exception {
        builder.userDetailsService(userDetailsService);
    }
```



**注意：创建 SimpleGrantedAuthority 对象添加角色时需要手动在角色名称前加“ROLE_”前缀**





# 实验九：应用密码加密规则



**自定义类应用密码加密规则：**



自定义类实现 PasswordEncoder接口

* encode()方法对明文进行加密。
* matches()方法对明文加密后和密文进行比较



```java
@Component
public class MyPasswordEncoder implements PasswordEncoder {

	@Override
	public String encode(CharSequence rawPassword) {
		return privateEncode(rawPassword);
	}

	@Override
	public boolean matches(CharSequence rawPassword, String encodedPassword) {
		// 1.对明文密码进行加密
		String formPassword = privateEncode(rawPassword);
		// 2.声明数据库密码
		String databasePassword = encodedPassword;
		// 3.比较
		return Objects.equals(formPassword, databasePassword);
	}
	
	private String privateEncode(CharSequence rawPassword) {
		try {
			// 1.创建MessageDigest对象
			String algorithm = "MD5";
			MessageDigest messageDigest = MessageDigest.getInstance(algorithm);
			// 2.获取rawPassword的字节数组
			byte[] input = ((String)rawPassword).getBytes();
			// 3.加密
			byte[] output = messageDigest.digest(input);
			// 4.转换为16进制数对应的字符
			String encoded = new BigInteger(1, output).toString(16).toUpperCase();
			return encoded;
		} catch (NoSuchAlgorithmException e) {
			e.printStackTrace();
			return null;
		}
	}
}
```



在配置类中的 configure(AuthenticationManagerBuilder)方法中应用自定义密码加密规则

```java
@Autowired
private MyPasswordEncoder myPasswordEncoder;

@Override
protected void configure(AuthenticationManagerBuilder builder) throws Exception {
    builder.userDetailsService(userDetailsService).passwordEncoder(myPasswordEncoder);
}
```





**使用SpringSecurity中更强到的BCryptPasswordEncoder加密类**

```java
@Bean
public BCryptPasswordEncoder bCryptPasswordEncoder() {
    return new BCryptPasswordEncoder();
}

@Override
protected void configure(AuthenticationManagerBuilder builder) throws Exception {
    builder.userDetailsService(userDetailsService).passwordEncoder(bCryptPasswordEncoder());
}
```

