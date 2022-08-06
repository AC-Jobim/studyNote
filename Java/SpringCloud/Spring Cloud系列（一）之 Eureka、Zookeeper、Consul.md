

> 使用SpringCloud版本：`2.1.1.RELEASE`

# 一、微服务架构介绍

## 1.1 架构的演变

**<font color='blue'>单一应用架构</font>**

* 当网站流量很小时，只需要一个应用，所有功能打成一个包部署在一起，减少部署节点成本的框架称之为集中式框架。此时，用于简化增删改查工作量的数据访问框架(ORM)是影响项目开发的关键。

  ![image-20210713202807818](https://gitee.com/jobim/blogimage/raw/master/img/20210904212757.png)

**优点：**

* 架构简单
* 部署成本低

**缺点：**

* 耦合度高（维护困难、升级困难）



**<font color='blue'>分布式服务架构</font>**

* 根据业务功能对系统做拆分，每个业务功能模块作为独立项目开发，称为一个服务。

  ![image-20210713203124797](https://gitee.com/jobim/blogimage/raw/master/img/20210904212948.png)

**优点：**

* 降低服务耦合
* 有利于服务升级和拓展

**缺点：**

* 服务调用关系错综复杂



分布式架构虽然降低了服务耦合，但是服务拆分时也有很多问题需要思考：

- 服务拆分的粒度如何界定？
- 服务之间如何调用？
- 服务的调用关系如何管理？

人们需要制定一套行之有效的标准来约束分布式架构。



**<font color='blue'>微服务架构：</font>**

微服务的架构特征：

* 单一职责：**微服务拆分粒度更小**，每一个服务都对应唯一的业务能力，做到单一职责
* 自治：团队独立、技术独立、数据独立，**独立部署和交付**
* 面向服务：服务提供统一标准的接口，与语言和技术无关
* 隔离性强：服务调用做好隔离、容错、降级，避免出现级联问题

微服务的上述特性其实是在给分布式架构制定一个标准，进一步降低服务之间的耦合度，提供服务的独立性和灵活性。做到高内聚，低耦合。

因此，可以认为**微服务**是一种经过良好架构设计的**分布式架构方案** 。其中微服务架构的最佳实践是SpringCloud。





## 1.2 Spring Cloud介绍

* Spring Cloud是一系列分布式微服务技术的有序整合，把非常流行的微服务的技术整合到一起。它利用Spring Boot的开发便利性巧妙地简化了分布式系统基础设施的开发。
* 官网地址：https://spring.io/projects/spring-cloud。



**其中常见的组件包括：**

![image-20210713204155887](https://gitee.com/jobim/blogimage/raw/master/img/20210904214143.png)

**由于升级和停更引发的技术变更：（圈起来的是阳哥推荐的技术）**

![image-20211125172027104](https://gitee.com/jobim/blogimage/raw/master/img/20211125172027.png)

* 另外，SpringCloud底层是依赖于SpringBoot的，并且有版本的兼容关系，如下：

  https://spring.io/projects/spring-cloud#overview
  
  ![image-20210713205003790](https://gitee.com/jobim/blogimage/raw/master/img/20210904214140.png)







# 二、微服务架构业务场景

* **目标：模拟一个最简单的服务调用场景，场景中保护微服务提供者(Producer)和微服务调用者(Consumer)**，方便后面学习微服务架构

* **注意：实际开发中，每个微服务为一个独立的SpringBoot工程**

**项目架构：**

![image-20210904220306317](https://gitee.com/jobim/blogimage/raw/master/img/20210904220306.png)



## 2.1 创建服务提供者(provider)工程

**主要工作**：创建SpringBoot工程（cloud-provider-payment8001）、加入依赖坐标、编写配置、编写MVC架构代码。

* **application.yml配置文件代码：**

  ```yml
  server:
    port: 8001
  
  spring:
    application:
      name: cloud-payment-service
    datasource:
      type: com.alibaba.druid.pool.DruidDataSource            # 当前数据源操作类型
      driver-class-name: org.gjt.mm.mysql.Driver              # mysql驱动包 com.mysql.jdbc.Driver
      url: jdbc:mysql://localhost:3306/db2019?useUnicode=true&characterEncoding=utf-8&useSSL=false
      username: root
      password: root
  
  mybatis:
    mapperLocations: classpath:mapper/*.xml
    type-aliases-package: com.zb.springcloud.entities    # 所有Entity别名类所在包
  ```



* **controller层代码：**

  注意不要忘记`@RequestBody`注解

  ```java
  @RestController
  @Slf4j
  public class PaymentController {
      @Autowired
      private PaymentService paymentService;
  
      @PostMapping(value = "/payment/create")
      public CommonResult crete(@RequestBody Payment payment){
          //注意不要忘记@RequestBody
          int result = paymentService.create(payment);
          log.info("******插入结果：" + result);
          if(result > 0) {
              return new CommonResult(200,"插入数据库成功",result);
          } else {
              return new CommonResult(444, "插入数据库失败", null);
          }
      }
  
      @GetMapping(value = "/payment/get/{id}")
      public CommonResult<Payment> getPaymentById(@PathVariable("id") Long id)
      {
          Payment payment = paymentService.getPaymentById(id);
          log.info("*****查询结果:{}",payment);
          int w = 5/2;
          if (payment != null) {
              return new CommonResult(200,"查询成功",payment);
          }else{
              return new CommonResult(444,"没有对应记录,查询ID: "+id,null);
          }
      }
  
  }
  ```

  

* 访问测试：http://localhost:8001/payment/get/3

  ![image-20210904220830133](https://gitee.com/jobim/blogimage/raw/master/img/20210904220830.png)

## 2.2 创建服务消费者(consumer)工程

**主要工作：**

1. 创建消费者SpringBoot工程（cloud-consumer-order80）、添加依赖
2. 注册http请求客户端对象RestTemplate
3. 编写Controller，用RestTemplate访问服务提供者



* **注册RestTemplate实例**

  RestTemplate是Spring提供的用于访问Rest服务的客户端，提供了多种便捷访问远程Http服务的方法。

  ```java
  @Configuration
  public class ApplicationContextConfig {
  
      @Bean //注册RestTemplate实例
      public RestTemplate getRestTemplate() {
          return new RestTemplate();
      }
  
  }
  ```

  ![image-20220105172001101](https://gitee.com/jobim/blogimage/raw/master/img/20220105172001.png)

* **实现远程调用**

  ```java
  @RestController
  @Slf4j
  public class OrderController {
  
      public static final String PAYMENT_URL = "http://localhost:8001";
  
      @Resource
      private RestTemplate restTemplate;
  
      @GetMapping("/consumer/payment/create")
      public CommonResult<Payment> create(Payment payment) {
          return restTemplate.postForObject(PAYMENT_URL + "/payment/create",payment,CommonResult.class);
      }
  
      @GetMapping("/consumer/payment/get/{id}")
      public CommonResult<Payment> getPayment(@PathVariable("id") Long id) {
          return restTemplate.getForObject(PAYMENT_URL+"/payment/get/"+id,CommonResult.class);
      }
  
  }
  ```

  



存在的问题：

1. **服务消费者在发起远程调用的时候，该如何得知服务提供者实例的ip地址和端口**。而且把url地址硬编码到代码中，不方便后期维护。
2. 有多个服务提供者实例地址，服务消费者调用时不知道怎么选择。
3. 服务消费者中，不清楚服务提供者的状态（是否宕机）
4. 服务提供者的如果出现故障，不能够及时发现。





# 三、Eureka服务注册与发现

上面那些问题都需要利用SpringCloud中的注册中心来解决，其中最广为人知的注册中心就是Eureka。



## 3.1 搭建注册中心

**项目架构：**

![image-20210905094226103](https://gitee.com/jobim/blogimage/raw/master/img/20210905094226.png)

新建SpringBoot注册中心服务端`cloud-eureka-server7001`

* **引入eureka依赖（eureka-server）**

  ```xml
  <!--eureka-server-->
  <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
  </dependency>
  ```

  

* **编写启动类，一定要添加一个`@EnableEurekaServer`注解**，开启eureka的注册中心功能：

  ```java
  @SpringBootApplication
  @EnableEurekaServer
  public class ServerMain7001 {
  
      public static void main(String[] args) {
          SpringApplication.run(ServerMain7001.class, args);
      }
  
  }
  ```

  

* **application.yml文件编写：**

  ```yml
  server:
    port: 7001
  
  eureka:
    instance:
      hostname: localhost #eureka服务端的实例名称
    client:
      #false表示不向注册中心注册自己。
      register-with-eureka: false
      #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
      fetch-registry: false
      service-url:
        #设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址。
        defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
  ```

  

* 启动微服务，然后在浏览器访问：http://127.0.0.1:7001

  ![image-20210905093906916](https://gitee.com/jobim/blogimage/raw/master/img/20210905093907.png)

  

## 3.2 服务注册（消费者集群）

下面，我们将服务消费者和服务提供者注册到eureka-server中。

**项目架构：**

![image-20220106153753896](https://gitee.com/jobim/blogimage/raw/master/img/20220106153753.png)

### 3.2.1 服务提供者注册

* **引入eureka-client依赖**

  ```xml
  <!--eureka-client-->
  <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
  </dependency>
  ```

* **修改application.yml文件，添加服务名称、eureka地址**

  ```yml
  server:
    port: 8001
  
  spring:
    application:
      name: cloud-payment-service # 指定服务名称
    datasource:
      type: com.alibaba.druid.pool.DruidDataSource            # 当前数据源操作类型
      driver-class-name: org.gjt.mm.mysql.Driver              # mysql驱动包 com.mysql.jdbc.Driver
      url: jdbc:mysql://localhost:3306/db2019?useUnicode=true&characterEncoding=utf-8&useSSL=false
      username: root
      password: root
  
  *****************************************************************************************************
  eureka:
    client:
      #表示是否将自己注册进EurekaServer默认为true。
      register-with-eureka: true
      #是否从EurekaServer抓取已有的注册信息，默认为true。单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
      fetchRegistry: true
      service-url:
        defaultZone: http://localhost:7001/eureka
  *****************************************************************************************************
  
  mybatis:
    mapperLocations: classpath:mapper/*.xml
    type-aliases-package: com.zb.springcloud.entities    # 所有Entity别名类所在包
  ```

* **在启动类上添加`@EnableEurekaClient`，表明是Eureka客户端**

  ```java
  @SpringBootApplication
  @EnableEurekaClient
  public class PaymentMain8001 {
      public static void main(String[] args) {
          SpringApplication.run(PaymentMain8001.class, args);
      }
  }
  ```

### 3.2.2 服务消费者注册

* **引入eureka-client依赖**

  ```xml
  <!--eureka-client-->
  <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
  </dependency>
  ```

* **修改application.yml文件，添加服务名称、eureka地址**

  ```yml
  server:
    port: 80
  
  spring:
    application:
      name: cloud-order-service
  
  eureka:
    client:
      #表示是否将自己注册进EurekaServer默认为true。
      register-with-eureka: true
      #是否从EurekaServer抓取已有的注册信息，默认为true。单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
      fetchRegistry: true
      service-url:
        defaultZone: http://localhost:7001/eureka
  ```

* **在启动类上添加`@EnableEurekaClient`，表明是Eureka客户端**

  ```java
  @SpringBootApplication
  @EnableEurekaClient
  public class MainApp80 {
      public static void main(String[] args) {
          SpringApplication.run(MainApp80.class,args);
      }
  }
  ```

* **查看eureka管理页面：**

  ![image-20220106154734015](https://gitee.com/jobim/blogimage/raw/master/img/20220106154734.png)



## 3.3 EurekaServer集群环境构建

**项目结构：**

![image-20220106154124009](https://gitee.com/jobim/blogimage/raw/master/img/20220106154124.png)

* 新建项目`cloud-eureka-server7002`，用来**搭建EurekaServer集群**。

* **修改C:\Windows\System32\drivers\etc路径下的hosts文件**，加入

  ```
  127.0.0.1  eureka7001.com
  127.0.0.1  eureka7002.com
  ```

  ![image-20210905111841174](https://gitee.com/jobim/blogimage/raw/master/img/20210905111841.png)

* **修改`eureka-server`的application.yml配置文件**

  ![image-20210905111620313](https://gitee.com/jobim/blogimage/raw/master/img/20210905111620.png)

* **将服务提供者和服务消费者同时注册到两个注册中心（可以只注册集群中的一个，信息会在集群中共享），修改每个项目的配置文件application.yml**

  ![image-20210905112748331](https://gitee.com/jobim/blogimage/raw/master/img/20210905112748.png)

  

* 查看eureka管理页面，可以发现**另一个已经成为其备份的注册中心**

  ![image-20210905112832611](https://gitee.com/jobim/blogimage/raw/master/img/20210905112832.png)

## 3.4 搭建服务提供者集群和负载均衡

**项目结构：**

![image-20220106155934958](https://gitee.com/jobim/blogimage/raw/master/img/20220106155935.png)

目标把搭建服务提供者集群注册到eureka-server中，并且实现负载均衡

* 配置服务提供者集群，新建cloud-provider-payment8002工程（参考8001）。只需要**修改配置文件的端口号为8002**即可。

* 修改服务消费者中的OrderController中的方法。**修改访问的url路径，用服务名代替ip、端口号**

  ```java
  @RestController
  @Slf4j
  public class OrderController {
  
  //    public static final String PAYMENT_URL = "http://localhost:8001";
      // 通过在eureka上注册过的微服务名称调用
      public static final String PAYMENT_URL = "http://cloud-payment-service";
  
      @Resource
      private RestTemplate restTemplate;
  
      @GetMapping("/consumer/payment/create")
      public CommonResult<Payment> create(Payment payment) {
          return restTemplate.postForObject(PAYMENT_URL + "/payment/create",payment,CommonResult.class);
      }
  
      @GetMapping("/consumer/payment/get/{id}")
      public CommonResult<Payment> getPayment(@PathVariable("id") Long id) {
          return restTemplate.getForObject(PAYMENT_URL+"/payment/get/"+id,CommonResult.class);
      }
  
  }
  ```

* 在服务消费者配置类中**给RestTemplate这个Bean添加一个`@LoadBalanced`注解，用来实现负载均衡功**

  ```java
    xxxxxxxxxx10 1@Configuration2public class ApplicationContextConfig {34    @Bean5    @LoadBalanced //使用@LoadBalanced注解赋予RestTemplate负载均衡的能力6    public RestTemplate getRestTemplate() {7        return new RestTemplate();8    }910}
  ```
  
* 查看eureka管理页面

  ![image-20220106162109525](https://gitee.com/jobim/blogimage/raw/master/img/20220106162109.png)

* 访问测试，spring会自动帮助我们从eureka-server端，**根据服务名称，获取实例列表，而后完成负载均衡**

  ![image-20210905104637384](https://gitee.com/jobim/blogimage/raw/master/img/20210905104637.png)

## 3.5 微服务信息完善

* **主机名称、服务名称修改**

  ```properties
  # 修改 Eureka Server 页面的 Status 值
  eureka.instance.instance-id=payment8001
  ```
  
  ![image-20220106162945957](https://gitee.com/jobim/blogimage/raw/master/img/20220106162946.png)
  
  ![image-20220106162817235](https://gitee.com/jobim/blogimage/raw/master/img/20220106162817.png)



* **访问信息有IP信息提示**

  ```properties
  # 带 ip + port 的超链接
  eureka.instance.prefer-ip-address=true
  ```

  ![image-20220106163035055](https://gitee.com/jobim/blogimage/raw/master/img/20220106163035.png)

  * 修改之前没有IP提示

    ![image-20210905145534948](https://gitee.com/jobim/blogimage/raw/master/img/20210905145535.png)

  * 修改之后有IP提示

    ![image-20210905145656401](https://gitee.com/jobim/blogimage/raw/master/img/20210905145656.png)



## 3.6 服务发现Discovery

> 可以通过discoveryClient查看Eureka所有注册服务：

1. **修改cloud-provider-payment8001的controller**

   ```java
   @RestController
   @Slf4j
   public class PaymentController {
   
       @Resource
       private DiscoveryClient discoveryClient; //用于服务发现
   
       @GetMapping(value = "/payment/discovery")
       public Object discovery() {
           List<String> services = discoveryClient.getServices();
           for (String element : services) {
               System.out.println(element);
           }
   
           List<ServiceInstance> instances = discoveryClient.getInstances("CLOUD-PAYMENT-SERVICE");
           for (ServiceInstance element : instances) {
               System.out.println(element.getServiceId() + "\t" + element.getHost() + "\t" + element.getPort() + "\t"
                       + element.getUri());
           }
           return this.discoveryClient;
       }
   }
   ```

2. **访问：http://127.0.0.1:8001/payment/discovery**

   ![image-20210905210947168](https://gitee.com/jobim/blogimage/raw/master/img/20210905210947.png)

   控制台打印结果：

   ![image-20220106163652846](https://gitee.com/jobim/blogimage/raw/master/img/20220106163652.png)

​	

## 3.7 Eureka自我保护

**什么是自我保护模式？**

* 默认情况下，服务每隔30秒会向注册中心续约(心跳)一次，如果没有续约，EurekaServer将会注销该实例（默认90秒）——**心跳检测**。但是当网络分区故障发生(延时、卡顿、拥挤)时，微服务与EurekaServer之间无法正常通信，以上行为可能变得非常危险了——因为微服务本身其实是健康的，**此时本不应该注销这个微服务。Eureka通过“自我保护模式”来解决这个问题**——当EurekaServer节点在短时间内丢失过多客户端时（可能发生了网络分区故障），那么这个节点就会进入自我保护模式。

* 保护模式主要用于一组客户端和Eureka Server之间存在网络分区场景下的保护。**在自我保护模式中，Eureka Server会保护服务注册表中的信息，<font color='red'>不再注销任何服务实例</font>。**

* 如果在Eureka Server的首页看到以下这段提示，则说明Eureka进入了保护模式

  ![image-20210905211305169](https://gitee.com/jobim/blogimage/raw/master/img/20210905211305.png)



**为什么会产生Eureka自我保护机制？**

* 是一种针对网络异常波动的安全保护措施，使用自我保护模式能使Eureka集群更加的健壮、稳定的运行，属于CAP里面的AP分支

**自我保护机制的工作机制?**

* **如果在15分钟内超过85%的客户端节点都没有正常的心跳，那么Eureka就认为客户端与注册中心出现了网络故障，Eureka Server自动进入自我保护机制**

<hr/>

**禁止自我保护：**

* 默认情况下自我保护机制是开启的。eureka.server.enable-self-preservation=true

* **关闭自我保护机制，修改检查失效服务的时间**

  ```yaml
  eureka:
    server:
       enable-self-preservation: false
       eviction-interval-timer-in-ms: 3000
  ```
  
  关闭效果：
  
  ![image-20210905214013213](https://gitee.com/jobim/blogimage/raw/master/img/20210905214013.png)



**服务续约过程：**

服务每隔30秒会向注册中心续约(心跳)一次，如果没有续约，租约在90秒后到期，然后服务会被失效。每隔30秒的续约操作我们称之为：`心跳`检测。

* lease-expiration-duration-in-seconds:30，租约续约间隔时间，默认30秒。

- lease-renewal-interval-seconds:90，租约到期时效时间，默认90秒

  ![image-20210905225854340](https://gitee.com/jobim/blogimage/raw/master/img/20210905225854.png)



# 四、Zookeeper服务注册与发现（待补）



# 五、Consul服务注册与发现

* Consul 是一套开源的**分布式服务发现和配置管理系统**，由 HashiCorp 公司用 Go 语言开发。

* 提供了微服务系统中的**服务治理、配置中心、控制总线**等功能。这些功能中的每一个都可以根据需要单独使用，也可以一起使用以构建全方位的服务网格，总之Consul提供了一种完整的服务网格解决方案。
* 它具有很多优点。包括： 基于 raft 协议，比较简洁； 支持健康检查, 同时支持 HTTP 和 DNS 协议 支持跨数据中心的 WAN 集群 提供图形界面 跨平台，支持 Linux、Mac、Windows
* 官网地址：https://www.consul.io/intro/index.html

## 5.1 安装并运行Consul



首先我们从官网下载Consul，地址：https://www.consul.io/downloads.html



* 下载完成后只有一个exe文件，双击运行

  ![image-20210905181629541](https://gitee.com/jobim/blogimage/raw/master/img/20210905181629.png)

* **通过`consul --version`查看版本号**

  ![image-20210905181944528](https://gitee.com/jobim/blogimage/raw/master/img/20210905181944.png)

* **使用开发模式启动`consul agent -dev`**

  ![image-20210905164340892](https://gitee.com/jobim/blogimage/raw/master/img/20210905164340.png)

* **通过以下地址可以访问Consul的首页：[http://localhost:8500](http://localhost:8500/)**

  ![image-20210905181915517](https://gitee.com/jobim/blogimage/raw/master/img/20210905181915.png)



## 5.2 服务提供者、服务消费者注册

**项目架构：**

![image-20210905203627717](https://gitee.com/jobim/blogimage/raw/master/img/20210905203627.png)

**服务提供者注册进Consul：**

* **加入依赖**

  ```xml
  <dependencies>
      <!--SpringCloud consul-server -->
      <dependency>
          <groupId>org.springframework.cloud</groupId>
          <artifactId>spring-cloud-starter-consul-discovery</artifactId>
      </dependency>
      <!-- SpringBoot整合Web组件 -->
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-web</artifactId>
      </dependency>
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-actuator</artifactId>
      </dependency>
      <!--日常通用jar包配置-->
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-devtools</artifactId>
          <scope>runtime</scope>
          <optional>true</optional>
      </dependency>
      <dependency>
          <groupId>org.projectlombok</groupId>
          <artifactId>lombok</artifactId>
          <optional>true</optional>
      </dependency>
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-test</artifactId>
          <scope>test</scope>
      </dependency>
  </dependencies>
  ```

* **application.yml配置**

  ```yml
  # consul服务端口号
  server:
    port: 8006
  
  spring:
    application:
      name: consul-provider-payment
    # consul注册中心地址
    cloud:
      consul:
        host: localhost
        port: 8500
        discovery:
          # hostname: 127.0.0.1
          service-name: ${spring.application.name}
  ```

* **主启动类**

  ```java
  @SpringBootApplication
  @EnableDiscoveryClient
  public class PaymentMain8006 {
      public static void main(String[] args){
          SpringApplication.run(PaymentMain8006.class,args);
      }
  }
  ```

* **Controller代码编写**

  ```java
  @RestController
  public class PaymentController {
  
      @Value("${server.port}")
      private String serverPort;
  
      @GetMapping("/payment/consul")
      public String paymentInfo() {
          return "springcloud with consul: "+serverPort+"\t\t"+ UUID.randomUUID().toString();
      }
  }
  ```

* **测试**

  ![image-20210905165324188](https://gitee.com/jobim/blogimage/raw/master/img/20210905165324.png)输入http://localhost:8006/payment/consul，可以看到服务提供者已经被注册

  ![image-20210905165119171](https://gitee.com/jobim/blogimage/raw/master/img/20210905165119.png)



**服务消费者注册进Consul**

* **加入依赖**

* **application.yml配置**

  ```yml
  # consul服务端口号
  server:
    port: 80
  
  spring:
    application:
      name: cloud-consumer-order
    # consul注册中心地址
    cloud:
      consul:
        host: localhost
        port: 8500
        discovery:
          # hostname: 127.0.0.1
          service-name: ${spring.application.name}
  ```

* **主启动类**

  ```java
  @SpringBootApplication
  @EnableDiscoveryClient //该注解用于向使用consul或者zookeeper作为注册中心时注册服务
  public class OrderConsulMain80 {
      public static void main(String[] args)
      {
          SpringApplication.run(OrderConsulMain80.class,args);
      }
  }
  ```

* **配置RestTemplate实体bean**

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

* **Controller代码编写，通过RestTemplate调用服务提供者**

  ```java
  @RestController
  public class OrderConsulController {
      public static final String INVOKE_URL = "http://consul-provider-payment"; //consul-provider-payment
  
      @Autowired
      private RestTemplate restTemplate;
  
      @GetMapping(value = "/consumer/payment/consul")
      public String paymentInfo()
      {
          String result = restTemplate.getForObject(INVOKE_URL+"/payment/consul", String.class);
          System.out.println("消费者调用支付服务(consule)--->result:" + result);
          return result;
      }
  }
  ```

* **测试**

  ![image-20210905174607265](https://gitee.com/jobim/blogimage/raw/master/img/20210905174607.png)

  访问服务消费者

  ![image-20210905205431794](https://gitee.com/jobim/blogimage/raw/master/img/20210905205431.png)

## 5.3 三个注册中心异同点



![image-20210905204333154](https://gitee.com/jobim/blogimage/raw/master/img/20210905204333.png)

**CAP理论：**

* 在一个分布式系统（指互相连接并共享数据节点的集合）中，当涉及读写操作时，只能保证`一致性（Consistence）`、`可用性（Availability）`、`分区容错性（Partition Tolerance）`三者中的两个，另外一个必须被牺牲。

* CAP原则：
  * **一致性（Consistency）** - 对某个指定的客户端来说，读操作保证能够返回最新的写操作结果。
  * **可用性（Availability）** - 非故障的节点在合理的时间内返回合理的响应（不是错误和超时的响应）。
  * **分区容错性（Partition Tolerance）** - 当出现网络分区后，系统能够继续履行职责

* **AP架构(Eureka)**

  当网络分区出现后，为了保证可用性，系统B可以返回旧值，保证系统的可用性

  ![image-20210905205152479](https://gitee.com/jobim/blogimage/raw/master/img/20210905205152.png)

* **CP架构(Zookeeper/Consul)**

  当网络分区出现后，为了保证一致性，就必须拒接请求，否则无法保证一致性

  ![image-20210905205204029](https://gitee.com/jobim/blogimage/raw/master/img/20210905205204.png)







