# 一、源码分析的入口

> 使用环境：SpringBoot2.4.11

* **Spring容器初始化入口，使用`AnnotationConfigApplicationContext`加载bean**

  ```java
  public static void main(String[] args)   {
     // 加载spring上下文
     AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(MainConfig.class);
  
     Car car =  context.getBean("car",Car.class);
     System.out.println(car.getName());
  }
  ```

  **MainConfig.class：**
  
  ```java
  @Configuration //作为配置类，替代 xml 配置文件
  @ComponentScan(basePackages = {"com.zb.demo.spring"})
  public class SpringConfig {
  }
  ```

* **AnnotationConfigApplicationContext的构造方法**

  ```java
  //根据参数类型可以知道，其实可以传入多个annotatedClasses，但是这种情况出现的比较少
  public AnnotationConfigApplicationContext(Class<?>... annotatedClasses) {
      //1、调用无参构造函数，会先调用父类GenericApplicationContext的构造函数,父类的构造函数里面就是初始化DefaultListableBeanFactory，并且赋值给beanFactory
      //2、本类的构造函数里面，初始化了一个读取器：AnnotatedBeanDefinitionReader read，一个扫描器ClassPathBeanDefinitionScanner scanner
      //scanner的用处不是很大，它仅仅是在我们外部手动调用 .scan 等方法才有用，常规方式是不会用到scanner对象的
      this();
      //3、把传入的类进行注册，这里有两个情况，
      //传入传统的配置类 和 传入bean（虽然一般没有人会这么做）
      //看到后面会知道spring把传统的带上@Configuration的配置类称之为FULL配置类，不带@Configuration的称之为Lite配置类
      //但是我们这里先把带上@Configuration的配置类称之为传统配置类，不带的称之为普通bean
      register(annotatedClasses);
      //刷新
      refresh();
  }
  ```

  **构造方法说明：**

  * 这是一个有参的构造方法，可以接收多个配置类，不过一般情况下，只会传入一个配置类。

  * 这个配置类有两种情况，
    * 一种是传统意义上的带上@Configuration注解的配置类，
    * 另一种是没有带上@Configuration，但是带有@Component，@Import，@ImportResouce，@Service，@ComponentScan等注解的配置类，在Spring内部把前者称为Full配置类，把后者称之为Lite配置类。在本源码分析中，有些地方也把Lite配置类称为**普通Bean**。



## 二、通过this()调用AnnotationConfigApplicationContext的无参构造函数

* **调用AnnotationConfigApplicationContext类的无参构造方法**

  ```java
  public class AnnotationConfigApplicationContext extends GenericApplicationContext implements AnnotationConfigRegistry {
  
      //注解bean定义读取器，主要作用是用来读取被注解的了bean
      private final AnnotatedBeanDefinitionReader reader;
  
      //扫描器，它仅仅是在我们外部手动调用 .scan 等方法才有用，常规方式是不会用到scanner对象的
      private final ClassPathBeanDefinitionScanner scanner;
  
      public AnnotationConfigApplicationContext() {
          //会隐式调用GenericApplicationContext父类的构造方法，初始化DefaultListableBeanFactory
  
          //初始化一个Bean读取器
          this.reader = new AnnotatedBeanDefinitionReader(this);
  
          //初始化一个扫描器，它仅仅是在我们外部手动调用 .scan 等方法才有用，常规方式是不会用到scanner对象的
          this.scanner = new ClassPathBeanDefinitionScanner(this);
      }
  }
  ```
  
  首先无参构造方法中就是对读取器`reader`和扫描器`scanner`进行了实例化
  
  * reader的类型是`AnnotatedBeanDefinitionReader`，可以看出它是一个 “**打了注解的Bean定义读取器**”，

  * scanner的类型是`ClassPathBeanDefinitionScanner`，它仅仅是在外面手动调用.scan方法，或者调用参数为String的构造方法，传入需要扫描的包名才会用到，像这样方式传入的配置类是不会用到这个scanner对象的。

* **其中会隐式调用父类的构造方法：实例化工厂DefaultListableBeanFactory**

  ```java
  public class GenericApplicationContext extends AbstractApplicationContext implements BeanDefinitionRegistry {
  
      private final DefaultListableBeanFactory beanFactory;
  
      @Nullable
      private ResourceLoader resourceLoader;
  
      private boolean customClassLoader = false;
  
      private final AtomicBoolean refreshed = new AtomicBoolean();
  
  
      /**
       * Create a new GenericApplicationContext.
       * @see #registerBeanDefinition
       * @see #refresh
       */
      public GenericApplicationContext() {
          this.beanFactory = new DefaultListableBeanFactory();
      }
  }
  ```

  DefaultListableBeanFactory的关系图：

  ![image-20211002112828624](https://gitee.com/jobim/blogimage/raw/master/img/20211002112828.png)

  DefaultListableBeanFactory 实际上也是一个集大成者。在 Spring 中，针对 Bean 的不同操作都有不同的接口进行规范，每个接口都有自己对应的实现，最终在 DefaultListableBeanFactory 中将所有的实现汇聚到一起



**3、实例化建BeanDefinition读取器： AnnotatedBeanDefinitionReader：**

其主要做了2件事情

1.注册内置BeanPostProcessor

2.注册相关的BeanDefinition









# 加载IOC容器的两种方式

> ApplicationContext 的继承关系
>
> ![image-20211002111413183](https://gitee.com/jobim/blogimage/raw/master/img/20211002111413.png)

* ClassPathXmlApplicationContext 通过XML配置

  ```java
  ApplicationContext context = new ClassPathXmlApplicationContext(“applicationContext.xml");
  ```

* AnnotationConfigApplicationContext 通过java config 类配置

  ```java
  @Configuration //作为配置类，替代 xml 配置文件
  @ComponentScan(basePackages = {"com.zb"})
  public class SpringConfig {
  }
  
  @Test
  public void testService2() {
  	//加载配置类
  	ApplicationContext context = new AnnotationConfigApplicationContext(SpringConfig.class);
  	UserService userService = context.getBean("userService", UserService.class);
  	System.out.println(userService);
  	userService.add();
  }
  ```



## BeanFactory和ApplicationContext的区别



ApplicationContext是BeanFactory的子接口；BeanFactory：bean工厂接口；负责创建bean实例；容器里面保存的所有单例bean其实是一个map；     Spring最底层的接口；ApplicationContext：是容器接口；更多的负责容器功能的实现；（可以基于beanFactory创建好的对象之上完成强大的容器）        容器可以从map获取这个bean，并且aop。di。在ApplicationContext接口的下的这些类里面；
BeanFactory最底层的接口，ApplicationContext留给程序员使用的ioc容器接口；ApplicationContext是BeanFactory的子接口；ApplicationContext





**为什么要使用三级缓存呢？二级缓存能解决循环依赖吗？**

* 如果要**使用二级缓存解决循环依赖，意味着所有Bean在实例化后就要完成AOP代理**，这样违背了Spring设计的原则，Spring在设计之初就是通过AnnotationAwareAspectJAutoProxyCreator这个后置处理器来在Bean生命周期的最后一步来完成AOP代理，而不是在实例化后就立马进行AOP代理。
* ![image-20211008005725779](https://gitee.com/jobim/blogimage/raw/master/img/20211008005725.png)



# FactoryBean和BeanFactory



在实例化剩余的单实例bean的时候会进行一个判断，如果该类对象实现了FactoryBean接口，则创建的实例通过调用getObject方法创建



















