# 一、Spring Security介绍
Spring 是非常流行和成功的 Java 应用开发框架，Spring Security 正是 Spring 家族中的成员。Spring Security 基于 Spring 框架，提供了一套 Web 应用安全性的完整解决方案。


正如你可能知道的关于安全方面的两个主要区域是“**认证**”和“**授权**”（或者访问控制），一般来说，Web 应用的安全性包括**用户认证（Authentication）和用户授权（Authorization）**两个部分，这两点也是 Spring Security 重要核心功能。

（1）用户认证指的是：验证某个用户是否为系统中的合法主体，也就是说用户能否访问该系统。用户认证一般要求用户提供用户名和密码。系统通过校验用户名和密码来完成认证过程。**通俗点说就是系统认为用户是否能登录**

（2）用户授权指的是验证某个用户是否有权限执行某个操作。在一个系统中，不同用户所具有的权限是不同的。比如对一个文件来说，有的用户只能进行读取，而有的用户可以进行修改。一般来说，系统会为不同的用户分配不同的角色，而每个角色则对应一系列的权限。**通俗点讲就是系统判断用户是否有权限去做某些事情**。



# 二、入门项目

## 1.1 配置文件中设置用户信息



>**对登录的用户名/密码进行配置，有三种不同的方式**
>1. **在 `application` 配置文件中声明**
>2. **在 `java` 代码配置在内存里**
>3. **通过获取 `数据库`**



1. **创建maven项目，导入依赖**

    ```xml
    <!--加入SpringBoot-->
    <parent>
        <artifactId>spring-boot-starter-parent</artifactId>
        <groupId>org.springframework.boot</groupId>
        <version>2.5.3</version>
    </parent>
    
    <dependencies>
        <!--web开发相关依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    
        <!--spring security-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
    
    </dependencies>
    
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
    ```

2. **创建应用启动类**

   ```java
   @SpringBootApplication
   public class FirstApplication {
   
       public static void main(String[] args) {
           SpringApplication.run(FirstApplication.class, args);
       }
   }
   ```

3. **创建Controller，接收请求**

   ```java
   @RestController
   public class HelloSecurityController {
       @RequestMapping("/hello")
       public String sayHello() {
           return "Hello Spring Security 安全管理框架";
       }
   }
   ```



