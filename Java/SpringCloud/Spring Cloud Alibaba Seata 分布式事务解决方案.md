# Spring Cloud Alibaba Seata 分布式事务解决方案

# 一、分布式事务问题

* **一次业务操作需要跨多个数据源或者需要跨多个系统进行远程调用，就会产生分布式事务问题。**



**举例：**

 **用户购买商品的业务逻辑。整个业务逻辑由3个微服务提供支持：**

 * 仓储服务：对给定的商品扣除仓储数量。 
 * 订单服务：根据采购需求创建订单。
 * 帐户服务：从用户帐户中扣除余额。

单体应用被拆分成微服务应用，原来的三个模块被拆分成三个独立的应用，分别使用三个独立的数据源，业务操作需要调用三个服务来完成。此时每个服务内部的数据一致性由本地事务来保证，但是全局数据一致性问题是无法保证的。

# 二、Seata简介

> Seata官网地址：http://seata.io/zh-cn/

* **Seata是一款开源的分布式事务解决方案，致力于在微服务架构下提供高性能和简单易用的分布式事务服务。Seata 将为用户提供了 AT、TCC、SAGA 和 XA 事务模式，为用户打造一站式的分布式解决方案。**



**Seata术语：**

* **TC (Transaction Coordinator) - 事务协调者**
  维护全局和分支事务的状态，驱动全局事务提交或回滚。

* **TM (Transaction Manager) - 事务管理器**
  定义全局事务的范围：开始全局事务、提交或回滚全局事务。

* **RM (Resource Manager) - 资源管理器**
  管理分支事务处理的资源，与TC交谈以注册分支事务和报告分支事务的状态，并驱动分支事务提交或回滚。

**Seata处理过程：**

1. TM向TC申请开启一个全局事务，全局事务创建成功并生成一个全局唯一的XID；
2. XID在微服务调用链路的上下文传播；
3. RM向TC注册分支事务，将其纳入XID对应全局事务的管辖；
4. TM向TC发起针对XID的全局提交或回滚决议；
5. TC调度XID下管辖的全部分支事务完成提交或者回滚请求

