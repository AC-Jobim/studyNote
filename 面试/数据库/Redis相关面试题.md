

# 一、Redis的应用场景

**1、缓存**

* 由于redis访问速度块、支持的数据类型比较丰富，所以redis很适合用来存储热点数据。同时Redis提供了键过期功能，也提供了灵活的键淘汰策略。

**2、分布式锁**

* 在分布式场景下，无法使用单机环境下的锁来对多个节点上的进程进行同步。可以使用 Redis 自带的`setnx`命令实现分布式锁，除此之外，还可以使用官方提供的 `RedLock` 分布式锁实现。

**3、自动过期**

* Redis针对数据都可以**设置过期时间**，过期的数据清理无需使用方去关注，所以开发效率也比较高，当然，性能也比较高。最常见的就是：短信验证码、具有时间性的商品展示等。无需像数据库还要去查时间进行对比。

**4、排行榜**

* 很多网站都有排行榜应用的，如淘宝的月度销量榜单、商品按时间的上新排行榜等。Redis提供的zset能实现各种复杂的排行榜应用。 

> **`zset`作为抖音热搜和根据商品销量进行排序**
>
> <img src="https://gitee.com/jobim/blogimage/raw/master/img/20210910093038.png" alt="image-20210910093038682" style="zoom:80%;" />
>
> ![image-20210910093252301](https://gitee.com/jobim/blogimage/raw/master/img/20210910093252.png)

**5、计数器**

* redis由于incrby命令可以实现原子性的递增，所以可以运用于高并发的秒杀活动、具体业务还体现在比如限制一个手机号发多少条短信、一个接口一分钟限制多少请求、一个接口一天限制调用多少次、电商网站商品的浏览量、视频网站视频的播放数、朋友圈点赞等。为了保证数据实时效，每次浏览都得给+1，并发量高时如果每次都请求数据库操作无疑是种挑战和压力。 

> **set用作朋友圈点赞**
>
> ![image-20210910093454074](https://gitee.com/jobim/blogimage/raw/master/img/20210910093454.png)



> **string用作数值增减：**
>
> * 递增数字 INCR key（可以不用预先设置key的数值。如果预先设置key但值不是数字，则会报错)
> * 增加指定的整数 INCRBY key increment
> * 递减数值 DECR key
> * 减少指定的整数 DECRBY key decrement



**6、消息队列(发布/订阅功能)**

* List 是一个双向链表，可以通过 lpush 和 rpop 写入和读取消息。不过最好使用 Kafka、RabbitMQ 等消息中间件。



**<font color='blue'>String的应用场景：</font>**

* **缓存功能：**字符串最经典的使用场景，redis最为缓存层，Mysql作为储存层，绝大部分请求数据都是 redis中获取，由于redis具有支撑高并发特性，所以缓存通常能起到加速读写和降低后端压力的作用。

* **计数器：**许多应用都会使用redis作为计数的基础工具，因为redis的`INCR`命令具有原子性的自增操作，在并发下也可以保证一个线程安全的问题。如果我们常见的论坛，网站的点赞数或者视频播放数就是使用redis作为计数的基础组件。

* **共享session：**出于负载均衡的考虑，分布式服务会将用户信息的访问均衡到不同服务器上，这样可能我们用户在第一次访问和第二次访问的时候不是同一台服务器的话，session不同步，就会导致重新登录。为避免这个问题可以使用redis将用户session集中管理。

  ![image-20210926094147261](https://gitee.com/jobim/blogimage/raw/master/img/20210926094147.png)



* **限流：**我们常见的限流算法有很多，如令牌桶，漏桶，计数器，滑动窗口等。而用String的话，就可以使用计数器，我们如果要设置一个一分钟最多只能访问100次的限流接口，只要设置键的一分钟过期时间就行，然后在一分钟之内通过计数器来进行计数。

**<font color='blue'>set的应用场景：</font>**

* **set可以用来好友等相互关系的存储（通过`并操作`公共认识的人）**

  ![image-20210910094019908](https://gitee.com/jobim/blogimage/raw/master/img/20210910094020.png)

* **set还可以用作抽奖，可以使用`spop/srandmember`命令来获取随机数**

  ![image-20210910125552405](https://gitee.com/jobim/blogimage/raw/master/img/20210910125552.png)

* **set用作朋友圈点赞**

  ![image-20210910093454074](https://gitee.com/jobim/blogimage/raw/master/img/20210910093454.png)

**8、hash应用场景**

* **hash可以用作对象缓存**，string类型和hash类型都可以用作对象缓存，对于**对象中单字段修改比较多**得可以用hash类型。

* **hash也可以用来实现小型的购物车**。

  ![image-20210910123142692](https://gitee.com/jobim/blogimage/raw/master/img/20210910123143.png)



<font color='blue'>**list应用场景：**</font>

* **list可用于微信文章订阅公众号**

  ![image-20210910125113965](https://gitee.com/jobim/blogimage/raw/master/img/20210910125114.png)


* **消息队列：**redis的`lpush+brpop`命令组合即可实现阻塞队列，生产者客户端是用`lupsh`从列表左侧插入元素，多个消费者客户端使用`brpop`命令阻塞时的“抢”列表尾部的元素，多个客户端保证了消费的负载均衡和高可用性。

  ![image-20210926093825029](https://gitee.com/jobim/blogimage/raw/master/img/20210926093825.png)

* **最新列表：**list类型的`lpush`命令和`lrange`命令能实现最新列表的功能，每次通过`lpush`命令往列表里插入新的元素，然后通过`lrange`命令读取最新的元素列表，如朋友圈的点赞列表、评论列表。



<font color='blue'>**Zset的应用场景：**</font>

* **实时排行榜：**比如我们要做一个一小时热搜，我们可以把当前的时间戳作为zset的key，把帖子ID作为member，点击数评论数作为score，当score发生变化时更新score。然后可以利用`zrevrange`或`zrange`来来查到对应数量的在时间内的记录。

* **延时队列：**zset会按照score进行排序，如果score代表想要执行时间的时间戳。在某个时间将它插入zset集合中，它便会按照时间戳大小进行排序，也就是对执行时间前后进行排序。

* **限流：**滑动窗口是限流常见的一种策略。如果我们把一个用户的 ID 作为 key 来定义一个 zset ，member 或者 score 都为访问时的时间戳。我们只需统计某个 key 下在指定时间戳区间内的个数，就能得到这个用户滑动窗口内访问频次，与最大通过次数比较，来决定是否允许通过。



# 二、Redis分布式锁



**通过`SETNX`命令来实现分布式锁：**

* Redis为**单线程模式**，可以使用`SETNX`命令实现分布式锁。当且仅当 key 不存在，将 key 的值设为 value。 若给定的 key 已经存在，则 SETNX 不做任何动作（成功返回1，失败返回0）。
* 为了防止获取锁后的程序出现异常、宕机等，导致其他线程调用`SETNX`命令总是返回0而进入死锁状态，需要通过`expire`指令为该key设置一个过期时间。Redis中提供了`set key value px milliseconds nx`命令保证了**加锁和设置时间两个命令为原子的**。
* 加锁的时候要**设置全局唯一的id**，用来标识这把锁是属于哪个请求加的，在解锁的时候保证只能由该请求释放锁。同时锁的释放要放在finally中。
* 释放锁时要验证 value 值，防止误解锁。同时为了保证**验证value和释放锁是原子的**，此时需要使用**Lua脚本**保证。（也通过redis中事务保证释放锁的原子性）

**为什么要保证验证value和释放锁是原子的？**

* 在多线程情况下，你的线程A执行了unlock，也判断完，这个时候恰恰在你执行到redisDao.delete(key)，还未执行的时候，失去了CPU执行权。这个时间 + 你执行业务代码的时间大于5秒，key过期。也就是说，这个时候线程A拿到的锁已经被释放掉。   
* 这个时候线程B拿到CPU执行权，并且执行了lock的逻辑，并且成功，然后恰巧，线程B在这个时候失去了CPU执行权，线程A又获得了执行权，并且成功的执行了redisDao.delete(key)，(线程A和线程B的key相同)， 这个时候线程A就把线程B获取的锁删除了，这个时候就会出现坏情况啦。

```java
@GetMapping("testLockLua")
public void testLockLua() {
    //1 声明一个uuid ,将做为一个value 放入我们的key所对应的值中
    String uuid = UUID.randomUUID().toString();
    //2 定义一个锁：lua 脚本可以使用同一把锁，来实现删除！
    String skuId = "25"; // 访问skuId 为25号的商品 100008348542
    String locKey = "lock:" + skuId; // 锁住的是每个商品的数据

    // 3 获取锁
    Boolean lock = redisTemplate.opsForValue().setIfAbsent(locKey, uuid, 3, TimeUnit.SECONDS);

    // 第一种： lock 与过期时间中间不写任何的代码。
    // redisTemplate.expire("lock",10, TimeUnit.SECONDS);//设置过期时间
    // 如果true
    if (lock) {
        // 执行的业务逻辑开始
        // 获取缓存中的num 数据
        Object value = redisTemplate.opsForValue().get("num");
        // 如果是空直接返回
        if (StringUtils.isEmpty(value)) {
            return;
        }
        // 不是空 如果说在这出现了异常！ 那么delete 就删除失败！ 也就是说锁永远存在！
        int num = Integer.parseInt(value + "");
        // 使num 每次+1 放入缓存
        redisTemplate.opsForValue().set("num", String.valueOf(++num));
        /*使用lua脚本来释放锁*/
        // 定义lua 脚本
        //String script = "if redis.call('get', KEYS[1]) == ARGV[1] then return redis.call('del', KEYS[1]) else return 0 end";
		String script = "if redis.call('get', KEYS[1]) == ARGV[1] "
    			+ "then "
    			+ "    return redis.call('del', KEYS[1]) "
    			+ "else "
    			+ "    return 0 "
    			+ "end";
    
		// 使用redis执行lua执行
        DefaultRedisScript<Long> redisScript = new DefaultRedisScript<>();
        redisScript.setScriptText(script);
        // 设置一下返回值类型 为Long
        // 因为删除判断的时候，返回的0,给其封装为数据类型。如果不封装那么默认返回String 类型，
        // 那么返回字符串与0 会有发生错误。
        redisScript.setResultType(Long.class);
        // 第一个要是script 脚本 ，第二个需要判断的key，第三个就是key所对应的值。
        redisTemplate.execute(redisScript, Arrays.asList(locKey), uuid);
    } else {
        // 其他线程等待
        try {
            // 睡眠
            Thread.sleep(1000);
            // 睡醒了之后，调用方法。
            testLockLua();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```



**上面分布式锁问题：**

1. **主节点宕机，锁安全得不到保障**。在Redis的master节点上拿到了锁；但是这个加锁的key还没有同步到slave节点；master故障，发生故障转移，slave节点升级为master节点；导致锁丢失。
2. **不能解决超时问题**。如果代码执行时间过长，超过了10秒， 那么分布式就自动释放了。
3. **不支持可重入锁**。



对于**主节点宕机，锁安全得不到保障的问题**，Redis作者antirez基于分布式环境下提出了一种更高级的分布式锁的实现方式：`Redlock`。

然而使用Redlock也不一定保险，此时需要引入`redisson`。

* **Redisson 分布式重入锁用法**，redisson这个框架重度依赖了Lua脚本和Netty。内部通过Watch Dog 机制来支持锁自动续期

  ![image-20210910174916803](https://gitee.com/jobim/blogimage/raw/master/img/20210910174916.png)





# 三、Redis内存淘汰策略



## 3.1 Redis过期键的删除策略

一般推荐Redis设置内存为最大物理内存的四分之三

* **定时过期**

  每个设置过期时间的key都需要创建一个定时器，到过期时间就会立即清除。该策略可以立即清除过期的数据，对内存很友好；但是会占用大量的CPU资源去处理过期的数据，从而影响缓存的响应时间和吞吐量。

* **惰性过期**

  只有当访问一个key时，才会判断该key是否已过期，过期则清除。该策略可以最大化地节省CPU资源，却对内存非常不友好。极端情况可能出现大量的过期key没有再次被访问，从而不会被清除，占用大量内存。

* **定期过期**

  每隔一定的时间，会扫描一定数量的数据库的expires字典中**一定数量的key**，并清除其中已过期的key。该策略是前两者的一个折中方案。通过调整定时扫描的时间间隔和每次扫描的限定耗时，可以在不同情况下使得CPU和内存资源达到最优的平衡效果。

* Redis中同时使用了**惰性过期和定期过期**两种过期策略。

## 3.2 内存淘汰策略

> 然而随着数据量的增大，一些过期数据
>
> 1. 定期删除时，从来没有被抽查到
>
> 2. 惰性删除时，也从来没有被点中使用过
>
> 此时大量过期的key堆积在内存中，导致redis内存空间紧张或者很快耗尽。此时**内存淘汰策略登场**



**Redis的内存淘汰策略**，是指当内存使用达到maxmemory极限时，需要使用内存淘汰算法来决定清理掉哪些数据，以保证新数据的存入。**对应的淘汰策略规则如下：**

* **全局的键空间选择性移除：**
  * noeviction：不会驱逐任何key。当内存不足以容纳新写入数据时，新写入操作会报错。
  * allkeys-lru：当内存不足以容纳新写入数据时，对所有key使用LRU算法进行删除。（这个是最常用的）
  * allkeys-lfu: 对所有key使用LFU算法进行删除。**淘汰最近一段时间被访问次数最少的数据，以次数作为参考。**
  * allkeys-random：当内存不足以容纳新写入数据时，在键空间中，**随机移除某个key。**
  
* **设置过期时间的键空间选择性移除：**

  * volatile-lru：对所有设置了过期时间的key使用LRU算法进行删除
  * volatile-lfu: 对所有设置了过期时间的key使用LFU算法进行删除
  * volatile-random：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，**随机移除某个key。**
  * volatile-ttl：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，**有更早过期时间的key优先移除。**
  
  

# 四、Redis的数据类型及底层实现

## 4.1 Redis的五大数据类型

Redis主要有**5种数据类型**，包括String，List，Set，Zset，Hash，满足大部分的使用要求

| 数据类型 | 编码方式                                                     | 操作                                                         |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| STRING   | INT（整型）、EMBSTR（简单动态字符串）、 RAW（简单动态字符串） | 对整个字符串或者字符串的其中一部分执行操作 对整数和浮点数执行自增或者自减操作 |
| LIST     | quicklist或ziplist                                           | 从两端压入或者弹出元素 对单个或者多个元素进行修剪， 只保留一个范围内的元素 |
| SET      | 集合对象的编码可以是 intset 或者 hashtable                   | Set是string类型的`无序集合`                                  |
| HASH     | 哈希对象的编码可以是 ziplist 或者 hashtable                  | Redis hash是一个string类型的`field`和`value`的映射表，hash特别适合用于存储对象。类似Java里面的`Map<String,Map<Object,Object>>` |
| ZSET     | 有序集合的编码可以是 ziplist 或者 skiplist                   | Redis有序集合zset与普通集合set非常相似，是一个`没有重复元素`的字符串集合。 不同之处是有序集合的每个成员都关联了一个`评分（score）`,这个评分（score）被用来按照从最低分到最高分的方式排序集合中的成员。`集合的成员是唯一的，但是评分可以是重复了` 。 |

## 4.2 五种基本数据结构底层实现

> 好的博客：[Redis5种基本数据结构底层实现 ](https://www.cnblogs.com/CryFace/p/13762241.html)

**Redis底层结构：**

![image-20210924093627630](https://gitee.com/jobim/blogimage/raw/master/img/20210924093627.png)



**<font color='blue' size='4'>渐近式 rehash：</font>**

* **dict结构中有两张哈希表（ht[2]）**：只有在重哈希的过程中，ht[0]和ht[1]才都有效。而在平常情况下，只有ht[0]有效，ht[1]里面没有任何数据。
* 扩展或收缩哈希表需要将 ht[0]里面的所有键值对 rehash 到 ht[1]里面， 但是， 这个 rehash 动作并不是一次性、集中式地完成的， 而是分多次、渐进式地完成的（redis为单线程，防止当哈希表中保存的键值对数量很大是，如果一次性rehash的话会导致服务器在一段时间内停止服务）。

**触发rehash的条件：**

* **负载因子计算：**哈希表的负载因子计算：负载因子 = 哈希表已保存节点数量 / 哈希表大小

* **rehash的条件**触发dict的rehash主要有两种：一种是触发扩容操作，另一种是触发收缩操作。
  * **触发扩容操作条件**：当以下条件中的任意一个被满足时， 程序会自动开始对哈希表执行扩展操作。
    * 服务器目前没有在执行 BGSAVE 命令或者 BGREWRITEAOF 命令，并且哈希表的负载因子大于等于 1。
    * 服务器目前正在执行 BGSAVE 命令或者 BGREWRITEAOF 命令，并且哈希表的负载因子大于等于 5。（此时存在子进程在执行aof文件重写或者生成RDB文件）
  * **触发收缩操作条件：**当哈希表的负载因子小于0.1时，程序自动开始对哈希表执行收缩操作

**渐进式 rehash：**

1. 为ht[1]分配空间，让dict字典同时持有 ht[0] 和 ht[1] 两个哈希表。扩容的大小是原数组的两倍。

2. 在字典中维持一个索引计数器变量rehashidx，并将它的值设置为0，表示rehash工作正式开始。

3. 在rehash进行期间，每次对字典执行添加、删除、查找或者更新操作时，程序除了执行指定的操作以外，还会顺带将ht[0]哈希表在 rehashidx索引(table[rehashidx]桶上的链表)上的所有键值对rehash到ht[1]上，当rehash工作完成之后，将rehashidx属性的值增一，表示下一次要迁移链表所在桶的位置。

   * **删除和查找**：在进行渐进式rehash的过程中，**字典会同时使用ht[0]和ht[1]两个哈希表**，所以在渐进式rehash进行期间，**字典的删除、查找、更新等操作会在两个哈希表上进行**。比如说，要在字典里面查找一个键的话，程序会先在ht[0]里面进行查找，如果没找到的话，就会继续到ht[1]里面进行查找，诸如此类。

   * **新增数据**：在渐进式 rehash 执行期间，**新添加到字典的键值对一律会被保存到ht[1]里面**，而ht[0]则不再进行任何添加操作。这一措施保证了ht[0]包含的键值对数量会只减不增，并随着rehash操作的执行而最终变成空表。

4. 随着字典操作的不断执行，最终在某个时间点上，ht[0]的所有桶对应的键值对都会被rehash至ht[1]，这时程序将rehashidx属性的值设为-1，表示rehash操作已完成。

   

**Redis中每个键值对都会对应一个`dictEntry`：**

* Redis中每个键值对都会有一个`dictEntry`，里面指向了key和value的指针，next 指向下一个 dictEntry。
* key 是字符串，但是 Redis 没有直接使用 C 的字符数组，而是**存储在redis自定义的 SDS中**。
* value 既不是直接作为字符串存储，也不是直接存储在 SDS 中，而是存储在redisObject 中。实际上五种常用的数据类型的任何一种，都是通过 redisObject 来存储的。

![image-20210911154947954](https://gitee.com/jobim/blogimage/raw/master/img/20210911154948.png)



**redisObject的结构：**

* 为了便于操作，Redis采用redisObjec结构来统一五种不同的数据类型。同时，为了识别不同的数据类型，redisObjec中定义了type和encoding字段对不同的数据类型加以区别。简单地说，redisObjec就是string、hash、list、set、zset的父类，

![image-20210911155059380](https://gitee.com/jobim/blogimage/raw/master/img/20210911155059.png)





### 4.1.2 String类型



**<font color='red'>SDS简单动态字符串：</font>**

* **embstr 与 raw 类型底层的数据结构其实都是 SDS** (简单动态字符串，Redis 内部定义 sdshdr 一种结构)。

* **SDS结构：**

  ![image-20210912175102283](https://gitee.com/jobim/blogimage/raw/master/img/20210912175102.png)

  * len 表示 SDS 的长度，使我们在获取字符串长度的时候可以在 O(1)情况下拿到，而不是像 C 那样需要遍历一遍字符串。
  * alloc 可以用来计算 free 就是字符串已经分配的未使用的空间，有了这个值就可以引入预分配空间的算法了，而不用去考虑内存分配的问题。
  * buf 表示字符串数组，真存数据的。

  * 底层源码：

    ![image-20210912175248437](https://gitee.com/jobim/blogimage/raw/master/img/20210912175248.png)
  
  选择不同的sdshdrn，根据len的大小分布同步的位数。（如果给len，alloc都分配32个字节浪费空间）

<hr/>



<font color='red'>**String类型三种编码格式：**</font>

* **int编码格式**：

  * **当字符串键值的内容可以用一个64位有符号整形来表示时，Redis会将键值转化为long型来进行存储**，此时即对应 `OBJ_ENCODING_INT` 编码类型。内部的内存结构表示如下:

    ![image-20210912175417721](https://gitee.com/jobim/blogimage/raw/master/img/20210912175417.png)

  * 注意：Redis 启动时会预先建立 **10000 个分别存储 0~9999 的 redisObject 变量作为共享对**象，这就意味着如果 set字符串的键值在 0~10000 之间的话，则可以 直接指向共享对象 而不需要再建立新对象，此时键值不占空间！（类似于Java中的常量池）

    ![image-20210912175554502](https://gitee.com/jobim/blogimage/raw/master/img/20210912175554.png)

  * 只有整数才会使用 int，如果是浮点数， Redis 内部其实先将浮点数转化为字符串值，然后再保存。



* **embstr编码格式**：

  * 对于**长度小于44的字符串**，Redis 对键值采用`OBJ_ENCODING_EMBSTR `方式，代表 embstr 格式的 SDS(Simple Dynamic String 简单动态字符串)

    CPU在读取数据的时候是以缓存行为单位的，一般一个缓存行的大小为64byte。

  * 从内存结构上来讲 即字符串 sds结构体与其对应的 redisObject 对象分配在同一块连续的内存空间，字符串sds嵌入在redisObject对象之中一样（减少内存碎片）。下图为底层源码：

    ![image-20210912175746487](https://gitee.com/jobim/blogimage/raw/master/img/20210912175746.png)

* **raw编码格式**：
  * 字符串的键值为**长度大于44的超长字符串时**，Redis 则会将键值的内部编码方式改为`OBJ_ENCODING_RAW`格式.此时动态字符串sds的内存与其依赖的redisObject的**内存不再连续了**
  * 注意：对于embstr，由于其实现是只读的，因此**在对embstr对象进行修改时，都会先转化为raw再进行修改**。因此，只要是修改embstr对象，修改后的对象一定是raw的，无论是否达到了44个字节。
    ![image-20210912180116239](https://gitee.com/jobim/blogimage/raw/master/img/20210912180116.png)





**int、embstr、raw的区别？**

| 类型   | 描述                                                         |
| ------ | ------------------------------------------------------------ |
| int    | Long类型整数时，RedisObject中的<font color='red'>ptr指针直接赋值为整数数据，不再额外的指针再指向整数了，节省了指针的空间开销</font>。 |
| embstr | 当保存的是字符串数据且字符串小于等于44字节时，<font color='red'>embstr类型将会调用内存分配函数，只分配一块连续的内存空间</font>，空间中依次包含 redisObject 与 sdshdr 两个数据结构，让元数据、指针和SDS是一块连续的内存区域，这样就可以避免内存碎片 |
| raw    | 当字符串大于44字节时，SDS的数据量变多变大了，SDS和RedisObject布局分家各自过，会给SDS分配多的空间并用指针指向SDS结构，<font color='red'>raw 类型将会调用两次内存分配函数，分配两块内存空间，一块用于包含 redisObject结构，而另一块用于包含 sdshdr 结构</font> |

> **流程图：**
>
> ![image-20210912223951498](https://gitee.com/jobim/blogimage/raw/master/img/20210912223951.png)



**Redis为什么重新设计一个 SDS 数据结构？**

|                                         | C语言                                                        | SDS                                                          |
| --------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| <font color='red'>字符串长度处理</font> | 需要从头开始遍历，直到遇到 '\0' 为止，时间复杂度O(N)         | **记录当前字符串的长度，直接读取即可，时间复杂度 O(1)**      |
| <font color='red'>内存重新分配</font>   | 分配内存空间超过后，会导致数组下标越级或者内存分配溢出       | 空间预分配<br/>SDS 修改后，**len 长度小于 1M，那么将会额外分配与 len 相同长度的未使用空间。如果修改后长度大于 1M，那么将分配1M的使用空间。**<br/>惰性空间释放<br/>有空间分配对应的就有空间释放。SDS 缩短时并不会回收多余的内存空间，而是使用 free 字段将多出来的空间记录下来。如果后续有变更操作，直接使用 free 中记录的空间，减少了内存的分配。 |
| <font color='red'>二进制安全</font>     | 二进制数据并不是规则的字符串格式，可能会包含一些特殊的字符，比如 '\0' 等。前面提到过，C中字符串遇到 '\0' 会结束，那 '\0' 之后的数据就读取不上了 | 根据 len 长度来判断字符串结束的，二进制安全的问题就解决了    |


​		

### 4.2.2 Hash数据结构

Hash的编码方式包括`ziplist`和`hashtable`两种，两者的转换规则：

> **`hash-max-ziplist-entries`：使用压缩列表保存时哈希集合中的元素个数。**
> **`hash-max-ziplist-value`：使用压缩列表保存时哈希集合中单个元素的最大长度。**
>
> * Hash类型键的字段个数 **小于 hash-max-ziplist-entries 并且每个字段名和字段值的长度 小于 hash-max-ziplist-value 时**，Redis才会使用 OBJ_ENCODING_ZIPLIST来存储该键，前述条件任意一个不满足则会转换为 OBJ_ENCODING_HT的编码方式
>
>   ![image-20210913092542082](https://gitee.com/jobim/blogimage/raw/master/img/20210913092542.png)
>
> * ziplist升级到hashtable可以，反过来降级不可以



<hr/>





<font color='red'>**Hash类型三种编码格式：**</font>

* **ziplist编码结构（重点）：**

  * 压缩列表（ziplist）是Redis为了节省内存而开发的，是由一系列特殊编码的连续内存块组成的顺序型数据结构，一个压缩列表可以包含任意多个节点（entry），每个节点可以保存一个字节数组或者一个整数值。

  * ziplist是一个经过特殊编码的双向链表，**它不存储指向上一个链表节点和指向下一个链表节点的指针，而是存储上一个节点长度和当前节点长度**，通过牺牲部分读写性能，来换取高效的内存空间利用率，节约内存，是一种时间换空间的思想。

  * **ziplist压缩列表节点的构成:**

    ![image-20210913093123529](https://gitee.com/jobim/blogimage/raw/master/img/20210913093123.png)

    ![image-20210925231522649](https://gitee.com/jobim/blogimage/raw/master/img/20210925231522.png)
  
  * **zlentry实体结构解析：**每个zlentry由前一个节点的长度、encoding和entry-data三部分组成
  
    > ![image-20210913093739347](https://gitee.com/jobim/blogimage/raw/master/img/20210913093739.png)
    >
    > ![image-20210913093755530](https://gitee.com/jobim/blogimage/raw/master/img/20210913093755.png)
    >
    > ![image-20210913093846317](https://gitee.com/jobim/blogimage/raw/master/img/20210913093846.png)





* **hashtable编码结构：**
  * 在 Redis 中，hashtable 被称为字典（dictionary），它是一个**数组+链表**的结构

  * 注意：使用hash数据结构时，hashtable 里面的key无法设置过期时间

  * **OBJ_ENCODING_HT源码分析：**

    ![image-20210913094715153](https://gitee.com/jobim/blogimage/raw/master/img/20210913094715.png)

    ![image-20210913094206510](https://gitee.com/jobim/blogimage/raw/master/img/20210913094206.png)

    可以看到我们每个dict结构里面都有**两个hashtable（用于扩容的时候拷贝）**。（ht[0]和ht[1]）。虽然dict结构有两个hashtable，但是通常情况下只有一个hashtable是有值的。在dict扩容缩容的时候，需要分配新的hashtable，然后进行渐近式搬迁，这时候两个hashtable存储的旧的hashtable和新的hashtable。搬迁结束后，旧hashtable删除，新的取而代之。

    ![image-20210913094343966](https://gitee.com/jobim/blogimage/raw/master/img/20210913094344.png)
  
  * hashtable的结构和Java的HashMap几乎是一样的，都是通过分桶的方式来解决hash冲突的。





**明明有链表了，为什么出来一个压缩链表?**



* 普通的双向链表会有两个指针，在存储数据很小的情况下，**我们存储的实际数据的大小可能还没有指针占用的内存大，得不偿失**。ziplist 是一个特殊的双向链表没有维护双向指针:prev next；而是**存储上一个 entry的长度和 当前entry的长度，通过长度推算下一个元素在什么地方**。牺牲读取的性能，获得高效的存储空间，因为(简短字符串的情况)存储指针比存储entry长度更费内存。这是典型的“时间换空间”。
* **链表在内存中一般是不连续的，遍历相对比较慢**，而ziplist可以很好的解决这个问题，普通数组的遍历是根据数组里存储的数据类型找到下一个元素的(例如int类型的数组访问下一个元素时每次只需要移动一个sizeof(int)就行)，但是ziplist的每个节点的长度是可以不一样的，而我们面对不同长度的节点又不可能直接sizeof(entry)，所以ziplist只好将一些必要的偏移量信息记录在了每一个节点里，使之能跳到上一个节点或下一个节点。
* **头节点里有头节点里同时还有一个参数 len**，和string类型提到的 SDS 类似，这里是用来记录链表长度的。因**此获取链表长度时不用再遍历整个链表，直接拿到len值就可以了**，这个时间复杂度是 O(1)



### 4.2.3 List数据结构

* 在低版本的Redis中，list采用的底层数据结构是**ziplist+linkedList**；高版本的Redis中底层数据结构是**quicklist**(相当于ziplist和linkedList的结合)和**ziplist**，而quicklist也用到了ziplist

**quicklist的配置参数：**

* ziplist压缩配置：list-compress-depth 0。**表示一个quicklist两端不被压缩的节点个数**。这里的节点是指quicklist双向链表的节点，而不是指ziplist里面的数据项个数

  > 参数list-compress-depth的取值含义如下：
  > 0: 是个特殊值，表示都不压缩。这是Redis的默认值。
  > 1: 表示quicklist两端各有1个节点不压缩，中间的节点压缩。
  > 2: 表示quicklist两端各有2个节点不压缩，中间的节点压缩。
  > 3: 表示quicklist两端各有3个节点不压缩，中间的节点压缩。
  > 依此类推…

* ziplist中entry配置：list-max-ziplist-size -2。**当取正值的时候，表示按照数据项个数来限定每个quicklist节点上的ziplist长度**。比如，当这个参数配置成5的时候，表示每个quicklist节点的ziplist最多包含5个数据项。**当取负值的时候，表示按照`占用字节数`来限定每个quicklist节点上的ziplist长度**。这时，它只能取-1到-5这五个值，

  > 每个值含义如下：
  > -5: 每个quicklist节点上的ziplist大小不能超过64 Kb。（注：1kb => 1024 bytes）
  > -4: 每个quicklist节点上的ziplist大小不能超过32 Kb。
  > -3: 每个quicklist节点上的ziplist大小不能超过16 Kb。
  > -2: 每个quicklist节点上的ziplist大小不能超过8 Kb。（-2是Redis给出的默认值）
  > -1: 每个quicklist节点上的ziplist大小不能超过4 Kb。



**quicklist的结构：**

* quicklist 实际上是 zipList 和 linkedList 的混合体，它将 linkedList按段切分，每一段使用 zipList 来紧凑存储，多个 zipList 之间使用双向指针串接起来。

  ![image-20210913103619615](https://gitee.com/jobim/blogimage/raw/master/img/20210913103619.png)

  ![image-20210913103433479](https://gitee.com/jobim/blogimage/raw/master/img/20210913103433.png)

  ![image-20210913103602109](https://gitee.com/jobim/blogimage/raw/master/img/20210913103602.png)





### 4.2.4 Set数据结构

* Redis使用`intset`或`hashtable`存储set。如果元素都是整数类型，就用intset存储（或者元素个数<=`set-max-intset-entries`）。如果不是整数类型，就用hashtable（数组+链表的存来储结构）。key就是元素的值，value为null。

  ![image-20210913103943482](https://gitee.com/jobim/blogimage/raw/master/img/20210913103943.png)



**intset：**

* 整数集合（intset）是Redis用于保存整数值的集合抽象数据类型，它可以保存类型为int16_t、int32_t 或者int64_t 的整数值，并且保证集合中不会出现重复元素。

* 定义如下：

  ```c++
  typedef struct intset{
       //编码方式
       uint32_t encoding;
       //集合包含的元素数量
       uint32_t length;
       //保存元素的数组
       int8_t contents[];
  }intset;
  ```

  整数集合的每个元素都是 contents 数组的一个数据项，它们按照从小到大的顺序排列，并且不包含任何重复项。

  length 属性记录了 contents 数组的大小。

* 当元素个数大于 set-max-intset-entries , 或者 元素无法用整形表示 将使用hashtable存储数据



### 4.2.5 ZSet数据结构

zet的底层编码有两种数据结构，一个`ziplist`，一个是`skiplist`（跳表）



<font color='red'>**ZSet类型三种编码格式：**</font>

* **ziplist(压缩列表)**

  * **当有序集合的元素个数小于`zset-max-ziplist- entries`配置(默认128个)**，**同时每个元素的值都小于`zset-max-ziplist-value`配置(默认64字节)时**，Redis会用ziplist来作为有序集合的内部实现，ziplist 可以有效减少内存的使用。

  * 压缩列表内的集合元素按分值从小到大进行排序，分值较小的元素被放置在靠近表头的方向，而分值较大的元素则被放在靠近表尾的方向。可以参考示意图如下：

    ![image-20210913124123249](https://gitee.com/jobim/blogimage/raw/master/img/20210913124123.png)



* **skiplist(跳跃表)**

  > 好的博客：[JavaGuide/Redis(2)——跳跃表](https://github.com/linmuhan/JavaGuide/blob/master/docs/database/Redis/redis-collection/Redis(2)——跳跃表.md)

  * 当ziplist条件不满足时，有序集合会使用skiplist作为内部实现，因为此时ziplist的读写效率会下降。

  * 跳表是可以实现二分查找的有序链表

  * 跳表是一个最典型的空间换时间解决方案，而且只有在数据量较大的情况下才能体现出来优势。而且应该是读多写少的情况下才能使用，所以它的适用范围应该还是比较有限的

    ![image-20210913111326745](https://gitee.com/jobim/blogimage/raw/master/img/20210913111326.png)
  
  *  时间复杂度是O(logN)，所以空间复杂度是O(N)



<font color='blue'>**跳跃表插入过程：**</font>

* 新插入一个节点之后，就会打乱上下相邻两层链表上节点个数严格的 2:1 的对应关系。如果要维持这种对应关系，就必须把新插入的节点后面的所有节点 *（也包括新插入的节点）* 重新进行调整，这会让时间复杂度重新蜕化成 *O(n)*。删除数据也有同样的问题。

* **skiplist** 为了避免这一问题，它不要求上下相邻两层链表之间的节点个数有严格的对应关系，而是 **为每个节点随机出一个层数(level)**。比如，一个节点随机出的层数是 3，那么就把它链入到第 1 层到第 3 层这三层链表中。为了表达清楚，下图展示了如何通过一步步的插入操作从而形成一个 skiplist 的过程：

* 每一个节点的层数（level）是随机出来的，而且新插入一个节点并不会影响到其他节点的层数，因此，**插入操作只需要修改节点前后的指针，而不需要对多个节点都进行调整**，这就降低了插入操作的复杂度。

  ![image-20210926085204437](https://gitee.com/jobim/blogimage/raw/master/img/20210926085204.png)

**跳跃表的底层实现：**

* `zskiplistNode`，跳跃表的节点信息。`zskiplist`跳跃表的相关信息。

  ```c++
  /* ZSETs use a specialized version of Skiplists */
  typedef struct zskiplistNode {
      // value
      sds ele;
      // 分值
      double score;
      // 后退指针
      struct zskiplistNode *backward;
      // 层
      struct zskiplistLevel {
          // 前进指针
          struct zskiplistNode *forward;
          // 跨度
          unsigned long span;
      } level[];
  } zskiplistNode;
  
  typedef struct zskiplist {
      // 跳跃表头指针
      struct zskiplistNode *header, *tail;
      // 表中节点的数量
      unsigned long length;
      // 表中层数最大的节点的层数
      int level;
  } zskiplist;
  ```

  ![image-20210926002534612](https://gitee.com/jobim/blogimage/raw/master/img/20210926002534.png)

* **随机层数：**

  ```c++
  int zslRandomLevel(void) {
      int level = 1;
      while ((random()&0xFFFF) < (ZSKIPLIST_P * 0xFFFF))
          level += 1;
      return (level<ZSKIPLIST_MAXLEVEL) ? level : ZSKIPLIST_MAXLEVEL;
  }
  ```

  期望的目标是 50% 的概率被分配到 `Level 1`，25% 的概率被分配到 `Level 2`，12.5% 的概率被分配到 `Level 3`，以此类推...有 2-63 的概率被分配到最顶层，因为这里每一层的晋升率都是 50%。





**为什么不使用平衡树/红黑树，而使用跳跃表？**

1. 在高并发的情况下，树形结构需要执行一些类似于 rebalance 这样的可能涉及整棵树的操作，相对来说跳跃表插入和删除只需要修改相邻节点的指针
2. 在复杂度与红黑树相同的情况下，跳跃表实现起来更简单，看起来也更加直观
3. 在做范围查找的时候，平衡树比skiplist操作要复杂





## 其他数据类型







# 五、Redis单线程VS多线程

> **Redis工作线程是单线程的，但是，整个Redis来说，是多线程的**



**Redis的版本发展：**

1. 版本3.x ，redis是单线程。Redis工作线程是单线程的，但Redis的其他功能，比如持久化、异步删除、集群数据同步等等，是由额外的线程执行的。

2. 版本4.x，严格意义来说也不是单线程，而是负责处理客户端请求的线程是单线程，但是开始加了点多线程的东西(异步删除，以及通过 Redis 模块实现的阻塞命令等)。

   * **4.0加入一点多线程的原因？**

   * 当我（Redis）**需要删除一个很大的数据时，因为是单线程同步操作，这就会导致 Redis 服务卡顿**，于是在 Redis 4.0 中就新增了多线程的模块，当然此版本中的多线程主要是为了解决删除数据效率比较低的问题的。

3. 最新版本的6.0.x后，在网络 IO 处理方面上了多线程，及**I/O多路复用**。



**Redis为什么这么快？**

1. **C语言实现**，C语言的执行速度快。

2. **Redis 是基于内存的，内存的读写速度非常快。**

3. Redis 是单线程的，**避免了不必要的上下文切换和竞争条件**，也不存在多线程导致的切换而消耗 CPU，不用去考虑各种锁的问题，不存在加锁释放锁操作，没有因为可能出现死锁而导致的性能消耗。

4. **丰富的数据结构**，对数据操作也简单，Redis 中的数据结构是专门进行设计的。这些简单的数据结构的查找和操作的时间大部分复杂度都是 O(1)，因此性能比较高

5. **基于非阻塞的I/O多路复用技术**，可以处理并发的连接。非阻塞 IO 部实现采用 epoll，采用了 epoll+自己实现的简单的事件框架。epoll 中的读、写、关闭、连接都转化成了事件，然后利用 epoll 的多路复用特性，绝不在 IO 上浪费一点时间。

   



**Redis6.0默认是否开启了多线程？**

>Redis将所有数据放在内存中，内存的响应时长大约为100纳秒，对于小数据包，Redis服务器可以处理8W到10W的QPS，这也是Redis处理的极限了，对于80%的公司来说，单线程的Redis已经足够使用了。

* 在Redis6.0中，**多线程机制默认是关闭的**，如果需要使用多线程功能，需要在redis.conf中完成两个设置

  ![image-20210913160220239](https://gitee.com/jobim/blogimage/raw/master/img/20210913160220.png)

  * 设置io-thread-do-reads配置项为yes，表示启动多线程。
  * 设置线程个数。关于线程数的设置，官方的建议是如果为 4 核的 CPU，建议线程数设置为 2 或 3，如果为 8 核 CPU 建议线程数设置为 6，线程数一定要小于机器核数，线程数并不是越大越好。



## 5.1 I/O多路复用

* 所谓 I/O 多路复用机制，就是说通过一种机制，可以**监视多个描述符，一旦某个描述符就绪（一般是读就绪或写就绪），能够通知程序进行相应的读写操作**。这种机制的使用需要` select 、 poll 、 epoll` 来配合。多个连接共用一个阻塞对象，应用程序只需要在一个阻塞对象上等待，无需阻塞等待所有连接。**当某条连接有新的数据可以处理时，操作系统通知应用程序，线程从阻塞状态返回，开始进行业务处理。**

* Redis利用epoll来实现IO多路复用，（多路指的是多个socket连接，复用指的是复用一个线程）**将连接信息和事件放到队列中**，再放到文件事件分派器，**事件分派器将事件分发给事件处理器**。

  ![image-20210913153354056](https://gitee.com/jobim/blogimage/raw/master/img/20210913153354.png)

* Redis基于Reactor模式开发了**文件事件处理器**。它的组成结构为4部分：**多个套接字、IO多路复用程序、文件事件分派器、事件处理器。**因为文件事件2分派器队列的消费是单线程的，所以Redis才叫单线程模型



**什么是Reactor设计模式？**

> Reactor模式及响应器模式，是指通过一个或多个输入同时传递给服务处理器的服务请求的事件驱动处理模式。服务端程序处理传入多路请求，并将它们同步分派给请求对应的处理线程，Reactor 模式也叫 Dispatcher 模式。即：通过回调机制实现。我们只需将事件的接口注册到Reactor上，当事件发生之后，会回调注册的接口。
>
> ![image-20210913165738068](https://gitee.com/jobim/blogimage/raw/master/img/20210913165738.png)
>
> Reactor 模式中有 2 个关键组成：
> 1、Reactor：Reactor 在一个单独的线程中运行，负责监听和分发事件，分发给适当的处理程序来对 IO 事件做出反应。 它就像公司的电话接线员，它接听来自客户的电话并将线路转移到适当的联系人；
> 2、Handlers：处理程序执行 I/O 事件要完成的实际事件，类似于客户想要与之交谈的公司中的实际办理人。Reactor 通过调度适当的处理程序来响应 I/O 事件，处理程序执行非阻塞操作。





**select、poll、epoll的区别？**

> **select、poll、epoll都是I/O多路复用的机制。I/O多路复用就是通过一种机制，一个进程可以监视多个文件描述符，一旦某个描述符就绪（读就绪或写就绪），能够通知程序进行相应的读写操作 。**
>
> **但是，select，poll，epoll本质还是同步I/O（I/O多路复用本身就是同步IO）的范畴，因为它们都需要在读写事件就绪后线程自己进行读写，读写的过程阻塞的。**



* **select**

  * select 其实就是把NIO中用户态要遍历的fd数组(我们的每一个socket链接，安装进ArrayList里面的那个)拷贝到了内核态，让内核态来遍历，因为用户态判断socket是否有数据还是要调用内核态的，所有拷贝到内核态后，这样遍历判断的时候就不用一直用户态和内核态频繁切换了。

  * 它仅仅知道了，有I/O事件发生了，却并不知道是哪那几个流（可能有一个，多个，甚至全部），我们只能无差别轮询所有流，找出能读出数据，或者写入数据的流，对他们进行操作。所以**select具有O(n)的无差别轮询复杂度**，同时处理的流越多，无差别轮询时间就越长。
  * **缺点：**
    1. bitmap最大1024位，一个进程最多只能处理1024个客户端
    2. 每次调用select，都需要把fd集合从用户态拷贝到内核态。文件描述符数组拷贝到了内核态(只不过无系统调用切换上下文的开销。（高并发场景下这样的拷贝消耗的资源是惊人的。）
    3. select并没有通知用户态哪一个socket有数据，对socket进行扫描时是线性扫描，即采用轮询的方法，需要O(n)的遍历。

* **poll**
  * poll本质上和select没有区别，它将用户传入的数组拷贝到内核空间，然后查询每个fd对应的设备状态， **但是它没有最大连接数的限制**，原因是它是基于链表来存储的.
  * 缺点：
    1. pollfds数组拷贝到了内核态，仍然有开销
    2. poll并没有通知用户态哪一个socket有数据，仍然需要O(n)的遍历

* **epoll**
  * epoll是基于事件驱动的I/O方式，使用event事件通知机制，**每次socket中有数据会主动通知内核，并加入到就绪链表中，不需要遍历所有的socket**。相对于select来说，epoll没有描述符个数限制；使用一个文件描述符管理多个描述符，将用户关心的文件描述符的事件存放到内核的一个事件表中，通过内存映射，使其**在用户空间也可直接访问，省去了拷贝带来的资源消耗。**
  * 三个函数：
    * **epoll_create**函数：创建一个epoll实例并返回，该实例可以用于监控__size个文件描
    * **epoll_ctl**函数：向epoll中注册事件，该函数如果调用成功返回0，否则返回-1。
    * **epoll_wait**函数：类似与select机制中的select函数，等待内核返回监听描述符的事件产生。该函数**返回已经就绪的事件的数量**，如果为-1表示出错。

![image-20210913175358200](https://gitee.com/jobim/blogimage/raw/master/img/20210913175358.png)





# 六、布隆过滤器BloomFilter

* 布隆过滤器实际上**是一个很长的二进制向量和一系列随机映射函数**。布隆过滤器可以用于检索一个元素是否在一个集合中。它的优点是空间效率和查询时间都远远超过一般的算法，缺点是有一定的误识别率和删除困难



**优点：**

- 时间复杂度低，增加和查询元素的时间复杂为O(N)，（N为哈希函数的个数，通常情况比较小）
- 存储空间小，如果允许存在一定的误判，布隆过滤器是非常节省空间的（相比其他数据结构如Set集合）

**缺点：**

* 不能删除元素，因为删掉元素会导致误判率增加。

* 有点一定的误判率，不同的数据可能出来相同的hash值

* 无法获取元素本身

  

**<font color='blue'>布隆过滤器原理：</font>**

* 由一个初值都为零的bit数组和多个个哈希函数构成，用来快速判断某个数据是否存在。

  ![image-20210921212251468](https://gitee.com/jobim/blogimage/raw/master/img/20210921212251.png)

* 当有变量被加入集合时，**通过N个映射函数将这个变量映射成位图中的N个点，把它们置为 1**（假定有两个变量都通过 3 个映射函数）。

* 查询某个变量的时候我们只要看看这些点是不是都是 1， 就可以大概率知道集合中有没有它了

* 注意：如果这些点，有任何一个为零则被查询变量一定不在，如果都是 1，则被查询变量很可能存在。

> 布谷鸟过滤器：解决了布隆过滤器不能删除元素的问题。



**布隆过滤器的使用场景：**

* 解决缓存穿透的问题。

  * 白名单实现：全部合法的key都需要放入过滤器+redis里面，不然数据就是返回null

  * 黑名单实现：对于所以数据库中不存在的key放入黑名单

* 黑名单校验









# 七、缓存雪崩、缓存穿透、缓存击穿



<font color='blue' size='4'>**缓存雪崩：**</font>

* **`缓存雪崩`**是指**缓存同一时间大面积的失效，所以，后面的请求都会落到数据库上，造成数据库短时间内承受大量请求而崩掉。**

**缓存雪崩原因：**

* 在一个较短的时间内，缓存中较多的key集中过期

**解决办法：**

1. **redis缓存集群实现高可用**，主从+哨兵+集群
2. **使用锁或队列**：
    用加锁或者队列的方式保证来保证不会有大量的线程对数据库一次性进行读写，从而避免失效时大量的并发请求落到底层存储系统上。不适用高并发情况
4. **将缓存失效时间分散开：**
比如我们可以在原有的失效时间基础上增加一个随机值，比如1-5分钟随机，这样每一个缓存的过期时间的重复率就会降低，就很难引发集体失效的事件。
4. **依赖隔离组件为后端限流熔断并降级。比如使用Sentinel或Hystrix限流降级组件**



<font color='blue' size='4'>**缓存穿透：**</font>

* **`缓存穿透`是指缓存和数据库中都没有的数据，导致所有的请求都落到数据库上，造成数据库短时间内承受大量请求而崩掉。**

**解决办法：**

1. **对空值缓存**：如果一个查询返回的数据为空（不管是数据是否不存在），我们仍然把这个空结果（null）进行缓存，设置空结果的过期时间会很短

2. **采用布隆过滤器**：

  * **Google布隆过滤器Guava解决缓存穿透**

    Guava是google开发的java基础库，其中提供了布隆过滤器的实现，即名为BloomFilter的类。但Guava只能单机使用。

  * **Redis布隆过滤器解决缓存穿透**



<font color='blue' size='4'>**缓存击穿：**</font>

* **大量的请求同时查询一个 key 时，此时这个key正好失效了，就会导致大量的请求都打到数据库上面去**

**解决办法：**

1. **实时调整过期时间：** 现场监控哪些数据热门，实时调整key的过期时长。或者设置key永不过期。

2. **使用分布式互斥锁：**只允许**一个线程重建缓存，其他线程等待重建缓存的线程执行完**，重新从缓存获取数据即可。set(key,value,timeout)

   ```java
   String get(String key) {
       // 从Redis中获取数据
       String value = redis.get(key);
       // 如果value为空， 则开始重构缓存
       if (value == null) {
           // 只允许一个线程重建缓存， 使用nx， 并设置过期时间ex
           String mutexKey = "mutext:key:" + key;
           if (redis.set(mutexKey, "1", "ex 180", "nx")) {
                // 从数据源获取数据
               value = db.get(key);
               // 回写Redis， 并设置过期时间
               redis.setex(key, timeout, value);
               // 删除key_mutex
               redis.delete(mutexKey);
           }// 其他线程休息50毫秒后重试
           else {
               Thread.sleep(50);
               get(key);
           }
       }
       return value;
   }
   ```

3. **分级缓存：**
  * 采用 L1 (一级缓存)和 L2(二级缓存) 缓存方式，L1 缓存失效时间短，L2 缓存失效时间长。 请求优先从 L1 缓存获取数据，如果 L1缓存未命中则加锁，只有 1 个线程获取到锁,这个线程再从数据库中读取数据并将数据再更新到到 L1 缓存和 L2 缓存中，而其他线程依旧从 L2 缓存获取数据并返回。
  * 这种通过避免缓存同时失效并结合锁机制实现。所以，当数据更 新时，只能淘汰 L1 缓存，不能同时将 L1 和 L2 中的缓存同时淘汰
  * 缺点：L2 缓存中 可能会存在脏数据，需要业务能够容忍这种短时间的不一致。而且，这种方案可能会造成额外的缓存空间浪费。



# 八、Redis 的持久化机制



<font color='blue' size='4'>**RDB快照（snapshot）:**</font>

* `RDB`（Redis DataBase）是将某一个时刻的内存快照（Snapshot），以二进制的方式写入磁盘的过程。在默认情况下保存在名字为 dump.rdb 的二进制文件中。（默认开启）



**save和bgsave命令：（手动触发）**

* **bgsave命令：**
  * Redis 借助操作系统提供的写时复制技术（Copy-On-Write, COW）。客户端可以使用`BGSAVE命令`来创建一个`快照`,当`redis服务器`接收到`客户端`的`BGSAVE`命令时,redis会调用`fork`来`创建一个子进程`，会先将数据写入到 一个`临时文件`中，待持久化过程都结束了，**再用这个临时文件替换上次持久化好的文件**。而**父进程则继续处理命令请求**。

* **save命令：**
  * 客户端还可以使用`SAVE命令`来创建一个`快照`,接收到SAVE命令的redis服务器在快照创建完毕之前将不再响应任何其他的命令。即：**使用SAVE命令在快照创建完毕之前,redis处于`阻塞状态`,无法对外服务(写操作)**

| **命令**              | **save**         | **bgsave**                                     |
| --------------------- | ---------------- | ---------------------------------------------- |
| IO类型                | 同步             | 异步                                           |
| 是否阻塞redis其它命令 | 是               | 否(在生成子进程执行调用fork函数时会有短暂阻塞) |
| 复杂度                | O(n)             | O(n)                                           |
| 优点                  | 不会消耗额外内存 | 不阻塞客户端命令                               |
| 缺点                  | 阻塞客户端命令   | 需要fork子进程，消耗内存                       |



**自动触发：**

- 如果用户在`redis.conf`中设置了`save配置选项`,redis会在save选项条件满足之后自动触发一次**`BGSAVE命令`**, **如果设置多个save配置选项,当任意一个save配置选项条件满足,redis也会触发一次BGSAVE命令**

  ![image-20210923145335750](https://gitee.com/jobim/blogimage/raw/master/img/20210923145335.png)

- 表示 `900S(15分钟)`, **key发生1次变化**, 就触发一次 bgsave命令, 持久化一次
- 表示`300S(5分钟)`, **key发生10次变化**, 就触发一次bgsave命令, 持久化一次
- 表示`60S(1分钟)`, **key发生10000次变化**, 就触发一次bgsave命令, 持久化一次

**RDB持久化的优缺点:**

* **优点：**

  1. RDB是一个紧凑压缩的二进制文件，存储效率较高
  2. RDB内部存储的是redis在某个时间点的数据快照，非常适合用于数据备份，全量复制等场景
  3. RDB恢复数据的速度要比AOF快很多

  应用：服务器中每X小时执行bgsave备份，并将RDB文件拷贝到远程机器中，用于灾难恢复

* **缺点：**

  1. RDB方式无论是执行指令还是利用配置，无法做到实时持久化，具有较大的可能性丢失数据
  2. bgsave指令每次运行要执行fork操作创建子进程，要牺牲掉一些性能
  3. 基于快照思想，每次读写都是全部数据，当数据量巨大时，效率非常低



<font color='blue' size='4'>**AOF（append-only file）:**</font>

* AOF 机制对每条写入命令作为日志，以 `append-only` 的模式写入一个日志文件中，在 redis 重启的时候，可以通过回放 AOF 日志中的写入指令来重新构建整个数据集。

* 注意：AOF和RDB同时开启，系统默认取AOF的数据（数据不会存在丢失）

> 可以通过修改配置文件来打开 AOF 功能：# appendonly yes



**AOF同步频率设置：**

* 修改`appendfsync everysec|always|no`指定

| 关键字      | 持久化时机 | 解释                                                         |
| ----------- | ---------- | ------------------------------------------------------------ |
| appendfsync | always     | 始终同步，每次Redis的写入都会立刻记入日志；性能较差但数据完整性比较好 |
| appendfsync | everysec   | 每秒将缓冲区中的指令同步到AOF文件中，数据准确性较高，性能较高，建议使用，也是默认配置。在系统突然宕机的情况下丢失1秒内的数据 |
| appendfsync | no         | 由操作系统控制每次同步到AOF文件的周期，整体过程不可控        |

推荐（并且也是默认）的措施为每秒 fsync 一次， 这种 fsync 策略可以兼顾速度和安全性。





**AOF重写：`重写aof文件`的操作，`并没有读取旧的aof文件`，而是将整个内存中的数据库内容用命令的方式重写了一个新的aof文件, 替换原有的文件这点和快照有点类似。**

* **手动重写：**进入redis客户端执行命令**bgrewriteaof**重写AOF

* 两个配置可以**控制AOF自动重写频率**
  * \# auto-aof-rewrite-min-size 64mb   //aof文件至少要达到64M才会自动重写，文件太小恢复速度本来就很快，重写的意义不大 
  * \# auto-aof-rewrite-percentage 100  //aof文件自上一次重写后文件大小增长了100%则再次触发重写



<font color='blue' size='4'>**Redis 4.0 混合持久化：**</font>

* 重启 Redis 时，我们很少使用 RDB来恢复内存状态，因为会丢失大量数据。我们通常使用 AOF 日志重放，但是重放 AOF 日志性能相对 RDB来说要慢很多，这样在 Redis 实例很大的情况下，启动需要花费很长的时间。 Redis 4.0 为了解决这个问题，带来了一个新的持久化选项——混合持久化。

* 通过如下配置可以开启混合持久化(**必须先开启aof**)：

  \# aof-use-rdb-preamble yes   （Redis 5默认开启）

如果开启了混合持久化，**AOF在重写时**，不再是单纯将内存数据转换为RESP命令写入AOF文件，而是将重写**这一刻之前**的内存做RDB快照处理，并且将RDB快照内容和**增量的**AOF修改内存数据的命令存在一起，都写入新的AOF文件，新的文件一开始不叫appendonly.aof，等到重写完新的AOF文件才会进行改名，覆盖原有的AOF文件，完成新旧两个AOF文件的替换。

于是在 Redis 重启的时候，可以先加载 RDB 的内容，然后再重放增量 AOF 日志就可以完全替代之前的 AOF 全量文件重放，因此重启效率大幅得到提升。

 

混合持久化AOF文件结构：

![img](https://gitee.com/jobim/blogimage/raw/master/img/20210923144222.png)





# 九、主从复制、哨兵模式、群集模式



## 9.1 主从复制

* 主（Master）和从（Slave）分别部署在不同的服务器上，**当主节点服务器写入数据时，同时也会将数据同步至从节点服务器**，通常情况下，**主节点负责写入数据，而从节点负责读取数据**。



**作用：**
- `读写分离`：master写、slave读，提高服务器的读写负载能力
- `负载均衡`：基于`主从结构`，配合`读写分离`，**由slave分担master负载，并根据需求的变化，`改变slave的数量，通过多个从节点分担数据读取负载，大大提高Redis服务器并发量与数据吞吐量`**
- `故障恢复`：当`master出现问题`时，由`slave`提供服务，实现快速的故障恢复
- `数据冗余`：实现数据热备份，是持久化之外的一种数据冗余方式, 因为`slave中和master数据是同步`的
- `高可用基石`：基于`主从复制`，构建`哨兵模式`与`集群`，实现`Redis的高可用`方案



注意：**当主服务器挂掉之后，从服务器不做任何事情。而主服务器重启之后，仍然是主服务器，其之前的从服务器依然存在。**



<font color='blue' size='4'>**全量复制和部分复制：**</font>

**全量复制：**

* 如果你为master配置了一个slave，不管这个slave是否是第一次连接上Master，它都会发送一个**PSYNC**命令给master请求复制数据。

* master收到PSYNC命令后，会在后台进行数据持久化通过bgsave生成最新的rdb快照文件，持久化期间，master会继续接收客户端的请求，它会把这些可能修改数据集的请求缓存在内存中。当持久化进行完毕以后，master会把这份rdb文件数据集发送给slave，slave会把接收到的数据进行持久化生成rdb，然后再加载到内存中。然后，master再将之前缓存在内存中的命令发送给slave。

* 当master与slave之间的连接由于某些原因而断开时，slave能够自动重连Master，如果master收到了多个slave并发连接请求，它只会进行一次持久化，而不是一个连接一次，然后再把这一份持久化的数据发送给多个并发连接的slave。

* 全量复制流程图：

  ![image-20210923182824350](https://gitee.com/jobim/blogimage/raw/master/img/20210923182824.png)



**部分复制：**

* 当master与slave之间的连接断开时，master在写入数据同时也会把写入的数据保存到repl_back_buffer复制缓冲区中，缓存最近一段时间的数据，master和它所有的slave都维护了复制的数据下标offset和master的进程id。

* 当master与slave之间的网络连通后，slave会执行psync {offset} {run_id}命令,offset是slave节点上的偏移量。master接收到slave传输的偏移量，会与repl_back_buffer复制缓冲区中的offset做对比，如果接收到的offset小于repl_back_buffer中记录的偏移量，master就会把两个偏移量之间的数据发送给slave，slave同步完成，slave中的数据就与master中的数据一致。

* 主从复制(部分复制，断点续传)流程图：

  ![image-20210923182842512](https://gitee.com/jobim/blogimage/raw/master/img/20210923182842.png)



## 9.2 哨兵模式

> 主从复制的缺点：当Redis的master就无法提供服务了，只有slave可以提供数据读取服务。此时只能把其中一个slave为成master,以提供写入数据功能，另外一台slave重新做为新的master的从节点，提供读取数据功能，这种解决方法依然需要手动完成。



* 哨兵(sentinel) 是一个分布式系统，用于对主从结构中的每台服务器进行监控，**当出现故障时通过投票机制选择新的master并将所有slave连接到新的master。**

**哨兵的作用：**

1. 监控
不断的检查master和slave是否正常运行。master存活检测、master与slave运行情况检测
2. 通知（提醒）
当被监控的服务器出现问题时，向其他（哨兵间，客户端）发送通知。
3. 自动故障转移
断开master与slave连接，选取一个slave作为master，将其他slave连接到新的master，并告知客户端新的服务器地址



### 9.2.1 Redis Sentinel实现原理



**<font color='blue' size='4'>Redis Sentinel内部的三个定时任务：</font>**

1. **每 10 秒**每个 sentinel 向 master、slaves 节点发送 `INFO` 命令。获得slave节点和确认主从关系。
2. **每 2 秒**每个 sentinel 通过 master 节点的 channel(名称为 `__sentinel__:hello`) 交换信息(pub/sub)，信息包括：
   1. Sentinel 自身信息（IP、Port、RunID、Epoch）
   2. 监视的 Master 节点的信息（Name、IP、Port、Epoch）
3. **每 1 秒**每个 sentinel 对其他 sentinel 和 redis master, slave 发送 `PING` 命令,用于心跳检测,作为节点存活的判断依据



- **每10秒每个sentinel对master和slave执行`info命令`，以发现slave节点和确认主从关系**

  Sentinel 向 master 节点发送 `INFO` 命令后获取到所有 slave 的信息

  Sentinel 与 slave 建立**命令连接**和**订阅连接**

  

  ![image-20210924174703143](https://gitee.com/jobim/blogimage/raw/master/img/20210924174703.png)

- **每2秒每个sentinel通过master节点的channel交换信息(发布订阅)**

  master节点上有一个发布订阅的channel频道：`__sentinel__:hello`，用于所有sentinel之间进行信息交换

  一个sentinel发布消息，消息包含当前sentinel节点的信息，对其他sentinel节点的判断以及当前sentinel对master节点和slave节点的一些判断，其他sentinel都可以接收到这条消息。新加入sentinel节点时，sentinel节点之间可以相互感知，以达到信息交互的功能

  ![image-20210924174917181](https://gitee.com/jobim/blogimage/raw/master/img/20210924174917.png)

- **每1秒每个sentinel对其他sentinel节点和Redis节点执行ping操作**

  每个sentinel都可以知道其他sentinel节点，当监控的master发生故障时，方便进行判断和新master的挑选，这个定时任务是master进行故障判定的依据

  ![image-20210924174943106](https://gitee.com/jobim/blogimage/raw/master/img/20210924174943.png)





**<font color='blue' size='4'>主观下线和客观下线：</font>**

* **主观下线（subjectively down，SDOWN）：当前 Sentinel 实例认为某个 redis 服务为”不可用”状态。**Sentinel 向 redis master 数据节点发送消息后 30s（down-after-milliseconds） 内没有收到有效回复（+PONG、-LOADING 或者 -MASTERDOWN），Sentinel 会将 master 标记为下线（打开 master 结构中 flags 的 `SRI_S_DOWN` 标记）
* **客观下线（objectively down，ODOWN）：多个 Sentinel 实例（超过半数）都认为 master 处于 SDOWN 状态，那么此时 master 将处于 客观下线`SRI_O_DOEN`， ODOWN可以简单理解为master已经被集群确定为”不可用”,将会开启故障转移机制。**



**<font color='blue' size='4'>sentinel领导者选举：</font>**

在多个`sentinel`中选举一个来清理; 在多个sentinel中通过`投票`来选举出来一个领头的`sentinel`;

完成sentinel领导者选举步骤：

1. 每个做主观下线的sentinel节点向其他sentinel节点发送命令，要求将自己设置为领导者
2. 收到命令的sentinel节点如果没有同意其他sentinel节点发送的命令，那么将同意该请求，否则拒绝。（即同意第一个请求的节点为领导者）
3. 如果该sentinel节点发现自己的票数已经超过sentinel集合半数且超过quorum（配置哨兵的参数），将成为领导者
4. 如果此过程中有多个sentinel节点成为领导者，那么将等待一段时间重新进行选举



<font color='blue' size='4'>**故障转移(由sentinel领导者节点完成)：**</font>

1. 从slave节点中选出一个合适的节点作为新的master节点（在线的，响应快的，与原master断开时间短的）
2. 对选出的slave节点执行`slaveof no one`命令，使成为新的master节点
3. **向剩余的slave节点发送命令，让slave节点成为新master节点的slave节点，然后从新master节点同步数据。**
4. sentinel领导者会把原来的master节点设置为slave节点，并保持对其'关注'，当原来的master节点恢复后，sentinel会使其去复制新master节点的数据

**slave节点的选择：**

1. 健康的节点：

2. 1. 在线的
   2. 最近成功通信过的（5s 内回复过 `PING` 命令）
   3. 数据比较新的（与 master 失联时间不超过 10*down-after-milliseconds）

3. slave-priority（slave节点优先级）最高的slave节点

4. 复制偏移量最大的 slave 节点（复制的最完整）

5. 选择 runId 最小的 slave 节点（启动最早的节点）









## 9.3 集群模式

* redis集群是一个由多个主从节点群组成的分布式服务器群，它具有**复制、高可用和分片**特性。Redis集群不需要sentinel哨兵，也能完成节点移除和故障转移的功能。需要将每个节点设置成集群模式，这种集群模式没有中心节点，**可水平扩展**。

  ![image-20210923231003362](https://gitee.com/jobim/blogimage/raw/master/img/20210923231003.png)



<font color='blue' size='4'>**Redis集群原理分析：**</font>

* Redis 集群使用数据分片来实现，Redis Cluster 将所有数据划分为 `16384` 个 哈希槽（hash slot），每个节点负责其中一部分槽位，每个节点只能对自己负责的槽进行读写操作，由于每个节点之间都彼此通信，每个节点都知道另外节点负责管理的槽范围 。

* 当客户端访问任意节点时，**对数据key按照CRC16算法进行hash运算，然后对运算结果对16384进行取模，`HASH_SLOT = CRC16(key) mod 16384`**，如果余数在当前访问的节点管理的槽范围内，则直接返回对应的数据 如果不在当前节点负责管理的槽范围内，则会告诉客户端去哪个节点获取数据，由客户端去正确的节点获取数据

* 这种将哈希槽分布到不同节点的做法使得用户**可以很容易地向集群中添加或者删除节点**，比如：如果用户将新节点 D 添加到集群中， 那么集群只需要将节点 A 、B 、 C 中的某些槽移动到节点 D 就可以了。

  ![image-20210923232332102](https://gitee.com/jobim/blogimage/raw/master/img/20210923232332.png)

**Redis集群选举原理：**

当slave发现自己的master变为FAIL状态时，便尝试进行Failover，以期成为新的master。由于挂掉的master可能会有多个slave，从而存在多个slave竞争成为master节点的过程， 其过程如下：

1. slave发现自己的master变为FAIL

2. 将自己记录的集群currentEpoch加1，并广播FAILOVER_AUTH_REQUEST 信息

3. 其他节点收到该信息，只有master响应，判断请求者的合法性，并发送FAILOVER_AUTH_ACK，对每一个epoch只发送一次ack

4. 尝试failover的slave收集master返回的FAILOVER_AUTH_ACK

5. slave收到超过半数master的ack后变成新Master(这里解释了集群为什么至少需要三个主节点，如果只有两个，当其中一个挂了，只剩一个主节点是不能选举成功的)

6. slave广播Pong消息通知其他集群节点。



# 十、缓存一致性

**缓存双写一致性：**

* 如果redis中有数据，需要和数据库中的值相同
* 如果redis中无数据，数据库中的值要是最新值

**给缓存设置过期时间，是保证最终一致性的解决方案。**



> **保持缓存一致性的策略：**

<font color='blue'>**先更新数据库，再更新缓存（不推荐）：**</font>

**使用该策略的问题：**

* **并发更新问题（双写不一致）**：比如线程1更新了数据库，线程2更新了数据库，线程2更新了缓存，线程1更新了缓存，这样最终存入的就是脏数据。

  ![image-20210926150248820](https://gitee.com/jobim/blogimage/raw/master/img/20210926150248.png)

* **更新失败问题**：若数据库更新成功缓存更新失败则会造成数据不一致问题

* **业务维护难度大**，比如有些更新操作多，但是读取时并不多，可能浪费更新到redis的资源，另外redis缓存的数据并不一定是直接写入数据库的，可能是经过刷选，过滤，复杂计算得出的，这个时候维护麻烦，每次写入数据库，都得更新缓存，重复计算，刷选。并且不一定是更新一张表的数据要更新缓存，可能缓存跟多张表的数据有关系。

<font color='blue'>**先删除缓存，再更新数据库：**</font>

**使用该策略的问题：**

* 存在脏数据的可能，比如线程A删除缓存，线程B查询缓存不存在数据，从数据库获取，获取成功后，数据存入缓存，现在A更新数据。这样缓存中的数据就是脏数据了

  ![image-20210926151150822](https://gitee.com/jobim/blogimage/raw/master/img/20210926151150.png)

  解决：延时双删策略

* 如果删除缓存后，系统处于高并发。此时会出现缓存击穿



**延时双删策略：** 

* 延时双删的方案的思路是，为了避免更新数据库的时候，其他线程从缓存中读取不到数据，就在更新完数据库之后，再 Sleep 一段时间，然后再次删除缓存。

  ![image-20210926140315744](https://gitee.com/jobim/blogimage/raw/master/img/20210926140315.png)

* **这个删除该休眠多久呢？**

  在业务程序运行的时候，统计下线程读数据和写缓存的操作时间，自行评估读数据业务逻辑的耗时，以此为基础来进行估算。然后写数据的休眠时间则在**读数据业务逻辑的耗时基础上加百毫秒即可**。

* **如果mysql主从读写分离架构怎么办？**

  （1）请求A进行写操作，删除缓存
  （2）请求A将数据写入数据库了，
  （3）请求B查询缓存发现，缓存没有值
  （4）请求B去从库查询，这时，还没有完成主从同步，因此查询到的是旧值
  （5）请求B将旧值写入缓存
  （6）数据库完成主从同步，从库变为新值 上述情形，就是数据不一致的原因。

  解决：还是使用双删延时策略。只是，睡眠时间修改为在主从同步的延时时间基础上，加几百ms。

* **缓存删除失败怎么办？**

  如果缓存因为种种问题删除失败，则将需要删除的key发送至消息队列。同时可以采用异步删除的方式提升吞吐量



<font color='blue'>**先更新数据库，再删除缓存：**</font>

**使用该策略的问题：**

* 请求A查询数据库，得一个旧值，请求B将新值写入数据库，请求B删除缓存，请求A将查到的旧值写入缓存。这种情况下会存在脏数据。（出现这种问题的概率极低，除非是查询比写入慢）

  ![image-20210926152112326](https://gitee.com/jobim/blogimage/raw/master/img/20210926152112.png)

* 假如缓存删除失败或者来不及，导致请求再次访问redis时缓存命中，读取到的是缓存旧值。

  ![image-20210926152033111](https://gitee.com/jobim/blogimage/raw/master/img/20210926152033.png)





**<font color='blue'>异步更新缓存（基于订阅binlog的同步机制）：</font>**

**技术整体思路：**

MySQL binlog增量订阅消费+消息队列+增量数据更新到redis

- **读Redis**：热数据基本都在Redis
- **写MySQL**:增删改都是操作MySQL
- **更新Redis数据**：MySQ的数据操作binlog，来更新到Redis



**读取binlog后分析 ，利用消息队列,推送更新各台的redis缓存数据。**

这样一旦MySQL中产生了新的写入、更新、删除等操作，就可以把binlog相关的消息推送至Redis，Redis再根据binlog中的记录，对Redis进行更新。

其实这种机制，很类似MySQL的主从备份机制，因为MySQL的主备也是通过binlog来实现的数据一致性。

**这里可以结合使用canal(阿里的一款开源框架)**，通过该框架可以对MySQL的binlog进行订阅，而canal正是模仿了mysql的slave数据库的备份请求，使得Redis的数据更新达到了相同的效果。



# 十一、Redis实战案例



<font color='blue' size='4'>**案例实战：微信抢红包：**</font>



**需求分析：**

1. 记录谁发了红包，可以使用redis中的list
2. 抢红包。使用redis中list的lpop()操作
3. 记录谁抢了多少，防止重复抢。使用redis中的hash结构
4. 如果红包到期没有抢完，需要回退账户。给list的key设置一个过期时间
5. 红包算法。二倍均值法，每次抢到的金额 = 随机区间 （0， (剩余红包金额M ÷ 剩余人数N ) X 2）

**代码实现：**

```java
import cn.hutool.core.util.IdUtil;
import com.google.common.primitives.Ints;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.annotation.Resource;
import java.util.Arrays;
import java.util.Random;
import java.util.concurrent.TimeUnit;

@RestController
public class RedPackageController {
    public static final String RED_PACKAGE_KEY = "redpackage:";
    public static final String RED_PACKAGE_CONSUME_KEY = "redpackage:consume:";

    @Resource
    private RedisTemplate redisTemplate;

    /**
     * 拆分+发送红包
     * http://localhost:5555/send?totalMoney=100&redPackageNumber=5
     * @param totalMoney
     * @param redPackageNumber
     * @return
     */
    @RequestMapping("/send")
    public String sendRedPackage(int totalMoney,int redPackageNumber) {
        //1 拆红包，总金额拆分成多少个红包，每个小红包里面包多少钱
        Integer[] splitRedPackages = splitRedPackage(totalMoney, redPackageNumber);
        //2 红包的全局ID
        String key = RED_PACKAGE_KEY+IdUtil.simpleUUID();
        //3 采用list存储红包并设置过期时间
        redisTemplate.opsForList().leftPushAll(key,splitRedPackages);
        redisTemplate.expire(key,1,TimeUnit.DAYS);
        return key+"\t"+"\t"+ Ints.asList(Arrays.stream(splitRedPackages).mapToInt(Integer::valueOf).toArray());
    }

    /**
     * http://localhost:5555/rob?redPackageKey=上一步的红包UUID&userId=1
     * @param redPackageKey
     * @param userId
     * @return
     */
    @RequestMapping("/rob")
    public String rodRedPackage(String redPackageKey,String userId) {
        //1 验证某个用户是否抢过红包
        Object redPackage = redisTemplate.opsForHash().get(RED_PACKAGE_CONSUME_KEY + redPackageKey, userId);
        //2 没有抢过就开抢，否则返回-2表示抢过
        if (redPackage == null) {
            // 2.1 从list里面出队一个红包，抢到了一个
            Object partRedPackage = redisTemplate.opsForList().leftPop(RED_PACKAGE_KEY + redPackageKey);
            if (partRedPackage != null) {
                //2.2 抢到手后，记录进去hash表示谁抢到了多少钱的某一个红包
                redisTemplate.opsForHash().put(RED_PACKAGE_CONSUME_KEY + redPackageKey,userId,partRedPackage);
                System.out.println("用户: "+userId+"\t 抢到多少钱红包: "+partRedPackage);
                //TODO 后续异步进mysql或者RabbitMQ进一步处理
                return String.valueOf(partRedPackage);
            }
            //抢完
            return "errorCode:-1,红包抢完了";
        }
        //3 某个用户抢过了，不可以作弊重新抢
        return "errorCode:-2,   message: "+"\t"+userId+" 用户你已经抢过红包了";
    }

    /**
     * 1 拆完红包总金额+每个小红包金额别太离谱
     * 2 算法：二倍均值法
     * @param totalMoney
     * @param redPackageNumber
     * @return
     */
    private Integer[] splitRedPackage(int totalMoney, int redPackageNumber) {
        int useMoney = 0;
        Integer[] redPackageNumbers = new Integer[redPackageNumber];
        Random random = new Random();

        for (int i = 0; i < redPackageNumber; i++) {
            if(i == redPackageNumber - 1) {
                redPackageNumbers[i] = totalMoney - useMoney;
            }else{
                //每次抢到的金额 = 随机区间 （0， (剩余红包金额M ÷ 剩余人数N ) X 2）
                int avgMoney = (totalMoney - useMoney) * 2 / (redPackageNumber - i);
                redPackageNumbers[i] = 1 + random.nextInt(avgMoney - 1);
            }
            useMoney = useMoney + redPackageNumbers[i];
        }
        return redPackageNumbers;
    }
}
```









Redis除了拿来做缓存，你还见过基于Redis的什么用法？
Redis做分布式锁的时候有需要注意的问题？
如果是Redis是单点部署的，会带来什么问题？那你准备怎么解决单点问题呢？
集群模式下，比如主从模式，有没有什么问题呢？
那你简单的介绍一下Redlock吧？你简历上写redisson，你谈谈。
Redis分布式锁如何续期？看门狗知道吗？



[redis](https://www.nowcoder.com/jump/super-jump/word?word=redis)的使用

[redis]()丢失怎么知道？[redis]()主从复制

[项目]()：基础内容，大差不差，还问了如果消息事务失败，[redis]()往回加失败等场景应该怎么办，叫给出解决策略

Redis为什么性能好（数据结构优化实例），哈希扩容的渐进式，[redis]()的pipeline管道，持久化rdb，aof等；

Redis为什么使用单线程；为什么又加回了多线程？

单线程多线程问题。

1. 
2. [redis]() 数据结构
3. [redis]()分布式锁，怎么实现，可靠吗
4. [redis]() zset内部怎么存储的

介绍一下[redis]()还有[redis]()的使用场景

[redis]()的数据类型 

  18、zset的底层实现？为什么使用跳表？跳表的结构？ 

  19、[redis]()的过期策略 

介绍一下[redis]() 

  14、[redis]()为什么可以抗住高并发？ 

  15、为什么[redis]()使用单线程性能会优于多线程？ 

  11.Redis在[项目]()中使用的场景

Redis做缓存的原因 

  Redis 数据类型 为什么快 缓存穿透 缓存击穿 缓存雪崩  区别及解决方式 RDB和AOF区别及原理

###  Redis 

1.  Redis有哪些数据结构

2.  String类型的底层实现:

3.  Hash字典类型

4.  Redis渐进式rehash 为什么

5.  Redis中zset的内部实现跳跃表 为什么

6.  Redis支持事务吗 
7.  Redis单线程还是多线程，为什么？

8.  Redis单线程为什么还并发量那么高

9.  Redis为什么快？ 
10.  缓存穿透以及解决

11.  缓存雪崩以及解决 
12.  缓存击穿以及解决

13.  [redis]()内存满了怎么办 
14.  Redis持久化的方法

15.  AOF重写

16.  [redis]()主从结构 
17.  [redis]()哨兵 
18.  Redis集群

19.  集群是如何判断是否有某个节点挂掉 
20.  分布式锁作用 
21.  一致性哈希 
22.  缓存与数据库双写一致 

、Redis缓存穿透 

1.  Redis为什么快（基于内存，IO多路复用，单线程，使用C语言并有很多优化） 
2.  Redis数据结构对于内存占用的优化（sdshdr5, sdshdr8, sdshdr16, sdshdr32, sdshdr64, 字符串越短，使用越少的内存存储额外信息；list 和 hash 元素少的时候使用 ziplist 编码） 

用过的Redis数据类型、分布式锁（set、Redisson）。         

 Redis持久化策略。 

 [项目]()哪些地方用到了[redis]() 
 • [redis]()用了哪些数据类型 

• 哪些用到了[redis]()缓存

 Redis数据类型，除了五种基本的还了解哪些数据类型？ 
 Redis持久化方法 

Redis如何保持缓存与数据库的一致性 

  Redis在[项目]()中咋使用的，为啥这么用，条数是多少。（我说就十几条他说那为啥不直接放在内存，尴尬了） 

  Redis宕机怎么办（直接在DB中查找） 

、[redis]()分布式锁怎么实现

、mysql和[redis]()区别（讲一下各自优缺点）

 10、为什么不用[redis]()存数据？

3 [redis]()哨兵心跳 事务 pipeline 

Redis分布式锁怎么实现的 

Redis为什么不用hashmap 

[redis]()数据结构 hashmap使用 

-  Redis有哪几种数据结构 
-  Redis数据结构的选取？有什么原则吗 

-  Redis机构树在数据库怎么存的 为什么要这么设计 
-  在[redis]()怎么存的 在数据库如何优化 

Redis 数据结构 应用场景 怎么解决并发访问 怎么解决数据一致性 用来缓存什么东西

Redis集群怎么部署的 一条写入什么时候可以当成提交了 

-  Redis你们用来做什么呢 
-  Redis为什么会比mysql快 从几个角度分析 
-  Redis上的数据是拿来缓存呢还是就只放在[redis]()上 
-  如何保证[redis]()和数据库的一致性问题 
-  如果[redis]()上三分钟有效期的临时数据在申请过程中过期了怎么办（看门狗）

[redis]()几种数据结构，底层的实现形式各是什么 

[redis]() mysql update时数据一致性 

[redis](https://www.nowcoder.com/jump/super-jump/word?word=redis)的底层数据结构

 6 [redis]()服务器是有单独的对不对？服务器访问延迟大概是多少？你觉得大概是什么数量级的？如果放在本地的内存里面，访问延迟会是多少？也是ms级别吗？

5、[redis]()被击穿，雪崩，你怎么解决

 7.[redis]() 高并发解决方案，持久化方案

1. Redis为什么单线程还快？ 
2. Redis什么操作能支持高并发？ 

1. Redis高级数据结构？布隆过滤器？hyperloglog？ 
2. Zset底层数据结构？ziplist + skipList 
3. [redis]()快的原理 

Dubbo注册中心，可以使用[redis]()来做吗？ 

1. Redis缓存过期key删除策略？惰性和定时 
2. 只有惰性行不行？定时有什么用？ 

Redis 底层数据结构类型都有哪些？比如list底层？（双向[链表]()+压缩[链表]()） 

[redis]()为什么比mysql快？

Redis中String的底层？

说说对Redis的理解？[redis]()单线程为什么会快？Redis的zset数据结构？跳跃表？[红黑树]()？ 

[redis]()缓存穿透？ 

1. Redis，zset的数据结构（ziplist（元素个数128个以下，长度小于64，字典（存放key和分值）+跳表）） 
2. [redis]()集群？hash槽，等 

[项目]()中用到Redis的实现原理（复制同步原理、高可用实现、key删除原理、常用数据结构实现原理等） 

[redis]()的基本数据结构？底层结构了解吗？[redis]()的用途？

 你用到了Redis，那么Redis还可以做什么？ 

   你说Redis是单线程的，那么比如遇到1000个并发请求它是怎么处理的，为什么单线程去处理并发量效率会高？  

  Redis 数据类型 为什么快 缓存穿透 缓存击穿 缓存雪崩  区别及解决方式 RDB和AOF区别及原理。。

作者：牛客401890116号
链接：https://www.nowcoder.com/discuss/628577?type=all&order=time&pos=&page=1&ncTraceId=&channel=-1&source_id=search_all_nctrack
来源：牛客网

select，poll，epoll 有什么区别

1 、什么是Redis? 

2 、Redis的数据类型？ 

3 、使用Redis有哪些好处？ 

4 、Redis相比Memcached有哪些优势？ 

5 、Memcache与Redis的区别都有哪些？ 

6 、Redis是单进程单线程的？ 

7 、一个字符串类型的值能存储最大容量是多少？ 

8 、Redis的持久化机制是什么？各自的优缺点？ 

9 、Redis常见性能问题和解决方案： 

10 、[redis]()过期键的删除策略？ 

11 、Redis的回收策略（淘汰策略）? 

12 、为什么edis需要把所有数据放到内存中？ 

13 、Redis的同步机制了解么？ 

14 、Pipeline有什么好处，为什么要用pipeline？ 

15 、是否使用过Redis集群，集群的原理是什么？ 

16 、Redis集群方案什么情况下会导致整个集群不可用？ 

17 、Redis支持的Java[客户端]()都有哪些？官方推荐用哪个？ 

18 、Jedis与Redisson对比有什么优缺点？ 

19 、Redis如何设置密码及验证密码？ 

20 、说说Redis哈希槽的概念？ 

21 、Redis集群的主从复制模型是怎样的？ 

22 、Redis集群会有写操作丢失吗？为什么？ 

23 、Redis集群之间是如何复制的？ 

24 、Redis集群最大节点个数是多少？ 

25 、Redis集群如何选择数据库？ 

26 、怎么测试Redis的连通性？ 

27 、怎么理解Redis事务？ 

28 、Redis事务相关的命令有哪几个？ 

29 、Rediskey的过期时间和永久有效分别怎么设置？ 

30 、Redis如何做内存优化？ 

31 、Redis回收进程如何工作的？ 

32 、都有哪些办法可以降低Redis的内存使用情况呢？ 

33 、Redis的内存用完了会发生什么？ 

34 、一个Redis实例最多能存放多少的keys？List、Set、SortedSet他们最多能存放多少元素？ 

35 、MySQL里有2000w数据，[redis]()中只存20w的数据，如何保证[redis]()中的数据都是热点数据？ 

36 、Redis最适合的场景？ 

37 、假如Redis里面有 1 亿个key，其中有10w个key是以某个固定的已知的前缀开头的，如果将它们全部找出来？ 

38 、如果有大量的key需要设置同一时间过期，一般需要注意什么？ 

39 、使用过Redis做异步队列么，你是怎么用的？ 

40 、使用过Redis分布式锁么，它是什么回事？



- 生产上你们你们的redis内存设置多少？
- 如何配置、修改redis的内存大小
- 如果内存满了你怎么办？
- redis清理内存的方式？定期删除和惰性删除了解过吗
- redis缓存淘汰策略
- redis的LRU了解过吗？可否手写一个LRU算法



作者：小谢backup
链接：https://www.nowcoder.com/discuss/596267?type=post&order=time&pos=&page=0&ncTraceId=&channel=-1&source_id=search_post_nctrack
来源：牛客网



## 1、Redis 的持久化机制有哪些？ 

 RDB（默认） 和 AOF 机制 



##  2、Redis主从复制用什么拓扑结构？ 

  **单向[链表]()结构**，即： 

  Master <- Slave1 <- Slave2 <- Slave3… 

 这样的结构方便解决单点故障问题，实现Slave对Master的替换。如果Master挂了，可以立刻启用Slave1做Master，其他不变。 



##  3、Redis的回收策略有哪些？ 

 \1. volatile-lru：从已设置过期时间的数据集中挑选最近最少使用的数据淘汰 
 \2. volatile-ttl：从已设置过期时间的数据集中挑选将要过期的数据淘汰 
 \3. volatile-random：从已设置过期时间的数据集中任意选择数据淘汰 
 \4. allkeys-lru：从数据集中挑选最近最少使用的数据淘汰 
 \5. allkeys-random：从数据集中任意选择数据淘汰 
 \6. no-enviction：禁止驱逐数据 



##  4、如何选择Redis的回收策略？ 

 如果数据呈现幂律分布，也就是一部分数据访问频率高，一部分数据访问频率低，则使用 lru； 
 如果数据呈现平等分布，也就是所有的数据访问频率都相同，则使用 random 



##  5、Redis的内存用完了会发生什么？ 

 如果达到设置的上限，Redis的写命令会返回错误信息（但是读命令还可以正常返回）。 
 解决思路：配置内存淘汰机制，当Redis达到内存上限时会冲刷掉旧的内容。 



##  6、Redis有哪些数据结构？ 

 String、Hash、List、Set、SortedSet。 

 如果你是Redis中高级用户，还需要加上下面几种数据结构：HyperLogLog、Geo、Pub/Sub。 
 如果你说还玩过Redis Module，像BloomFilter，RedisSearch，Redis-ML，面试官得眼睛就开始发亮了。 



##  7、Pipeline有什么好处？为什么要用pipeline？ 

 可以将多次IO往返的时间缩减为一次，前提是pipeline执行的指令之间没有因果相关性。 
 Pipeline可以提高Redis的QPS峰值。 



##  8、Redis 两种集群方案的区别？ 

 Redis Sentinal：着眼于高可用，在master宕机时会自动将slave提升为master，继续提供服务。 
 Redis Cluster：着眼于扩展性，在单个[redis]()内存不足时，使用Cluster进行分片存储。



##  9、Redis持久化数据和缓存怎么做扩容？ 

 如果Redis被当做缓存使用，使用一致性哈希实现动态扩容缩容。 
 如果Redis被当做一个持久化存储使用，必须使用固定的keys-to-nodes映射关系，节点的数量一旦确定不能变化。否则必须使用可以在运行时进行数据再平衡的一套系统，而当前只有Redis集群可以做到这样。 



##  10、如果有大量的key需要设置同一时间过期，一般需要注意什么？ 

  如果大量的key过期时间设置的过于集中，到过期的那个时间点，[redis]()可能会出现短暂的卡顿现象。 

  一般需要在时间上加一个随机值，使得过期时间分散一些。





作者：🐡赵小胖
链接：https://www.nowcoder.com/discuss/314937?type=post&order=time&pos=&page=1&ncTraceId=&channel=-1&source_id=search_post_nctrack
来源：牛客网



我将 Redis 面试中常见的题目划分为如下几大部分： 

1. Redis 的概念理解 
2. Redis 基本数据结构详解 
3. Redis 高并发问题策略 
4. Redis 集群结构以及设计理念 
5. Redis 持久化机制 
6. Redis 应用场景设计 

部分涉及到的题目如下：

- 什么是 Redis? 
- Redis 的特点有哪些？ 
- Redis 支持的数据类型 
- 为什么 Redis 需要把所有数据放到内存中？ 
- Redis 适用场景有哪些？ 
- Redis常用的业务场景有哪些？ 
- Mem*** 与 Redis 的区别都有哪些？ 
- Redis 相比 mem***d 有哪些优势？ 
- Redis常用的命令有哪些？ 
- Redis 是单线程的吗？ 
- Redis 为什么设计成单线程的？ 
- 一个字符串类型的值能存储最大容量是多少？ 
- Redis各个数据类型最大存储量分别是多少? 
- Redis 持久化机制有哪些？ 区别是什么？ 
- 请介绍一下 RDB, AOF两种持久化机制的优缺点？ 
- 什么是缓存穿透？怎么解决？ 
- 什么是缓存雪崩？ 怎么解决？ 
- Redis支持的额Java[客户端]()有哪些？ 简单说明一下特点。 
- 缓存的更新策略有几种？分别有什么注意事项？ 
- 什么是分布式锁？有什么作用？ 
- 分布式锁可以通过什么来实现？ 
- 介绍一下分布式锁实现需要注意的事项？ 
- Redis怎么实现分布式锁？ 
- 常见的淘汰[算法]()有哪些？ 
- Redis 淘汰策略有哪些？ 
- Redis 缓存失效策略有哪些？ 
- Redis 的持久化机制有几种方式？ 
- 请介绍一下持久化机制 RDB, AOF的优缺点分别是什么？ 
- Redis 通讯协议是什么？有什么特点？ 
- 请介绍一下 Redis 的数据类型 SortedSet(zset) 以及底层实现机制？ 
- Redis 集群最大节点个数是多少？ 
- Redis 集群的主从复制模型是怎样的？ 
- Redis 如何做内存优化？ 
- Redis 事务相关命令有哪些？ 
- 什么是 Redis 事务？原理是什么？ 
- Redis 事务的注意点有哪些？ 
- Redis 为什么不支持回滚？ 
- 请介绍一下 Redis 集群实现方案 
- 请介绍一下 Redis 常见的业务使用场景？ 
- Redis 集群会有写操作丢失吗？为什么？ 
- 请介绍一下 Redis 的 Pipeline (管道)，以及使用场景 
- 请说明一下 Redis 的批量命令与 Pipeline 有什么不同？ 
- Redis 慢查询是什么？通过什么配置？ 
- Redis 的慢查询修复经验有哪些？ 怎么修复的？ 
- 请介绍一下 Redis 的发布订阅功能 
- 请介绍几个可能导致 Redis 阻塞的原因 
- 怎么去发现 Redis 阻塞异常情况？ 
- 如何发现大对象 
- Redis 的内存消耗分类有哪些？内存统计使用什么命令？ 
- 简单介绍一下 Redis 的内存管理方式有哪些？ 
- 如何设置 Redis 的内存上限？有什么作用？ 
- 什么是 bigkey？ 有什么影响？ 
- 怎么发现bigkey? 
- 请简单描述一下 Jedis 的基本使用方法？ 
- Jedis连接池链接方法有什么优点？ 
- 冷热数据表示什么意思？ 
- 缓存命中率表示什么？ 
- 怎么提高缓存命中率？ 
- 如何优化 Redis 服务的性能？ 
- 如何实现本地缓存？请描述一下你知道的方式 
- 请介绍一下 Spring 注解缓存 
- 如果 AOF 文件的数据出现异常， Redis服务怎么处理？ 
- Redis 的主从复制模式有什么优缺点? 
- Redis sentinel (哨兵) 模式优缺点有哪些？ 
- Redis 集群架构模式有哪几种？ 
- 如何设置 Redis 的最大连接数？查看Redis的最大连接数？查看Redis的当前连接数？ 
- Redis 的[链表]()数据结构的特征有哪些？ 
- 请介绍一下 Redis 的 String 类型底层实现？ 
- Redis 的 String 类型使用 SSD 方式实现的好处？ 
- 设计一下在交易网站首页展示当天最热门售卖商品的前五十名商品列表?





作者：Tronhon
链接：https://www.nowcoder.com/discuss/412166?type=post&order=time&pos=&page=1&ncTraceId=&channel=-1&source_id=search_post_nctrack
来源：牛客网

**1.[redis]() 缓存穿透，缓存雪崩，缓存击穿**

   **（1）缓存穿透**  

   缓存穿透，是指查询一个数据库一定不存在的数据。正常的使用缓存流程大致是，数据查询先进行缓存查询，如果key不存在或者key已经过期，再对数据库进行查询，并把查询到的对象，放进缓存。如果数据库查询对象为空，则不放进缓存。  

   **（2）缓存雪崩**  

   缓存雪崩，是指在某一个时间段，缓存集中过期失效。  

   产生雪崩的原因之一，比如在写本文的时候，马上就要到双十二零点，很快就会迎来一波抢购，这波商品时间比较集中的放入了缓存，假设缓存一个小时。那么到了凌晨一点钟的时候，这批商品的缓存就都过期了。而对这批商品的访问查询，都落到了数据库上，对于数据库而言，就会产生周期性的压力波峰。  

   **（3）缓存击穿**  

   缓存击穿，是指一个key非常热点，在不停的扛着大并发，大并发集中对这一个点进行访问，当这个key在失效的瞬间，持续的大并发就穿破缓存，直接请求数据库，就像在一个屏障上凿开了一个洞。  

  **
**  

  **2.Redis 和 MemeCache 有什么区别？** 

 1)、存储方式 Memecache把数据全部存在内存之中，断电后会挂掉，数据不能超过内存大小。 Redis有部份存在硬盘上，这样能保证数据的持久性。 2)、数据支持类型 Memcache对数据类型支持相对简单。 Redis有复杂的数据类型。 3)、使用底层模型不同 它们之间底层实现方式 以及与[客户端]()之间通信的应用协议不一样。 Redis直接自己构建了VM 机制 ，因为一般的系统调用系统函数的话，会浪费一定的时间去移动和请求。

**3.Redis 为什么是单线程的？**
 因为[redis]()利用队列技术将并发访问变为串行访问，消除了传统数据库串行控制的开销。

**4.什么是缓存穿透？怎么解决？**

  传送门：  https://www.cnblogs.com/jamaler/p/12453800.html 

  


**5.Redis 支持的数据类型有哪些？**
 1.[redis]()的5种数据类型： string 字符串（可以为整形、浮点型和字符串，统称为元素） list 列表（实现队列,元素不唯一，先入先出原则） set 集合（各不相同的元素）
 hash hash散列值（hash的key必须是唯一的） sort set 有序集合 

**6.Redis 支持的 Java [客户端]()都有哪些？**
 Redis Desktop Manager 、Redis Client、Redis Studio、jedis 

**7.Jedis 和 Redisson 有哪些区别？**
 Jedis是Redis的java实现[客户端]()，其API提供了比较全面的Redis命令的支持；Redisson实现了分布式和可扩展的Java数据结构，和Jedis相比，功能较为简单，不支持字符串操作，不支持[排序]()、事务‘管道、分区等Redis特性。Redisson的宗旨是促进使用者对Redis的关注分离，从而让使用者能够将精力更集中地放在处理业务逻辑上。

**8.怎么保证缓存和数据库数据的一致性？**
 由于数据库层面的读写并发，引发的数据库与缓存数据不一致的问题（本质是后发生的读请求先返回了），可能通过两个小的改动解决：（1）修改服务Service连接池，id取模选取服务连接，能够保证同一个数据的读写都落在同一个后端服务上（2）修改数据库DB连接池，id取模选取DB连接，能够保证同一个数据的读写在数据库层面是串行的 

**9.Redis 持久化有几种方式？**
 Redis是一种高级key-value数据库。它跟memcached类似，不过数据可以持久化，而且支持的数据类型很丰富。有字符串，[链表]()，集 合和有序集合。支持在服务器端计算集合的并，交和补集(difference)等，还支持多种[排序]()功能。所以Redis也可以被看成是一个数据结构服务 器。Redis的所有数据都是保存在内存中，然后不定期的通过异步方式保存到磁盘上(这称为“半持久化模式”)；也可以把每一次数据变化都写入到一个append only file(aof)里面(这称为“全持久化模式”)。

**10.Redis 怎么实现分布式锁？**
 Redis的实现主要基于setnx 和给予一个超时时间（防止释放锁失败）。 多个尝试获取锁的[客户端]()使用同一个key做为目标数据的唯一键，value为锁的期望超时时间点； 首先进行一次setnx命令，尝试获取锁，如果获取成功，则设置锁的最终超时时间（以防在当前进程获取锁后奔溃导致锁无法释放）这里利用 Redis set key 时的一个 NX 参数可以保证在这个 key 不存在的情况下写入成功。并且再加上 EX 参数可以让该 key 在超时之后自动删除。一定不要把两个命令(NX EX)分开执行，如果在 NX 之后程序出现问题就有可能产生死锁。非阻塞锁、阻塞锁、解锁、为了更好的健壮性，将该操作封装为一个lua脚本，这样即可保证其原子性 [redis]()分布式的实现原理：
 1、通过setNX操作，如果存在key，不操作；不存在，才会set值，保证锁的互斥性2、value设置锁的到期时间，当锁超时时，进行getAndSet操作，先get旧值，再set新值，避免发生死锁。这里也可以通过设置key的有效期来避免死锁，但是setNx和exprise（设置有效期）操作非原子性，可能发生锁没有设置有效时间的问题，从而发生死锁。实现：spring boot 通过jdeis连接redsi集群。 
 加锁过程： 
 1、获得当前系统时间，计算锁的到期时间 
 2、setNx操作，加锁 
 3、如果，加锁成功，设置锁的到期时间，返回true；取锁失败，取出当前锁的value(到期时间) 
 4、如果value不为空而且小于当前系统时间，进行getAndSet操作，重新设置value，并取出旧value；否则，等待间隔时间后，重复步骤2； 
 5、如果步骤3和4取出的value一样，加锁成功，设置锁的到期时间，返回true；否则，别人加锁成功，恢复锁的value，等待间隔时间后，重复步骤2。 

**11.Redis 分布式锁有什么缺陷？**
 在工作和网络上看到过各个版本的Redis分布式锁实现，每种实现都有一些不严谨的地方，甚至有可能是错误的实现，包括在代码中，如果不能正确的使用分布式锁，可能造成严重的生产环境故障。 

  [redis]()分布式锁的缺陷 

  传送门： https://blog.csdn.net/matt8/article/details/64442064 



 

**12.Redis 如何做内存优化？**
 1，使用对象共享池优化小整数对象。 
 2，数据优先使用整数，比字符串更节省空间。3，操作优化。尽量避免字符串的追加操作，因为字符串存在预分配机制。追加操作后字符串对象会分配一倍的容量作为于预留空间。4，编码优化。list，hash，set，zset尽可能使用ziplist编码。好处是内存下降，坏处是操作变慢。一般大小不超过1000。 

**13.Redis 淘汰策略有哪些？**
 [redis]() 提供 6种数据淘汰策略：
 volatile-lru：从已设置过期时间的数据集（server.db[i].expires）中挑选最近最少使用的数据淘汰 
 volatile-ttl：从已设置过期时间的数据集（server.db[i].expires）中挑选将要过期的数据淘汰 
 volatile-random：从已设置过期时间的数据集（server.db[i].expires）中任意选择数据淘汰 
 allkeys-lru：从数据集（server.db[i].dict）中挑选最近最少使用的数据淘汰 
 allkeys-random：从数据集（server.db[i].dict）中任意选择数据淘汰 
 no-enviction（驱逐）：禁止驱逐数据 

**14.Redis 常见的性能问题有哪些？**

  该如何解决？1.master写内存快照，seve命令调度rdbsave函数，会阻塞主线程的工程，当快照比较大的时候对性能的影响是非常大的，会间断性暂停服务 。所以master最好不要写内存快照。2.master AOF持久化，如果不重写AOF文件，这个持久化方式对性能的影响是最小的，但是AOF文件会不断增大，AOF文件过大会影响master重启时的恢复速度。master最好不要做任何持久化工作，包括内存快照和AOF日志文件，特别是不要启用内存快照做持久化，如果数据比较关键，某个slave开启AOF备份数据，策略每秒为同步一次。3.master调用BGREWRITEAOF重写AOF文件，AOF在重写的时候会占大量的CPU和内存资源，导致服务load过高，出现短暂的服务暂停现象。4.[redis]()主从复制的性能问题，为了主从复制的速度和连接的稳定性，slave和master最好在同一个局域网内。





作者：牛客401890116号
链接：https://www.nowcoder.com/discuss/578480?type=post&order=time&pos=&page=0&ncTraceId=&channel=-1&source_id=search_post_nctrack
来源：牛客网



- 什么是Redis 
-  Redis有哪些优缺点 
-  为什么要用 Redis /为什么要用缓存 
-  为什么要用 Redis 而不用 map/guava 做缓存? 
-  Redis为什么这么快 

  数据类型 

-  Redis有哪些数据类型 
-  Redis的应用场景 

  持久化 

-  什么是Redis持久化？ 
-  Redis 的持久化机制是什么？各自的优缺点？ 
-  如何选择合适的持久化方式 
-  Redis持久化数据和缓存怎么做扩容？ 

  过期键的删除策略 

-  Redis的过期键的删除策略 
-  Redis key的过期时间和永久有效分别怎么设置？ 
-  我们知道通过expire来设置key 的过期时间，那么对过期的数据怎么处理呢? 

  内存相关 

-  MySQL里有2000w数据，[redis]()中只存20w的数据，如何保证[redis]()中的数据都是热点数据 
-  Redis的内存淘汰策略有哪些 
-  Redis主要消耗什么物理资源？ 
-  Redis的内存用完了会发生什么？ 
-  Redis如何做内存优化？ 

  线程模型 

-  Redis线程模型 

  事务 

-  什么是事务？ 
-  Redis事务的概念 
-  Redis事务的三个阶段 
-  Redis事务相关命令 
-  事务管理（ACID）概述 
-  Redis事务支持隔离性吗 
-  Redis事务保证原子性吗，支持回滚吗 
-  Redis事务其他实现 

  集群方案 

-  哨兵模式 
-  官方Redis Cluster 方案(服务端路由查询) 
-  基于[客户端]()分配 
-  基于代理服务器分片 
-  Redis 主从架构 
-  Redis集群的主从复制模型是怎样的？ 
-  生产环境中的 [redis]() 是怎么部署的？ 
-  说说Redis哈希槽的概念？ 
-  Redis集群会有写操作丢失吗？为什么？ 
-  Redis集群之间是如何复制的？ 
-  Redis集群最大节点个数是多少？ 
-  Redis集群如何选择数据库？ 

  分区 

-  Redis是单线程的，如何提高多核CPU的利用率？ 
-  为什么要做Redis分区？ 
-  你知道有哪些Redis分区实现方案？ 
-  Redis分区有什么缺点？ 

  分布式问题 

-  Redis实现分布式锁 
-  如何解决 Redis 的并发竞争 Key 问题 
-  分布式Redis是前期做还是后期规模上来了再做好？为什么？ 
-  什么是 RedLock 

  缓存异常 

-  缓存雪崩 
-  缓存穿透 
-  缓存击穿 
-  缓存预热 
-  缓存降级 
-  热点数据和冷数据 
-  缓存热点key 

  常用工具 

-  Redis支持的Java[客户端]()都有哪些？官方推荐用哪个？ 
-  Redis和Redisson有什么关系？ 
-  Jedis与Redisson对比有什么优缺点？ 

  其他问题 

-  Redis与Memcached的区别 
-  如何保证缓存与数据库双写时的数据一致性？ 
-  Redis常见性能问题和解决方案？ 
-  Redis官方为什么不提供Windows版本？ 
-  一个字符串类型的值能存储最大容量是多少？ 
-  Redis如何做大量数据插入？ 
-  假如Redis里面有1亿个key，其中有10w个key是以某个固定的已知的前缀开头的，如果将它们全部找出来？ 
-  使用Redis做过异步队列吗，是如何实现的 
-  Redis如何实现延时队列 
-  Redis回收进程如何工作的？ 
-  Redis回收使用的是什么[算法]()？

