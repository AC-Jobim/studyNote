# docker中部署并启动redis

**使用`docker pull 镜像名称`拉取镜像**

```javascript
docker pull redis //不指定版本号，默认拉取最新。
docker pull redis:6.0.8
```

**拉取完镜像后，使用docker images查看已经拉取的镜像**

```javascript
docker images
```

**在运行之前对redis进行一些配置**

* redis.conf的配置文件可以在 http://download.redis.io/redis-stable/redis.conf 上下载

  ```
  使用 mkdir /usr/local/docker 在宿主机上创建存放docker目录
  vi /usr/local/docker/redis.conf 在docker中创建redis的配置文件redis.conf
  将下载好的redis.conf文件替换或将内容复制到自己创建的配置文件中
  
  然后修改配置
  
  bind 127.0.0.1  //127.0.0.1 限制只能本机访问 将其改为0.0.0.0
  
  protected-mode no # 默认yes，开启保护模式，限制为本地访问
  
  daemonize no 默认no，改为yes意为以守护进程方式启动，yes会使配置文件方式启动redis失败（一开启就退出）
  ```

**运行指定镜像**

```javascript
docker run -itd -p 16379:6379 -v /mydocker/myredis/data:/data -v /mydocker/myredis/conf/redis.conf:/etc/redis/redis.conf  redis:latest redis-server /etc/redis/redis.conf --requirepass "password"

-d 以守护线程的方式运行（后台运行）
-i 以交互模式运行容器
-t 为容器重新分配一个伪输入终端 ，通常与 -i 同时使用；
-p 映射容器服务的 6379 端口到宿主机的 6379 端口。外部可以直接通过宿主机ip:6379 访问到 Redis 的服务。

-v /mydocker/myredis/data:/data  //把redis持久化的数据挂载到宿主机内，做数据备份

-v /mydocker/myredis/conf/redis.conf:/etc/redis/redis.conf //把宿主机配置好的redis.conf挂载到容器内的指定位置

redis-server /etc/redis/redis.conf  //使redis按照redis.conf的配置启动

--requirepass "password"

--appendonly yes //redis启动后数据持久化
```

![image-20220409134903387](https://gitee.com/jobim/blogimage/raw/master/img/202204091349475.png)



**连接redis客户端：**

```dockerfile
docker exec -it 1b52df719983 redis-cli
```

![image-20220423182048325](https://blog.zhaobincode.cn/blogimages/202204231820369.png)





# ZUNIONSTORE 命令和 ZINTERSTORE 命令

> 参考博客：[Redis 的 ZUNIONSTORE 命令和 ZINTERSTORE 命令的另一种用法](https://blog.huangz.me/diary/2013/another-usage-of-zunionstore-and-zinterstore.html)



**`ZUNIONSTORE`和 `ZINTERSTORE` 是用于有序集（sorted set）的聚合命令。**

- **`ZUNIONSTORE` 用于计算给定的一个或多个有序集的并集。**
- **而 `ZINTERSTORE` 则用于计算给定的一个或多个有序集的交集。**

* **除此之外，这两个命令除了可以用于有序集之外，还可以用于集合（set）。**



**ZINTERSTORE的使用：**

* **命令计算给定的一个或多个有序集的交集，其中给定 key 的数量必须以 numkeys 参数指定，并将该交集(结果集)储存到 destination 。默认情况下，结果集中某个成员的分数值是所有给定集下该成员分数值之和**

  ```
  ZINTERSTORE destination numkeys key [key ...] [WEIGHTS weight] [AGGREGATE SUM|MIN|MAX]
  ```

* **时间复杂度：** O（N * K）+ O（M * log（M））最坏的情况，N 是最小的输入排序集合，K 是输入排序集合的数量，M 是结果排序集合中的元素数量。

**使用示例：**

```
$ redis-cli -h 127.0.0.1 -p 6379
127.0.0.1:6379> zadd z1 2 cat 3 dog
(integer) 2
127.0.0.1:6379> zadd z2 1 cat 4 goose
(integer) 2
// 返回交集里成员的数量。新集合中成员分数值默认是给定集合下成员分数值之和
127.0.0.1:6379> ZINTERSTORE z3 2 z1 z2
(integer) 1
127.0.0.1:6379> ZRANGE z3 0 -1 withscores
1) "cat"
2) "3"
// 使用权重时，也返回交集里成员的数量。默认依然是分数值相加，但是要先乘以权重值，此处即是2*1+1*2=4
127.0.0.1:6379> ZINTERSTORE z4 2 z1 z2 WEIGHTS 1 2
(integer) 1
127.0.0.1:6379> ZRANGE z4 0 -1 withscores
1) "cat"
2) "4"
// AGGREGATE包含SUM/MIN/MAX。ZINTERSTORE默认使用SUM。
// MIN即为取给定集合里最小的分数值，MAX则反之
127.0.0.1:6379> ZINTERSTORE z5 2 z1 z2 AGGREGATE MIN
(integer) 1
127.0.0.1:6379> ZRANGE z5 0 -1 withscores
1) "cat"
2) "1"
127.0.0.1:6379> ZINTERSTORE z6 2 z1 z2 AGGREGATE MAX
(integer) 1
127.0.0.1:6379> ZRANGE z6 0 -1 withscores
1) "cat"
2) "2"
//AGGREGATE和权重WEIGHTS一起使用时，取经过权重计算后的结果
//此处即2*3>1*4
127.0.0.1:6379> ZINTERSTORE z7 2 z1 z2 WEIGHTS 3 4 AGGREGATE MAX
(integer) 1
127.0.0.1:6379> ZRANGE z7 0 -1 withscores
1) "cat"
2) "6"
```

**使用zset和set进行操作，其中set集合中的value默认为1**

```
127.0.0.1:6379[1]> sadd s1 cat dog pig
(integer) 3
127.0.0.1:6379[1]> zadd z1 2 cat 3 dog
(integer) 2
// set集合中的value默认为1
127.0.0.1:6379[1]> ZINTERSTORE z3 2 z1 s1 AGGREGATE MIN
(integer) 2
127.0.0.1:6379[1]> ZRANGE z3 0 -1 withscores
1) "cat"
2) "1"
3) "dog"
4) "1"
127.0.0.1:6379[1]> ZINTERSTORE z4 2 z1 s1
(integer) 2
127.0.0.1:6379[1]> ZRANGE z4 0 -1 withscores
1) "cat"
2) "3"
3) "dog"
4) "4"
127.0.0.1:6379[1]> 
```





