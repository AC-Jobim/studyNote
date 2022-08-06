由于 Spring Cloud Netflix 项目进入维护模式（将模块置于维护模式意味着 Spring Cloud 团队将不会再向模块中添加新功能，只会修复 block 级别的 bug 以及安全问题），阿里巴巴团队为我们提供了一套新的微服务开发一站式解决方案

详见官方介绍：https://github.com/alibaba/spring-cloud-alibaba/blob/master/README-zh.md



# Spring Cloud Alibaba Nacos服务注册和配置中心

# 一、Nacos 概述

<font color='blue' size='4'>**什么是Nacos？**</font>

* Nacos（Dynamic Naming and Configuration Service）：**`一个更易于构建云原生应用的动态服务发现，配置管理和服务管理中心`**。我们可以理解为：**Nacos = 服务注册中心 + 配置中心；等价于 Nacos = Eureka + Spring Cloud Config + Spring Cloud Bus**

* Nacos 可以替代 Eureka 来实现 `服务注册中心`、可以替代 Spring Cloud Config 来实现`服务配置中心`、可以替代 Spring Cloud Bus 来实现 `配置的全局广播`。Nacos 是更强调云原生时代支持 “服务治理、服务沉淀、共享、持续发展” 理念的注册中心和配置中心。(附：[Nacos 官网](https://nacos.io/zh-cn/index.html))

**各注册中心比较：**

![image-20220121185031380](https://gitee.com/jobim/blogimage/raw/master/img/20220121185031.png)



<font color='blue' size='4'>**Nacos的安装运行：**</font>

* 下载地址：https://github.com/alibaba/nacos/releases

* 下载解压后，打开 bin 目录，打双击 startup.cmd 均可启动

  ![image-20220121184334396](https://gitee.com/jobim/blogimage/raw/master/img/20220121184334.png)

* 浏览器地址栏输入 `localhost:8848/nacos` 登录，默认用户名密码都是 nacos，主页面如下：

  ![image-20220121184606972](https://gitee.com/jobim/blogimage/raw/master/img/20220121184607.png)



# 二、Nacos作为注册中心

项目结构：

![image-20220121203719294](https://gitee.com/jobim/blogimage/raw/master/img/20220121203719.png)

## 2.1 基于Nacos的服务提供者

**1、建moudle作为服务端`cloudalibaba-provider-payment9001`**

**2、引入依赖**

* 父pom引入spring-cloud-alibaba 依赖

  ```xml
  <!--spring cloud alibaba 2.1.0.RELEASE-->
  <dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-alibaba-dependencies</artifactId>
    <version>2.1.0.RELEASE</version>
    <type>pom</type>
    <scope>import</scope>
  </dependency>
  ```

* 当前模块pom引入 nacos-discovery 依赖

  ```xml
  <!--SpringCloud ailibaba nacos -->
  <dependency>
      <groupId>com.alibaba.cloud</groupId>
      <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
  </dependency>
  ```

**3、applicaiton.yml 文件配置**

```yaml
server:
  port: 9001

spring:
  application:
    name: nacos-payment-provider
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 #配置Nacos地址
# 做监控需要把这个全部暴露出来
management:
  endpoints:
    web:
      exposure:
        include: '*'
```

**4、主启动类，需要使用`@EnableDiscoveryClient`注解**

```java
@EnableDiscoveryClient
@SpringBootApplication
public class PaymentMain9001 {
    public static void main(String[] args) {
            SpringApplication.run(PaymentMain9001.class, args);
    }
}
```

**5、业务类**

```java
@RestController
public class PaymentController {

    @Value("${server.port}")
    private String serverPort;

    @GetMapping(value = "/payment/nacos/{id}")
    public String getPayment(@PathVariable("id") Integer id) {
        return "nacos registry, serverPort: "+ serverPort+"\t id"+id;
    }

}
```

**6、测试**

* 这里不像eureka还要写注册中心微服务，直接安装打开nacos即可。

* 启动服务模块，进入 Nacos 控制台，在 **`服务管理 → 服务列表`** 中可以看到，我们定义的服务名 `nacos-payment-provider` 已经成功注册到 Nacos 注册中心。

  ![image-20220121192710004](https://gitee.com/jobim/blogimage/raw/master/img/20220121192710.png)

* 访问http://localhost:9001/payment/nacos/1

  ![image-20220121192726100](https://gitee.com/jobim/blogimage/raw/master/img/20220121192726.png)

**7、相同配置再新建一个module 9002**

再来个 `cloudalibaba-provider-payment9002`模块作为服务端，将其与 9001 形成集群，以集群的方式对外提供服务，并将其也注册到 Nacos 中。

![image-20220121192802414](https://gitee.com/jobim/blogimage/raw/master/img/20220121192802.png)



## 2.2 基于Nacos的服务消费者

**Nacos 默认支持负载均衡：**

* 使用 Eureka 时，服务调用使用的是 **`基于Ribbon 的 RestTemplate + @LoadBalance`** 或 **`Feign`** 的方式进行。**`Feign 整合 Ribbon 默认支持负载均衡`**。 **`我们的 Nacos 也整合了 Ribbon 默认支持负载均衡。`**

  ![image-20220121193011924](https://gitee.com/jobim/blogimage/raw/master/img/20220121193011.png)



**1、新建module作为服务消费者`cloudalibaba-consumer-nacos-order83`**

**2、引入nacos-discovery依赖**

```xml
<!--SpringCloud ailibaba nacos -->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

**3、applicaiton.yml 文件配置**

```yaml
server:
  port: 83

spring:
  application:
    name: nacos-order-consumer
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848

# 消费者将要去访问的微服务名称(注册成功进nacos的微服务提供者)
service-url:
  nacos-user-service: http://nacos-payment-provider
```

**4、主启动类，需要使用`@EnableDiscoveryClient`注解**

```java
@EnableDiscoveryClient
@SpringBootApplication
public class OrderNacosMain83 {
    public static void main(String[] args) {
        SpringApplication.run(OrderNacosMain83.class,args);
    }
} 
```

**5、业务类**

```java
@Configuration
public class ApplicationContextBean {

    @Bean
    @LoadBalanced
    public RestTemplate getRestTemplate() {
        return new RestTemplate();
    }

}
```

```java
@RestController
public class OrderNacosController {

    @Resource
    private RestTemplate restTemplate;

    @Value("${service-url.nacos-user-service}")
    private String serverURL;

    @GetMapping("/consumer/payment/nacos/{id}")
    public String paymentInfo(@PathVariable("id") Long id) {
        return restTemplate.getForObject(serverURL+"/payment/nacos/"+id,String.class);
    }

}
```

**6、测试**

* 启动9001、9002、83

  ![image-20220121194241090](https://gitee.com/jobim/blogimage/raw/master/img/20220121194241.png)

* 测试链接：http://localhost:83/consumer/payment/nacos/1，发现9001与9002交替出现，即轮询负载均衡

  ![image-20220121194438492](https://gitee.com/jobim/blogimage/raw/master/img/20220121194438.png)



## 2.3 Nacos与其他注册中心对比

![image-20220121194502219](https://gitee.com/jobim/blogimage/raw/master/img/20220121194502.png)



<font color='blue' size='4'>**Nacos可以在CP与AP之间切换：**</font>

* `C` **一致性** `A` **高可用** `P` **容错性**。参考：[CAP原则](https://baike.baidu.com/item/CAP原则/5712863?fr=aladdin)，主流选用的都是 AP 模式，保证系统的高可用。

**何时选择使用何种模式？**

* **AP:**

  一般来说，**如果不需要存储服务级别的信息且服务实例是通过nacos-client注册，并能够保持心跳上报，那么就可以选择AP模式**。当前主流的服务如 Spring cloud 和 Dubbo 服务，都适用于AP模式，AP模式为了服务的可能性而减弱了一致性，因此AP模式下只支持注册临时实例。

* **CP:**

  **如果需要在服务级别编辑或者存储配置信息，那么 CP 是必须**，K8S服务和DNS服务则适用于CP模式。CP模式下则支持注册持久化实例，此时则是以 Raft 协议为集群运行模式，该模式下注册实例之前必须先注册服务，如果服务不存在，则会返回错误。

Nacos 集群默认支持的是CAP原则中的 **`AP原则`**，但是 **`也可切换为CP原则`**，**切换命令如下：**

```shell
curl -X PUT '$NACOS_SERVER:8848/nacos/v1/ns/operator/switches?entry=serverMode&value=CP'
```

# 三、Nacos作为配置中心

Nacos 可以替代 Config + Bus 来用作 **`服务配置中心`**。附：[Nacos服务配置中心官方文档](https://spring-cloud-alibaba-group.github.io/github-pages/hoxton/en-us/index.html#_spring_cloud_alibaba_nacos_config)。Nacos 用作服务配置中心，分为 **`基础配置(简单使用)`** 和 **`分类配置(多环境使用 dev、test、prod等环境)`**。接下来就介绍 Nacos 用作服务配置中心。

## 3.1 Nacos基础配置

**1、新建模块 `cloudalibaba-config-nacos-client3377` 用作配置中心，并将其注册到 Nacos 中**

**2、pom 引入 nacos-config 依赖**

```xml
<!--引入nacos-config配置-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
<!--注册到 Nacos，需引入nacos-discovery配置-->
<!--引入nacos-discovery配置-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

**3、进行yml配置**

此处需要配置 **bootstrap.yml** 和 **application.yml** 两个文件。

原因：Nacos同springcloud-config一样，在项目初始化时，要保证先从配置中心进行配置拉取，拉取配置之后，才能保证项目的正常启动。springboot中配置文件的加载是存在优先级顺序的，**bootstrap优先级高于application**。**`bootstrap.yml 用作系统级资源配置项，application.yml 用作用户级的资源配置项。在项目中两者配合共同生效，bootstrap.yml 优先级更高`**

* **bootstrap.yml**

  ```yaml
  # nacos配置
  server:
    port: 3377
  
  spring:
    application:
      name: nacos-config-client
    cloud:
      nacos:
        discovery:
          server-addr: localhost:8848 #Nacos服务注册中心地址
        config:
          server-addr: localhost:8848 #Nacos作为配置中心地址
          file-extension: yaml #指定yaml格式的配置
  
  # ${spring.application.name}-${spring.profile.active}.${spring.cloud.nacos.config.file-extension}
  # nacos-config-client-dev.yaml
  ```

* **application.yml**

  ```yaml
  spring:
    profiles:
      active: dev # 表示开发环境
  ```

  

**4、主启动类，添加`@EnableDiscoveryClient`注解**

```java
@EnableDiscoveryClient
@SpringBootApplication
public class NacosConfigClientMain3377 {
    public static void main(String[] args) {
            SpringApplication.run(NacosConfigClientMain3377.class, args);
    }
}
```

**5、业务类添加 @RefreshScope 实现配置自动更新**

```java
@RestController
@RefreshScope //在控制器类加入@RefreshScope注解使当前类下的配置支持Nacos的动态刷新功能。
public class ConfigClientController {
    @Value("${config.info}")
    private String configInfo;

    @GetMapping("/config/info")
    public String getConfigInfo() {
        return configInfo;
    }
}
```



**6、Nacos中添加配置项**

进入 **`Nacos → 配置管理 → 配置列表 → + 号`** 添加配置项。Data ID 按规则编写。

![image-20220122181744075](https://gitee.com/jobim/blogimage/raw/master/img/20220122181744.png)

<font color='blue'>**Data ID 命名规则：**</font>

>官网：https://nacos.io/zh-cn/docs/quick-start-spring-cloud.html

`dataId` 的完整格式如下：

```yaml
${prefix}-${spring.profile.active}.${file-extension}
```

- `prefix` 默认为 `spring.application.name` 的值，也可以通过配置项 `spring.cloud.nacos.config.prefix`来配置。
- `spring.profile.active` 即为当前环境对应的 profile。注意：当 `spring.profile.active` 为空时，**对应的连接符 - 也将不存在**，dataId 的拼接格式变成 `${prefix}.${file-extension}`**（建议：不要让 `spring.profile.active` 为空，或许会有一些意外的问题）**
- `file-exetension` 为配置内容的数据格式，可以通过配置项 `spring.cloud.nacos.config.file-extension` 来配置。目前只支持 `properties` 和 `yaml` 类型。

![image-20220122182810583](https://gitee.com/jobim/blogimage/raw/master/img/20220122182810.png)

**七、测试**

通过 http://localhost:3377/config/info 测试，发现可以正确获取 Nacos 配置中心的配置信息。

![image-20220122183505669](https://gitee.com/jobim/blogimage/raw/master/img/20220122183505.png)

**自带动态刷新：修改nacos中的yaml配置文件，再次调用查看配置，发现配置刷新了**

![image-20220122183609612](https://gitee.com/jobim/blogimage/raw/master/img/20220122183609.png)

## 3.2 Nacos分类配置

项目开发中，一定会遇到 **`多环境`**、**`多项目管理`** 问题。遇到下面问题时，Nacos 基础配置显然无法解决这些问题，接下来就对 **`Nacos 命名空间`** 及 **`Group`** 相关概念的了解。

### 3.2.1 Nacos的命名规则说明

Nacos 命名由 **`Namespace(命名空间)`** + **`Group(分组)`** + **`Data ID(实例ID)`** 三部分组成。最外层 Namespace 用于区分部署环境；Group 和 Data ID 逻辑上用于区分两个目标对象。
![image-20220122215016660](https://gitee.com/jobim/blogimage/raw/master/img/20220122215016.png)
默认情况下：**Namespace = public**，**Group = DEFAULT_GROUP**，**Cluster=DEFAULT**

* **`Namespace`** 主要用来实现隔离，Nacos 默认的命名空间是 `public`。比方说我们现在有三个环境：`开发`、`测试`、`生产`环境，我们就可以创建三个 Namespace，不同的 Namespace 之间是隔离的；

* **`Group`** 是一组配置集，是组织配置的维度之一。默认是 `DEFAULT_GROUP`。通过一个有意义的 **`名称`** 对配置集进行分组，从而区分 Data ID 相同的配置集。配置分组的常见场景：不同的应用或组件使用了相同的配置类型，就可以把不同的微服务划分到同一个分组里面去，从而解决问题2；如 database_url 配置和 MQ_topic 配置。

* **`Service 微服务`**；一个 Service 可以包含多个 Cluster(集群)，Nacos 默认 Cluster 是 DEFAULT，Cluster 是对指定微服务的一个虚拟划分。比方说为了容灾，将 Service 微服务分别部署在了杭州机房和广州机房，这是就可以给杭州机房的 Service 微服务起一个集群名称(HZ)，给广州机房的 Service 微服务起一个集群名称(GZ)，还可以尽量让同一个机房的微服务互相调用，以提升性能。

* **`Instance`**，就是一个个微服务实例。



### 3.2.2 DataID方案

* 指定spring.profile.active和配置文件的DataID来使不同环境下读取不同的配置

  ![image-20220122220850861](https://gitee.com/jobim/blogimage/raw/master/img/20220122220850.png)

  这里命名空间是默认的public，Group也是默认的。

* 通过spring.profile.active属性就能进行多环境下配置文件的读取

  ![image-20220122221207131](https://gitee.com/jobim/blogimage/raw/master/img/20220122221207.png)

* 重启3377 测试http://localhost:3377/config/info，成功读取到test配置下的config.info

![img](https://cdn.nlark.com/yuque/0/2021/png/22423156/1635687508172-571f6d42-9516-4ca1-a1a3-8d75bea9b912.png)

### 3.2.3 Group方案

* 默认Group是DEFAULT_GROUP，现在通过Group实现环境分区

**1、新建一个配置文件，添加到DEV_GROUP分组**

![image-20220122225828026](https://gitee.com/jobim/blogimage/raw/master/img/20220122225828.png)

**2、新建一个配置文件，添加到TEST_GROUP分组**

![image-20220122230409156](https://gitee.com/jobim/blogimage/raw/master/img/20220122230409.png)

![image-20220122230423789](https://gitee.com/jobim/blogimage/raw/master/img/20220122230423.png)

**3、在config下增加一条group的配置即可。可配置为DEV_GROUP或TEST_GROUP![image-20220122230445981](https://gitee.com/jobim/blogimage/raw/master/img/20220122230446.png)**



**4、测试：**

![image-20220122230505746](https://gitee.com/jobim/blogimage/raw/master/img/20220122230505.png)

### 3.2.4 namesapce方案

**1、新建dev/test的Namesapce**

![image-20220124141130569](https://gitee.com/jobim/blogimage/raw/master/img/20220124141130.png)

* 服务列表中可以看到不同的Namespace

  ![image-20220124141209859](https://gitee.com/jobim/blogimage/raw/master/img/20220124141209.png)

**2、在这两个新建的namespace中分别新建三个不同分组的配置文件**

![image-20220124141305432](https://gitee.com/jobim/blogimage/raw/master/img/20220124141305.png)

**3、修改3377的yml文件**

* **bootstrap：**

  ![image-20220124141343855](https://gitee.com/jobim/blogimage/raw/master/img/20220124141343.png)

* **application：**

  ![image-20220124141354509](https://gitee.com/jobim/blogimage/raw/master/img/20220124141354.png)

**4、测试**

* **dev namesapce：**

  ![image-20220124141424072](https://gitee.com/jobim/blogimage/raw/master/img/20220124141424.png)

* **test namesapce：**

  ![image-20220124141445200](https://gitee.com/jobim/blogimage/raw/master/img/20220124141445.png)



# 四、Nacos 集群搭建和持久化配置（待补）

参考博客：[Nacos 集群搭建和持久化配置(Linux)](https://blog.csdn.net/lzb348110175/article/details/107489625)





