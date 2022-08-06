#### Spring

bean的生命周期、循环依赖问题、spring cloud（如项目中有用过）、AOP的实现、spring事务传播

#### 常见问题

- java动态代理和cglib动态代理的区别（经常结合spring一起问所以就放这里了）
- spring中bean的生命周期是怎样的？
- 属性注入和构造器注入哪种会有循环依赖的问题？

12.你用过mybatis吗？说一下你mybatis运作的流程吧?如何绑定到xml？（这一块简直胡言乱语）

 

13.说一下#{}和${}的区别？（说了很多，sql注入都说了，但是第一次面试太紧张，说反了。。。）

9.说说spring,spring mvc springboot,springcloud的区别

spring 类加载方式、实例保存在哪、aop ioc、反射机制

Spring：有没有用过 Spring，Spring IOC、AOP 机制与实现，Spring MVC

9.问道spring，问我都用到了哪些里面的东西，我说ioc，本来问我ioc的实现，突然，问我控制反转了什么，最后说完这个，就没问实现了，直接问我还有啥问他。



•MyBatis 包含一个非常强大的查询缓存特性,它可以非常方便地配置和定制。缓存可以极大的提升查询效率。

•

•MyBatis系统中默认定义了两级缓存。

•**一级****缓存**和**二级缓存**。

–1、默认情况下，只有一级缓存（SqlSession级别的缓存，也称为本地缓存）开启。

–2、二级缓存需要手动开启和配置，他是基于namespace级别的缓存。

–3、为了提高扩展性。MyBatis定义了缓存接口Cache。我们可以通过实现Cache接口来自定义二级缓存



#### 1.1 缓存介绍





## 什么是缓存

缓存就是内存中的一个对象，用于对数据库查询结果的保存，用于减少与数据库的交互次数从而降低数据库的压力，进而提高响应速度。

## 什么是MyBatis中的缓存

**MyBatis 中的缓存就是说 MyBatis 在执行一次SQL查询或者SQL更新之后，这条SQL语句并不会消失，而是被MyBatis 缓存起来，当再次执行相同SQL语句的时候，就会直接从缓存中进行提取，而不是再次执行SQL命令。**

MyBatis中的缓存分为一级缓存和二级缓存，一级缓存又被称为 SqlSession 级别的缓存，二级缓存又被称为表级缓存。

> SqlSession是什么？SqlSession 是SqlSessionFactory会话工厂创建出来的一个会话的对象，这个SqlSession对象用于执行具体的SQL语句并返回给用户请求的结果。
>
> SqlSession级别的缓存是什么意思？SqlSession级别的缓存表示的就是每当执行一条SQL语句后，默认就会把该SQL语句缓存起来，也被称为会话缓存







```
/**
 * 一级缓存失效的几种情况；(一级缓存是SqlSession级别的缓存)
 * 1、不同的sqlSession，使用不同的一级缓存；
 *        只有在同一个sqlSession期间查询到的数据会保存在这个sqlSession的缓存中；
 *        下次使用这个sqlSession查询会从缓存中拿
 * 2、同一个方法，不同的参数，由于可能之前没查询过，所有还会发新的sql；
 * 3、在这个sqlSession期间执行上任何一次增删改操作，增删改操作会把缓存清空；
 * 4、手动清空了缓存；
 *
 * 每次查询，先看一级缓存中有没有，如果没有就去发送新的sql；每个sqlSession拥有自己的一级缓存
 */
```





　虚拟机安装完成之后，我们启动Linux系统，将第二步下载的 redis-4.0.9.tar.gz 文件通过工具复制到 /opt 目录下，然后在通过如下命令进行解压：

```
1 tar -zxf redis-4.0.9.tar.gz
```

　　解压之后如下图所示：

　　![img](https://images2018.cnblogs.com/blog/1120165/201805/1120165-20180522220340824-311785863.png)

　　由于在安装过程中需要对源码进行编译，而编译依赖 gcc 环境。如下图所示，则是没有进行 gcc 的安装：

　　![img](https://images2018.cnblogs.com/blog/1120165/201805/1120165-20180522220544196-1242289705.png)

　　下面，我们通过如下命令进行 gcc 的安装（yum 方式需要联网）：

```
1 yum install gcc-c++
```

　　安装完成之后，在输入 gcc -v 命令，则不会出现上面的提示信息了。

[回到顶部](https://www.cnblogs.com/ysocean/p/9074353.html#_labelTop)

### 4、编译安装

　　进入到第二步解压的Redis文件目录，然后输入 make 命令进行编译：

```
1 cd /opt/redis-4.0.9
2 make
```

　　![img](https://images2018.cnblogs.com/blog/1120165/201805/1120165-20180522221252985-1154819980.png)

　　编译完成之后，还是在该目录下输入 make install 进行构建：

　　该命令会生成 Redis的5个二进制文件，默认是在 /usr/local/bin 路径下，但是我们可以手动指定生成的文件位置，将 make install 变成：

```
1 make PREFIX=/usr/local/redis install 
```

　　![img](https://images2018.cnblogs.com/blog/1120165/201805/1120165-20180522221722393-1839471658.png)

　　完成之后，就会在 /usr/local/redis/bin 目录下生成如下几个二进制文件：

　　![img](https://images2018.cnblogs.com/blog/1120165/201805/1120165-20180522221821479-1540414236.png)





ps:大家不懂这些配置意思没关系，后面会在具体实例中进行介绍，先过个眼熟即可。

[回到顶部](https://www.cnblogs.com/ysocean/p/9074787.html#_labelTop)

### 1、开头说明





 
