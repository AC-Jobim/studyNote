> 使用SpringCloud版本：`2.1.1.RELEASE`

# 一、Hystrix断路器

## 1.1 Hystrix 简介

**服务雪崩：**

* 在微服务架构中，服务与服务之间通过远程调用的方式进行通信，一旦某个被调用的服务发生了故障，其依赖服务也会发生故障，此时就会发生故障的蔓延，最终导致系统瘫痪。这就是所谓的 **`"雪崩效应"`**。



**什么是Hystrix?**

* 是一个用于处理分布式系统的 **`延迟`** 和 **`容错`** 的开源库。在分布式系统中，许多依赖不可避免的会调用失败，比如：超时、异常等原因。Hystrix 能够保证在一个依赖出现问题的情况下， **`不会导致整体服务失败，避免级联故障，以提高分布式系统的弹性。`**

* **Hystrix实现了断路器模式，当某个服务发生故障时，通过断路器的监控，给调用方返回一个错误响应，而不是长时间的等待，这样就不会使得调用方由于长时间得不到响应而占用线程，从而防止故障的蔓延**。
* Hystrix具备服务降级、服务熔断、线程隔离、请求缓存、请求合并及服务监控等强大功能。



> 目前Hystrix停更进入维护，官网推荐我们使用 resilience4j，在国外比较流行。但是在国内，结合国内情况，推荐大家使用阿里的 **Sentineal**



**Hystrix重要概念：**

* **服务降级**

  * 服务降级是指当服务器压力剧增的情况下，根据实际业务情况及流量，对一些服务和页面有策略的不处理或换种简单的方式处理，从而释放服务器资源以保证核心业务正常运作或高效运作
  * 服务降级处理是在客户端(Consumer 端)实现完成的，与服务端(Provider 端)没有关系。当某个 Consumer 访问一个 Provider 却迟迟得不到响应时执行预先设定好的一个解决方案，而不是一直等待。

* **服务熔断**

  * 熔断机制是应对雪崩效应的一种微服务链路保护机制。当扇出链路的某个微服务不可用或者响应时间太长时，会进行服务的降级，进而熔断该节点微服务的调用，快速响应错误信息。当检测到该节点微服务调用响应正常后恢复调用链路。

  **服务降级和服务熔断的区别？**

  1. **`服务降级 → 当前服务还是可用的；`**（比如有10个线程，谁抢到谁用，抢不到如果超时报提错误提示，下一次抢到还能继续提供服务）
  2. **`服务熔断 → 当前服务不可用，但是它会逐渐恢复服务提供；`**(拉闸，整个家用电器都不能用；然后测试开5个电器没问题，6个也没问题，逐渐的就恢复服务提供)

* **服务限流**

  服务限流场景：一般应用在 **`秒杀`**，**`高并发`** 等操作。严禁一窝蜂的过来拥挤，大家排队，一秒钟只允许通过 N 个，有序进行



**Hystrix应用场景分析：**

* 对于OrderFeign的案例，在非高并发的情况下，请求可以正常执行。但在高并发的情况下，tomcat 线程里面的工作线程被抢占，可能会导致请求响应缓慢地问题，所有才有 **`服务降级`** / **`服务熔断`** / **`服务限流`** 等技术诞生



## 1.2 Hystrix实现服务降级(重点)

**哪些情况会触发降级？**

* 程序运行异常
* 超时
* 服务熔断触发服务降级
* 线程池/信号量打满也会导致服务降级

> 服务降级即可以放在服务提供端，也可以放在服务消费端

