# 一、源码分析的入口

> 使用环境：SpringBoot2.1.16.RELEASE

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



# 二、this()调用构造函数

**调用AnnotationConfigApplicationContext类的无参构造方法**

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

* 首先无参构造方法中就是对读取器`reader`和扫描器`scanner`进行了实例化
  * reader的类型是`AnnotatedBeanDefinitionReader`，可以看出它是一个 “**打了注解的Bean定义读取器**”，
  * scanner的类型是`ClassPathBeanDefinitionScanner`，它仅仅是在外面手动调用.scan方法，或者调用参数为String的构造方法，传入需要扫描的包名才会用到，像这样方式传入的配置类是不会用到这个scanner对象的。

## 2.1 super()隐式调用父类的构造方法

* **其中会隐式调用父类GenericApplicationContext的构造方法：实例化工厂DefaultListableBeanFactory**

  ```java
  public class GenericApplicationContext extends AbstractApplicationContext implements BeanDefinitionRegistry {
  
      private final DefaultListableBeanFactory beanFactory;
  
      @Nullable
      private ResourceLoader resourceLoader;
  
      private boolean customClassLoader = false;
  
      private final AtomicBoolean refreshed = new AtomicBoolean();
  
  
      /**
       * Create a new GenericApplicationContext.
       */
  	public GenericApplicationContext() {
  		/**
  		 * 调用父类的构造函数,为ApplicationContext spring上下文对象初始beanFactory
  		 * 为啥是DefaultListableBeanFactory？我们去看BeanFactory接口的时候
  		 * 发现DefaultListableBeanFactory是最底层的实现，功能是最全的
  		 */
  		this.beanFactory = new DefaultListableBeanFactory();
  	}
  }
  ```

  > 问题: BeanFactory有很多, 为什么初始化的时候选择DefaultListableBeanFactory呢?
  >
  > DefaultListableBeanFactory的关系图：
  >
  > ![image-20211002112828624](https://gitee.com/jobim/blogimage/raw/master/img/20211002112828.png)
  >
  > 通过观察, 我们发现, DefaultListableBeanFactory实现了各种各样的BeanFactory接口, 同时还是先了BeanDefinitionRegistry接口。
  >
  > 也就是说, DefaultListableBeanFactory不仅仅有BeanFactory的能力, 同时还有BeanDefinitionRegistry的能力. 它的功能是最全的。所以，我们使用的是一个功能非常强大的类Bean工厂类

## 2.2 初始化AnnotatedBeanDefinitionReader

```java
this.reader = new AnnotatedBeanDefinitionReader(this);
```

```java
public AnnotatedBeanDefinitionReader(BeanDefinitionRegistry registry, Environment environment) {
   Assert.notNull(registry, "BeanDefinitionRegistry must not be null");
   Assert.notNull(environment, "Environment must not be null");
   //把ApplicationContext对象赋值给AnnotatedBeanDefinitionReader
   this.registry = registry;
   //用户处理条件注解 @Conditional os.name
   this.conditionEvaluator = new ConditionEvaluator(registry, environment, null);
   //注册一些内置的后置处理器
   AnnotationConfigUtils.registerAnnotationConfigProcessors(this.registry);
}
```

注册一些内置的后置处理器：AnnotationConfigUtils.registerAnnotationConfigProcessors(this.registry);

```java
public static Set<BeanDefinitionHolder> registerAnnotationConfigProcessors(
		BeanDefinitionRegistry registry, @Nullable Object source) {

	DefaultListableBeanFactory beanFactory = unwrapDefaultListableBeanFactory(registry);
	if (beanFactory != null) {
		if (!(beanFactory.getDependencyComparator() instanceof AnnotationAwareOrderComparator)) {
			//注册了实现Order接口的排序器
			beanFactory.setDependencyComparator(AnnotationAwareOrderComparator.INSTANCE);
		}
		//设置@AutoWired的候选的解析器：ContextAnnotationAutowireCandidateResolver
		// getLazyResolutionProxyIfNecessary方法，它也是唯一实现。
		//如果字段上带有@Lazy注解，表示进行懒加载 Spring不会立即创建注入属性的实例，而是生成代理对象，来代替实例
		if (!(beanFactory.getAutowireCandidateResolver() instanceof ContextAnnotationAutowireCandidateResolver)) {
			beanFactory.setAutowireCandidateResolver(new ContextAnnotationAutowireCandidateResolver());
		}
	}

	Set<BeanDefinitionHolder> beanDefs = new LinkedHashSet<>(8);

	/**
	 * 1.为我们容器中注册了解析我们配置类的后置处理器ConfigurationClassPostProcessor
	 * 名字叫:org.springframework.context.annotation.internalConfigurationAnnotationProcessor
	 */
	if (!registry.containsBeanDefinition(CONFIGURATION_ANNOTATION_PROCESSOR_BEAN_NAME)) {
		RootBeanDefinition def = new RootBeanDefinition(ConfigurationClassPostProcessor.class);
		def.setSource(source);
		beanDefs.add(registerPostProcessor(registry, def, CONFIGURATION_ANNOTATION_PROCESSOR_BEAN_NAME));
	}

	/**
	 * 2.为我们容器中注册了处理@Autowired 注解的处理器AutowiredAnnotationBeanPostProcessor
	 * 名字叫:org.springframework.context.annotation.internalAutowiredAnnotationProcessor
	 */
	if (!registry.containsBeanDefinition(AUTOWIRED_ANNOTATION_PROCESSOR_BEAN_NAME)) {
		RootBeanDefinition def = new RootBeanDefinition(AutowiredAnnotationBeanPostProcessor.class);
		def.setSource(source);
		beanDefs.add(registerPostProcessor(registry, def, AUTOWIRED_ANNOTATION_PROCESSOR_BEAN_NAME));
	}

	/**
	 * 3.为我们容器注册处理JSR250规范的注解处理器CommonAnnotationBeanPostProcessor
	 * org.springframework.context.annotation.internalCommonAnnotationProcessor
	 */
	if (jsr250Present && !registry.containsBeanDefinition(COMMON_ANNOTATION_PROCESSOR_BEAN_NAME)) {
		RootBeanDefinition def = new RootBeanDefinition(CommonAnnotationBeanPostProcessor.class);
		def.setSource(source);
		beanDefs.add(registerPostProcessor(registry, def, COMMON_ANNOTATION_PROCESSOR_BEAN_NAME));
	}

	/**
	 * 4.处理jpa注解的处理器org.springframework.orm.jpa.support.PersistenceAnnotationBeanPostProcessor
	 */
	if (jpaPresent && !registry.containsBeanDefinition(PERSISTENCE_ANNOTATION_PROCESSOR_BEAN_NAME)) {
		RootBeanDefinition def = new RootBeanDefinition();
		try {
			def.setBeanClass(ClassUtils.forName(PERSISTENCE_ANNOTATION_PROCESSOR_CLASS_NAME,
					AnnotationConfigUtils.class.getClassLoader()));
		}
		catch (ClassNotFoundException ex) {
			throw new IllegalStateException(
					"Cannot load optional framework class: " + PERSISTENCE_ANNOTATION_PROCESSOR_CLASS_NAME, ex);
		}
		def.setSource(source);
		beanDefs.add(registerPostProcessor(registry, def, PERSISTENCE_ANNOTATION_PROCESSOR_BEAN_NAME));
	}

	/**
	 * 5.处理监听方法的注解@EventListener解析器EventListenerMethodProcessor
	 */
	if (!registry.containsBeanDefinition(EVENT_LISTENER_PROCESSOR_BEAN_NAME)) {
		RootBeanDefinition def = new RootBeanDefinition(EventListenerMethodProcessor.class);
		def.setSource(source);
		beanDefs.add(registerPostProcessor(registry, def, EVENT_LISTENER_PROCESSOR_BEAN_NAME));
	}

	/**
	 * 6.注册事件监听器工厂
	 */
	if (!registry.containsBeanDefinition(EVENT_LISTENER_FACTORY_BEAN_NAME)) {
		RootBeanDefinition def = new RootBeanDefinition(DefaultEventListenerFactory.class);
		def.setSource(source);
		beanDefs.add(registerPostProcessor(registry, def, EVENT_LISTENER_FACTORY_BEAN_NAME));
	}

	return beanDefs;
}
```

**在这里注册了6个后置处理器的Bean定义：**

![image-20211118114621517](https://gitee.com/jobim/blogimage/raw/master/img/20211118114621.png)



## 2.3 初始化bean定义扫描ClassPathBeanDefinitionScanner

```java
this.scanner = new ClassPathBeanDefinitionScanner(this);
```

* **创建BeanDefinition扫描器，可以用来扫描包或者类，进而转换为bd**
* **Spring默认的扫描包不是这个scanner对象，而是自己new的一个ClassPathBeanDefinitionScanner，Spring在执行后置处理器ConfigurationClassPostProcessor时, 去扫描包时会new一个ClassPathBeanDefinitionScanner**
* **这里的scanner仅仅是为了程序员可以手动调用，AnnotationConfigApplicationContext对象的scan方法, 通过调用context.scan("package name");扫描处理配置类。**

比如，可以这样使用：

```java
public static void main(String[] args) {
   // 第一步: 通过AnnotationConfigApplicationContext读取一个配置类
   AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(MainStarter.class);
   context.scan("package name");
   Car car = (Car) context.getBean("car");
   System.out.println(car.getName());
   context.close();
}
```





# 三、register(annotatedClasses)注册配置类为BeanDefinition

```java
register(annotatedClasses);
```

**最终调用doRegisterBean(beanClass, null, null, null)方法将配置类注册成bean定义**

```java
<T> void doRegisterBean(Class<T> annotatedClass, @Nullable Supplier<T> instanceSupplier, @Nullable String name,
						@Nullable Class<? extends Annotation>[] qualifiers, BeanDefinitionCustomizer... definitionCustomizers) {
	//AnnotatedGenericBeanDefinition可以理解为一种数据结构，是用来描述Bean的，这里的作用就是把传入的标记了注解的类
	//转为AnnotatedGenericBeanDefinition数据结构，里面有一个getMetadata方法，可以拿到类上的注解
	AnnotatedGenericBeanDefinition abd = new AnnotatedGenericBeanDefinition(annotatedClass);

	//判断是否需要跳过注解，spring中有一个@Condition注解，当不满足条件，这个bean就不会被解析
	if (this.conditionEvaluator.shouldSkip(abd.getMetadata())) {
		return;
	}

	abd.setInstanceSupplier(instanceSupplier);

	//解析bean的作用域，如果没有设置的话，默认为单例
	ScopeMetadata scopeMetadata = this.scopeMetadataResolver.resolveScopeMetadata(abd);
	abd.setScope(scopeMetadata.getScopeName());

	//获得beanName
	String beanName = (name != null ? name : this.beanNameGenerator.generateBeanName(abd, this.registry));

	//解析通用注解，填充到AnnotatedGenericBeanDefinition，解析的注解为Lazy，Primary，DependsOn，Role，Description
	AnnotationConfigUtils.processCommonDefinitionAnnotations(abd);

	//限定符处理，不是特指@Qualifier注解，也有可能是Primary,或者是Lazy，或者是其他（理论上是任何注解，这里没有判断注解的有效性），如果我们在外面，以类似这种
	//AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(Appconfig.class);常规方式去初始化spring，
	//qualifiers永远都是空的，包括上面的name和instanceSupplier都是同样的道理
	//但是spring提供了其他方式去注册bean，就可能会传入了
	if (qualifiers != null) {
		//可以传入qualifier数组，所以需要循环处理
		for (Class<? extends Annotation> qualifier : qualifiers) {
			//Primary注解优先
			if (Primary.class == qualifier) {
				abd.setPrimary(true);
			}
			//Lazy注解
			else if (Lazy.class == qualifier) {
				abd.setLazyInit(true);
			}
			//其他，AnnotatedGenericBeanDefinition有个Map<String,AutowireCandidateQualifier>属性，直接push进去
			else {
				abd.addQualifier(new AutowireCandidateQualifier(qualifier));
			}
		}
	}

	for (BeanDefinitionCustomizer customizer : definitionCustomizers) {
		customizer.customize(abd);
	}

	//这个方法用处不大，就是把AnnotatedGenericBeanDefinition数据结构和beanName封装到一个对象中
	BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(abd, beanName);

	definitionHolder = AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry);

	//注册，最终会调用DefaultListableBeanFactory中的registerBeanDefinition方法去注册，
	//DefaultListableBeanFactory维护着一系列信息，比如beanDefinitionNames，beanDefinitionMap
	//beanDefinitionNames是一个List<String>,用来保存beanName
	//beanDefinitionMap是一个Map,用来保存beanName和beanDefinition
	BeanDefinitionReaderUtils.registerBeanDefinition(definitionHolder, this.registry);
}
```



在这里又要说明下，以常规方式去注册配置类，此方法中除了第一个参数，其他参数都是默认值。

1. 通过AnnotatedGenericBeanDefinition的构造方法，获得配置类的BeanDefinition。
2. 判断需不需要跳过注册，Spring中有一个@Condition注解，如果不满足条件，就会跳过这个类的注册。
3. 然后是解析作用域，如果没有设置的话，默认为单例。
4. 获得BeanName。
5. **解析通用注解，填充到AnnotatedGenericBeanDefinition，解析的注解为Lazy，Primary，DependsOn，Role，Description。**
6. 限定符处理，不是特指@Qualifier注解，也有可能是Primary，或者是Lazy，或者是其他（理论上是任何注解，这里没有判断注解的有效性）。
7. 把AnnotatedGenericBeanDefinition数据结构和beanName封装到一个对象中（这个不是很重要，可以简单的理解为方便传参）。
8. **注册，最终会调用DefaultListableBeanFactory中的registerBeanDefinition方法去注册**



# 四、refresh()

`refresh()`是 Spring 最核心的方法，没有之一，上帝就是用这个方法创造了 Spring 的世界。这是一个同步方法，用`synchronized`关键字来实现的。该方法包含以下12个核心方法。**用来加载活刷新Spring配置，使配置生效**

```java
@Override
public void refresh() throws BeansException, IllegalStateException {
	synchronized (this.startupShutdownMonitor) {
		//1.准备刷新上下文环境
		prepareRefresh();

		//2.获取告诉子类初始化Bean工厂不同工厂不同实现
		ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

		//还是一些准备工作，添加了两个后置处理器：ApplicationContextAwareProcessor，ApplicationListenerDetector
        //还设置了忽略自动装配和允许自动装配 的接口,如果不存在某个bean的时候，spring就自动注册singleton bean
        //还设置了bean表达式解析器等
		prepareBeanFactory(beanFactory);

		try {
			//4.留个子类去实现该接口
			postProcessBeanFactory(beanFactory);

			//5.调用我们的bean工厂的后置处理器. 1. 会在此将class扫描成beanDefinition  2.bean工厂的后置处理器调用
			invokeBeanFactoryPostProcessors(beanFactory);

			//6.注册我们bean的后置处理器
			registerBeanPostProcessors(beanFactory);

			//7.初始化国际化资源处理器.
			initMessageSource();

			//8.创建事件多播器
			initApplicationEventMulticaster();

			//9.这个方法同样也是留个子类实现的springboot也是从这个方法进行启动tomcat的.
			onRefresh();

			//10.把我们的事件监听器注册到多播器上
			registerListeners();

			//11.实例化我们剩余的单实例bean.
			finishBeanFactoryInitialization(beanFactory);

			//12.最后容器刷新 发布刷新事件(Spring cloud也是从这里启动的)
			finishRefresh();
		}

		catch (BeansException ex) {
			if (logger.isWarnEnabled()) {
				logger.warn("Exception  encountered during context initialization - " +
						"cancelling refresh attempt: " + ex);
			}

			// Destroy already created singletons to avoid dangling resources.
			destroyBeans();

			// Reset 'active' flag.
			cancelRefresh(ex);

			// Propagate exception to caller.
			throw ex;
		}

		finally {
			// Reset common introspection caches in Spring's core, since we
			// might not ever need metadata for singleton beans anymore...
			resetCommonCaches();
		}
	}
}
```



## 4.1 prepareRefresh()

该方法用于容器刷新前的准备，包括设置上下文状态，获取属性，验证必要的属性等

```java
// 设置启动时间
this.startupDate = System.currentTimeMillis();
// 1交给子类实现，初始化属性源
initPropertySources();
// 验证所有标记为必须的属性
getEnvironment().validateRequiredProperties();
```

## 4.2 obtainFreshBeanFactory()

该方法获取新的beanFactory。该方法很简单，刷新 BeanFactory 和获取 getBeanFactory

```java
protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
   refreshBeanFactory();
   return getBeanFactory();//返回我们的bean工厂，this()中new的工厂
}
```



## 4.3 prepareBeanFactory(beanFactory)

```java
protected void prepareBeanFactory(ConfigurableListableBeanFactory beanFactory) {
    // Tell the internal bean factory to use the context's class loader etc.
    beanFactory.setBeanClassLoader(getClassLoader());//设置类加载器

    //设置bean表达式解析器
    beanFactory.setBeanExpressionResolver(new StandardBeanExpressionResolver(beanFactory.getBeanClassLoader()));

    //属性编辑器支持
    beanFactory.addPropertyEditorRegistrar(new ResourceEditorRegistrar(this, getEnvironment()));

    // Configure the bean factory with context callbacks.
    //添加一个后置处理器：ApplicationContextAwareProcessor，此后置处理处理器实现了BeanPostProcessor接口
    beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this));

    //以下接口，忽略自动装配
    beanFactory.ignoreDependencyInterface(EnvironmentAware.class);
    beanFactory.ignoreDependencyInterface(EmbeddedValueResolverAware.class);
    beanFactory.ignoreDependencyInterface(ResourceLoaderAware.class);
    beanFactory.ignoreDependencyInterface(ApplicationEventPublisherAware.class);
    beanFactory.ignoreDependencyInterface(MessageSourceAware.class);
    beanFactory.ignoreDependencyInterface(ApplicationContextAware.class);

    // BeanFactory interface not registered as resolvable type in a plain factory.
    // MessageSource registered (and found for autowiring) as a bean.
    //以下接口，允许自动装配,第一个参数是自动装配的类型，，第二个字段是自动装配的值
    beanFactory.registerResolvableDependency(BeanFactory.class, beanFactory);
    beanFactory.registerResolvableDependency(ResourceLoader.class, this);
    beanFactory.registerResolvableDependency(ApplicationEventPublisher.class, this);
    beanFactory.registerResolvableDependency(ApplicationContext.class, this);

    // Register early post-processor for detecting inner beans as ApplicationListeners.
    //添加一个后置处理器：ApplicationListenerDetector，此后置处理器实现了BeanPostProcessor接口
    beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(this));

    // Detect a LoadTimeWeaver and prepare for weaving, if found.
    if (beanFactory.containsBean(LOAD_TIME_WEAVER_BEAN_NAME)) {
        beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));
        // Set a temporary ClassLoader for type matching.
        beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));
    }

    //如果没有注册过bean名称为XXX，spring就自己创建一个名称为XXX的singleton bean
    //Register default environment beans.

    if (!beanFactory.containsLocalBean(ENVIRONMENT_BEAN_NAME)) {
        beanFactory.registerSingleton(ENVIRONMENT_BEAN_NAME, getEnvironment());
    }
    if (!beanFactory.containsLocalBean(SYSTEM_PROPERTIES_BEAN_NAME)) {
        beanFactory.registerSingleton(SYSTEM_PROPERTIES_BEAN_NAME, getEnvironment().getSystemProperties());
    }
    if (!beanFactory.containsLocalBean(SYSTEM_ENVIRONMENT_BEAN_NAME)) {
        beanFactory.registerSingleton(SYSTEM_ENVIRONMENT_BEAN_NAME, getEnvironment().getSystemEnvironment());
    }
}
```

主要做了如下的操作：

1. 设置了一个类加载器
2. 设置了bean表达式解析器
3. 添加了属性编辑器的支持
4. **添加了一个后置处理器：ApplicationContextAwareProcessor（后置处理器用来处理ApplicationContextAware接口的回调方法），此后置处理器实现了BeanPostProcessor接口**
5. 设置了一些忽略自动装配的接口
6. 设置了一些允许自动装配的接口，并且进行了赋值操作
7. 在容器中还没有XX的bean的时候，帮我们注册beanName为XX的singleton bean

## 4.4 postProcessBeanFactory(beanFactory)

* 模板方法，此时，所有的 beanDefinition 已经加载，但是还没有实例化允许在子类中对 beanFactory 进行扩展处理。比如添加 ware 相关接口自动装配设置，添加后置处理器等,是子类扩展 prepareBeanFactory(beanFactory) 的方法。

## 4.5 invokeBeanFactoryPostProcessors(beanFactory)（重点）

* **实例化并调用所有注册的beanFactory后置处理器**
* 最终调用`invokeBeanFactoryPostProcessors(beanFactory, getBeanFactoryPostProcessors())`方法
* 在`invokeBeanFactoryPostProcessors`方法中，Spring 会先去找到所有的实现了`BeanDefinitionRegistryPostProcessor`的 BeanFactory 后置处理器，然后先执行实现`PriorityOrdered`的，再执行实现了`Ordered`的。其中最著名的就是`ConfigurationClassPostProcessor`，用来扫描被 @Component 和 @Bean 标记的对象，并注册其 BeanDefinition 元数据到 Spring 容器的 BeanDefinitionMap 中。
* 查找实现了BeanFactoryPostProcessor的后置处理器，并且执行后置处理器中的方法。

```java
public static void invokeBeanFactoryPostProcessors(
		ConfigurableListableBeanFactory beanFactory, List<BeanFactoryPostProcessor> beanFactoryPostProcessors) {

	//调用BeanDefinitionRegistryPostProcessor的后置处理器 Begin
	// 定义已处理的后置处理器
	Set<String> processedBeans = new HashSet<>();

	//判断我们的beanFactory实现了BeanDefinitionRegistry(实现了该结构就有注册和获取Bean定义的能力）
	if (beanFactory instanceof BeanDefinitionRegistry) {
		//强行把我们的bean工厂转为BeanDefinitionRegistry，因为待会需要注册Bean定义
		BeanDefinitionRegistry registry = (BeanDefinitionRegistry) beanFactory;
		//保存BeanFactoryPostProcessor类型的后置   BeanFactoryPostProcessor 提供修改
		List<BeanFactoryPostProcessor> regularPostProcessors = new ArrayList<>();
		//保存BeanDefinitionRegistryPostProcessor类型的后置处理器 BeanDefinitionRegistryPostProcessor 提供注册
		List<BeanDefinitionRegistryPostProcessor> registryProcessors = new ArrayList<>();

		//循环我们传递进来的beanFactoryPostProcessors
		for (BeanFactoryPostProcessor postProcessor : beanFactoryPostProcessors) {
			//判断我们的后置处理器是不是BeanDefinitionRegistryPostProcessor
			if (postProcessor instanceof BeanDefinitionRegistryPostProcessor) {
				//进行强制转化
				BeanDefinitionRegistryPostProcessor registryProcessor =
						(BeanDefinitionRegistryPostProcessor) postProcessor;
				//调用他作为BeanDefinitionRegistryPostProcessor的处理器的后置方法
				registryProcessor.postProcessBeanDefinitionRegistry(registry);
				//添加到我们用于保存的BeanDefinitionRegistryPostProcessor的集合中
				registryProcessors.add(registryProcessor);
			}
			else {//若没有实现BeanDefinitionRegistryPostProcessor 接口，那么他就是BeanFactoryPostProcessor
				//把当前的后置处理器加入到regularPostProcessors中
				regularPostProcessors.add(postProcessor);
			}
		}

		//定义一个集合用户保存当前准备创建的BeanDefinitionRegistryPostProcessor
		List<BeanDefinitionRegistryPostProcessor> currentRegistryProcessors = new ArrayList<>();

		//第一步:去容器中获取BeanDefinitionRegistryPostProcessor的bean的处理器名称
		String[] postProcessorNames =
				beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
		//循环筛选出来的匹配BeanDefinitionRegistryPostProcessor的类型名称
		for (String ppName : postProcessorNames) {
			//判断是否实现了PriorityOrdered接口的
			if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
				//显示的调用getBean()的方式获取出该对象然后加入到currentRegistryProcessors集合中去
				currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
				//同时也加入到processedBeans集合中去
				processedBeans.add(ppName);
			}
		}
		//对currentRegistryProcessors集合中BeanDefinitionRegistryPostProcessor进行排序
		sortPostProcessors(currentRegistryProcessors, beanFactory);
		// 把当前的加入到总的里面去
		registryProcessors.addAll(currentRegistryProcessors);
		/**
		 * 在这里典型的BeanDefinitionRegistryPostProcessor就是ConfigurationClassPostProcessor
		 * 用于进行bean定义的加载 比如我们的包扫描，@import  等等。。。。。。。。。
		 */
		invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry);
		//调用完之后，马上clear掉
		currentRegistryProcessors.clear();
//---------------------------------------调用内置实现PriorityOrdered接口ConfigurationClassPostProcessor完毕--优先级No1-End----------------------------------------------------------------------------------------------------------------------------
		//去容器中获取BeanDefinitionRegistryPostProcessor的bean的处理器名称（内置的和上面注册的）
		postProcessorNames = beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
		//循环上一步获取的BeanDefinitionRegistryPostProcessor的类型名称
		for (String ppName : postProcessorNames) {
			//表示没有被处理过,且实现了Ordered接口的
			if (!processedBeans.contains(ppName) && beanFactory.isTypeMatch(ppName, Ordered.class)) {
				//显示的调用getBean()的方式获取出该对象然后加入到currentRegistryProcessors集合中去
				currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
				//同时也加入到processedBeans集合中去
				processedBeans.add(ppName);
			}
		}
		//对currentRegistryProcessors集合中BeanDefinitionRegistryPostProcessor进行排序
		sortPostProcessors(currentRegistryProcessors, beanFactory);
		//把他加入到用于保存到registryProcessors中
		registryProcessors.addAll(currentRegistryProcessors);
		//调用他的后置处理方法
		invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry);
		//调用完之后，马上clea掉
		currentRegistryProcessors.clear();
//-----------------------------------------调用自定义Order接口BeanDefinitionRegistryPostProcessor完毕-优先级No2-End-----------------------------------------------------------------------------------------------------------------------------
		//调用没有实现任何优先级接口的BeanDefinitionRegistryPostProcessor
		//定义一个重复处理的开关变量 默认值为true
		boolean reiterate = true;
		//第一次就可以进来
		while (reiterate) {
			//进入循环马上把开关变量给改为false
			reiterate = false;
			//去容器中获取BeanDefinitionRegistryPostProcessor的bean的处理器名称
			postProcessorNames = beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
			//循环上一步获取的BeanDefinitionRegistryPostProcessor的类型名称
			for (String ppName : postProcessorNames) {
				//没有被处理过的
				if (!processedBeans.contains(ppName)) {
					//显示的调用getBean()的方式获取出该对象然后加入到currentRegistryProcessors集合中去
					currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
					//同时也加入到processedBeans集合中去
					processedBeans.add(ppName);
					//再次设置为true
					reiterate = true;
				}
			}
			//对currentRegistryProcessors集合中BeanDefinitionRegistryPostProcessor进行排序
			sortPostProcessors(currentRegistryProcessors, beanFactory);
			//把他加入到用于保存到registryProcessors中
			registryProcessors.addAll(currentRegistryProcessors);
			//调用他的后置处理方法
			invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry);
			//进行clear
			currentRegistryProcessors.clear();
		}
//-----------------------------------------调用没有实现任何优先级接口自定义BeanDefinitionRegistryPostProcessor完毕--End-----------------------------------------------------------------------------------------------------------------------------
		//调用 BeanDefinitionRegistryPostProcessor.postProcessBeanFactory方法
		invokeBeanFactoryPostProcessors(registryProcessors, beanFactory);
		//调用BeanFactoryPostProcessor 自设的（没有）
		invokeBeanFactoryPostProcessors(regularPostProcessors, beanFactory);
	}

	else {
		 //若当前的beanFactory没有实现了BeanDefinitionRegistry 说明没有注册Bean定义的能力
		 // 那么就直接调用BeanDefinitionRegistryPostProcessor.postProcessBeanFactory方法
		invokeBeanFactoryPostProcessors(beanFactoryPostProcessors, beanFactory);
	}

//-----------------------------------------所有BeanDefinitionRegistryPostProcessor调用完毕--End-----------------------------------------------------------------------------------------------------------------------------


//-----------------------------------------处理BeanFactoryPostProcessor --Begin-----------------------------------------------------------------------------------------------------------------------------

	//获取容器中所有的 BeanFactoryPostProcessor
	String[] postProcessorNames =
			beanFactory.getBeanNamesForType(BeanFactoryPostProcessor.class, true, false);

	//保存BeanFactoryPostProcessor类型实现了priorityOrdered
	List<BeanFactoryPostProcessor> priorityOrderedPostProcessors = new ArrayList<>();
	//保存BeanFactoryPostProcessor类型实现了Ordered接口的
	List<String> orderedPostProcessorNames = new ArrayList<>();
	//保存BeanFactoryPostProcessor没有实现任何优先级接口的
	List<String> nonOrderedPostProcessorNames = new ArrayList<>();
	for (String ppName : postProcessorNames) {
		//processedBeans包含的话，表示在上面处理BeanDefinitionRegistryPostProcessor的时候处理过了
		if (processedBeans.contains(ppName)) {
			// skip - already processed in first phase above
		}
		//判断是否实现了PriorityOrdered 优先级最高
		else if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
			priorityOrderedPostProcessors.add(beanFactory.getBean(ppName, BeanFactoryPostProcessor.class));
		}
		//判断是否实现了Ordered  优先级 其次
		else if (beanFactory.isTypeMatch(ppName, Ordered.class)) {
			orderedPostProcessorNames.add(ppName);
		}
		//没有实现任何的优先级接口的  最后调用
		else {
			nonOrderedPostProcessorNames.add(ppName);
		}
	}
	//  排序
	sortPostProcessors(priorityOrderedPostProcessors, beanFactory);
	// 先调用BeanFactoryPostProcessor实现了 PriorityOrdered接口的
	invokeBeanFactoryPostProcessors(priorityOrderedPostProcessors, beanFactory);

	//再调用BeanFactoryPostProcessor实现了 Ordered.
	List<BeanFactoryPostProcessor> orderedPostProcessors = new ArrayList<>();
	for (String postProcessorName : orderedPostProcessorNames) {
		orderedPostProcessors.add(beanFactory.getBean(postProcessorName, BeanFactoryPostProcessor.class));
	}
	sortPostProcessors(orderedPostProcessors, beanFactory);
	invokeBeanFactoryPostProcessors(orderedPostProcessors, beanFactory);

	//调用没有实现任何方法接口的
	List<BeanFactoryPostProcessor> nonOrderedPostProcessors = new ArrayList<>();
	for (String postProcessorName : nonOrderedPostProcessorNames) {
		nonOrderedPostProcessors.add(beanFactory.getBean(postProcessorName, BeanFactoryPostProcessor.class));
	}
	invokeBeanFactoryPostProcessors(nonOrderedPostProcessors, beanFactory);
//-----------------------------------------处理BeanFactoryPostProcessor --End-----------------------------------------------------------------------------------------------------------------------------

	// Clear cached merged bean definitions since the post-processors might have
	// modified the original metadata, e.g. replacing placeholders in values...
	beanFactory.clearMetadataCache();

//------------------------- BeanFactoryPostProcessor和BeanDefinitionRegistryPostProcessor调用完毕 --End-----------------------------------------------------------------------------------------------------------------------------

}
```

看着挺长，其实里面的内容，比较简单。

* 首先判断一下传入的beanFactory是不是BeanDefinitionRegistry的实例，当然肯定是的。
* 定义了一个Set，processedBeans代表了一个已经处理过的Bean的容器，装载BeanName，后面会根据这个Set，来判断后置处理器是否被执行过了。
* 定义了两个List，一个是regularPostProcessors，用来装载BeanFactoryPostProcessor，一个是registryProcessors用来装载BeanDefinitionRegistryPostProcessor，其中BeanDefinitionRegistryPostProcessor扩展了BeanFactoryPostProcessor。
* 循环传进来的beanFactoryPostProcessors，一般情况下，这里永远都是空的，只有手动add beanFactoryPostProcessor，这里才会有数据。
* getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false)，找到类型为BeanDefinitionRegistryPostProcessor的后置处理器，如果是PriorityOrdered类型的，那么会放入priorityOrderedPostProcessors容器中，然后排序，执行BeanDefinitionRegistryPostProcessor的生命周期方法postProcessBeanDefinitionRegistry。相同的方式再处理Ordered类型的，最后剩下的再单独处理。
* **其中解析配置类的后置处理器ConfigurationAnnotationProcessor在此工作，用于进行bean定义的加载**
* 之后查找实现了BeanFactoryPostProcessor的后置处理器，并且执行后置处理器中的方法。和上面的逻辑差不多。



## 4.6 registerBeanPostProcessors(beanFactory)

**该方法是注册 Bean 的后置处理器**

核心代码：

```java
// 1. 获取所有的 Bean 后置处理器的名字
String[] postProcessorNames = beanFactory.getBeanNamesForType(BeanPostProcessor.class, true, false);
// 2. 对 Bean 后置处理器分类
List<BeanPostProcessor> priorityOrderedPostProcessors = new ArrayList<>();
List<BeanPostProcessor> internalPostProcessors = new ArrayList<>();
List<String> orderedPostProcessorNames = new ArrayList<>();
List<String> nonOrderedPostProcessorNames = new ArrayList<>();
// 3. 注册 Bean 后置处理器
registerBeanPostProcessors(beanFactory, priorityOrderedPostProcessors);
registerBeanPostProcessors(beanFactory, orderedPostProcessors);
registerBeanPostProcessors(beanFactory, nonOrderedPostProcessors);
registerBeanPostProcessors(beanFactory, internalPostProcessors);

// 4. 注册 ApplicationListener 探测器
beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(applicationContext));
```

1. 获取所有的 Bean 后置处理器的名字。
2. 对 Bean 后置处理器分类。执行完分类之后，所有的`priorityOrderedPostProcessors`都将成为一个 Bean 进入 Spring 容器中
   1. `priorityOrderedPostProcessors`是所有实现了`PriorityOrdered`接口的后置处理器。
   2. `internalPostProcessors`是所有内置的后置处理器
   3. `orderedPostProcessorNames`实现了`ordered`接口的后置处理器。
   4. `nonOrderedPostProcessorNames`没有排序的后置处理器。

## 4.7 initMessageSource()

为上下文初始化 Message 源，即对不同语言的消息体进行国际化处理

## 4.8 initApplicationEventMulticaster()

初始化事件多播放器组件：

1. 看容器中是否有applicationEventMulticaster的定义信息，按照id找
2. 如果没有就注册一个默认的
3. 把事件多播器组件（ApplicationEventMulticaster）放到单例池中

## 4.9 onRefresh()

模板方法，在容器刷新的时候可以自定义逻辑，不同的Spring容器做不同的事情

## 4.10 registerListeners()

注册监听器：

```java
protected void registerListeners() {
   //获取容器中所有的监听器对象
   // 这个时候正常流程是不会有监听器的
   // （因为监听器不会在这之前注册，在initApplicationEventMulticaster后在registerListeners之前，只有一个可能在：在onRefresh里面注册了监听器）
   for (ApplicationListener<?> listener : getApplicationListeners()) {
      //把监听器挨个的注册到我们的多播器上去
      getApplicationEventMulticaster().addApplicationListener(listener);
   }

   //获取bean定义中的监听器对象
   String[] listenerBeanNames = getBeanNamesForType(ApplicationListener.class, true, false);
   //把监听器的名称注册到我们的多播器上
   for (String listenerBeanName : listenerBeanNames) {
      getApplicationEventMulticaster().addApplicationListenerBean(listenerBeanName);
   }

   //在这里获取我们的早期事件
   Set<ApplicationEvent> earlyEventsToProcess = this.earlyApplicationEvents;
   // 在这里赋null。  也就是值此之后都将没有早期事件了
   this.earlyApplicationEvents = null;
   if (earlyEventsToProcess != null) {
      //通过多播器进行播发早期事件
      for (ApplicationEvent earlyEvent : earlyEventsToProcess) {
         getApplicationEventMulticaster().multicastEvent(earlyEvent);
      }
   }
}
```

## 4.11 finishBeanFactoryInitialization(beanFactory)（重点）

实例化所有剩余的非懒加载单例,比如`invokeBeanFactoryPostProcessors`方法中根据各种注解解析出来的类，在这个时候都会被初始化。实例化的过程各种`BeanPostProcessor`开始起作用。

* **finishBeanFactoryInitialization(beanFactory)：**

  ```java
  protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
      // Initialize conversion service for this context.
      // 1.初始化此上下文的转换服务
      if (beanFactory.containsBean(CONVERSION_SERVICE_BEAN_NAME) &&
              beanFactory.isTypeMatch(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class)) {
          beanFactory.setConversionService(
                  beanFactory.getBean(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class));
      }
   
      // Register a default embedded value resolver if no bean post-processor
      // (such as a PropertyPlaceholderConfigurer bean) registered any before:
      // at this point, primarily for resolution in annotation attribute values.
      // 2.如果beanFactory之前没有注册嵌入值解析器，则注册默认的嵌入值解析器：主要用于注解属性值的解析。
      if (!beanFactory.hasEmbeddedValueResolver()) {
          beanFactory.addEmbeddedValueResolver(new StringValueResolver() {
              @Override
              public String resolveStringValue(String strVal) {
                  return getEnvironment().resolvePlaceholders(strVal);
              }
          });
      }
   
      // Initialize LoadTimeWeaverAware beans early to allow for registering their transformers early.
      // 3.初始化LoadTimeWeaverAware Bean实例对象
      String[] weaverAwareNames = beanFactory.getBeanNamesForType(LoadTimeWeaverAware.class, false, false);
      for (String weaverAwareName : weaverAwareNames) {
          getBean(weaverAwareName);
      }
   
      // Stop using the temporary ClassLoader for type matching.
      beanFactory.setTempClassLoader(null);
   
      // Allow for caching all bean definition metadata, not expecting further changes.
      // 4.冻结所有bean定义，注册的bean定义不会被修改或进一步后处理，因为马上要创建 Bean 实例对象了
      beanFactory.freezeConfiguration();
   
      // Instantiate all remaining (non-lazy-init) singletons.
      // 5.实例化所有剩余（非懒加载）单例对象
      beanFactory.preInstantiateSingletons();
  }
  ```

* **preInstantiateSingletons()**

  ```java
  @Override
  public void preInstantiateSingletons() throws BeansException {
      if (this.logger.isDebugEnabled()) {
          this.logger.debug("Pre-instantiating singletons in " + this);
      }
   
      // Iterate over a copy to allow for init methods which in turn register new bean definitions.
      // While this may not be part of the regular factory bootstrap, it does otherwise work fine.
      // 1.创建beanDefinitionNames的副本beanNames用于后续的遍历，以允许init等方法注册新的bean定义
      List<String> beanNames = new ArrayList<String>(this.beanDefinitionNames);
   
      // Trigger initialization of all non-lazy singleton beans...
      // 2.遍历beanNames，触发所有非懒加载单例bean的初始化
      for (String beanName : beanNames) {
          // 3.获取beanName对应的MergedBeanDefinition
          RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
          // 4.bd对应的Bean实例：不是抽象类 && 是单例 && 不是懒加载
          if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
              // 5.判断beanName对应的bean是否为FactoryBean
              if (isFactoryBean(beanName)) {
                  // 5.1 通过beanName获取FactoryBean实例
                  // 通过getBean(&beanName)拿到的是FactoryBean本身；通过getBean(beanName)拿到的是FactoryBean创建的Bean实例
                  final FactoryBean<?> factory = (FactoryBean<?>) getBean(FACTORY_BEAN_PREFIX + beanName);
                  // 5.2 判断这个FactoryBean是否希望急切的初始化
                  boolean isEagerInit;
                  if (System.getSecurityManager() != null && factory instanceof SmartFactoryBean) {
                      isEagerInit = AccessController.doPrivileged(new PrivilegedAction<Boolean>() {
                          @Override
                          public Boolean run() {
                              return ((SmartFactoryBean<?>) factory).isEagerInit();
                          }
                      }, getAccessControlContext());
                  } else {
                      isEagerInit = (factory instanceof SmartFactoryBean &&
                              ((SmartFactoryBean<?>) factory).isEagerInit());
                  }
                  if (isEagerInit) {
                      // 5.3 如果希望急切的初始化，则通过beanName获取bean实例
                      getBean(beanName);
                  }
              } else {
                  // 6.如果beanName对应的bean不是FactoryBean，只是普通Bean，通过beanName获取bean实例
                  getBean(beanName);
              }
          }
      }
   
      // Trigger post-initialization callback for all applicable beans...
      // 7.遍历beanNames，触发所有SmartInitializingSingleton的后初始化回调
      for (String beanName : beanNames) {
          // 7.1 拿到beanName对应的bean实例
          Object singletonInstance = getSingleton(beanName);
          // 7.2 判断singletonInstance是否实现了SmartInitializingSingleton接口
          if (singletonInstance instanceof SmartInitializingSingleton) {
              final SmartInitializingSingleton smartSingleton = (SmartInitializingSingleton) singletonInstance;
              // 7.3 触发SmartInitializingSingleton实现类的afterSingletonsInstantiated方法
              if (System.getSecurityManager() != null) {
                  AccessController.doPrivileged(new PrivilegedAction<Object>() {
                      @Override
                      public Object run() {
                          smartSingleton.afterSingletonsInstantiated();
                          return null;
                      }
                  }, getAccessControlContext());
              } else {
                  smartSingleton.afterSingletonsInstantiated();
              }
          }
      }
  }
  ```

  *  这个方法主要是循环遍历BeanDefinitionMap, 调用getBean, 去生产bean
  * getBean()方法参考Bean的声明周期

## 4.12 finishRefresh()

最后的一些清理、事件发送等

```java
protected void finishRefresh() {
    // 清除上下文资源缓存（如扫描中的ASM元数据） scanning).
    clearResourceCaches();
    // 初始化上下文的生命周期处理器，并刷新（找出Spring容器中实现了Lifecycle接口的bean并执行start()方法）
    initLifecycleProcessor();
    getLifecycleProcessor().onRefresh();
    // 发布ContextRefreshedEvent事件告知对应的ApplicationListener进行响应的操作
    publishEvent(new ContextRefreshedEvent(this));

}
```