> - **启动后访问 `localhost:8080/hello`会自动跳到 `localhost:8080/login`**
> - **需要登录后才能访问 `/hello`**
>
>   ![image-20210807123240137](https://gitee.com/jobim/blogimage/raw/master/img/image-20210807123240137.png)
>
> - **默认情况下用户名是 `user` ,而密码会在项目启动时 `控制台` 打印出一串随机 `字符串`,这就是密码.每次启动项目,密码都不一样**
>
>   ![image-20210807123220386](https://gitee.com/jobim/blogimage/raw/master/img/image-20210807123220386.png)



4. **在配置文件中定义用户名和密码**

   ```yml
   spring:
     security:
       user:
         name: zb
         password: 123
   ```

   此时登录就可以使用zb，123登陆了

   

* **也可以设置关闭验证**，只需要修改启动类

  ```java
  //排除Secuirty的配置，让他不启用
  @SpringBootApplication(exclude = {SecurityAutoConfiguration.class})
  public class FirstApplication {
  
      public static void main(String[] args) {
          SpringApplication.run(FirstApplication.class, args);
      }
  }
  ```
  
  

## 1.2 使用内存中的信息认证



创建一个`SecurityConfig`配置类,继承 `WebSecurityConfigurerAdapter`并重写`protected void configure(AuthenticationManagerBuilder auth)`方法

```java
@Configuration
@EnableWebSecurity //表示启用 spring security 安全框架的功能
public class MyWebSecurityConfig extends WebSecurityConfigurerAdapter {

    //在方法中配置 用户和密码的信息，作为登录的数据
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        PasswordEncoder pe = passwordEncoder();
        //可以使用zhangsan,123登录
        auth.inMemoryAuthentication()
                .withUser("zhangsan")
                .password(pe.encode("123"))
                .roles();
        //可以使用lisi,123登录
        auth.inMemoryAuthentication().withUser("lisi")
                .password(pe.encode("123"))
                .roles();
        //可以使用admin,123登录
        auth.inMemoryAuthentication()
                .withUser("admin")
                .password(pe.encode("123"))
                .roles();
    }

    //创建密码的加密类
    @Bean
    public PasswordEncoder passwordEncoder() {
        //创建PasswordEncoder的实现类， 实现类是加密算法
        return new BCryptPasswordEncoder();
    }

}
```



注意：spring security 5 版本要求密码比较加密，必须给密码进行加密，否则报错：

![image-20210807144509054](https://gitee.com/jobim/blogimage/raw/master/img/image-20210807144509054.png)

加密步骤：

> 1、创建用来加密的实现类
>
> ```java
> //创建密码的加密类
> @Bean
> public PasswordEncoder passwordEncoder() {
>  //创建PasswordEncoder的实现类， 实现类是加密算法
>  return new BCryptPasswordEncoder();
> }
> ```
>
> 2. 给密码进行加密
>
> 
>
> ```java
> PasswordEncoder pe = passwordEncoder();
> pe.encode("123456")
> ```



这样通过java将用户信息配置在内存中，即可成功登录。

## 1.3 基于角色Role的身份认证

同一个用户可以有不同的角色。同时可以开启对方法级别的认证



**基于角色的实现步骤：**



1. 设置用户的角色，定义配置类继承 WebSecurityConfigurerAdapter重写 configure 方法，指定用户的 role。并在类上标记注解`@EnableGlobalMethodSecurity(prePostEnabled = true)`，表示启动方法级别的注解

   ```java
   /**
    * @EnableGlobalMethodSecurity:启用方法级别的认证
    *      prePostEnabled：boolean 默认是false
    *          true:表示可以使用@PreAuthorize注解 和 @PostAuthorize
    */
   @Configuration
   @EnableWebSecurity //表示启用 spring security 安全框架的功能
   @EnableGlobalMethodSecurity(prePostEnabled = true)
   public class MyWebSecurityConfig extends WebSecurityConfigurerAdapter {
   
       //在方法中配置 用户和密码的信息，作为登录的数据
       @Override
       protected void configure(AuthenticationManagerBuilder auth) throws Exception {
           PasswordEncoder pe = passwordEncoder();
           //定义两个角色：normal，admin
           auth.inMemoryAuthentication()
                   .withUser("zhangsan")
                   .password(pe.encode("123"))
                   .roles("noraml");
           auth.inMemoryAuthentication()
                   .withUser("lisi")
                   .password(pe.encode("123"))
                   .roles("normal");
           auth.inMemoryAuthentication()
                   .withUser("admin")
                   .password(pe.encode("123"))
                   .roles("admin","normal");
       }
   
       //创建密码的加密类
       @Bean
       public PasswordEncoder passwordEncoder() {
           //创建PasswordEncoder的实现类， 实现类是加密算法
           return new BCryptPasswordEncoder();
       }
   
   }
   
   ```

   

   

2. 在处理器方法使用`@PreAuthorize`加入角色的信息，指定方法可以访问的角色列表。

   

   ```java
   @RestController
   public class HelloController {
       
       //指定 normal 和admin 角色都可以访问的方法
       @RequestMapping("/helloUser")
       @PreAuthorize(value = "hasAnyRole('admin','normal')")
       public String helloCommonUser(){
           return "Hello 拥有normal, admin角色的用户";
       }
   
       //指定admin角色的访问方法
       @RequestMapping("/helloAdmin")
       @PreAuthorize("hasAnyRole('admin')")
       public String helloAdmin(){
           return "Hello admin角色的用户可以访问";
       }
   }
   ```

   

此时这有登录admin的角色才能访问`http://localhost:8080/helloAdmin`






# spring specurity 中相关接口和类



## UserDetailsService

常在spring security应用中，我们会自定义一个UserDetailsService来实现UserDetailsService接口，并实现其loadUserByUsername(final String login);方法。我们在实现loadUserByUsername方法的时候，就可以通过查询数据库（或者是缓存、或者是其他的存储形式）来获取用户信息，然后组装成一个UserDetails,(通常是一个org.springframework.security.core.userdetails.User，它继承自UserDetails) 并返回。

在实现loadUserByUsername方法的时候，如果我们通过查库没有查到相关记录，需要抛出一个异常来告诉spring security来“善后”。这个异常是org.springframework.security.core.userdetails.UsernameNotFoundException。




## UserDetails





1. UserDetails存储的就是用户信息

   ![image-20210808144638913](https://gitee.com/jobim/blogimage/raw/master/img/image-20210808144638913.png)

   

## PasswordEncoder






# 三、基于角色的权限





## 3.2 认证和授权





## 3.3 RBAC



## 3.4 Spring Security中认证的接口和类













设计用户角色表：

![image-20210807185025903](https://gitee.com/jobim/blogimage/raw/master/img/image-20210807185025903.png)





![image-20210807185249771](https://gitee.com/jobim/blogimage/raw/master/img/image-20210807185249771.png)





![image-20210807185258390](https://gitee.com/jobim/blogimage/raw/master/img/image-20210807185258390.png)







通过程序初始化SysUser账号数据



手工初始化角色数据：

![image-20210807193624616](https://gitee.com/jobim/blogimage/raw/master/img/image-20210807193624616.png)





设置url权限定义，访问index.html不需要认证









# 实战案例（jdbc、Ajax、验证码）



**实现效果：**





项目结构：

![image-20210808155147644](https://gitee.com/jobim/blogimage/raw/master/img/image-20210808155147644.png)





## 准备工作



1. **导入依赖**

   ```xml
   <!--加入SpringBoot-->
     <parent>
       <artifactId>spring-boot-starter-parent</artifactId>
       <groupId>org.springframework.boot</groupId>
       <version>2.5.3</version>
     </parent>
   
     <dependencies>
   
       <dependency>
         <groupId>junit</groupId>
         <artifactId>junit</artifactId>
         <version>4.11</version>
         <scope>test</scope>
       </dependency>
   
       <!--web开发相关依赖-->
       <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-web</artifactId>
       </dependency>
   
       <!--spring security-->
       <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-security</artifactId>
       </dependency>
   
       <!--spring-jdbc-->
       <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-jdbc</artifactId>
       </dependency>
   
       <!--mysql驱动-->
       <dependency>
         <groupId>mysql</groupId>
         <artifactId>mysql-connector-java</artifactId>
         <version>5.1.9</version>
       </dependency>
   
       <!--mybatis-->
       <dependency>
         <groupId>org.mybatis.spring.boot</groupId>
         <artifactId>mybatis-spring-boot-starter</artifactId>
         <version>2.0.1</version>
       </dependency>
   
       <!--jackson-->
       <dependency>
         <groupId>com.fasterxml.jackson.core</groupId>
         <artifactId>jackson-core</artifactId>
       </dependency>
       <dependency>
         <groupId>com.fasterxml.jackson.core</groupId>
         <artifactId>jackson-databind</artifactId>
       </dependency>
   
     </dependencies>
   
     <build>
       <plugins>
         <plugin>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-maven-plugin</artifactId>
         </plugin>
       </plugins>
     </build>
   ```

   



1. **定义用户，角色，角色关系表并添加数据**

   用户信息表：`sys_user`
       ![image-20210808155601955](https://gitee.com/jobim/blogimage/raw/master/img/image-20210808155601955.png)

   角色表：`sys_role`
       ​	![image-20210808155625375](https://gitee.com/jobim/blogimage/raw/master/img/image-20210808155625375.png)

   

   用户-角色关系表：`sys_user_role`

   ![image-20210808155639043](https://gitee.com/jobim/blogimage/raw/master/img/image-20210808155639043.png)

   

2. **创建实体类**

   定义用户信息类，实现SpringSecurity框架中的UserDetails

   ```java
   public class SysUser implements UserDetails {
   
       private Integer id;
       private String username;
       private String password;
       private String realname;
   
       private boolean isExpired; //是否过期
       private boolean isLocked; //是否锁定
       private boolean isCredentials;  //是否有凭证
       private boolean isEnabled; //是否能用
   
       private Date createTime;
       private Date loginTime;
   
       private List<GrantedAuthority> authorities;
   
   
       @Override
       public Collection<? extends GrantedAuthority> getAuthorities() {
           return authorities;
       }
   
       @Override
       public String getPassword() {
           return password;
       }
   
       @Override
       public String getUsername() {
           return username;
       }
   
       @Override
       public boolean isAccountNonExpired() {
           return isExpired;
       }
   
       @Override
       public boolean isAccountNonLocked() {
           return isLocked;
       }
   
       @Override
       public boolean isCredentialsNonExpired() {
           return isCredentials;
       }
   
       @Override
       public boolean isEnabled() {
           return isEnabled;
       }
       //set、get方法、构造方法
   }
   ```

   SysRole 类：

   ```java
   public class SysRole {
       private Integer id;
       private String name;
       private String memo;
   }
   ```

   

3. **创建mapper文件**

   SysUserMapper

   ```java
   public interface SysUserMapper {
   
       int insertSysUser(SysUser user);
   
       //根据账号名称，获取用户信息
       SysUser selectSysUser(String username);
   
   }
   ```

   SysRoleMapper

   ```java
   public interface SysRoleMapper {
       //通过用户id查询用户角色
       List<SysRole> selectRoleByUser(Integer userId);
   }
   ```

   mapper.xml省略



## 创建UserDetailsService接口的实现类

根据 username 从数据查询账号信息 SysUser 对象。在根据 user 的 id， 去查 Sys_Role 表获取 Role 信息

```java
@Service("JdbcUserDetailsService")
public class JdbcUserDetailsService implements UserDetailsService {

    @Autowired
    private SysUserMapper userMapper;

    @Autowired
    private SysRoleMapper roleMapper;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        //1. 根据username 获取SysUser
        SysUser user  = userMapper.selectSysUser(username);
        System.out.println("loadUserByUsername user:"+user);
        if( user != null){
            //2. 根据userid的，获取role
            List<SysRole> roleList = roleMapper.selectRoleByUser(user.getId());
            System.out.println("roleList:"+ roleList);

            List<GrantedAuthority> authorities = new ArrayList<>();
            String roleName = "";
            for (SysRole role : roleList) {
                roleName  = role.getName();
                GrantedAuthority authority = new SimpleGrantedAuthority("ROLE_"+roleName);
                authorities.add(authority);
            }
            user.setAuthorities(authorities);
            return user;
        }
        return user;
    }
}
```





好的博客：

[SpringSecurity系列](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzI1NDY0MTkzNQ==&action=getalbum&album_id=1319828555819286528&scene=173&from_msgid=2247489405&from_itemidx=2&count=3&nolastread=1#wechat_redirect)

