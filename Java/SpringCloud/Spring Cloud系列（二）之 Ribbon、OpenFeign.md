> 使用SpringCloud版本：`2.1.1.RELEASE`

# 一、Ribbon负载均衡服务调用

* Ribbon是Netflix发布的负载均衡器，有助于控制HTTP客户端行为。为Ribbon配置服务提供者地址列表后，Ribbon就可基于负载均衡算法，自动帮助服务消费者请求。

* Ribbon默认提供的负载均衡算法：轮询，随机其他…。当然，我们可用自己定义负载均衡算法

  ![image-20210906100708602](https://gitee.com/jobim/blogimage/raw/master/img/20210906100708.png)

**Ribbon 与 Nginx 区别？**

* **服务器端负载均衡 Nginx**

  nginx 是客户端所有请求统一交给 nginx，由 nginx 进行实现负载均衡请求转发，属于服务器端负载均衡。即请求由 nginx 服务器端进行转发。

* **客户端负载均衡 Ribbon**

  Ribbon 是从 eureka 注册中心服务器端上获取服务注册信息列表，缓存到本地，然后在本地实现轮询负载均衡策略。即在客户端实现负载均衡。



## 1.1 Ribbon负载均衡实现过程

**RestTemplate的使用：**

* **GET请求方法**

  ```java
  <T> T getForObject(String url, Class<T> responseType, Object... uriVariables);
  
  <T> T getForObject(String url, Class<T> responseType, Map<String, ?> uriVariables);
  
  <T> T getForObject(URI url, Class<T> responseType);
  
  <T> ResponseEntity<T> getForEntity(String url, Class<T> responseType, Object... uriVariables);
  
  <T> ResponseEntity<T> getForEntity(String url, Class<T> responseType, Map<String, ?> uriVariables);
  
  <T> ResponseEntity<T> getForEntity(URI var1, Class<T> responseType);
  ```

  * **getForObject方法：**

    返回对象为响应体中数据转化成的对象。

    ```java
    @GetMapping("/consumer/payment/get/{id}")
    public CommonResult<Payment> getPayment(@PathVariable("id") Long id) {
        return restTemplate.getForObject(PAYMENT_URL+"/payment/get/"+id,CommonResult.class);
    }
    ```

    

  * **getForEntity方法**

    返回对象为ResponseEntity对象，包含了响应中的一些重要信息，比如响应头、响应状态码、响应体等，举例如下：

    ```java
    @GetMapping("/consumer/payment/getFroEntity/{id}")
    public CommonResult<Payment> getPayment2(@PathVariable("id") Long id) {
        ResponseEntity<CommonResult> entity = restTemplate.getForEntity(PAYMENT_URL + "/payment/get/" + id, CommonResult.class);
        if(entity.getStatusCode().is2xxSuccessful()) {
            return entity.getBody();
        } else {
            return new CommonResult<>(444,"操作失败");
        }
    }
    ```

* **POST请求方法**

  ```java
  <T> T postForObject(String url, @Nullable Object request, Class<T> responseType, Object... uriVariables);
  
  <T> T postForObject(String url, @Nullable Object request, Class<T> responseType, Map<String, ?> uriVariables);
  
  <T> T postForObject(URI url, @Nullable Object request, Class<T> responseType);
  
  <T> ResponseEntity<T> postForEntity(String url, @Nullable Object request, Class<T> responseType, Object... uriVariables);
  
  <T> ResponseEntity<T> postForEntity(String url, @Nullable Object request, Class<T> responseType, Map<String, ?> uriVariables);
  
  <T> ResponseEntity<T> postForEntity(URI url, @Nullable Object request, Class<T> responseType);
  ```

  代码示例：

  ```java
  @PostMapping("/insert")
  public Result insert(@RequestBody User user) {
      return restTemplate.postForObject(userServiceUrl + "/user/insert", user, Result.class);
  }
  ```



<hr/>

**实现过程：**

1. **导入依赖**

   ```xml
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
   </dependency>
   ```

   注意eureka的客户端中包含ribbon依赖，所以如果导入了eureka依赖就不用再导入ribbon依赖了。

   ![image-20210906103358288](https://gitee.com/jobim/blogimage/raw/master/img/20210906103358.png)

2. **在Eureka中注册服务提供者集群**（8001、8002）

   ![image-20210906104331549](https://gitee.com/jobim/blogimage/raw/master/img/20210906104331.png)

3. **编写服务提供者的Controller方法**

   ```java
   @GetMapping(value = "/payment/get/{id}")
   public CommonResult<Payment> getPaymentById(@PathVariable("id") Long id){
       Payment payment = paymentService.getPaymentById(id);
       log.info("*****查询结果:{}",payment);
       int w = 5/2;
       if (payment != null) {
           return new CommonResult(200,"查询成功"+"， 服务端口："+serverPort,payment);
       }else{
           return new CommonResult(444,"没有对应记录,查询ID: "+id,null);
       }
   }
   ```

4. 在服务消费者配置类中**给RestTemplate这个Bean添加一个`@LoadBalanced`注解，用来实现负载均衡功**

   ```java
   @Configuration
   public class ApplicationContextConfig {
   
       @Bean
       @LoadBalanced //使用@LoadBalanced注解赋予RestTemplate负载均衡的能力
       public RestTemplate getRestTemplate() {
           return new RestTemplate();
       }
   
   }
   ```

5. 编写服务消费者中的Controller中的方法。**不再手动获取ip和端口，而是直接通过服务名称调用**

   ```java
   @Resource
   private RestTemplate restTemplate;
   public static final String PAYMENT_URL = "http://CLOUD-PAYMENT-SERVICE";
   @GetMapping("/consumer/payment/get/{id}")
   public CommonResult<Payment> getPayment(@PathVariable("id") Long id) {
       return restTemplate.getForObject(PAYMENT_URL+"/payment/get/"+id,CommonResult.class);
   }
   ```

6. 访问测试，spring会自动帮助我们从eureka-server端，**根据服务名称，获取实例列表，而后完成负载均衡**

   ![image-20210905104637384](https://gitee.com/jobim/blogimage/raw/master/img/20210905104637.png)



## 1.2 Ribbon的负载均衡策略

所谓的负载均衡策略，就是当A服务调用B服务时，此时B服务有多个实例，这时A服务**以何种方式**来选择调用的B实例，ribbon可以选择以下几种负载均衡策略。

| **内置负载均衡规则类**    | **规则描述**                                                 |
| ------------------------- | ------------------------------------------------------------ |
| **RoundRobinRule**        | 简单**轮询**服务列表来选择服务器。它是Ribbon默认的负载均衡规则。 |
| AvailabilityFilteringRule | 先过滤掉故障实例，再选择并发较小的实例                       |
| WeightedResponseTimeRule  | 对RoundRobinRule的扩展，响应速度越快的实例选择权重越大，越容易被选择 |
| ZoneAvoidanceRule         | 采用双重过滤，同时过滤不是同一区域的实例和故障实例，选择并发较小的实例。 |
| BestAvailableRule         | 会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务，然后选择一个并发量最小的服务 |
| RandomRule                | **随机**选择一个可用的服务器                                 |
| RetryRule                 | 先按照RoundRobinRule的策略获取服务，如果获取服务失败则在指定时间内会进行重试，获取可用的服务 |

**继承关系：**

![image-20210906100730837](https://gitee.com/jobim/blogimage/raw/master/img/20210906100731.png)





**修改Ribbon负载均衡策略（修改成随机选择）：**

> 注意：官方文档明确给出了警告
> 这个自定义配置类不能放在@ComponentScan所扫描的当前包下以及子包下，否则我们自定义的这个配置类就会被所有的Ribbon客户端所共享，达不到特殊化定制的目的了。

1. **在@ComponentScan所扫描不到的包中新建配置类`MySelfRule`**

   ![image-20210906110011717](https://gitee.com/jobim/blogimage/raw/master/img/20210906110011.png)

   ```java
   @Configuration
   public class MySelfRule {
       @Bean
       public IRule myRule(){
           return new RandomRule();//定义为随机
       }
   }
   ```

2. **在主启动类添加@RibbonClient注解**

   ```java
   @SpringBootApplication
   @EnableEurekaClient
   // 用来指定CLOUD-PAYMENT-SERVICE服务采用MySelfRule配置类中配置的负载均衡策略
   @RibbonClient(name = "CLOUD-PAYMENT-SERVICE",configuration= MySelfRule.class)
   public class OrderMain80 {
   
       public static void main(String[] args) {
           SpringApplication.run(OrderMain80.class, args);
       }
   
   }
   ```

3. 测试可以发现restTemplate将使用随机调用（注意使用restTemplate调用时服务名称改成大写，否则不起作用）

## 1.3 负载均衡算法原理分析

负载均衡算法：**rest接口第几次请求数 % 服务器集群总数量 = 实际调用服务器位置下标  ，每次服务重启动后rest接口计数从1开始。**

```
List<ServiceInstance> instances = discoveryClient.getInstances("CLOUD-PAYMENT-SERVICE");
如：	   List [0] instances = 127.0.0.1:8002
		List [1] instances = 127.0.0.1:8001
 
8001 + 8002 组合成为集群，它们共计2台机器，集群总数为2， 按照轮询算法原理：
 
当总请求数为1时： 1 % 2 =1 对应下标位置为1 ，则获得服务地址为127.0.0.1:8001
当总请求数位2时： 2 % 2 =0 对应下标位置为0 ，则获得服务地址为127.0.0.1:8002
当总请求数位3时： 3 % 2 =1 对应下标位置为1 ，则获得服务地址为127.0.0.1:8001
当总请求数位4时： 4 % 2 =0 对应下标位置为0 ，则获得服务地址为127.0.0.1:8002
```





**RoundRobinRule源码：**

```java
public class RoundRobinRule extends AbstractLoadBalancerRule {
    private AtomicInteger nextServerCyclicCounter;
    private static final boolean AVAILABLE_ONLY_SERVERS = true;
    private static final boolean ALL_SERVERS = false;
    private static Logger log = LoggerFactory.getLogger(RoundRobinRule.class);

    public RoundRobinRule() {
        this.nextServerCyclicCounter = new AtomicInteger(0);
    }

    public RoundRobinRule(ILoadBalancer lb) {
        this();
        this.setLoadBalancer(lb);
    }

    public Server choose(ILoadBalancer lb, Object key) {
        if (lb == null) {
            log.warn("no load balancer");
            return null;
        } else {
            Server server = null;
            int count = 0;

            while(true) {
                if (server == null && count++ < 10) {
                    List<Server> reachableServers = lb.getReachableServers();//得到可以连接的server
                    List<Server> allServers = lb.getAllServers();//得到所有的server  并且放在list中了。
                    int upCount = reachableServers.size();//在注册中心连接的。得到数量
                    int serverCount = allServers.size();// 所有server的数量
                    if (upCount != 0 && serverCount != 0) {
                        int nextServerIndex = this.incrementAndGetModulo(serverCount);//关键在于这个方法的的next值。得到list集合中要访问服务器的下标
                        server = (Server)allServers.get(nextServerIndex);
                        if (server == null) {
                            Thread.yield();
                        } else {
                            if (server.isAlive() && server.isReadyToServe()) {
                                return server;
                            }

                            server = null;
                        }
                        continue;
                    }

                    log.warn("No up servers available from load balancer: " + lb);
                    return null;
                }

                if (count >= 10) {
                    log.warn("No available alive servers after 10 tries from load balancer: " + lb);
                }

                return server;
            }
        }
    }
	
    // 通过CAS保证线程安全
    private int incrementAndGetModulo(int modulo) {
        int current;
        int next;
        do {
            current = this.nextServerCyclicCounter.get();//这个是我们的atmic原子类。得到默认的值0
            next = (current + 1) % modulo;//这个是服务的个数。这个next的值如果这一样取，其实就是和serverlist的下标对应，并且，每次加1.
        } while(!this.nextServerCyclicCounter.compareAndSet(current, next));//因为是多线程，我们要保证安全，所以一用了循环的CAS

        return next;
    }

    public Server choose(Object key) {
        return this.choose(this.getLoadBalancer(), key);
    }

    public void initWithNiwsConfig(IClientConfig clientConfig) {
    }
}
```





# 二、OpenFeign服务接口调用

* 在认识 Feign 之前，微服务之间调用。我们使用的是 **`Ribbon + RestTemplate`** 方式

* Feign是**声明式的服务调用工具，我们只需创建一个接口并用注解的方式来配置它，就可以实现对某个服务接口的调用**，简化了直接使用RestTemplate来调用服务接口的开发量。Feign具备可插拔的注解支持，同时支持Feign注解、JAX-RS注解及SpringMvc注解。当使用Feign时，Spring Cloud集成了Ribbon和Eureka以提供负载均衡的服务调用及基于Hystrix的服务容错保护功能。



## 2.1 OpenFeign使用步骤

**项目架构：**

![image-20210906220338017](https://gitee.com/jobim/blogimage/raw/master/img/20210906220338.png)

1. 新建一个`cloud-consumer-feign-order80`模块

2. **导入依赖**

   ```xml
   <!--openfeign-->
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-openfeign</artifactId>
   </dependency>
   <!--eureka client-->
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
   </dependency>
   <!-- 引入自己定义的api通用包，可以使用Payment支付Entity -->
   <dependency>
       <groupId>com.zb.springcloud</groupId>
       <artifactId>cloud-api-commons</artifactId>
       <version>1.0-SNAPSHOT</version>
   </dependency>
   <!--web-->
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-web</artifactId>
   </dependency>
   ```

3. **编写application.yml文件**

   ```yml
   server:
     port: 80
   
   eureka:
     client:
       # 表示是否将自己注册进EurekaServer默认为true。
       register-with-eureka: false
       service-url:
         defaultZone: http://eureka7001.com:7001/eureka/
   ```

4. **启动类上添加`@EnableFeignClients`注解来启用Feign的客户端功能**

   ```java
   @SpringBootApplication
   @EnableFeignClients // 激活并开启Feign
   public class OrderFeignMain80 {
   
       public static void main(String[] args) {
           SpringApplication.run(OrderFeignMain80.class, args);
       }
   
   }
   ```

5. **添加Service接口完成对服务提供者接口绑定**

   >我们通过@FeignClient注解实现了一个Feign客户端，其中的value指定服务提供方服务名称

   ```java
   @Component
   @FeignClient(value = "CLOUD-PAYMENT-SERVICE") //添加@FeignClient注解,指定服务提供方服务名称
   public interface PaymentFeignService {
   
       @GetMapping(value = "/payment/get/{id}")
       CommonResult<Payment> getPaymentById(@PathVariable("id") Long id);
   
       @GetMapping(value = "/payment/feign/timeout")
       String paymentFeignTimeOut();
   
   }
   ```

6. **添加Controller调用Service实现服务调用**

   ```java
   @RestController
   @Slf4j
   public class OrderFeignController {
   
       @Autowired
       private PaymentFeignService paymentFeignService;
   
       @GetMapping(value = "/consumer/payment/get/{id}")
       public CommonResult<Payment> getPaymentById(@PathVariable("id") Long id) {
           return paymentFeignService.getPaymentById(id);
       }
   
       @GetMapping(value = "/payment/feign/timeout")
       public String paymentFeignTimeOut(){
           // openfeign-ribbon,客户端一般默认等待1秒钟
           return paymentFeignService.paymentFeignTimeOut();
       }
   
   }
   ```

7. 测试，可以发现运行在8001和8002的服务提供者交替打印（**Feign自带负载均衡配置项**）

   ![image-20210906215921268](https://gitee.com/jobim/blogimage/raw/master/img/20210906215921.png)

**关联关系：**

![image-20210906220126490](https://gitee.com/jobim/blogimage/raw/master/img/20210906220126.png)



## 2.2 超时控制

**`默认 Feign 客户端只等待 1 秒钟`**，但是服务端处理需要超过1秒钟，从而导致 Feign 客户端不再继续等待，**`直接以超时报错的方式返回`**。**为了避免这类情况，我们就需要设置 Feign 客户端的超时控制。**



* **服务端处理需要超过 1 秒钟，从而导致 Feign 客户端直接以报错的方式返回**

  ![image-20210906220626086](https://gitee.com/jobim/blogimage/raw/master/img/20210906220626.png)

* **设置 Feign 超时时间（此时访问正常显示）**

  ```yml
  #设置feign客户端超时时间(OpenFeign默认支持ribbon)
  ribbon:
    # 指的是建立连接后从服务器读取到可用资源所用的时间
    ReadTimeout: 5000
    # 指的是建立连接所用的时间，适用于网络状况正常的情况下,两端连接所用的时间
    ConnectTimeout: 5000
  ```

  

## 2.3 日志打印


Feign 提供了日志打印功能，我们可以通过配置来调整日志级别，从而了解 Feign 中 Http 请求的细节。就是对Feign接口的调用情况进行监控和输出

| 日志级别 | 说明                                      |
| -------- | ----------------------------------------- |
| NONE     | 默认的，不显示任何日志                    |
| BASIC    | 仅记录请求方法、URL、响应状态码、执行时间 |
| HEADERS | 除了 BASIC 中定义的信息外，还有请求和响应的头信息           |
| FULL    | 除了 HEADERS 中定义的信息之外，还有请求和响应的正文及元数据 |



**设置 Feign 日志级别：**

1. **定义Feign 配置类**

   ![image-20210906220929096](https://gitee.com/jobim/blogimage/raw/master/img/20210906220929.png)

2. **application.yml 配置日志输出级别**

   ```yml
   logging:
     level:
       # feign日志以什么级别监控哪个接口
       com.zb.springcloud.service.PaymentFeignService: debug
   ```

3. **Feign日志输出**

   ![image-20210906221105158](https://gitee.com/jobim/blogimage/raw/master/img/20210906221105.png)

