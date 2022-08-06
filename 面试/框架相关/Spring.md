

# 一、Spring概述

## 1.1 什么是spring?

Spring是**一个轻量级Java开发框架**，最早有**Rod Johnson**创建，目的是为了解决企业级应用开发的业务逻辑层和其他各层的耦合问题。它是一个分层的JavaSE/JavaEE full-stack（一站式）轻量级开源框架，为开发Java应用程序提供全面的基础架构支持。Spring负责基础架构，因此Java开发者可以专注于应用程序的开发

Spring最根本的使命是**解决企业级应用开发的复杂性，即简化Java开发**。

Spring可以做很多事情，它为企业级开发提供给了丰富的功能，但是这些功能的底层都依赖于它的两个核心特性，也就是**依赖注入（dependency injection，DI）**和**面向切面编程（aspect-oriented programming，AOP）**。

为了降低Java开发的复杂性，Spring采取了以下4种关键策略

- 基于POJO的轻量级和最小侵入性编程；
- 通过依赖注入和面向接口实现松耦合；
- 基于切面和惯例进行声明式编程；
- 通过切面和模板减少样板式代码。



## 1.2 Spring的优缺点是什么？

**优点：**

- **方便解耦，简化开发**
  通过Spring提供的IOC容器，可以将所有对象的创建和依赖关系的维护，交给Spring管理。集中管理对象，对象与对象之间的耦合度降低。
- **AOP编程的支持**
  Spring提供面向切面编程，在不修改代码的情况下可以对业务代码进行增强，可以方便的实现对程序进行权限拦截、运行监控等功能。
- **声明式事务的支持**
  只需要通过配置就可以完成对事务的管理，而无需手动编程。
- **方便程序的测试**
  Spring对Junit4支持，可以通过注解方便的测试Spring程序。

- **方便集成各种优秀框架**
  Spring不排斥各种优秀的开源框架，其内部提供了对各种优秀框架的直接支持（如：Struts、Hibernate、MyBatis等）。
- **降低JavaEE API的使用难度**
  Spring对JavaEE开发中非常难用的一些API（JDBC、JavaMail、远程调用等），都提供了封装，使这些API应用难度大大降低。

**缺点：**

- Spring明明一个很轻量级的框架，却给人感觉大而全
- Spring依赖反射，反射影响性能
- 简化开发，但如果想深入到底层了解就有不少难度。（上层使用越简单，底层封装得就越复制）



## Spring控制反转(IOC)



控制反转即IOC，就是将原本在程序中手动创建对象的控制权，交由 Spring 框架来管理。

**为什么叫控制反转？**

- **控制** ：指的是对象创建（实例化、管理）的权力
- **反转** ：控制权交给外部环境（Spring 框架、IoC 容器）



**控制反转(IoC)有什么作用？**

* 管理对象的创建和依赖关系的维护。

* 解耦，对象之间的耦合度或者说依赖程度降低
* IOC容器支持加载服务时的饿汉式初始化和懒加载



> 在实际项目中一个 Service 类可能依赖了很多其他的类，假如我们需要实例化这个 Service，你可能要每次都要搞清这个 Service 所有底层类的构造函数，这可能会把人逼疯。如果利用 IoC 的话，你只需要配置好，然后在需要的地方引用就行了，这大大增加了项目的可维护性且降低了开发难度。



## Spring IoC 的实现机制

**IOC的实现原理主要是xml 解析、工厂模式、反射**

```java
interface Fruit {
   public abstract void eat();
 }

class Apple implements Fruit {
    public void eat(){
        System.out.println("Apple");
    }
}

class Orange implements Fruit {
    public void eat(){
        System.out.println("Orange");
    }
}

class Factory {
    public static Fruit getInstance(String ClassName) {
        Fruit f=null;
        try {
            //通过反射创建对象
            f=(Fruit)Class.forName(ClassName).newInstance();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return f;
    }
}

class Client {
    public static void main(String[] a) {
        Fruit f=Factory.getInstance("io.github.dunwu.spring.Apple");
        if(f!=null){
            f.eat();
        }
    }
}
```





# Spring的AOP



## Spring中有哪些不同的通知类型

