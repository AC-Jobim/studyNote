

# 一、循环依赖介绍

> 好的博客：[面试必杀技，讲一讲Spring中的循环依赖 ](https://www.cnblogs.com/daimzh/p/13256413.html)

**<font color='blue' size='4'>什么是循环依赖？</font>**

* 循环依赖是指，A 依赖 B，B 又依赖 A，它们之间形成了循环依赖。或者是 A 依赖 B，B 依赖 C，C 又依赖 A。它们之间的依赖关系如下

  ![image-20211007222808829](https://gitee.com/jobim/blogimage/raw/master/img/20211007222815.png)



<font color='blue' size='4'>**什么情况下循环依赖可以被处理？**</font>

1. 出现循环依赖的Bean必须要是单例
2. 依赖注入的方式不能全是构造器注入的方式



**<font color='blue' size='4'>如何解决循环依赖？</font>**

* Spring通过**三级缓存解决了循环依赖**，其中一级缓存为单例池（singletonObjects）,二级缓存为早期曝光对象earlySingletonObjects，三级缓存为早期曝光对象工厂（singletonFactories）。
* 当A、B两个类发生循环引用时，在A完成实例化后，就使用实例化后的对象去创建一个对象工厂，并添加到三级缓存中。
* 当A进行属性注入时，会去创建B，同时B又依赖了A，所以创建B的同时又会去调用getBean(a)来获取需要的依赖，此时的getBean(a)会从缓存中获取，第一步，先获取到三级缓存中的工厂；第二步，调用对象工工厂的getObject方法来获取到对应的对象，同时将半成品对象放到二级缓存（如果A被AOP代理，那么通过这个工厂获取到的就是A代理后的对象，如果A没有被AOP代理，那么这个工厂获取到的就是A实例化的对象），得到这个对象后将其注入到B中。紧接着B会走完它的生命周期流程，包括初始化、后置处理器等。当B创建完后，会将B再注入到A中，此时A再完成它的整个生命周期。





**三级缓存在循环依赖中的使用：**

> 这里我们假设一个场景进行讲解：ServiceA、ServiceB相互依赖。

1. Spring容器依次创建两个bean时，发现在缓存中没有ServiceA，因此将新创建好的未注入属性的ServiceA放到三级缓存中去。然后ServiceA进行属性注入时，发现依赖ServiceB，转而去实例化B。
2. 同样创建对象ServiceB，注入属性时发现依赖ServiceA，依次从一级到三级缓存查询ServiceA，从三级缓存通过对象工厂拿到ServiceA（可能为代理对象），把ServiceA放入二级缓存，同时删除三级缓存中的ServiceA，此时，ServiceB已经实例化并且初始化完成，把ServiceB放入一级缓存。
3. 接着继续创建ServiceA，顺利从一级缓存拿到实例化且初始化完成的ServiceB对象，ServiceA对象创建也完成，删除二级缓存中的ServiceA，同时把ServiceA放入一级缓存
4. 最后，一级缓存中保存着实例化、初始化都完成的ServiceA、ServiceB对象



# 二、源码分析，循环依赖



先来分析一个最简单的例子

```java
@Component
public class A {
    // A中注入了B
	@Autowired
	private B b;
}

@Component
public class B {
    // B中也注入了A
	@Autowired
	private A a;
}
```

**几个缓存：**

```java
/** 一级缓存：单例缓存池，用于保存所有的单实例bean */
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);

/** 三级缓存：key为beanName，value为ObjectFactory(包装为早期对象) */
private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);

/** 二级缓存：key为beanName，value是早期对象(对象属性还没有来得及进行赋值) */
private final Map<String, Object> earlySingletonObjects = new HashMap<>(16);

/** 该集合缓存当前正在创建bean的名称 */
private final Set<String> singletonsCurrentlyInCreation = Collections.newSetFromMap(new ConcurrentHashMap<>(16));
```

![image-20211125155511873](https://gitee.com/jobim/blogimage/raw/master/img/20211125155511.png)





## 2.1 获取A：getBean(a)

* **调用doGetBean()方法，想要获取beanA，于是调用getSingleton()方法从缓存中查找beanA**

### 2.1.1 检查缓存getSingleton(a)

![image-20211125111019501](https://gitee.com/jobim/blogimage/raw/master/img/20211125111026.png)

* **此时的beanName是“a“ 先从单例池中找，此时肯定是找不到的，返回null。**

* **isSingletonCurrentlyCreation方法是判断当前Bean是否在singletonsCurrentlyInCreation集合当中。此时也是没有的，返回false。那么就不会进第一个if。直接返回null值。**

* **表示从缓存中获取不到 “a” 对应的Bean**

* **因为上面没有获取到Bean，这里就要去准备创建一个Bean，进入`getSingleton(beanName,singletonFactory)`方法**
  ![image-20211125111212606](https://gitee.com/jobim/blogimage/raw/master/img/20211125111212.png)

### 2.1.2 执行getSingleton(beanName,singletonFactory)

* **首先从一级缓存中查找，没有，返回null。**

  ![image-20211125111417983](https://gitee.com/jobim/blogimage/raw/master/img/20211125111418.png)

* **beforeSingletonCreation(beanName)，创建单例前的操作，将beanName（a）加入singletonsCurrentlyInCreation（该集合用户缓存当前正在创建bean的名称）**

  ![image-20211125111546609](https://gitee.com/jobim/blogimage/raw/master/img/20211125111546.png)

* **执行singletonFactory的getObject()方法获取bean实例，其实是调用`createBean()`方法。之后调用doCreateBean(beanName, mbdToUse, args)方法，真正创建A实例对象。**

  ![image-20211125112400904](https://gitee.com/jobim/blogimage/raw/master/img/20211125112400.png)



### 2.1.3 真正创建A：doCreateBean(beanName, mbdToUse, args)

* **实例化A：createBeanInstance();**

* **`addSingletonFactory`，在填充A的属性之前，会将创建Bean A的工厂放入到singletonFactories（三级缓存）中。**

  ![image-20211125112817353](https://gitee.com/jobim/blogimage/raw/master/img/20211125112817.png)

* **属性赋值：populateBean()**

  ![image-20211125113230036](https://gitee.com/jobim/blogimage/raw/master/img/20211125113230.png)

  **在属性填充之前，会先去获得属性值，但是此时的B对象并没有被实例化，所以需要先去实例化B才可以。此时递归的去调用getBean(b)构建B对象，再调用doGetBean()方法**

  ![image-20211125113357974](https://gitee.com/jobim/blogimage/raw/master/img/20211125113358.png)



#### 2.1.4.1 获取B：调用doGetBean()方法

* **和之前一样先调用getSingleton方法，4个集合中，都没有和“b“有关的缓存，所以返回null**

  ![image-20211125151915442](https://gitee.com/jobim/blogimage/raw/master/img/20211125151915.png)

* **进入getSingleton(beanName,singletonFactory)方法，将“b“ 放入到 singletonsCurrentlyInCreation 集合中**

  ![image-20211125120022321](https://gitee.com/jobim/blogimage/raw/master/img/20211125120022.png)

* **执行singletonFactory的getObject()方法获取bean实例，其实是调用`createBean()`方法。之后调用doCreateBean(beanName, mbdToUse, args)方法，真正创建B实例对象**

#### 2.1.4.2 真正创建B对象：doCreateBean()

* 和之前一样，在填充B的属性之前，会将创建Bean B的工厂放入到singletonFactories中

  ![image-20211125120350870](https://gitee.com/jobim/blogimage/raw/master/img/20211125120350.png)

  > **此时四个集合的状态**
  >
  > ![image-20211125120540923](https://gitee.com/jobim/blogimage/raw/master/img/20211125120540.png)

* **属性赋值：populateBean()**

  * **B的属性有A，也就是去获取A实例Bean。**

  * **代码又回来到AbstractBeanFactory# doGetBean**

    ![image-20211125145521271](https://gitee.com/jobim/blogimage/raw/master/img/20211125145521.png)

  * **首先调用getSingleton 来从缓存中获取。进入getSingleton 看看能不能得到A的实例**

    ![image-20211125150813243](https://gitee.com/jobim/blogimage/raw/master/img/20211125150813.png)

    * **获取到三级缓存中的工厂**

    * **调用对象工工厂的getObject方法来获取到对应的对象，其实调用getEarlyBeanReference()方法，（如果A被AOP代理，那么通过这个工厂获取到的就是A代理后的对象，如果A没有被AOP代理，那么这个工厂获取到的就是A实例化的对象）**

      ![image-20211125161636678](https://gitee.com/jobim/blogimage/raw/master/img/20211125161636.png)

    * **同时将半成品对象放到二级缓存并将包装对象从三级缓存中删除掉**

    > 此时4个集合的状态：
    >
    > ![image-20211125150949907](https://gitee.com/jobim/blogimage/raw/master/img/20211125150950.png)



#### 2.1.4.3 B对象创建完后续操作

* **afterSingletonCreation(name)，创建单例后的操作，把singletonsCurrentlyInCreation标记正在创建的bean从集合中移除**

* **addSingleton(beanName, singletonObject)，把对象加入到单例缓存池中(所谓的一级缓存 并且考虑循环依赖和正常情况下,移除二三级缓存)**

  ![image-20211125152536454](https://gitee.com/jobim/blogimage/raw/master/img/20211125152536.png)



### 2.1.4 A对象完成属性赋值之后

* **此时构成了一个完整的B，A属性填充完毕**

* **afterSingletonCreation(name)，创建单例后的操作，把singletonsCurrentlyInCreation标记正在创建的bean从集合中移除**

* **addSingleton(beanName, singletonObject)，把对象加入到单例缓存池中(所谓的一级缓存 并且考虑循环依赖和正常情况下,移除二三级缓存)**

  ![image-20211125152536454](https://gitee.com/jobim/blogimage/raw/master/img/20211125152536.png)



## 2.2 获取B：getBean(b)

* **直接从一级缓存中获取B对象**

# 三、为什么需要三级缓存

> 好的博客：[Spring 为何需要三级缓存解决循环依赖，而不是二级缓存](https://www.cnblogs.com/semi-sub/p/13548479.html)
>
> [面试必杀技，讲一讲Spring中的循环依赖 ](https://www.cnblogs.com/daimzh/p/13256413.html#三级缓存真的提高了效率了吗？)
>
> [Spring循环依赖三级缓存是否可以去掉第三级缓存？](https://segmentfault.com/a/1190000023647227)

* **所以如果没有AOP的话确实可以两级缓存就可以解决循环依赖的问题，如果加上AOP，两级缓存是无法解决的**



**<font color='red'>Spring什么时候创建代理对象？</font>**

* **正常Bean在生命周期的最后一步完成代理**

  ![image-20211125162051636](https://gitee.com/jobim/blogimage/raw/master/img/20211125162051.png)

* **涉及循环依赖的代理对象在实例化后创建，循环依赖调用getSingleton查看缓存时，会调用getEarlyBeanReference()，其中AOP的AbstractAutoProxyCreator.getEarlyBeanReference()方法也在此执行，但只有循环引用的对象为代理对象的时候才会进行动态代理**

  ![image-20211125161636678](https://gitee.com/jobim/blogimage/raw/master/img/20211125161636.png)





**<font color='blue'>去掉第三级缓存的情况：</font>**

* **如果要使用`二级缓存`解决`循环依赖`，意味着Bean在`构造`完后就创建`代理对象`，这样违背了`Spring设计原则`。Spring结合AOP跟Bean的生命周期，是在`Bean创建完全`之后通过`AnnotationAwareAspectJAutoProxyCreator`这个后置处理器来完成的，在这个后置处理的`postProcessAfterInitialization`方法中对初始化后的Bean完成AOP代理。如果出现了`循环依赖`，那没有办法，只有给Bean先创建代理，但是没有出现循环依赖的情况下，设计之初就是让Bean在生命周期的最后一步完成代理而不是在实例化后就立马完成代理。**



**<font color='blue'>去掉第二级缓存的情况：</font>**

![image-20211125150813243](https://gitee.com/jobim/blogimage/raw/master/img/20211125150813.png)

* **拿到ObjectFactory对象后，调用ObjectFactory.getObject()方法最终会调用getEarlyBeanReference()方法，getEarlyBeanReference这个方法主要逻辑大概描述下如果bean被AOP切面代理则返回的是beanProxy对象，如果未被代理则返回的是原bean实例。**

* **由于去除了第二级缓存，所以不能二级缓存earlySingletonObjects中，只能通过每次调用来获取**

* **但不可能每次执行singleFactory.getObject()方法都给我产生一个新的代理对象，所以还要借助另外一个缓存来保存产生的代理对象**