![image-20220119160920877](https://gitee.com/jobim/blogimage/raw/master/img/20220119160920.png)

# 三、Seata的部署

> **Seata的版本选择： [Spring Cloud Alibaba与Spring Boot、Spring Cloud版本对应关系](https://blog.csdn.net/Thinkingcao/article/details/105652632)**
>
> **[官网版本说明](https://github.com/alibaba/spring-cloud-alibaba/wiki/版本说明)**

**本次使用：`Seata 1.2.0`**

## 3.1 Seata Server端配置

> **官方配置文档：[Seata部署指南](http://seata.io/zh-cn/docs/ops/deploy-guide-beginner.html)**

下载seata-server-1.2.0和seata-1.2.0源码

* **seate-server下载： https://seata.io/zh-cn/blog/download.html**
* **seata-1.2.0源码下载： https://github.com/seata/seata/releases**



### 3.1.1 修改配置文件

解压后，进入 **`conf`** 目录开始参数的配置。我们修改 **`file.conf`** 和 **`registry.conf`** 这两个文件。

![image-20220121145621877](https://gitee.com/jobim/blogimage/raw/master/img/20220121145621.png)

**1、进入conf文件夹，修改file.conf文件** 

![image-20220121150258191](https://gitee.com/jobim/blogimage/raw/master/img/20220121150258.png)

**2、修改registry.conf文件**

* **如果config设置成了file，则不需要网上的设置nacos的配置**

![image-20220121151610692](https://gitee.com/jobim/blogimage/raw/master/img/20220121151610.png)





### 3.1.2 MySQL 数据库配置

**我们先创建数据库seata（数据库要与file.conf中db设置那里对应），数据库的建表语句在README文件的server连接中**

![image-20220121152050143](https://gitee.com/jobim/blogimage/raw/master/img/20220121152050.png)

让后执行mysql.sql

```sql
-- -------------------------------- The script used when storeMode is 'db' --------------------------------
-- the table to store GlobalSession data
CREATE TABLE IF NOT EXISTS `global_table`
(
    `xid`                       VARCHAR(128) NOT NULL,
    `transaction_id`            BIGINT,
    `status`                    TINYINT      NOT NULL,
    `application_id`            VARCHAR(32),
    `transaction_service_group` VARCHAR(32),
    `transaction_name`          VARCHAR(128),
    `timeout`                   INT,
    `begin_time`                BIGINT,
    `application_data`          VARCHAR(2000),
    `gmt_create`                DATETIME,
    `gmt_modified`              DATETIME,
    PRIMARY KEY (`xid`),
    KEY `idx_gmt_modified_status` (`gmt_modified`, `status`),
    KEY `idx_transaction_id` (`transaction_id`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8;

-- the table to store BranchSession data
CREATE TABLE IF NOT EXISTS `branch_table`
(
    `branch_id`         BIGINT       NOT NULL,
    `xid`               VARCHAR(128) NOT NULL,
    `transaction_id`    BIGINT,
    `resource_group_id` VARCHAR(32),
    `resource_id`       VARCHAR(256),
    `branch_type`       VARCHAR(8),
    `status`            TINYINT,
    `client_id`         VARCHAR(64),
    `application_data`  VARCHAR(2000),
    `gmt_create`        DATETIME(6),
    `gmt_modified`      DATETIME(6),
    PRIMARY KEY (`branch_id`),
    KEY `idx_xid` (`xid`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8;

-- the table to store lock data
CREATE TABLE IF NOT EXISTS `lock_table`
(
    `row_key`        VARCHAR(128) NOT NULL,
    `xid`            VARCHAR(96),
    `transaction_id` BIGINT,
    `branch_id`      BIGINT       NOT NULL,
    `resource_id`    VARCHAR(256),
    `table_name`     VARCHAR(32),
    `pk`             VARCHAR(36),
    `gmt_create`     DATETIME,
    `gmt_modified`   DATETIME,
    PRIMARY KEY (`row_key`),
    KEY `idx_branch_id` (`branch_id`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8;

```

![image-20220121152613835](https://gitee.com/jobim/blogimage/raw/master/img/20220121152613.png)

### 3.1.3 启动Seata Server端

![image-20220121153557919](https://gitee.com/jobim/blogimage/raw/master/img/20220121153558.png)



## 3.2 Seata Client 客户端配置

### 3.2.1 业务前置准备

**业务场景：用户购买商品的业务逻辑。整个业务逻辑由3个微服务提供支持：**

用户A购买商品，调用 **`A服务`** 创建订单完成，调用 **`B服务`** 扣减库存，然后调用 **`C服务`** 扣减账户余额。每个服务内部的数据一致性由本地事务来保证，多个服务调用来完成业务，全局事务数据一致性则由 Seata 来保证。

- 订单服务A：根据采购需求创建订单。

- 仓储服务B：对给定的商品扣除仓储数量。

- 帐户服务C：从用户帐户中扣除余额。

  

**业务数据库准备：**配置三个业务分别对应各自的数据库。

- A服务 对应数据库：seata_order ；表：t_order
- B服务 对应数据库：seata_storage ；表：t_storage
- C服务 对应数据库：seata_account ；表：t_account



**建表语句：**

```sql
# 创建seata_order数据库
CREATE DATABASE seata_order;

# 创建t_order表
CREATE TABLE seata_order.t_order(
    `id` BIGINT(11) NOT NULL AUTO_INCREMENT PRIMARY KEY,
    `user_id` BIGINT(11) DEFAULT NULL COMMENT '用户id',
    `product_id` BIGINT(11) DEFAULT NULL COMMENT '产品id',
    `count` INT(11) DEFAULT NULL COMMENT '数量',
    `money` DECIMAL(11,0) DEFAULT NULL COMMENT '金额',
    `status` INT(1) DEFAULT NULL COMMENT '订单状态：0：创建中; 1：已完结'
) ENGINE=INNODB AUTO_INCREMENT=7 DEFAULT CHARSET=utf8;
```

```sql
# 创建seata_storage数据库
CREATE DATABASE seata_storage;

# 创建t_storage表
CREATE TABLE seata_storage.t_storage(
    `id` BIGINT(11) NOT NULL AUTO_INCREMENT PRIMARY KEY,
    `product_id` BIGINT(11) DEFAULT NULL COMMENT '产品id',
    `total` INT(11) DEFAULT NULL COMMENT '总库存',
    `used` INT(11) DEFAULT NULL COMMENT '已用库存',
    `residue` INT(11) DEFAULT NULL COMMENT '剩余库存'
) ENGINE=INNODB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8;

# 插入一条数据 
INSERT INTO seata_storage.t_storage(`id`,`product_id`,`total`,`used`,`residue`)
VALUES('1','1','100','0','100');
```

```sql
# 创建seata_account数据库
CREATE DATABASE seata_account;

# 创建t_account表
CREATE TABLE seata_account.t_account(
    `id` BIGINT(11) NOT NULL AUTO_INCREMENT PRIMARY KEY COMMENT 'id',
    `user_id` BIGINT(11) DEFAULT NULL COMMENT '用户id',
    `total` DECIMAL(10,0) DEFAULT NULL COMMENT '总额度',
    `used` DECIMAL(10,0) DEFAULT NULL COMMENT '已用余额',
    `residue` DECIMAL(10,0) DEFAULT '0' COMMENT '剩余可用额度'
) ENGINE=INNODB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8;

# 插入一条数据 
INSERT INTO seata_account.t_account(`id`,`user_id`,`total`,`used`,`residue`) VALUES('1','1','1000','0','1000');
```



### 3.2.2 创建undo_log表

按照上述3库分别建对应的回滚日志表，建表文件在README_ZH文件中的client中

![image-20220121160604988](https://gitee.com/jobim/blogimage/raw/master/img/20220121160605.png)

**undo_log表建表语句如下：**

```sql
-- for AT mode you must to init this sql for you business database. the seata server not need it.
CREATE TABLE IF NOT EXISTS `undo_log`
(
    `id`            BIGINT(20)   NOT NULL AUTO_INCREMENT COMMENT 'increment id',
    `branch_id`     BIGINT(20)   NOT NULL COMMENT 'branch transaction id',
    `xid`           VARCHAR(100) NOT NULL COMMENT 'global transaction id',
    `context`       VARCHAR(128) NOT NULL COMMENT 'undo_log context,such as serialization',
    `rollback_info` LONGBLOB     NOT NULL COMMENT 'rollback info',
    `log_status`    INT(11)      NOT NULL COMMENT '0:normal status,1:defense status',
    `log_created`   DATETIME     NOT NULL COMMENT 'create datetime',
    `log_modified`  DATETIME     NOT NULL COMMENT 'modify datetime',
    PRIMARY KEY (`id`),
    UNIQUE KEY `ux_undo_log` (`xid`, `branch_id`)
) ENGINE = InnoDB
  AUTO_INCREMENT = 1
  DEFAULT CHARSET = utf8 COMMENT ='AT transaction mode undo table';
```

**完成图示：**

![image-20220121160809982](https://gitee.com/jobim/blogimage/raw/master/img/20220121160810.png)



> **以下配置以`seata-order-service2001`为例**
>
> * 项目结构：
>
>   ![image-20220121161527271](https://gitee.com/jobim/blogimage/raw/master/img/20220121161527.png)

### 3.2.3 Seata Client配置文件

配置文件的编写参考README_ZH文件中的client中：

![image-20220121161822797](https://gitee.com/jobim/blogimage/raw/master/img/20220121161822.png)

**1、file.conf，修改这一处**

![image-20220121162353321](https://gitee.com/jobim/blogimage/raw/master/img/20220121162353.png)

```
transport {
  # tcp, unix-domain-socket
  type = "TCP"
  #NIO, NATIVE
  server = "NIO"
  #enable heartbeat
  heartbeat = true
  # the tm client batch send request enable
  enableTmClientBatchSendRequest = false
  # the rm client batch send request enable
  enableRmClientBatchSendRequest = true
   # the rm client rpc request timeout
  rpcRmRequestTimeout = 2000
  # the tm client rpc request timeout
  rpcTmRequestTimeout = 10000
  # the tc client rpc request timeout
  rpcTcRequestTimeout = 5000
  #thread factory for netty
  threadFactory {
    bossThreadPrefix = "NettyBoss"
    workerThreadPrefix = "NettyServerNIOWorker"
    serverExecutorThread-prefix = "NettyServerBizHandler"
    shareBossWorker = false
    clientSelectorThreadPrefix = "NettyClientSelector"
    clientSelectorThreadSize = 1
    clientWorkerThreadPrefix = "NettyClientWorkerThread"
    # netty boss thread size
    bossThreadSize = 1
    #auto default pin or 8
    workerThreadSize = "default"
  }
  shutdown {
    # when destroy server, wait seconds
    wait = 3
  }
  serialization = "seata"
  compressor = "none"
}
service {
  #transaction service group mapping
  vgroupMapping.my_test_tx_group = "default"
  #only support when registry.type=file, please don't set multiple addresses
  default.grouplist = "127.0.0.1:8091"
  #degrade, current not support
  enableDegrade = false
  #disable seata
  disableGlobalTransaction = false
}

client {
  rm {
    asyncCommitBufferLimit = 10000
    lock {
      retryInterval = 10
      retryTimes = 30
      retryPolicyBranchRollbackOnConflict = true
    }
    reportRetryCount = 5
    tableMetaCheckEnable = false
    tableMetaCheckerInterval = 60000
    reportSuccessEnable = false
    sagaBranchRegisterEnable = false
    sagaJsonParser = "fastjson"
    sagaRetryPersistModeUpdate = false
    sagaCompensatePersistModeUpdate = false
    tccActionInterceptorOrder = -2147482648 #Ordered.HIGHEST_PRECEDENCE + 1000
  }
  tm {
    commitRetryCount = 5
    rollbackRetryCount = 5
    defaultGlobalTransactionTimeout = 60000
    degradeCheck = false
    degradeCheckPeriod = 2000
    degradeCheckAllowTimes = 10
    interceptorOrder = -2147482648 #Ordered.HIGHEST_PRECEDENCE + 1000
  }
  undo {
    dataValidation = true
    onlyCareUpdateColumns = true
    logSerialization = "jackson"
    logTable = "undo_log"
    compress {
      enable = true
      # allow zip, gzip, deflater, 7z, lz4, bzip2, zstd default is zip
      type = zip
      # if rollback info size > threshold, then will be compress
      # allow k m g t
      threshold = 64k
    }
  }
  loadBalance {
      type = "RandomLoadBalance"
      virtualNodes = 10
  }
}
log {
  exceptionRate = 100
}
tcc {
  fence {
    # tcc fence log table name
    logTableName = tcc_fence_log
    # tcc fence log clean period
    cleanPeriod = 1h
  }
}
```

**2、registry.conf，修改这一处**

![image-20220121162506377](https://gitee.com/jobim/blogimage/raw/master/img/20220121162506.png)

```json
registry {
  # file 、nacos 、eureka、redis、zk、consul、etcd3、sofa、custom
  type = "nacos"

  nacos {
    application = "seata-server"
    serverAddr = "127.0.0.1:8848"
    namespace = ""
    username = ""
    password = ""
    ##if use MSE Nacos with auth, mutex with username/password attribute
    #accessKey = ""
    #secretKey = ""
  }
  eureka {
    serviceUrl = "http://localhost:8761/eureka"
    weight = "1"
  }
  redis {
    serverAddr = "localhost:6379"
    db = "0"
    password = ""
    timeout = "0"
  }
  zk {
    serverAddr = "127.0.0.1:2181"
    sessionTimeout = 6000
    connectTimeout = 2000
    username = ""
    password = ""
  }
  consul {
    serverAddr = "127.0.0.1:8500"
    aclToken = ""
  }
  etcd3 {
    serverAddr = "http://localhost:2379"
  }
  sofa {
    serverAddr = "127.0.0.1:9603"
    region = "DEFAULT_ZONE"
    datacenter = "DefaultDataCenter"
    group = "SEATA_GROUP"
    addressWaitTime = "3000"
  }
  file {
    name = "file.conf"
  }
  custom {
    name = ""
  }
}

config {
  # file、nacos 、apollo、zk、consul、etcd3、springCloudConfig、custom
  type = "file"

  nacos {
    serverAddr = "127.0.0.1:8848"
    namespace = ""
    group = "SEATA_GROUP"
    username = ""
    password = ""
    ##if use MSE Nacos with auth, mutex with username/password attribute
    #accessKey = ""
    #secretKey = ""
    dataId = "seata.properties"
  }
  consul {
    serverAddr = "127.0.0.1:8500"
	key = "seata.properties"
    aclToken = ""
  }
  apollo {
    appId = "seata-server"
    apolloMeta = "http://192.168.1.204:8801"
    namespace = "application"
    apolloAccesskeySecret = ""
  }
  zk {
    serverAddr = "127.0.0.1:2181"
    sessionTimeout = 6000
    connectTimeout = 2000
    username = ""
    password = ""
    nodePath = "/seata/seata.properties"
  }
  etcd3 {
    serverAddr = "http://localhost:2379"
    key = "seata.properties"
  }
  file {
    name = "file.conf"
  }
  custom {
    name = ""
  }
}
```

### 3.2.4 pom.xml文件

```xml
<dependencies>
    <!--nacos-->
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    </dependency>
    <!--seata-->
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
        <exclusions>
            <exclusion>
                <artifactId>seata-all</artifactId>
                <groupId>io.seata</groupId>
            </exclusion>
        </exclusions>
    </dependency>
    <dependency>
        <groupId>io.seata</groupId>
        <artifactId>seata-all</artifactId>
        <version>1.2.0</version>
    </dependency>
    <!--feign-->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-openfeign</artifactId>
    </dependency>
    <!--web-actuator-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    <!--mysql-druid-->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>5.1.37</version>
    </dependency>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid-spring-boot-starter</artifactId>
        <version>1.1.10</version>
    </dependency>
    <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>2.0.0</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>
```





### 3.2.5 application.yml文件

```yaml
server:
  port: 2001

spring:
  application:
    name: seata-order-service
  cloud:
    alibaba:
      seata:
        # 自定义事务组名称需要与seata-server中的对应
        tx-service-group: my_test_tx_group #因为seata的file.conf文件中没有service模块，事务组名默认为my_test_tx_group
        #service要与tx-service-group对齐，vgroupMapping和grouplist在service的下一级，my_test_tx_group在再下一级
        service:
          vgroupMapping:
            #要和tx-service-group的值一致
            my_test_tx_group: default
          grouplist:
            # seata seaver的 地址配置，此处可以集群配置是个数组
            default: 127.0.0.1:8091
    nacos:
      discovery:
        server-addr: localhost:8848
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://localhost:3306/seata_order?useUnicode=true&characterEncoding=UTF-8&useSSL=false&serverTimezone=GMT%2B8
    username: root
    password: root

feign:
  hystrix:
    enabled: false

logging:
  level:
    io:
      seata:

mybatis:
  mapperLocations: classpath:mapper/*.xml
```



### 3.2.6 业务代码编写：加@GlobalTransactional

> dao层、mapper层、controller层代码省略

业务代码使用OpenFeign 的方式进行服务间的调用，来实现一个简单的业务功能。AT模式只用一个 **`@GlobalTransactional`** 注解即可实现分布式事务。**`name 属性为事务唯一性表示，可以随意定义。rollbackFor 属性为指定Exception异常才进行事务回滚。`**

OrderServiceImpl.java

```java
@Service
@Slf4j
public class OrderServiceImpl implements OrderService {
    @Resource
    private OrderDao orderDao;

    @Resource
    private StorageService storageService;

    @Resource
    private AccountService accountService;

    /**
     * 创建订单->调用库存服务扣减库存->调用账户服务扣减账户余额->修改订单状态
     * 简单说：
     * 下订单->减库存->减余额->改状态
     */
    @Override
    @GlobalTransactional(name = "fsp-create-order",rollbackFor = Exception.class)
    public void create(Order order) {
        log.info("------->下单开始");
        //本应用创建订单
        orderDao.create(order);

        //远程调用库存服务扣减库存
        log.info("------->order-service中扣减库存开始");
        storageService.decrease(order.getProductId(),order.getCount());
        log.info("------->order-service中扣减库存结束");

        //远程调用账户服务扣减余额
        log.info("------->order-service中扣减余额开始");
        accountService.decrease(order.getUserId(),order.getMoney());
        log.info("------->order-service中扣减余额结束");

        //修改订单状态为已完成
        log.info("------->order-service中修改订单状态开始");
        orderDao.update(order.getUserId(),0);
        log.info("------->order-service中修改订单状态结束");

        log.info("------->下单结束");
    }
}
```



### 3.2.7 config代码编写



**1、MybatisConfig.java**

```java
@Configuration
@MapperScan({"com.zb.springcloud.dao"})
public class MyBatisConfig {
}
```

**2、DataSourceProxyConfig.java**

```java
package com.zb.springcloud.config;

import com.alibaba.druid.pool.DruidDataSource;
import io.seata.rm.datasource.DataSourceProxy;
import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.transaction.SpringManagedTransactionFactory;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;

import javax.sql.DataSource;

@Configuration
public class DataSourceProxyConfig {

    @Value("${mybatis.mapperLocations}")
    private String mapperLocations;

    @Bean
    @ConfigurationProperties(prefix = "spring.datasource")
    public DataSource druidDataSource(){
        return new DruidDataSource();
    }

    @Bean
    public DataSourceProxy dataSourceProxy(DataSource dataSource) {
        return new DataSourceProxy(dataSource);
    }

    @Bean
    public SqlSessionFactory sqlSessionFactoryBean(DataSourceProxy dataSourceProxy) throws Exception {
        SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
        sqlSessionFactoryBean.setDataSource(dataSourceProxy);
        sqlSessionFactoryBean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources(mapperLocations));
        sqlSessionFactoryBean.setTransactionFactory(new SpringManagedTransactionFactory());
        return sqlSessionFactoryBean.getObject();
    }

}
```



### 3.2.8 主启动类

```java
@EnableDiscoveryClient
@EnableFeignClients
@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)//取消数据源的自动创建
public class SeataOrderMainApp2001 {

    public static void main(String[] args)
    {
        SpringApplication.run(SeataOrderMainApp2001.class, args);
    }
}
```



## 3.3 测试

* 分别启三个服务，可以看到在项目启动过程中，Seata Sever 服务端会有相对应提示

  ![image-20220119174242644](https://gitee.com/jobim/blogimage/raw/master/img/20220119174242.png)



发送请求 http://localhost:2001/order/create?userId=1&productId=1&count=10&money=100 来模拟下单请求。发送多次请求，可以看到下单都成功了，Seata Server 端也有事务在进行处理的过程。（此时数据库中，因为事务处理完成，并没有任何数据）







>  参考博客：
>
>  * [Spring Cloud Alibaba Seata 分布式事务解决方案简介](https://blog.csdn.net/lzb348110175/article/details/107699201)
>
>  * [SpringCloud 整合Seata 解决分布式事务](https://thinkingcao.blog.csdn.net/article/details/109320221)