![image-20220110170854846](https://gitee.com/jobim/blogimage/raw/master/img/20220110170855.png)

### 1.2.1 服务端提供端实现服务降级



> 模块名称定义为：**`cloud-provider-hystrix-payment8001`**，来充当提供服务的角色。用到的是 **`@HystrixCommand`** 注解

* **引入pom.xml 依赖**

  ```xml
  <!--引入 spring-cloud-hystrix 依赖-->
  <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
  </dependency>
  ```

* **在主启动类，添加`@EnableCircuitBreaker`注解**

  ```java
  @SpringBootApplication
  @EnableEurekaClient
  @EnableCircuitBreaker  //激活Hystrix断路器
  public class PaymentHystrixMain8001 {
      public static void main(String[] args) {
          SpringApplication.run(PaymentHystrixMain8001.class,args);
      }
  }
  ```

* **在服务提供端Service中添加调用方法与服务降级方法，方法上需要添加`@HystrixCommand`注解**

  参数 **`execution.isolation.thread.timeoutInMilliseconds`** ，在设定的时间范围内返回，即执行正常流程；超过指定时间，则执行指定的服务降级方法。

  ```java
  @Service
  public class PaymentService {
  
      /**
       * 超时访问，演示降级
       */
      @HystrixCommand(fallbackMethod = "paymentInfo_TimeOutHandler", commandProperties = {
              @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "5000") //规定该线程的超时时间为5秒的时候，执行服务降级方法
      })
      public String paymentInfo_TimeOut(Integer id) {
          int timeNumber = 10; // 设置睡眠10秒
  //        int age = 10/0; // 出现异常也发生服务降级
          try {
              TimeUnit.SECONDS.sleep(timeNumber);
          } catch (InterruptedException e) {
              e.printStackTrace();
          }
          return "线程池:"+Thread.currentThread().getName()+"paymentInfo_TimeOut,id: "+id+"\t"+"O(∩_∩)O，耗费"+timeNumber+"秒";
      }
  
      public String paymentInfo_TimeOutHandler(Integer id) {
          return "/(ㄒoㄒ)/调用支付接口超时或异常：\t"+ "\t当前线程池名字" + Thread.currentThread().getName();
      }
      
  }
  ```

* 测试：

  1. 当设置 **`TimeUnit.SECONDS.sleep(3)`** ，这种情况下，便会执行正常的方法；
  2. 当设置 **`TimeUnit.SECONDS.sleep(10)`**，由于10s > 5s 已经超时，便执行降级方法；
  3. 当设置 **`int age = 10/0`**，由于异常原因，会直接进入降级方法



### 1.2.2 消费端实现服务降级（重点）

>模块名称定义为：**`cloud-consumer-feign-hystrix-order80`**，来充当提供消费者的角色。

1. **application.yml 配置文件修改**

   消费端添加配置 **`feign.hystrix.enabled=true`**。如果在消费端调用服务端时，此处涉及到 **`Feign 超时控制`** 这个概念。需要配置 **`ribbon.ConnectTimeout 和 ribbon.ReadTimeout`** 属性。

   ```yml
   server:
     port: 80
   spring:
     application:
       name: cloud-provider-hystrix-order
   eureka:
     client:
       register-with-eureka: true    #示表不向注册中心注册自己
       fetch-registry: true   #表示自己就是注册中心，职责是维护服务实例，并不需要去检索服务
       service-url:
         defaultZone: http://eureka7001.com:7001/eureka/
         
   ####################以下配置需要新增(ribbon相关配置和Feign超时相关)##################
   
   #全局配置
   # 请求连接的超时时间 默认的时间为 1 秒
   # 设置feign客户端超时时间(OpenFeign默认支持ribbon)
   ribbon:
     #指的是建立连接后从服务器读取到可用资源所用的时间
     ReadTimeout: 5000
     #指的是建立连接所用的时间，适用于网络状况正常的情况下,两端连接所用的时间
     ConnectTimeout: 5000
   ```

2. **主启动类，添加`@EnableHystrix`注解**

   ```java
   @SpringBootApplication
   @EnableFeignClients
   @EnableHystrix
   public class OrderHystrixMain80 {
   
       public static void main(String[] args) {
           SpringApplication.run(OrderHystrixMain80.class,args);
       }
   
   }
   ```

3. **消费端在controller层实现自己的服务降级**

   ```java
   @RestController
   @Slf4j
   public class OrderHystrixController {
       @Resource
       private PaymentHystrixService paymentHystrixService;
   
       @GetMapping("/consumer/payment/hystrix/ok/{id}")
       public String paymentInfo_OK(@PathVariable("id") Integer id) {
           String result = paymentHystrixService.paymentInfo_OK(id);
           return result;
       }
   
       @GetMapping("/consumer/payment/hystrix/timeout/{id}")
       @HystrixCommand(fallbackMethod = "paymentTimeOutFallbackMethod",commandProperties = {
               @HystrixProperty(name="execution.isolation.thread.timeoutInMilliseconds",value="1500")
       })
       public String paymentInfo_TimeOut(@PathVariable("id") Integer id) throws InterruptedException {
   //        System.out.println(10/0);
           TimeUnit.SECONDS.sleep(3);
           String result = paymentHystrixService.paymentInfo_TimeOut(id);
           return result;
       }
   
       public String paymentTimeOutFallbackMethod(@PathVariable("id") Integer id) {
           return "我是消费者80,对方支付系统繁忙请10秒钟后再试或者自己运行出错请检查自己,o(╥﹏╥)o";
       }
   
   }
   ```

4. **测试**

   消费端根据自己的情况，**进行服务降级设置，设置为1.5s**。服务端 **`TimeUnit.SECONDS.sleep(3)`** 设置**睡眠 3s**（或者抛出异常）。显然在设置的 1.5s 内无法返回数据。最终便会**执行服务端的降级方法**。返回 **`我是消费者80，对付支付系统繁忙请10秒钟后再试或者自己运行出错请检查自己,(┬＿┬)`** 提示

   ![image-20210908165154289](https://gitee.com/jobim/blogimage/raw/master/img/20210908165154.png)



### 1.2.2 服务降级配置存在的问题及解决

1. 每个业务方法对应一个兜底的方法，代码膨胀
2. **`业务方法`** 和 **`自定义降级方法`** 混合在一起(业务逻辑方法和处理服务降级、熔断方法 揉在一块)



#### 1.2.2.1 代码膨胀问题

* 服务间使用 Feign 进行接口调用，针对每个业务一个兜底方法的问题。**`解决方式：`** 我们可以定义一个**全局通用的 `服务降级` 方法**，这样就可以使用全局 **`服务降级`** 方法，解决代码膨胀问题。

**步骤：**

1. **使用 `@DefaultProperties(defaultFallback = "xxx")` 的方式，定义全局 `服务降级` 方法；**
2. 在类中，定义 **`全局降级方法`**，名称需要与 **`defaultFallback = "xxx"`** 定义一致；
3. 在需要 **`服务降级`** 的方法上，添加 **`@HystrixCommand`**；
4. 只有一个 **`@HystrixCommand 标签`**，则使用全局 **`"服务降级"`** 方法；如果是 **`@HystrixCommand(fallbackMethod = "xxx",commandProperties = {xxx})`** 这样定义，则使用指定的 **`"服务降级"`** 方法。

```java
@RestController
@Slf4j
//使用@DefaultProperties 定义全局服务降级方法
@DefaultProperties(defaultFallback = "payment_Global_FallbackMethod")
public class OrderHystrixController {
    @Resource
    private PaymentHystrixService paymentHystrixService;

    /**
     * 使用全局服务降级方法
     */
    @GetMapping("/consumer/payment/hystrix/ok/{id}")
    @HystrixCommand //加了@DefaultProperties属性注解，并且没有写具体方法名字，就用统一全局的
    public String paymentInfo_OK(@PathVariable("id") Integer id) {
        String result = paymentHystrixService.paymentInfo_OK(id);
        return result;
    }

    /**
     * 使用指定的服务降级方法
     */
    @GetMapping("/consumer/payment/hystrix/timeout/{id}")
    @HystrixCommand(fallbackMethod = "paymentTimeOutFallbackMethod",commandProperties = {
            @HystrixProperty(name="execution.isolation.thread.timeoutInMilliseconds",value="1500")
    })
    public String paymentInfo_TimeOut(@PathVariable("id") Integer id) {
        String result = paymentHystrixService.paymentInfo_TimeOut(id);
        return result;
    }
    //指定服务降级方法
    public String paymentTimeOutFallbackMethod(@PathVariable("id") Integer id) {
        return "我是消费者80,对方支付系统繁忙请10秒钟后再试或者自己运行出错请检查自己,o(╥﹏╥)o";
    }
    //定义全局服务降级方法(不能有参数)
    public String payment_Global_FallbackMethod() {
        return "Global异常处理信息，请稍后再试，/(ㄒoㄒ)/~~";
    }

}
```



#### 1.2.2.1 业务方法和降级方法混合在一起问题（重点）

* 解决混合在一起的问题，只需要为 Feign 定义的接口 **`添加统一的服务降级类`** 即可实现解耦。**`即：来一个实现类，实现Feign 接口，并重写方法`**（重写的就是"服务降级"处理方法）。



**步骤：**

1. 客户端 **`针对使用 Feign 组件调用的接口，重新新建一个类来实现该接口，统一为接口里面的方法进行异常处理`**；
2. 客户端 Feign 接口，添加注解 **`@FeignClient(value = "服务名",fallback = 接口实现类.class)`** 进行指定。

![image-20210908170953710](https://gitee.com/jobim/blogimage/raw/master/img/20210908170953.png)

**测试：**
  在 **`客户端`** 调用 **`服务端`** 接口，如果客户端此时突然 **`宕机`**，便会触发客户端服务降级方法。最终返回的是 **`Feign 接口实现类`** 中重写的内容。

![image-20210908171111858](https://gitee.com/jobim/blogimage/raw/master/img/20210908171111.png)

## 1.3 Hystrix 实现服务熔断（重点）

* 类似于我们家用的保险丝，当某服务出现不可用或响应超时的情况时，为了防止整个系统出现雪崩，暂时 **`停止对该服务的调用`** 。过一段时间，服务器会慢慢进行恢复，直到完全恢复服务提供。

* **服务熔断过程：** **`请求太多`** → **`服务器扛不住，停止服务`** → **`一段时间后`** → **`逐渐接受请求处理`** → **`最终恢复服务`**



服务熔断，在 **`服务端`** 进行配置。使用到的还是 **`@HystrixCommand`** 注解



1. **配置服务熔断（`@HystrixCommand` 注解）**

   1. **`fallbackMethod`** 来定义服务降级处理方式；
   2. **`@HystrixProperty`** 属性，用来定义服务熔断相关参数（具体有哪些参数可配置，你可以在HystrixCommandProperties.java 类中查看）
   3. **下面配置的含义：** 在 10s 内，10次请求如果失败率 > 60%，此时断路器开关打开，整个服务不可用。随着请求正确率的提升，服务也会逐渐恢复。

   ```java
   @Service
   public class PaymentService {
       //=========服务熔断，配置在10秒内，10次请求中有60%都是失败的，则触发服务熔断
       @HystrixCommand(fallbackMethod = "paymentCircuitBreaker_fallback",commandProperties = {
               @HystrixProperty(name = "circuitBreaker.enabled",value = "true"), // 是否开启断路器
               @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold",value = "10"), // 请求次数
               @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds",value = "10000"), // 时间窗口期
               @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage",value = "60"), // 失败率达到多少后跳闸
       })
       public String paymentCircuitBreaker(@PathVariable("id") Integer id) {
           if(id < 0) {
               throw new RuntimeException("******id 不能负数");
           }
           String serialNumber = IdUtil.simpleUUID(); // 等价于UUID.randomUUID().toString();
   
           return Thread.currentThread().getName()+"\t"+"调用成功，流水号: " + serialNumber;
       }
   
       public String paymentCircuitBreaker_fallback(@PathVariable("id") Integer id) {
           return "id 不能负数，请稍后再试，/(ㄒoㄒ)/~~   id: " +id;
       }
   
   
   }
   ```

2. **编写服务提供者Controller**

   ```java
   @RestController
   @Slf4j
   public class PaymentController {
       @Resource
       private PaymentService paymentService;
       //===服务熔断
       @GetMapping("/payment/circuit/{id}")
       public String paymentCircuitBreaker(@PathVariable("id") Integer id){
           String result = paymentService.paymentCircuitBreaker(id);
           log.info("*******result:"+result);
           return result;
       }
   }
   ```

3. **服务熔断测试**

   * 进行服务熔断测试，**`http://localhost:8001/payment/circuit/{id}`** 请求调用。当 id > 0，正确返回；当 id < 0 时，抛出异常，进入 **`服务降级`** 方法。

   * 当我们输入 id < 0，连续发送请求，满足 10s 内，10次请求如果失败率 > 60%，此时断路器开关打开。此时再次输入 id > 0 时，由于整个服务不可用，还是会进入到 **`服务降级`** 方法。连续多次发送 id > 0 请求，此时正确率逐渐提高，服务也会逐渐恢复。

   ![20200712182829463](https://gitee.com/jobim/blogimage/raw/master/img/20210908172544.gif)

<hr/>



**断路器开启/关闭条件：**

![image-20210908172810994](https://gitee.com/jobim/blogimage/raw/master/img/20210908172811.png)





<hr/>



**Hystrix 全部配置：**

```java
@HystrixCommand(fallbackMethod = "str_fallbackMethod",
    groupKey = "strGroupCommand",
    commandKey = "strCommand",
    threadPoolKey = "strThreadPool",
    commandProperties = {
        //设置执行隔离策略，THREAD 表示线程池   SEMAPHORE:信号量隔离    默认为THREAD线程池
        @HystrixProperty(name = "execution.isolation.strategy", value = "THREAD"),
        // 当隔离策略选择信号池隔离的时候，用来设置信号池的大小(最大并发数)
        @HystrixProperty(name = "execution.isolation.semaphore.maxConcurrentRequests", value = "10"),
        // 配置命令执行的超时时间
        @HystrixProperty(name = "execution.isolation.thread.timeoutinMilliseconds", value = "10"),
        // 是否启用超时时间
        @HystrixProperty(name = "execution.timeout.enabled", value = "true"),
        // 执行超时的时候是否中断
        @HystrixProperty(name = "execution.isolation.thread.interruptOnTimeout", value = "true"),
        // 执行被取消的时候是否中断
        @HystrixProperty(name = "execution.isolation.thread.interruptOnCancel", value = "true"),
        // 允许回调方法执行的最大并发数
        @HystrixProperty(name = "fallback.isolation.semaphore.maxConcurrentRequests", value = "10"),
        // 服务降级是否启用，是否执行回调函数
        @HystrixProperty(name = "fallback.enabled", value = "true"),
        // 设置断路器是否起作用。
        @HystrixProperty(name = "circuitBreaker.enabled", value = "true"),
        // 该属性用来设置在滚动时间窗中，断路器熔断的最小请求数。例如，默认该值为 20 的时候，
        // 如果滚动时间窗(默认10s)内仅收到了19个请求，及时这19个请求都失败了，断路也不会打开。
        @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "20"),
        // 该属性用来设置在滚动时间窗中，表示在滚动时间窗中，在请求数量超过 circuitBreaker.requestVolumeThreshold 的情况下，
        // 如果错误请求数的百分比超过 50，就把断路器设置为"打开"状态，否则就设置为"关闭"状态
        @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage", value = "50"),
        // 该属性用来设置当断路器打开之后的休眠时间窗。休眠时间窗结束之后，会将断路器置为"半开"状态，
        // 尝试熔断的请求命令，如果依然失败就将断路器继续设置为"打开"状态，如果成功就设置为"关闭"状态
        @HystrixProperty(name = "circuitBreaker.sleepWindowinMilliseconds", value = "5000"),
        // 断路器强制打开
        @HystrixProperty(name = "circuitBreaker.forceOpen", value = "false"),
        // 断路器强制关闭
        @HystrixProperty(name = "circuitBreaker.forceClosed", value = "false"),
        // 滚动时间窗设置，该时间用于断路器判断健康度时，需要收集信息的持续时间
        @HystrixProperty(name = "metrics.rollingStats.timeinMilliseconds", value = "10000"),
        // 该属性用来设置滚动时间窗统计指标信息时，划分"桶"的数量，断路器在手机指标信息的时候会根据设置的时间窗长度拆分成多个"桶"来累计各度量值，每个
        // "桶"记录了一段时间内的采集指标。比如 10 秒内拆分成 10 个"桶'收集这样，所以 timeinMilliseconds 必须能被 numBuckets 整除。否则会抛异常
        @HystrixProperty(name = "metrics.rollingStats.numBuckets", value = "10"),
        // 该属性用来设置对命令执行的延迟是否采用百分位数来跟踪和计算。如果设置为 false，name所有的概要统计都将返回-1
        @HystrixProperty(name = "metrics.rollingPercentile.enabled", value = "false"),
        // 该属性用来设置百分位统计的滚动窗口的持续时间，单位为毫秒
        @HystrixProperty(name = "metrics.rollingPercentile.timeInMilliseconds", value = "60000"),
        // 该属性用来设置百分位统计滚动窗口中使用 "桶" 的数量
        @HystrixProperty(name = "metrics.rollingPercentile.numBuckets", value = "60000"),
        // 该属性用来设置在执行过程中每个"桶"中保留的最大执行次数。如果在滚动时间窗内发生超过该设定值的执行次数
        // 就从最初的位置开始重写。例如，将该值设置为100，滚动窗口为10秒，若在10秒内一个"桶"中发生了500次执行，
        // 那么该"桶"中只保留最后的100次执行的统计。另外，增加该值的大小将会增加内存量的消耗，并增加排序百分位数所需的计算时间
        @HystrixProperty(name = "metrics.rollingPercentile.bucketSize", value = "100"),
        // 该属性用来设置采集意向断路器状态的健康快照(请求的成功、错误百分比)的间隔等待时间
        @HystrixProperty(name = "metrics.healthSnapshot.intervalinMilliseconds", value = "500"),
        // 是否开启请求缓存
        @HystrixProperty(name = "requestCache.enabled", value = "true"),
        // HystrixCommand 的执行和事件是否打印日志到 HystrixRequestLog 中
        @HystrixProperty(name = "requestLog.enabled", value = "true")
    },
    threadPoolProperties = {
        // 该参数用来设置执行命令线程池的核心线程数，该值也就是命令执行的最大并发量
        @HystrixProperty(name = "coreSize", value = "10"),
        // 该参数用来设置线程池的最大队列大小。当设置为 -1 时，线程池将使用 SynchronousQueue 实现的队列，否则将使用 LinkedBlockingQueue 实现的队列
        @HystrixProperty(name = "maxQueueSize", value = "-1"),
        // 该参数用来为队列设置拒绝阈值。通过该参数，即使队列没有达到最大值也能拒绝请求。该参数主要是对 LinkedBlockingQueue 队列的补充，因为LinkedBlockingQueue
        // 队列不能动态修改它的对象大小，而通过该属性就可以调整拒绝请求的队列大小了
        @HystrixProperty(name = "queueSizeRejectionThreshold", value = "5")
    }
)
```



**Hystrix Dashboard 可视化监控：**[Hystrix Dashboard 可视化监控](https://blog.csdn.net/m0_37989980/article/details/108508173)

# 二、Zuul路由网关（待补）



# 三、SpringCloud Gateway网关

## 3.1 Gateway介绍

> 好的博客：[SpringCloud gateway （史上最全）](https://www.cnblogs.com/crazymakercircle/p/11704077.html)
>
> [Gateway网关简介及使用](https://blog.csdn.net/rain_web/article/details/102469745)



* **Spring Cloud Gateway 是在 Spring Cloud 的一个全新项目，基于 `Spring5`、`Spring Boot 2` 和 `Project Reactor` 等技术开发的网关，`它旨在为微服务提供一种简单有效的统一的 API 路由管理`。**

* Spring Cloud Gateway 作为 Spring Cloud 生态系统中的网关，目标是替代 Zuul。在 Spring Cloud 2.0 以上版本中，没有对新版本的 Zuul 2.0 以上版本进行集成，使用的使用时 Zuul 1.x 非 Reactor 模式的老版本。

* **为了提升网关的性能，Spring Cloud Gateway 基于 WebFlux 框架实现，WebFlux 框架底层使用的时高性能 Reactor 模式通信框架 Netty，对于高并发、非阻塞式通信有非常大的优势。**

* **Spring Cloud Gateway 的目标是：提供统一的路由方式，基于 Filter 链的方式提供了网关的基本功能，例如：`安全`、`监控/指标`、`权限管理`、`限流` 等**



**GateWay 具有的特性：**

1. 基于 Spring Framework 5、Project Reactor 和 Spring Boot 2.0 进行构建；
2. 动态路由：能够匹配任何请求属性；
3. 可以对路由指定 **`Predicate`**(断言) 和 **`Filter`**(过滤器)，易于编写；
4. 集成 Hystrix 的断路器功能；
5. 集成 Spring Cloud 服务发现功能（Gateway一样可注册到 Eureka）；
6. 请求限流功能；
7. 支持路径重写；
8. Spring 自家产品，更稳定。



**微服务架构网关所在位置：**

![image-20210908180427510](https://gitee.com/jobim/blogimage/raw/master/img/20210908180427.png)



<hr/>

**Gateway 的三大概念**

**Route（路由）**：路由是构建网关的基本模块，它由 ID、目标 URI、一系列的断言和过滤器组成，如果断言为 true 则匹配该路由

**Predicate（断言）**：*参考的是 Java8 中的 java.util.function.Predicate*。开发人员可以匹配 HTTP 请求中的所有内容（例如请求头或请求参数），如果请求与断言相匹配则进行路由

**Filter（过滤）**：指的是 Spring 框架中 GatewayFilter 的实例，使用过滤器，可以在请求被路由之前或之后对请求进行修改

![image-20220112193001909](https://gitee.com/jobim/blogimage/raw/master/img/20220112193001.png)



<hr/>

**工作流程：**

![image-20220112193016456](https://gitee.com/jobim/blogimage/raw/master/img/20220112193016.png)

客户端向 Spring Cloud Gateway 发出请求。如果网关处理程序映射确定请求与路由匹配，则将其发送到网关 Web 处理程序。该处理程序通过特定于请求的过滤器链来运行请求。 **筛选器由虚线分隔的原因是，筛选器可以在发送代理请求之前和之后运行逻辑。**所有 “前置“ 过滤器逻辑均被执行，然后发出代理请求，发出代理请求后，将运行“ 后置 ”过滤器逻辑。

**总结：路由转发 + 执行过滤器链**



## 3.2 Gateway快速入门

新建一个网关模块，名称为：`cloud-gateway-gateway9527` 

**项目结构：**

![image-20220112211922931](https://gitee.com/jobim/blogimage/raw/master/img/20220112211922.png)

* 在 application.yml 中，需要进行详细的路由规则配置。当前场景，多个微服务对应端口分别为：

| 端口 | 功能                | 提供服务说明                                                 |
| ---- | ------------------- | ------------------------------------------------------------ |
| 7001 | 注册服务中心 Eureka | 8001、8002、9527 进行服务注册                                |
| 8001 | 服务提供方(服务端)  | 1.提供 **`/payment/get/{id}`** 服务接口(返回一个字符串) 2.提供 **`/payment/lb`** 服务接口(负载均衡，该接口返回集群中具体提供服务的端口号) |
| 8002 | 服务提供方(服务端)  | 同 8001 组成集群方式                                         |
| 9527 | 网关                |                                                              |



**1、引入 pom.xml 依赖**

```xml
<!--gateway-->
<!-- gateway和spring web+actuator不能同时存在，即web相关jar包不能导入 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
<!--eureka-client-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

**`注意：`** 引入 spring-cloud-starter-gateway 依赖，需要移除 spring-boot-starter-web 和 spring-boot-starter-actuator 这两个依赖，否则会报错：**`'org.springframework.http.codec.ServerCodecConfigurer' that could not be found.`**

 **2、配置文件 application.yml 修改**

```yml
# 端口
server:
  port: 9527
# Eureka 相关配置
eureka:
  instance:
    hostname: cloud-gateway-service
  client:
    service-url:
      register-with-eureka: true
      fetch-registry: true
      defaultZone: http://eureka7001.com:7001/eureka
# 应用服务名   
spring:
  application:
    name: cloud-gateway
  # 以下会是网关具体配置，此处先不介绍，接下来会介绍
  cloud:
  	routes:
	  # gateway网关详细配置
```

**3、主启动类**

```java
@SpringBootApplication
@EnableEurekaClient
public class GateWayMain9527 {
    public static void main(String[] args) {
        SpringApplication.run(GateWayMain9527.class,args);
    }
}
```



**4、gateway 路由规则配置（重点）**

* **单机版(仅使用 8001 单机提供服务)**

  **网关也可以起到变相保护服务端口的作用：我们目前不想暴露8001端口，希望在8001外面套一层9527。**以下就是对8001端口服务 url 的网关配置信息：

  ```yml
  server:
    port: 9527
  spring:
    application:
      name: cloud-gateway
    cloud:
    	# gateway 网关配置
      gateway:
        routes:
        - id: payment_routh #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: http://localhost:8001   #匹配后提供服务的路由地址
          predicates:
          - Path=/payment/get/**   #断言,路径相匹配的进行路由
     	  - id: payment_routh2
          uri: http://localhost:8001
          predicates:
          - Path=/payment/lb/**   #断言,路径相匹配的进行路由
  ```

  **服务调用测试结果：**

  ![image-20220113110507190](https://gitee.com/jobim/blogimage/raw/master/img/20220113110507.png)

* **集群版( 8001、8002 集群提供服务，通过微服务名实现动态路由)**

  开启集群方式提供服务，网关配置部分， uri 就不能采用 **`http://localhost:8081`** 直接写死的方式，这样是无法达到负载均衡效果的。此处需要使用 **`服务名`** 的方式进行调用。

  在配置上，需要两个地方修改：

  1. **`spring.cloud.gateway.discovery.locator.enabled = true`** 开启从注册中心动态获取路由的功能，利用微服务名进行路由；
  2. **`uri`** 配置，使用 **`lb://服务名`** 的方式 **（uri 以 lb:// 开头，lb 代表从注册中心获取服务）**

  ```yml
  server:
    port: 9527
  spring:
    application:
      name: cloud-gateway
    cloud:
      gateway:
        discovery:
          locator:
            enabled: true  #开启从注册中心动态创建路由的功能，利用微服务名进行路由
        routes:
        - id: payment_routh #路由的ID，没有固定规则但要求唯一，建议配合服务名
  #        uri: http://localhost:8001   #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service #集群服务路由地址
          predicates:
          - Path=/payment/get/**   #断言,路径相匹配的进行路由  
        - id: payment_routh2
  #        uri: http://localhost:8001
          uri: lb://cloud-payment-service
          predicates:
          - Path=/payment/lb/**   #断言,路径相匹配的进行路由
  ```

  **服务调用测试结果： 通过 9527，已经实现负载均衡(轮询)**

  ![image-20220113110845205](https://gitee.com/jobim/blogimage/raw/master/img/20220113110845.png)

### 3.2.1 两种配置方式

**1、application.yml 中配置 `（推荐）`**

* 参考快速入门配置

2、代码方式配置 **`(使用 @Bean 代码中注入 RouteLocator 的方式 )`** **`（不推荐）`**

> 实现目标：通过网关访问百度的网站

```java
@Configuration
public class GateWayConfig {
    /**
     * 配置了一个id为route-name的路由规则，
     * 当访问地址 http://localhost:9527/guonei时会自动转发到地址：http://news.baidu.com/guonei
     * @param builder
     * @return
     */
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        RouteLocatorBuilder.Builder routes = builder.routes();
        routes.route("path_route_atguigu", r -> r.path("/guonei").uri("http://news.baidu.com/guonei")).build();
        return routes.build();

    }
    @Bean
    public RouteLocator customRouteLocator2(RouteLocatorBuilder builder) {
        RouteLocatorBuilder.Builder routes = builder.routes();
        routes.route("path_route_atguigu2", r -> r.path("/guoji").uri("http://news.baidu.com/guoji")).build();
        return routes.build();
    }
}
```



## 3.3 Predicate断言（转发规则）

* **Predicate 就是为了实现一组匹配规则，方便让请求过来找到对应的 Route 进行处理，接下来我们接下 Spring Cloud GateWay 内置几种 Predicate 的使用。**



**转发规则**（predicates），假设 转发uri都设定为***[http://localhost:9023](http://localhost:9023/)***

| 规则        | 实例                                                         | 说明                                                         |
| ----------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **Path**    | - Path=/gate/**,/rule/**                                     | 当请求的路径为gate、rule开头的时，转发到http://localhost:9023服务器上 |
| **Before**  | - Before=2017-01-20T17:42:47.789-07:00[America/Denver]       | 在某个时间之前的请求才会被转发到 http://localhost:9023服务器上 |
| **After**   | - After=2017-01-20T17:42:47.789-07:00[America/Denver]        | 在某个时间之后的请求才会被转发                               |
| **Between** | - Between=2017-01-20T17:42:47.789-07:00[America/Denver],2017-01-21T17:42:47.789-07:00[America/Denver] | 在某个时间段之间的才会被转发                                 |
| **Cookie**  | - Cookie=chocolate, ch.p                                     | 名为chocolate的表单或者满足正则ch.p的表单才会被匹配到进行请求转发 |
| **Header**  | - Header=X-Request-Id, \d+                                   | 携带参数X-Request-Id或者满足\d+的请求头才会匹配              |
| **Host**    | - Host=www.hd123.com                                         | 当主机名为www.hd123.com的时候直接转发到http://localhost:9023服务器上 |
| **Method**  | - Method=GET                                                 | 只有GET方法才会匹配转发请求，还可以限定POST、PUT等请求方式   |



**1、After Route Predicate**

**application.yml**

```yml
spring:
  cloud:
    gateway:
      routes:
      - id: after_route
        uri: https://example.org
        predicates:
        - After=2017-01-20T17:42:47.789-07:00[America/Denver]
```

**说明：** **`这个路由规则，在 2017年1月29号 17:42:47.789 后访问才会生效。`** 此处日期可以使用 **`ZonedDateTime zonedDateTime = ZonedDateTime.now();`** 来生成



**2、Cookie Route Predicate**

**application.yml**

**说明： 这个路由规则，需要两个参数，一个是 Cookie name，一个是正则表达式。如果能够匹配 Cookie name值 和正则表达式，就会执行路由，如果没有匹配上则不执行。**

```yml
spring:
  cloud:
    gateway:
      routes:
		- id: payment_routh2
        uri: http://localhost:8001
        predicates:
        - Path=/payment/lb/**   #断言,路径相匹配的进行路由
        - Cookie=username, ch.p\d
```

Predicate 规则为：**`- Cookie=username, ch.p\d`**。**`ch.p\d`** 为正则表达式，**`\d`** 代表数字。即只有cookie 为 **`username=ch.p3`** 能够匹配；**`username=ch.a`**、**`username=ch.pabc`** 能是不能够匹配的；

**测试结果：**

![image-20220113113831187](https://gitee.com/jobim/blogimage/raw/master/img/20220113113831.png)

**3、Others Route Predicate**

  Predicate 断言这块内容，在官网都有很详细的介绍。其他 9 个断言，此处不做过多说明

  官网 Predicate 地址：[Route Predicate Gactories](https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.2.1.RELEASE/reference/html/#gateway-request-predicates-factories)



## 3.4 Gateway网关的过滤器

* Spring-Cloud-Gateway 基于过滤器实现，同 zuul 类似，有**pre**和**post**两种方式的 filter,分别处理**前置逻辑**和**后置逻辑**。客户端的请求先经过**pre**类型的 filter，然后将请求转发到具体的业务服务，收到业务服务的响应之后，再经过**post**类型的 filter 处理，最后返回响应到客户端。

* 过滤器执行流程如下，**order 越大，优先级越低**

  ![image-20220113114407179](https://gitee.com/jobim/blogimage/raw/master/img/20220113114407.png)



分为全局过滤器和局部过滤器（这里主要介绍自定义全局过滤器）

- **全局过滤器：**对所有路由生效

- **局部过滤器：**对指定路由生效



### 3.4.1 自定义全局过滤器

* **自定义全局过滤器，可以帮助我们进行 `全局日志记录`、`统一网关鉴权` 等功能。**

* **我们需要定义一个类，并实现 `GlobalFilter` ，`Ordered` 两个接口，重写里面的 `filter、order` 方法即可（加入到spring容器管理即可，无需配置，全局过滤器对所有的路由都有效）。**
* **主要是 filter 方法，编写详细的过滤器逻辑；  `order`** 方法是用来定义加载过滤器的优先级，返回一个 int 数值，值越小优先级越高



**自定义过滤器：**

* 以组件的方式，在 Spring Boot 启动时被加载。使用 **`@Component`** 进行注释。

  ```java
  @Component
  @Slf4j
  public class MyLogGatewayFilter implements GlobalFilter,Ordered {
      @Override
      public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
          log.info("*********come in MyLogGateWayFilter: "+new Date());
          log.info("*********进入全局过滤器： "+new Date());
          String username = exchange.getRequest().getQueryParams().getFirst("username");
          if(StringUtils.isEmpty(username)){
              log.info("*****用户名为Null 非法用户,(┬＿┬)");
              exchange.getResponse().setStatusCode(HttpStatus.NOT_ACCEPTABLE);//给人家一个回应，设置http状态码
              return exchange.getResponse().setComplete();
          }
          return chain.filter(exchange);
      }
      @Override
      public int getOrder() { //加载过滤器优先级，越小优先级越高
          return 0;
      }
  }
  
  ```

* **测试自定义过滤器**：过滤器需要满足带参数：username。**`链接不带 username 参数，会被过滤掉。`**

  ![image-20220113115015602](https://gitee.com/jobim/blogimage/raw/master/img/20220113115015.png)



* **日志返回：**

  ![image-20220113115044413](https://gitee.com/jobim/blogimage/raw/master/img/20220113115044.png)