![image-20210929175347497](https://gitee.com/jobim/blogimage/raw/master/img/20210929175347.png)

![image-20210929174346347](https://gitee.com/jobim/blogimage/raw/master/img/20210929174346.png)



## 循环依赖



```
//1
private final Map<String, Object> singletonObjects = new ConcurrentHashMap(256);
//3
private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap(16);

private final Map<String, Object> earlySingletonObjects = new ConcurrentHashMap(16);
```



# 声明式事务





# SpringMVC







## 如何解决POST请求中文乱码问题，GET的又如何处理呢？

**解决post请求乱码问题：**

* 在web.xml中配置一个**CharacterEncodingFilter过滤器**，设置成utf-8；

  ```xml
  <filter>
      <filter-name>CharacterEncodingFilter</filter-name>
      <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
  
      <init-param>
          <param-name>encoding</param-name>
          <param-value>utf-8</param-value>
      </init-param>
  </filter>
  
  <filter-mapping>
      <filter-name>CharacterEncodingFilter</filter-name>
      <url-pattern>/*</url-pattern>
  </filter-mapping>
  ```

**get请求中文参数出现乱码解决方法有两个：**

* **修改tomcat配置文件conf/server.xml中添加编码与工程编码一致**

  ```
  <ConnectorURIEncoding="utf-8" connectionTimeout="20000" port="8080" protocol="HTTP/1.1" redirectPort="8443"/>
  ```

* **对参数进行重新编码：解码（iso-8859-1解码） 再编码（utf-8再编码） 再使用**

  ```java
  String UserID  = new String(request.getParameter("UserID").getBytes("iso-8859-1"), "utf-8");
  ```



## SpringMVC的执行流程

![image-20211020170642910](https://gitee.com/jobim/blogimage/raw/master/img/20211020170642.png)

1、所有请求，前端控制器（DispatcherServlet）收到请求，调用doDispatch进行处理
2、根据HandlerMapping中保存的请求映射信息找到，处理当前请求的，处理器执行链（包含拦截器）
3、根据当前处理器找到他的HandlerAdapter（处理器适配器）
4、拦截器的preHandle先执行
5、适配器执行目标方法，并返回ModelAndView
6、拦截器的postHandle执行
7、视图解析器ViewReslover 解析后返回具体 View 对象。

8、对 View 进行渲染视图（即将模型数据填充至视图中）。

9、把View返回给请求者（浏览器）



为什么使用了springboot

1. spring aop怎么理解，你在日常会用在哪些方面，举例

2. spring事务基于注解的方式能用于分布式吗

3. spring [@Transactional使用过程中踩过什么坑 ]()

    

  25、spring采用了哪些设计模式？ 

  26、spring创建和管理bean有几种方式？ 

  27、bean的作用域 

  Spring 核心理解 IOC理解 Bean的生命周期理解  常用注解有哪些  为什么要用Component Controller Service  都用Component不可以吗。。。 

  SpringMVC 入口工作流程说明。。。 

  SpringBoot 怎么用的 有什么好处  为什么要用。。。

1. springBoot和spring的区别
2. springBoot自动加载的功能说一说
3. spring的bean的生命周期说一下

4，Spring底层实现原理？ 

1.  springboot 最大的特点（自动配置），启动注解和配置文件（记不清了） 
2.  

 spring boot和spring的区别？ 

 对spring的理解与看法？ 

 • spring设计模式 
 • springMVC流程  
 • spring aop 和aspectj aop区别  

 • springMVC的发送请求处理过程 

  spring bean的生命周期 

 spring ioc和aop，aop原理，动态代理的原理，基于java的什么机制 
 springboot用过吗，有什么优点（我用的是ssm，只写过springboot demo就直接实话说了，简单说了一下springboot的优点） 

、Spring的AOP是什么，自己实现过吗 

 六、Spring的 IOC 和AOP 

  解释了之后问我控制反转(IOC)是啥意思，面试官的意思是IOC不是Spring中独有的概念，我就不道了 

  问我有没有用过AOP，我说简单的用来权限校验 

#### 框架（Spring）

1. Spring中事务的ACID四个属性指什么？
2. 事务的隔离属性有哪些？事务的传播属性有哪些？默认的是什么？
3. 你的[项目]()中有没有用到事务？怎么配置的？

-  了解过springboot吗 
-  了解springcloud 它的各个组件是怎么样的 

了解springboot吗 了解微服务吗 

springmvc常用的注解 

了解springcloud吗 讲讲你对spirngcloud几个组件的理解 

springboot的启动流程 

  10.SpringBoot[项目]()中是怎样用的，它的自动装配原理是什么

 在spring怎么用 

 aop和ioc注入bean什么什么的（问题叙述不上来了） 

 事务在spring中咋用的 

**SpringMVC的设计模式**

 \2.   Spring @Bean @Service @Component等注解的区别 

 7、springboot bean是单例的吗，想变多例的话怎么办 



 10、spring ioc、aop 

1. Spring Boot相比如Spring提供了哪些特性？ 
2. Spring Boot注解自动装配过程是什么样？ 
3. BeanFactory和FactoryBean有什么区别？ 
4. Bean相关可以进行扩展的接口？ 

1. spring boot自动装配实现原理 
2. Spring 解决循环依赖 

为什么要有spring事务传播？ 

 说一下spring的IOC和AOP？

 5.Spring事务 
 6.Spring的通知等级 

 Spring 核心理解 IOC理解 Bean的生命周期理解  常用注解有哪些  为什么要用Component Controller Service  都用Component不可以吗。。。 

  SpringMVC 入口工作流程说明。。。 

  SpringBoot 怎么用的 有什么好处  为什么要用。。。

springMVC的理解

spring ioc怎么理解，怎么实现