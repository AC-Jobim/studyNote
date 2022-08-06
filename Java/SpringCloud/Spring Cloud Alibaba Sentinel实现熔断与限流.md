# Spring Cloud Alibaba Sentinel实现熔断与限流



# 一、Sentinel介绍与安装

<font color='blue' size='4'>**Sentinel是什么？**</font>

* **Sentinel是阿里开源的项目，提供了流量控制、熔断降级、系统负载保护等多个维度来保障服务之间的稳定性。**

> 中文文档：[介绍 · alibaba/Sentinel Wiki](https://github.com/alibaba/Sentinel/wiki/介绍#sentinel-是什么)



**Sentinel 的主要特性：**

![image-20220124145151264](https://gitee.com/jobim/blogimage/raw/master/img/20220124145151.png)



<font color='blue' size='4'>**Sentinel控制台的安装：**</font>

> **下载地址：[Sentinel的下载地址](https://github.com/alibaba/Sentinel/releases)**

* 下载 **`sentinel-dashboard-1.7.0.jar`** ，环境：JDK 8，端口：8080 不被占用。进入 cmd 控制台，使用 **`java -jar sentinel-dashboard-1.7.0.jar`** 方式直接运行。

  ![image-20220124150112653](https://gitee.com/jobim/blogimage/raw/master/img/20220124150112.png)

* **访问 localhost:8080，账号密码均为sentinel**

  ![image-20220124150318182](https://gitee.com/jobim/blogimage/raw/master/img/20220124150318.png)



# 二、微服务项目整合Sentinel

**1、启动Nacos服务**

**2、新建Module cloudalibaba-sentinel-service8401**

**3、添加pom依赖**

```xml
<!--引入 sentinel 依赖-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>

<!--nacos服务注册依赖-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>

<!--sentinel持久化需要的依赖(后续持久化会用到,此处可有可无)-->
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-datasource-nacos</artifactId>
</dependency>
```

**4、application.yml 配置**

```yaml
server:
  port: 8401

spring:
  application:
    name: cloudalibaba-sentinel-service
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 #Nacos服务注册中心地址
    # 添加sentinel相关配置
    sentinel:
      transport:
        dashboard: localhost:8080 #配置sentinel dashboard地址
        port: 8719 #sentinel默认8719，假如被占用了会自动从8719开始依次+1扫描。直至找到未被占用的端口

#暴露，用于监控等
management:
  endpoints:
    web:
      exposure:
        include: '*'
```

**5、主启动类添加 @EnableDiscoveryClient 注解**

```java
@EnableDiscoveryClient
@SpringBootApplication
public class SentinelMain8401 {
    public static void main(String[] args) {
        SpringApplication.run(SentinelMain8401.class, args);
    }
}
```

**6、业务类**

```java
@RestController
public class FlowLimitController {

    @GetMapping("/testA")
    public String testA() {
        return "-----testA";
    }

    @GetMapping("/testB")
    public String testB(){
        return "-----testB";
    }
}
```

**7、启动微服务8401，查看Sentinel监控信息**

* **查看Sentinel控制台，发现什么也没有，这是因为Sentinel 采用的懒加载机制，只有执行一次方法调用，才能被Sentinel监控到。**

  ![image-20220124150318182](https://gitee.com/jobim/blogimage/raw/master/img/20220124150318.png)

* **多次调用 /testA 接口，在实时监控便能够看到接口 `调用时间`、`QPS`、`响应时间` 等内容**

  ![image-20220124153641883](https://gitee.com/jobim/blogimage/raw/master/img/20220124153642.png)

# 三、流控规则

* **流控规则，即：流量控制规则。具体配置有 `资源名`、`针对来源`、`阈值类型`、`是否集群`、`流控模式`、`单机阈值`、`流控效果` 这几项 。**

![image-20220124182334173](https://gitee.com/jobim/blogimage/raw/master/img/20220124182334.png)

![image-20220124160353180](https://gitee.com/jobim/blogimage/raw/master/img/20220124160353.png)



## 3.1 阈值类型：QPS

* **QPS（每秒钟的请求数量）**：当调用该 API 的 **`QPS`** 达到阈值的时候，进行限流。

* 下面设置表示1秒钟内查询一次就是OK，若QPS>1，就直接-快速失败，报默认错误

  ![image-20220124183204107](C:\Users\ZB\AppData\Roaming\Typora\typora-user-images\image-20220124183204107.png)

  ![image-20220126134416152](https://gitee.com/jobim/blogimage/raw/master/img/20220126134416.png)

* 测试：**当/testA的访问超过1次/s是，就会进行流量控制：`快速失败`（流控 Sentinel 默认提示：`Blocked by Sentinel(flow limiting)`）**

  ![image-20220126134458504](https://gitee.com/jobim/blogimage/raw/master/img/20220126134458.png)

## 3.2 阈值类型：线程数

* **线程数**：当调用该 API 的 **`线程数`** 达到阈值的时候，进行限流。

  ![image-20220126135245568](https://gitee.com/jobim/blogimage/raw/master/img/20220126135245.png)



* **先修改一下8401的业务代码：**

  ![image-20220126135135042](https://gitee.com/jobim/blogimage/raw/master/img/20220126135135.png)

* 测试：然后重启8401，测试/testA，最好用两个浏览器访问，效果更明显

  ![image-20220126140004180](https://gitee.com/jobim/blogimage/raw/master/img/20220126140004.png)

## 3.3 流控模式：直接

* **已经介绍，就是直接失败**

## 3.4 流控模式：关联

* **关联**：当关联的资源达到阈值时，就限流自己。**`当与 A 资源关联的 B 资源达到阈值时，就限流自己(A)`**，即：B惹事，A挂了

* **应用场景：**双十一，`支付接口`和`下单接口`关联。当支付接口达到阈值，就限流下单接口

  ![image-20220126141329207](https://gitee.com/jobim/blogimage/raw/master/img/20220126141329.png)

* **测试：开始，/testA 和 /testB 1次/s 可以正常调用，突然大批量访问打到 /testB 请求。由于关联的 /testB 请求超过设定的阈值 QPS = 1，导致 /testA 请求被限流了。**

  ![image-20220126141721653](https://gitee.com/jobim/blogimage/raw/master/img/20220126141721.png)

## 3.5 流控模式：链路

**链路**：当链路中的资源达到阈值时，就会对使用到该资源的链路进行流控。**`当 A01 资源达到设定阈值时，所有调用该服务的链路，都会被限流`**，即：**`A01 挂了，用到我的链路都得挂`**

此处会用到 **`@SentinelResource`** 注解 value 属性值 **`作为资源名`**。

**模拟两条请求链路：**

1. **A链路：** **`A → A01 → A04 → A05`**
2. **B链路：** **`B → A01 → A02 → A03`**
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200728093358122.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x6YjM0ODExMDE3NQ==,size_16,color_FFFFFF,t_70)

**配置：**

![image-20220126145053505](https://gitee.com/jobim/blogimage/raw/master/img/20220126145053.png)

**配置说明：对 message 服务进行 链路 流控，该服务关联有 A 和 B 两条链路。当 A 链路1s 调用 1次，服务正常。当该链路调用 超出阈值 QPS = 1 后，此时A链路都会被限流，同时因为B链路也调用 message，所以B链路也会同时被限流调用**

**业务代码：**

```java
@Service
public class FlowLimitService {

    @SentinelResource("message")
    public String message() {
        return "success";
    }
}
```

```java
@RestController
@Slf4j
public class FlowLimitController {

    @Autowired
    FlowLimitService flowLimitService;
    
    //   链路测试
    @GetMapping("/linkTestA")
    public String linkTestA() {
        return flowLimitService.message();
    }

    @GetMapping("/linkTestB")
    public String linkTestB() {
        return flowLimitService.message();
    }

}
```

**测试：**开始，对 **`/linkTestA请求`** 1次/s 可以正常调用，当 **`/linkTestA请求`** QPS > 1 后，满足设定的 **`message链路流控`** 规则 ，所以 **`/linkTestA请求`** 会被限流。同时 **`/linkTestB请求`** 也会被限流。 

![image-20220126145803869](https://gitee.com/jobim/blogimage/raw/master/img/20220126145803.png)



## 3.6 流控效果：快速失败

**快速失败**是默认的流控效果，直接失败。

![image-20220126150917624](https://gitee.com/jobim/blogimage/raw/master/img/20220126150917.png)

源码：com.alibaba.csp.sentinel.slots.block.flow.controller.DefaultController

## 3.7 流控效果：Warm Up

* **Warm Up**：某个服务，日常访问量很少，基本为 0，突然1s访问量 10w，这种极端情况，会直接将服务击垮。所以通过配置 **`流控效果：Warm Up`**，允许系统慢慢呼呼的进行预热，经预热时长逐渐升至设定的QPS阈值。

  **即：默认coldFactor为3，请求QPS从（threshold / 3）开始，经多少预热时长才逐渐升至设定的QPS阈值。**

* **配置：**
  ![image-20220126151347913](https://gitee.com/jobim/blogimage/raw/master/img/20220126151347.png)
  **配置说明：**`/testA` 服务，设置 QPS 单机阈值为 10，采用 Warm Up 预热的方式，预热时长为 5s。根据计算公式 **`10 / 3 = 3`**，前 5s 的阈值为 3，预热 5s 后阈值增长到 10。

* **测试：**开始调用 /testA 服务，狂点刷新访问打到 /testA 请求。配置 `Warm Up` 流控效果，**在前 5s 内，通过公式计算阈值为 `10/3 = 3`，访问超过 3 次便会被限流；5s 后，阈值增长到 10，此时访问超过 3 次也不会被限流**，**这就是 Warm Up 预热效果。**

  ![image-20220126151753693](https://gitee.com/jobim/blogimage/raw/master/img/20220126151753.png)

## 3.8 流控效果：排队等待

* **排队等待**：让请求以均匀的速度通过，对应的是漏桶算法。这种方式主要用于处理间隔性突发的流量，例如消息队列。在某一秒有大亮的请求到来，而接下来的几秒则处于空闲状态。我们希望系统能够在接下来的空闲期间逐渐处理这些请求，而不是在第一秒直接拒绝多余的请求。

* **配置： `/testA` 服务，设置 QPS 单机阈值为 1，每秒只接收 1 个请求。设置超时时间 2s。采用漏斗算法，让后台匀速的处理请求，而不是直接拒绝更多的请求。超时的请求则被抛弃，返回错误信息。**

  ![image-20220126154009810](https://gitee.com/jobim/blogimage/raw/master/img/20220126154009.png)

* 修改一下业务代码，把线程名打印出来以验证是否排队

  ![image-20220126154029104](https://gitee.com/jobim/blogimage/raw/master/img/20220126154029.png)

* 测试：使用postman发送10个请求

  ![image-20220126154457144](https://gitee.com/jobim/blogimage/raw/master/img/20220126154457.png)

  

  ![image-20220126154512439](https://gitee.com/jobim/blogimage/raw/master/img/20220126154512.png)

  可以看到刚好满足1s一个请求，说明请求的执行进行了排队  

# 四、降级规则

## 4.1 慢调用比例

* **慢调用比例 (SLOW_REQUEST_RATIO)：选择以慢调用比例作为阈值，需要设置允许的慢调用 RT（即最大的响应时间），请求的响应时间大于该值则统计为慢调用。当单位统计时长（statIntervalMs）内请求数目大于设置的最小请求数目，并且慢调用的比例大于阈值，则接下来的熔断时长内请求会自动被熔断。**经过熔断时长后熔断器会进入探测恢复状态（HALF-OPEN 状态），若接下来的一个请求响应时间小于设置的慢调用 RT 则结束熔断，若大于设置的慢调用 RT 则会再次被熔断。

* **执行流程：**

  ![image-20220203115038000](https://gitee.com/jobim/blogimage/raw/master/img/20220203115038.png)

* **举例：**

  ![image-20220203114848877](https://gitee.com/jobim/blogimage/raw/master/img/20220203114856.png)



## 4.2 异常比例

- **异常比例 (ERROR_RATIO)：当单位统计时长（statIntervalMs）内请求数目大于设置的最小请求数目，并且异常的比例大于阈值，则接下来的熔断时长内请求会自动被熔断。**经过熔断时长后熔断器会进入探测恢复状态（HALF-OPEN 状态），若接下来的一个请求成功完成（没有错误）则结束熔断，否则会再次被熔断。异常比率的阈值范围是 [0.0, 1.0]，代表 0% - 100%。

* **举例：**

  ![image-20220203115702354](https://gitee.com/jobim/blogimage/raw/master/img/20220203115702.png)



## 4.3 异常数

- **异常数 (ERROR_COUNT)：当单位统计时长内的异常数目超过阈值之后会自动进行熔断。**经过熔断时长后熔断器会进入探测恢复状态（HALF-OPEN 状态），若接下来的一个请求成功完成（没有错误）则结束熔断，否则会再次被熔断。

* **举例：**

  ![image-20220203115756910](https://gitee.com/jobim/blogimage/raw/master/img/20220203115756.png)



# 五、热点key限流

## 5.1 基本介绍

(官网地址：[Github 热点规则官方介绍](https://github.com/alibaba/Sentinel/wiki/热点参数限流))

何为热点？热点即经常访问的数据。很多时候我们希望统计某个热点数据中访问频次最高的 Top K 数据，并对其访问进行限制。比如：

* 商品 ID 为参数，统计一段时间内最常购买的商品 ID 并进行限制
* 用户 ID 为参数，针对一段时间内频繁访问的用户 ID 进行限制

**热点参数限流会统计传入参数中的热点参数，并根据配置的限流阈值与模式，对包含热点参数的资源调用进行限流。 热点参数限流可以看做是一种特殊的流量控制，仅对包含热点参数的资源调用生效。**

![image-20220203122753778](https://gitee.com/jobim/blogimage/raw/master/img/20220203122753.png)

## 5.2 基本使用

<font color='blue' size='4'>**编写测试方法**</font>

这里需要使用**@SentinelResource**注解，与 @HystrixCommand 类似，也是用来定义服务降级 **`兜底方法`** 的注解。

```java
@GetMapping("/testHotKey")
@SentinelResource(value = "testHotKey", blockHandler = "deal_testHotKey")   //这里的名称可以随便写，但是一般跟rest地址一样
public String testHotKey(@RequestParam(value = "p1", required = false) String p1,
                         @RequestParam(value = "p1", required = false) String p2) {
    return "------testHotkey";
}

//这里是我们自定义的兜底方法，BlockException不要打成了BlockedException
public String deal_testHotKey(String p1, String p2, BlockException e) {
    return "这次不用默认的兜底提示Blocked by Sentinel(flow limiting)，自定义提示：del_testHotKey o(╥﹏╥)o...";
}
```

**分析：**

- 其中 **value = "testHotKey"** 是一个标识（**Sentinel资源名**），与rest的/testHotKey对应，这里value的值可以任意写，但是我们约定与rest地址一致，唯一区别是没有/。
- blockHandler = "del_testHotKey" 则表示如果违背了Sentinel中配置的流控规则，就会**调用我们自己的兜底方法del_testHotKey**

<font color='blue' size='4'>**配置热点key限流规则**</font>

设定热点限流规则：方法testHotKey里面第一个参数只要QPS超过每秒1次，马上降级处理，产生限流并执行自定义的del_testHotKey兜底方法。

![image-20220203140625942](https://gitee.com/jobim/blogimage/raw/master/img/20220203140626.png)



<font color='blue' size='4'>**测试**</font>

* 访问http://localhost:8401/testHotKey?p1=a&p2=b，仅传入参数p2没有任何影响，1次/s正常显示，迅速点击两次，触发热点限流，执行自定义兜底方法：

  ![image-20220203151201884](https://gitee.com/jobim/blogimage/raw/master/img/20220203151201.png

* **不配置blockeHandler（兜底方法）**

  ![image-20220203151238352](https://gitee.com/jobim/blogimage/raw/master/img/20220203151238.png)

  触发热点限流降级会出现error page，对用户不友好。

  ![image-20220203150839666](https://gitee.com/jobim/blogimage/raw/master/img/20220203150839.png)



## 5.3 参数例外项

上述案例演示了第一个参数p1，当QPS超过1秒1次点击后马上被限流

**设置参数例外项**：我们期望p1参数当它是某个特殊值时，它的限流值和平时不一样，比如当p1的值等于5时，它的阈值可以达到200。

![image-20220203151325020](C:\Users\ZB\AppData\Roaming\Typora\typora-user-images\image-20220203151325020.png)

测试：狂点http://localhost:8401/testHotKey?p1=5&p2=b 没有限流

![image-20220203152016976](https://gitee.com/jobim/blogimage/raw/master/img/20220203152017.png)

## 5.4 其他

手动添加一个异常：

![image-20220203152902501](https://gitee.com/jobim/blogimage/raw/master/img/20220203152902.png)

测试直接错误页面。

![image-20220203152928842](https://gitee.com/jobim/blogimage/raw/master/img/20220203152928.png)

注意：Sentinel它只管你有没有触发它的限流规则，也可以说只管这个web交互页面（控制台）里面的东西。 配置类的东西Sentinel可以管，java异常的错误我不管。

* **@SentinelResource**处理的是Sentinel控制台配置的违规情况，由blockHandler方法配置的兜底处理；

* int age = 10/0,这个是java运行时报出的运行时异常RunTimeException，@SentinelResource不管



# 六、@SentinelResource注解详解

**`Sentinel 提供了 @SentinelResource 注解用于定义资源，并提供了AspectJ的扩展用于自动定义资源、处理BlockException等。`**

## 6.1 @SentinelResource 属性介绍

| 属性名                                      | 是否必填 | 说明                                                         |
| ------------------------------------------- | -------- | ------------------------------------------------------------ |
| <font color='red'>value</font>              | 是       | 资源名称 。（必填项，需要通过 `value` 值找到对应的规则进行配置） |
| entryType                                   | 否       | entry类型，标记流量的方向，取值IN/OUT，默认是OUT             |
| <font color='red'>blockHandler</font>       | 否       | **处理BlockException的函数名称(可以理解为对Sentinel的配置进行方法兜底)**。<br/>函数要求： <br/>1.必须是 public 修饰 <br/>2.返回类型与原方法一致 <br/>3. 参数类型需要和原方法相匹配，并在最后加 BlockException 类型的参数。<br/>4. 默认需和原方法在同一个类中。若希望使用其他类的函数，可配置 blockHandlerClass ，并指定blockHandlerClass里面的方法。 |
| <font color='red'>blockHandlerClass</font>  | 否       | **存放blockHandler的类**。 <br/>对应的处理函数必须 public static 修饰，否则无法解析，其他要求：同blockHandler。 |
| <font color='red'>fallback</font>           | 否       | **用于在抛出异常的时候提供fallback处理逻辑(可以理解为对Java异常情况方法兜底)**。<br/> fallback函数可以针对所有类型的异常（除了 exceptionsToIgnore 里面排除掉的异常类型）进行处理。函数要求： <br/>1.返回类型与原方法一致 。<br/>2.参数类型需要和原方法相匹配，Sentinel 1.6开始，也可在方法最后加 Throwable 类型的参数。<br/>3.默认需和原方法在同一个类中。若希望使用其他类的函数，可配置 fallbackClass ，并指定fallbackClass里面的方法。 |
| <font color='red'>fallbackClass</font>      | 否       | **存放fallback的类**。 <br/>对应的处理函数必须static修饰，否则无法解析，其他要求：同fallback。 |
| defaultFallback                             | 否       | **用于通用的 fallback 逻辑**。 <br/>默认 fallback 函数可以针对所有类型的异常（除了 exceptionsToIgnore 里面排除掉的异常类型）进行处理。若同时配置了 fallback 和 defaultFallback，以fallback为准。函数要求：<br/>1.返回类型与原方法一致 <br/>2.方法参数列表为空，或者有一个 Throwable 类型的参数。<br/>3.默认需要和原方法在同一个类中。若希望使用其他类的函数，可配置 fallbackClass ，并指定 fallbackClass 里面的方法。 |
| <font color='red'>exceptionsToIgnore</font> | 否       | **指定排除掉哪些异常。** <br/>排除的异常不会计入异常统计，也不会进入fallback逻辑，而是原样抛出。 |
| exceptionsToTrace                           | 否       | 需要trace的异常                                              |

## 6.2 fallback 指定Java异常兜底方法

**`fallback只用来处理与Java逻辑异常相关的兜底`**。比如：NullPointerException、ArrayIndexOutOfBoundsException 等Java代码中的异常，fallback 指定的兜底方法便会生效。

<font color='blue' size='4'>**兜底方法与业务方法耦合**</font>

```java
@GetMapping("/testA")
@SentinelResource(value = "testA", fallback = "fallbackMethod")
public String testA() {
    int i = 10 / 0;
    return "-----testA";
}

public String fallbackMethod(Throwable e) {
    return "限流请求连接(Java类异常)的兜底方法：" + e.getMessage();
}
```

<font color='blue' size='4'>**使用 fallbackClass 将兜底方法与业务解耦合**</font>

```java
/**
 * 业务逻辑
 */
@GetMapping("/testA")
@SentinelResource(value = "testA", fallback = "fallbackMethod", fallbackClass = CustomerFallback.class)
public String testA() {
    int i = 10 / 0;
    return "-----testA";
}

/**
 * 单独一个类，存放兜底方法
 */
public class CustomerFallback {

    public static String fallbackMethod(Throwable e) {
        return "限流请求连接(Java类异常)的兜底方法：" + e.getMessage();
    }
    
}
```

![image-20220204111320068](https://gitee.com/jobim/blogimage/raw/master/img/20220204111320.png)

**测试兜底结果返回：**

![image-20220204111424301](https://gitee.com/jobim/blogimage/raw/master/img/20220204111424.png)

## 6.3 blockHandler 指定 Sentinel 配置兜底方法

**`blockHandler 只用来处理 与 Sentinel 配置有关的兜底`**。比如：配置某资源 QPS =1，当 QPS >1 时，blockHandler 指定的兜底方法便会生效。

<font color='blue' size='4'>**兜底方法与业务方法耦合**</font>

```java
/**
 * 业务逻辑
 */
@GetMapping("/testB")
@SentinelResource(value = "testB",blockHandler = "exceptionMethod")
public String testB() {
    return "-----testB";
}

public String exceptionMethod(BlockException exception) {
    return "限流@SentinelResource value 属性的兜底方法:" + exception;
}
```

<font color='blue' size='4'>**使用 fallbackClass 将兜底方法与业务解耦合**</font>

```java
@GetMapping("/testB")
@SentinelResource(value = "testB",blockHandler = "exceptionMethod",blockHandlerClass = CustomerBlockHandler.class)
public String testB() {
    return "-----testB";
}

/**
 * 单独一个类，存放兜底方法
 */
public class CustomerBlockHandler {

    public static String exceptionMethod(BlockException exception) {
        return "处理与 Sentinel 配置相关的兜底方法:" + exception;
    }
}

```

**测试：兜底结果返回**

![image-20220204111952797](https://gitee.com/jobim/blogimage/raw/master/img/20220204111952.png)

## 6.4 exceptionsToIgnore 用于指定异常不走兜底方法

使用 **`exceptionsTolgnore`** 属性，来 **`指定某些异常不执行兜底方法，直接显示错误信息`**。配置 ArithmeticException 异常不走兜底方法。**`java.lang.ArithmeticException: / by zero`** ，便不会再执行兜底方法，直接显示错误信息给前台页面。

```java
@GetMapping("/testA")
@SentinelResource(value = "testA", 
					fallback = "fallbackMethod",
					fallbackClass = CustomerFallback.class, 
					exceptionsToIgnore = ArithmeticException.class)
public String testA() {
    int i = 10 / 0;
    return "-----testA";
}
123456789
```

测试：不走兜底方法，直接返回异常到页面

![image-20220204112117014](https://gitee.com/jobim/blogimage/raw/master/img/20220204112117.png)

## 6.5 defaultFallback 用于指定通用的 fallback 兜底方法

使用 **`defaultFallback`** 来指定通用的 fallback 兜底方法。

1. 如果当前业务配置有 **`defaultFallback`** 和 **`fallback`** 两个属性，则优先执行 **`fallback`** 指定的方法。
2. 如果 **`fallback`** 指定的方法不存在，还会执行 **`defaultFallback`** 指定的方法。

```java
/**
 * 业务逻辑
 */
@GetMapping("/testA")
@SentinelResource(value = "testA",
        fallback = "fallbackMethod",
        fallbackClass = CustomerFallback.class,
        defaultFallback = "defaultFallbackMethod" //直接指定即可，使用比较简单
)
public String testA() {
    int i = 10 / 0;
    return "-----testA";
}

/**
 * 单独一个类，存放兜底方法
 */
public class CustomerFallback {

    public static String defaultFallbackMethod(Throwable e) {
        return "通用的fallback兜底方法";
    }

    public static String fallbackMethod(Throwable e) {
        return "限流请求连接(Java类异常)的兜底方法：" + e.getMessage();
    }
}
```



