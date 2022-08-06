# 四、Golang操作Redis（redigo）

Golang 操作 Redis 的三方库主要有 **go-redis** 和 **redigo**，这里主要介绍 redigo，它也是 Golang 官方推荐使用的 Redis 客户端。

go-redis 三方库为我们封装了很多函数来执行 Redis 命令，而 redigo 三方库只有一个 Do 函数执行 Redis 命令，更接近使用 redis-cli 操作 Redis。

> go-redis的使用参考：[Go操作Redis实战](https://www.cnblogs.com/itbsl/p/14198111.html)

## 4.1 安装redigo库

* **使用 go get 命令安装 redigo：（在GOPATH路径下执行）**

  ```
  go get github.com/gomodule/redigo/redis
  ```

  安装成功后，可以看到如下包

  ![image-20220316150444343](https://gitee.com/jobim/blogimage/raw/master/img/20220316150444.png) 

## 4.2 Redis的连接和操作

<font color='blue' size='4'>**Redis连接和关闭：**</font>

* **使用`Dail()`方法来连接服务：**

  ```go
  func Dial(network, address string, options ...DialOption) (Conn, error) {}
  ```

  `network`表示网络类型，`address`是服务地址，`options`是一些可选的选项，如果连接成功将返回一个`redis.Conn`对象，在连接使用完毕后，使用`Close()`关闭。



<font color='blue' size='4'>**执行命令：**</font>

* **执行redis命令使用`Do()`方法：**

  ```go
  Do(commandName string, args ...interface{}) (reply interface{}, err error)
  ```

  `commandName`是redis命令，后面接参数

  ```go
  package main
  import (
  	"fmt"
  	"github.com/gomodule/redigo/redis" //引入redis包
  )
  
  func main() {
  	//通过go 向redis 写入数据和读取数据
  	//1. 链接到redis
  	conn, err := redis.Dial("tcp", "127.0.0.1:6379")
  	if err != nil {
  		fmt.Println("redis.Dial err=", err)
  		return 
  	}
  	defer conn.Close() //关闭..
  
  	//2. 通过go 向redis写入数据 string [key-val]
  	_, err = conn.Do("Set", "name", "tomjerry猫猫")
  	if err != nil {
  		fmt.Println("set  err=", err)
  		return 
  	}
  
  	//3. 通过go 向redis读取数据 string [key-val]
  
  	r, err := redis.String(conn.Do("Get", "name"))
  	if err != nil {
  		fmt.Println("set  err=", err)
  		return 
  	}
  
  	//因为返回 r是 interface{}
  	//因为 name 对应的值是string ,因此我们需要转换
  	//nameString := r.(string)
  
  	fmt.Println("操作ok ", r)
  }
  ```

**操作string类型：**

```go
//string类型数据命令
//redis命令：set key val
func set(key,val string){
	_,err := c.Do("set",key,val)

	if err != nil {
		log.Fatal(err)
	}
}
//redis命令：mset key1 val1 key2 val2 key3 val3 ...
func mset(key1,val1,key2,val2,key3,val3 string){
	_,err := c.Do("mset",key1,val1,key2,val2,key3,val3)

	if err != nil {
		log.Fatal(err)
	}
}
//redis命令：get key
func get(key string){
	val,err := redis.String(c.Do("get",key))

	if err != nil {
		log.Fatal(err)
	}

	fmt.Println(val)
}
//redis命令：mget key1 key2 key3 ...
func mget(key1,key2,key3 string){
	vals,err := redis.Values(c.Do("mget",key1,key2,key3))

	if err != nil {
		log.Fatal(err)
	}

	for k,v := range vals {
		fmt.Printf("k = %v  v = %s\n",k,v)
	}
}
//redis命令：del key1 key2 ...
func del(key1,key2,key3 string){
	_,err := c.Do("del",key1,key2,key3)

	if err != nil {
		log.Fatal(err)
	}
}
//redis命令：getrange key start end
func getrange(key string,start,end int){
	s,err := redis.String(c.Do("getrange",key,start,end))

	if err != nil{
		log.Fatal(err)
	}

	fmt.Println(s)
}
//redis命令：exists key
func exists(key string){
	isExists,err := redis.Bool(c.Do("exists",key))

	if err != nil {
		log.Fatal(err)
	}

	fmt.Println(isExists)
}
//redis命令：setex key 10 val
func setex(key,val string,expire int){
	_,err := c.Do("setex",key,expire,val)

	if err != nil {
		log.Fatal(err)
	}
}
```

**操作Hash类型：**

```go
//hash类型数据命令
//redis命令：hset hashTable key val
func hset(hashTable,key,val string){
	_,err := c.Do("hset",hashTable,key,val)

	if err != nil {
		log.Fatal(err)
	}
}
//redis命令：hmset hashTable key1 val1 key2 val2 key3 val3 ...
func hmset(hashTable,key1,val1,key2,val2,key3,val3 string){
	_,err := c.Do("hmset",hashTable,key1,val1,key2,val2,key3,val3)

	if err != nil {
		log.Fatal(err)
	}
}
//redis命令：hget hashTable key
func hget(hashTable,key string){
	val,err := redis.String(c.Do("hget",hashTable,key))

	if err != nil {
		log.Fatal(err)
	}

	fmt.Println(val)
}
//redis命令：hmget hashTable key1 key2 key3 ...
func hmget(hashTable,key1,key2,key3 string){
	vals,err := redis.Values(c.Do("hmget",hashTable,key1,key2,key3))

	if err != nil {
		log.Fatal(err)
	}

	for k,v := range vals {
		fmt.Printf("k = %v  v = %s\n",k,v)
	}
}
//redis命令：hdel hashTable key1 key2 key3
func hdel(hashTable,key1,key2,key3 string){
	//批量删除一个或若干个hash表中的键值对，如果是一次性删除多个，只要至少删除掉一个即返回true
	isDelSuccessful,err := redis.Bool(c.Do("hdel",hashTable,key1,key2,key3))

	if err != nil {
		log.Fatal(err)
	}

	fmt.Println(isDelSuccessful)
}
//redis命令：hgetall hashTable
func hgetall(hashTable string){
	vals,err := redis.Values(c.Do("hgetall",hashTable))

	if err != nil {
		log.Fatal(err)
	}

	for k,v := range vals {
		fmt.Printf("k = %v  v = %s\n",k,v)
	}
}
//redis命令：hkeys hashTable
func hkeys(hashTable string){
	keys,err := redis.Values(c.Do("hkeys",hashTable))

	if err != nil {
		log.Fatal(err)
	}

	for k,v := range keys {
		fmt.Printf("k = %v v = %s\n",k,v)
	}
}
//redis命令：hvals hashTable
func hvals(hashTable string){
	vals,err := redis.Values(c.Do("hvals",hashTable))

	if err != nil {
		log.Fatal(err)
	}

	for k,v := range vals {
		fmt.Printf("k = %v v = %s\n",k,v)
	}
}
//redis命令：hlen hashTable
func hlen(hashTable string){
	len,err := redis.Int(c.Do("hlen",hashTable))

	if err != nil {
		log.Fatal(err)
	}

	fmt.Println(len)
}
//redis命令：hexists hashTable
func hexists(hashTable,key string){
	isExists,err := redis.Bool(c.Do("hexists",hashTable,key))

	if err != nil {
		log.Fatal(err)
	}

	fmt.Println(isExists)
}
```

**操作list类型：**

```go
//list类型数据命令
//redis命令：lpush list val1 val2 val3 ...
func lpush(list,val1,val2,val3 string){
	_,err := c.Do("lpush",list,val1,val2,val3)

	if err != nil {
		log.Fatal(err)
	}
}
//redis命令：rpush list val1 val2 val3 ...
func rpush(list,val1,val2,val3 string){
	_,err := c.Do("rpush",list,val1,val2,val3)

	if err != nil {
		log.Fatal(err)
	}
}
//redis命令：lpop list
func lpop(list string){
	v,err := redis.String(c.Do("lpop",list))

	if err != nil {
		log.Fatal(err)
	}

	fmt.Println("删除列表首个元素并返回：" + v)
}
//redis命令：rpop list
func rpop(list string){
	v,err := redis.String(c.Do("rpop",list))

	if err != nil {
		log.Fatal(err)
	}

	fmt.Println("删除列表末尾元素并返回：" + v)
}
//redis命令：lrem list count val
func lrem(list,val string,count int){
	n,err := redis.Int(c.Do("lrem",list,count,val))

	if err != nil {
		log.Fatal(err)
	}

	fmt.Println(n)
}
//redis命令：lset list index val
func lset(list,val string,index int){
	result,err := c.Do("lset",list,index,val)

	if err != nil {
		log.Fatal(err)
	}
	//成功修改则返回OK
	fmt.Println(result)
}
//redis命令：lindex list index
func lindex(list string,index int){
	v,err := redis.String(c.Do("lindex",list,index))

	if err != nil {
		log.Fatal(err)
	}

	fmt.Printf("列表%s中索引为%d对应的字符串为%s\n",list,index,v)
}
//redis命令：lrange list start end
func lrange(list string,start,end int){
	vals,err := redis.Values(c.Do("lrange",list,start,end))

	if err != nil {
		log.Fatal(err)
	}

	for k,v := range vals {
		fmt.Printf("k = %v v = %s\n",k,v)
	}
}
//redis命令：llen list
func llen(list string){
	len,err := redis.Int(c.Do("llen",list))

	if err != nil {
		log.Fatal(err)
	}

	fmt.Println(len)
}
//redis命令：ltrim list start end
func ltrim(list string,start,end int){
	//这里加不加redis.String都可以，如果操作成功返回的都是字符串OK
	result,err := redis.String(c.Do("ltrim",list,start,end))

	if err != nil {
		log.Fatal(err)
	}

	fmt.Println(result)
}
```

**操作无序集合set类型：**

```go
//无序集合set数据类型命令
//redis命令：sadd myset val1 val2 val3 ...
func sadd(myset,val1,val2,val3 string){
	_,err := c.Do("sadd",myset,val1,val2,val3)

	if err != nil {
		log.Fatal(err)
	}
}
//redis命令：srem myset val
func srem(myset,val string){
	//c.Do删除成功返回1，然后用redis.Bool转换成bool值true
	isDel,err := redis.Bool(c.Do("srem",myset,val))

	if err != nil {
		log.Fatal(err)
	}

	fmt.Println(isDel)
}
//redis命令：spop myset
func spop(myset string){
	val,err := redis.String(c.Do("spop",myset))

	if err != nil {
		log.Fatal(err)
	}

	fmt.Println("随机删除一个元素并返回：" + val)
}
//redis命令：smembers myset
func smembers(myset string){
	vals,err := redis.Values(c.Do("smembers",myset))

	if err != nil {
		log.Fatal(err)
	}

	for k,v := range vals {
		fmt.Printf("k = %v v = %s\n",k,v)
	}
}
//redis命令：scard myset
func scard(myset string){
	len,err := redis.Int(c.Do("scard",myset))

	if err != nil {
		log.Fatal(err)
	}

	fmt.Printf("集合长度为：%d\n",len)
}
//redis命令：sismember myset val
func sismember(myset,val string){
	isMember,err := redis.Bool(c.Do("sismember",myset,val))

	if err != nil {
		log.Fatal(err)
	}

	fmt.Println(isMember)
}
//redis命令：srandmember myset count
func srandmember(myset string,count int){
	vals,err := redis.Values(c.Do("srandmember",myset,count))

	if err != nil {
		log.Fatal(err)
	}

	for k,v := range vals {
		fmt.Printf("k = %v v = %s\n",k,v)
	}
}
//redis命令：smove myset myset2 val
func smove(myset,myset2,val string){
	isMoveSuccessful,err := redis.Bool(c.Do("smove",myset,myset2,val))

	if err != nil {
		log.Fatal(err)
	}

	fmt.Printf("从集合%s中移动元素%s到集合%s，移动结果：" + strconv.FormatBool(isMoveSuccessful) + "\n",myset,val,myset2)
}
//redis命令：sunion myset myset2 ...
func sunion(myset,myset2 string){
	vals,err := redis.Values(c.Do("sunion",myset,myset2))

	if err != nil {
		log.Fatal(err)
	}

	for k,v := range vals {
		fmt.Printf("k = %v v = %s\n",k,v)
	}
}
//redis命令：sunionstore myset3 myset myset2 ...
func sunionstore(myset,myset2,myset3 string){
	//将集合myset、myset2取并集后存入myset3集合
	len,err := redis.Int(c.Do("sunionstore",myset3,myset,myset2))

	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf("集合%s与集合%s取交集后得到的集合%s有%d个元素\n",myset,myset2,myset3,len)

	//打印集合myset3验证结果
	smembers("myset3")
}
//redis命令：sinter myset myset2 ...
func sinter(myset,myset2 string){
	vals,err := redis.Values(c.Do("sinter",myset,myset2))

	if err != nil {
		log.Fatal(err)
	}

	for k,v := range vals {
		fmt.Printf("k = %v v = %s\n",k,v)
	}
}
//redis命令：sinterstore myset3 myset myset2 ...
func sinterstore(myset,myset2,myset3 string){
	len,err := redis.Int(c.Do("sinterstore",myset3,myset,myset2))

	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf("集合%s与集合%s取交集后得到的集合%s有%d个元素\n",myset,myset2,myset3,len)

	//打印集合myset3验证结果
	smembers("myset3")
}
//redis命令：sdiff myset myset2 ...
func sdiff(myset,myset2 string){
	vals,err := redis.Values(c.Do("sdiff",myset,myset2))

	if err != nil {
		log.Fatal(err)
	}

	for k,v := range vals {
		fmt.Printf("k = %v v = %s\n",k,v)
	}
}
//redis命令：sdiffstore myset3 myset myset2 ...
func sdiffstore(myset,myset2,myset3 string){
	len,err := redis.Int(c.Do("sdiffstore",myset3,myset,myset2))

	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf("集合%s与集合%s取交集后得到的集合%s有%d个元素\n",myset,myset2,myset3,len)

	//打印集合myset3验证结果
	smembers("myset3")

}
```

**操作有序集合zset类型：**

```go
//有序集合zset数据类型命令
//redis命令：zadd myzset val1 val2 val3 ...
func zadd(myzset,val1,val2,val3 string,score1,score2,score3 float64){
	_,err := c.Do("zadd",myzset,score1,val1,score2,val2,score3,val3)

	if err != nil {
		log.Fatal(err)
	}
}
//redis命令：zrem myzset val1 val2 val3 ...
func zrem(myzset,val1,val2,val3 string){
	len,err := redis.Int(c.Do("zrem",myzset,val1,val2,val3))

	if err != nil {
		log.Fatal(err)
	}

	fmt.Printf("本次有%d个元素被成功删除\n",len)
}
//redis命令：zrange myzset start end [withscores]
func zrange(myzset string,start,end,flag int){
	var vals []interface{}
	var err error

	if flag == 0 {
		//不加withscores
		vals,err = redis.Values(c.Do("zrange","myzset",start,end))
	}else if flag == 1{
		//加withscores
		vals,err = redis.Values(c.Do("zrange","myzset",start,end,"withscores"))
	}

	if err != nil {
		log.Fatal(err)
	}

	for k,v := range vals {
		fmt.Printf("k = %v v = %s\n",k,v)
	}
}
//redis命令：zrevrange myzset start end [withscores]
func zrevrange(myzset string,start,end,flag int){
	var vals []interface{}
	var err error

	if flag == 0 {
		//不加withscores
		vals,err = redis.Values(c.Do("zrevrange",myzset,start,end))
	}else if flag == 1 {
		//加withscores
		vals,err = redis.Values(c.Do("zrevrange",myzset,start,end,"withscores"))
	}

	if err != nil {
		log.Fatal(err)
	}

	for k,v := range vals {
		fmt.Printf("k = %v v = %s\n",k,v)
	}

}
//redis命令：redis zrangebyscore myzset start end [withscores]
func zrangebyscore(myzset string,start,end,flag int){
	var vals []interface{}
	var err error

	if flag == 0 {
		//不加withscores
		vals,err = redis.Values(c.Do("zrangebyscore",myzset,start,end))
	}else if flag == 1 {
		//加withscores
		vals,err = redis.Values(c.Do("zrangebyscore",myzset,start,end,"withscores"))
	}

	if err != nil {
		log.Fatal(err)
	}

	for k,v := range vals {
		fmt.Printf("k = %v v = %s\n",k,v)
	}
}
//redis命令：zcard myzset
func zcard(myzset string){
	len,err := redis.Int(c.Do("zcard",myzset))

	if err != nil {
		log.Fatal(err)
	}

	fmt.Printf("有序集合%s中有%d个元素\n",myzset,len)
}
//redis命令：zcount myzset minscore maxscore
func zcount(myzset string,minscore,maxscore float64){
	len,err := redis.Int(c.Do("zcount",myzset,minscore,maxscore))

	if err != nil {
		log.Fatal(err)
	}

	fmt.Printf("有序集合%s位于%.2f和%.2f分数区间内的元素有%d个\n",myzset,minscore,maxscore,len)
}
//redis命令：zrank myzset val
func zrank(myzset,val string){
	index,err := redis.Int(c.Do("zrank",myzset,val))

	if err != nil {
		log.Fatal(err)
	}

	fmt.Printf("有序集合%s中的元素%s对应的索引为%d\n",myzset,val,index)
}
//redis命令：zscore myzset val
func zscore(myzset,val string){
	score,err := redis.Float64(c.Do("zscore",myzset,val))

	if err != nil {
		log.Fatal(err)
	}

	fmt.Printf("有序集合%s中的元素%s对应的分数为%.2f\n",myzset,val,score)
}
```



## 4.3 Redis连接池

**连接池：事先初始化一定数量的链接，放入到链接池，当Go需要操作Redis 时，直接从 Redis链接池取出链接即可。这样可以节省临时获取Redis 链接的时间，从而提高效率。**

* **应用程序调用Get方法从池中获取连接，并使用连接的Close方法将连接的资源返回到池**

**提供方法：**

* `func (*Pool) ActiveCount` 返回active的连接数，包含空闲的和正在使用的
* `func (*Pool) Close` 关闭连接
* `func (*Pool) Get` 获取一个连接
* `func (*Pool) GetContext` GetContext使用提供的上下文获取连接
* `func (*Pool) IdleCount` 空闲连接数
* `func (*Pool) Stats` 连接池统计信息



代码示例：

```go
package main
import (
	"fmt"
	"github.com/gomodule/redigo/redis"
)

//定义一个全局的pool
var pool *redis.Pool

//当启动程序时，就初始化连接池
func init() {
	pool = &redis.Pool{
		MaxIdle: 8, //最大空闲链接数
		MaxActive: 0, // 表示和数据库的最大链接数， 0 表示没有限制
		IdleTimeout: 100, // 最大空闲时间
		Dial: func() (redis.Conn, error) { // 初始化链接的代码， 链接哪个ip的redis
		return redis.Dial("tcp", "localhost:6379")
		},
	}
}

func main() {
	//先从pool 取出一个链接
	conn := pool.Get()
	defer conn.Close()

	_, err := conn.Do("Set", "name", "汤姆猫~~")
	if err != nil {
		fmt.Println("conn.Do err=", err)
		return
	}

	//取出
	r, err := redis.String(conn.Do("Get", "name"))
	if err != nil {
		fmt.Println("conn.Do err=", err)
		return
	}

	fmt.Println("r=", r)

	//如果我们要从pool 取出链接，一定保证链接池是没有关闭
	//pool.Close()
	conn2 := pool.Get()

	_, err = conn2.Do("Set", "name2", "汤姆猫~~2")
	if err != nil {
		fmt.Println("conn.Do err~~~~=", err)
		return
	}

	//取出
	r2, err := redis.String(conn2.Do("Get", "name2"))
	if err != nil {
		fmt.Println("conn.Do err=", err)
		return
	}

	fmt.Println("r=", r2)

	//fmt.Println("conn2=", conn2)

}
```



# 四、Golang操作Redis（go-redis）

> 好的博客：[Go操作Redis实战](https://www.cnblogs.com/itbsl/p/14198111.html)

## 安装

Go语言中使用第三方库https://github.com/go-redis/redis连接Redis数据库并进行操作。使用以下命令下载并安装：

```go
go get -u github.com/go-redis/redis

//安装新版本
go get github.com/go-redis/redis/v8
```

## 连接

* 普通连接

  ```go
  // 声明一个全局的rdb变量
  var rdb *redis.Client
  
  // 初始化连接
  func initClient() (err error) {
  	rdb = redis.NewClient(&redis.Options{
  		Addr:     "localhost:6379",
  		Password: "", // no password set
  		DB:       0,  // use default DB
  	})
  
  	_, err = rdb.Ping().Result()
  	if err != nil {
  		return err
  	}
  	return nil
  }
  ```

* 连接Redis哨兵模式

  ```go
  func initClient()(err error){
  	rdb := redis.NewFailoverClient(&redis.FailoverOptions{
  		MasterName:    "master",
  		SentinelAddrs: []string{"x.x.x.x:26379", "xx.xx.xx.xx:26379", "xxx.xxx.xxx.xxx:26379"},
  	})
  	_, err = rdb.Ping().Result()
  	if err != nil {
  		return err
  	}
  	return nil
  }
  ```

* 连接Redis集群

  ```go
  func initClient()(err error){
  	rdb := redis.NewClusterClient(&redis.ClusterOptions{
  		Addrs: []string{":7000", ":7001", ":7002", ":7003", ":7004", ":7005"},
  	})
  	_, err = rdb.Ping().Result()
  	if err != nil {
  		return err
  	}
  	return nil
  }
  ```

## 基本使用

### set/get示例

```go
func redisExample() {
	err := rdb.Set("score", 100, 0).Err()
	if err != nil {
		fmt.Printf("set score failed, err:%v\n", err)
		return
	}

	val, err := rdb.Get("score").Result()
	//val = rdb.Get("score").Val()	//Val()函数只取出来值
	if err != nil {
		fmt.Printf("get score failed, err:%v\n", err)
		return
	}
	fmt.Println("score", val)

	val2, err := rdb.Get("name").Result()
	//注意：key不存在也是一种错误类型，等于redis.Nil
	if err == redis.Nil {
		fmt.Println("name does not exist")
	} else if err != nil {
		fmt.Printf("get name failed, err:%v\n", err)
		return
	} else {
		fmt.Println("name", val2)
	}
}
```

### zset示例

```go
func redisExample2() {
	zsetKey := "language_rank"
	languages := []redis.Z{
		redis.Z{Score: 90.0, Member: "Golang"},
		redis.Z{Score: 98.0, Member: "Java"},
		redis.Z{Score: 95.0, Member: "Python"},
		redis.Z{Score: 97.0, Member: "JavaScript"},
		redis.Z{Score: 99.0, Member: "C/C++"},
	}
	// ZADD
	num, err := rdb.ZAdd(zsetKey, languages...).Result()
	if err != nil {
		fmt.Printf("zadd failed, err:%v\n", err)
		return
	}
	fmt.Printf("zadd %d succ.\n", num)

	// 把Golang的分数加10
	newScore, err := rdb.ZIncrBy(zsetKey, 10.0, "Golang").Result()
	if err != nil {
		fmt.Printf("zincrby failed, err:%v\n", err)
		return
	}
	fmt.Printf("Golang's score is %f now.\n", newScore)

	// 取分数最高的3个
	ret, err := rdb.ZRevRangeWithScores(zsetKey, 0, 2).Result()
	if err != nil {
		fmt.Printf("zrevrange failed, err:%v\n", err)
		return
	}
	for _, z := range ret {
		fmt.Println(z.Member, z.Score)
	}

	// 取95~100分的
	op := redis.ZRangeBy{
		Min: "95",
		Max: "100",
	}
	ret, err = rdb.ZRangeByScoreWithScores(zsetKey, op).Result()
	if err != nil {
		fmt.Printf("zrangebyscore failed, err:%v\n", err)
		return
	}
	for _, z := range ret {
		fmt.Println(z.Member, z.Score)
	}
}
```

### Pipeline

`Pipeline` 主要是一种网络优化。它本质上意味着客户端缓冲一堆命令并一次性将它们发送到服务器。这些命令不能保证在事务中执行。这样做的好处是节省了每个命令的网络往返时间（RTT）。

`Pipeline` 基本示例如下：

```go
pipe := rdb.Pipeline()

incr := pipe.Incr("pipeline_counter")
pipe.Expire("pipeline_counter", time.Hour)

_, err := pipe.Exec()
fmt.Println(incr.Val(), err)
```

上面的代码相当于将以下两个命令一次发给redis server端执行，与不使用`Pipeline`相比能减少一次RTT。

```bash
INCR pipeline_counter
EXPIRE pipeline_counts 3600
```

也可以使用`Pipelined`：

```go
var incr *redis.IntCmd
_, err := rdb.Pipelined(func(pipe redis.Pipeliner) error {
	incr = pipe.Incr("pipelined_counter")
	pipe.Expire("pipelined_counter", time.Hour)
	return nil
})
fmt.Println(incr.Val(), err)
```

在某些场景下，当我们有多条命令要执行时，就可以考虑使用pipeline来优化。



### 事务

Redis是单线程的，因此单个命令始终是原子的，但是来自不同客户端的两个给定命令可以依次执行，例如在它们之间交替执行。但是，`Multi/exec`能够确保在`multi/exec`两个语句之间的命令之间没有其他客户端正在执行命令。

在这种场景我们需要使用`TxPipeline`。`TxPipeline`总体上类似于上面的`Pipeline`，但是它内部会使用`MULTI/EXEC`包裹排队的命令。例如：

```go
pipe := rdb.TxPipeline()

incr := pipe.Incr("tx_pipeline_counter")
pipe.Expire("tx_pipeline_counter", time.Hour)

_, err := pipe.Exec()
fmt.Println(incr.Val(), err)
```

上面代码相当于在一个RTT下执行了下面的redis命令：

```bash
MULTI
INCR pipeline_counter
EXPIRE pipeline_counts 3600
EXEC
```

还有一个与上文类似的`TxPipelined`方法，使用方法如下：

```go
var incr *redis.IntCmd
_, err := rdb.TxPipelined(func(pipe redis.Pipeliner) error {
	incr = pipe.Incr("tx_pipelined_counter")
	pipe.Expire("tx_pipelined_counter", time.Hour)
	return nil
})
fmt.Println(incr.Val(), err)
```

### Watch

在某些场景下，我们除了要使用`MULTI/EXEC`命令外，还需要配合使用`WATCH`命令。在用户使用`WATCH`命令监视某个键之后，直到该用户执行`EXEC`命令的这段时间里，如果有其他用户抢先对被监视的键进行了替换、更新、删除等操作，那么当用户尝试执行`EXEC`的时候，事务将失败并返回一个错误，用户可以根据这个错误选择重试事务或者放弃事务。

```go
Watch(fn func(*Tx) error, keys ...string) error
```

Watch方法接收一个函数和一个或多个key作为参数。基本使用示例如下：

```go
func watchDemo() {
	// 监视watch_count的值，并在值不变的前提下将其值+1
	key := "watch_count"
	err := rdb.Watch(func(tx *redis.Tx) error {
		n, err := tx.Get(key).Int()
		if err != nil && err != redis.Nil {
			return err
		}
		_, err = tx.Pipelined(func(pipe redis.Pipeliner) error {
			//time.Sleep(time.Second * 5)	//如果此时对key对应value进行修改，下面会执行错误
			pipe.Set(key, n+1, 0)
			return nil
		})
		return err
	}, key)
	if err != nil {
		fmt.Printf("watch操作失败：%v\n", err)
		return
	}
	fmt.Println("watch操作成功")
}
```

## V8新版本

* **安装：**

  ```go
  go get github.com/go-redis/redis/v8
  ```

* 最新版本的`go-redis`库的相关命令都需要传递`context.Context`参数，例如：

  ```go
  package main
  
  import (
  	"context"
  	"fmt"
  	"time"
  
  	"github.com/go-redis/redis/v8" // 注意导入的是新版本
  )
  
  var (
  	rdb *redis.Client
  )
  
  // 初始化连接
  func initClient() (err error) {
  	rdb = redis.NewClient(&redis.Options{
  		Addr:     "localhost:16379",
  		Password: "",  // no password set
  		DB:       0,   // use default DB
  		PoolSize: 100, // 连接池大小
  	})
  
  	ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
  	defer cancel()
  
  	_, err = rdb.Ping(ctx).Result()
  	return err
  }
  
  func V8Example() {
  	ctx := context.Background()
  	if err := initClient(); err != nil {
  		return
  	}
  
  	err := rdb.Set(ctx, "key", "value", 0).Err()
  	if err != nil {
  		panic(err)
  	}
  
  	val, err := rdb.Get(ctx, "key").Result()
  	if err != nil {
  		panic(err)
  	}
  	fmt.Println("key", val)
  
  	val2, err := rdb.Get(ctx, "key2").Result()
  	if err == redis.Nil {
  		fmt.Println("key2 does not exist")
  	} else if err != nil {
  		panic(err)
  	} else {
  		fmt.Println("key2", val2)
  	}
  	// Output: key value
  	// key2 does not exist
  }
  ```

  

# 五、操作MySQL

## 5.1 获取数据库链接

* **Go语言中的`database/sql`包提供了保证SQL或类SQL数据库的泛用接口，并不提供具体的数据库驱动。`database/sql/driver` 包定义了应被数据库驱动实现的接口，这些接口会被 sql 包使用。所有使用`database/sql`包时必须注入（至少）一个数据库驱动。**

* 我们常用的数据库基本上都有完整的第三方实现。例如：MySQL驱动

**下载依赖：**

```bash
go get -u github.com/go-sql-driver/mysql
```



<font color='blue' size='4'>**初始化连接：**</font>

* **Open打开数据库函数**

  ```go
  func Open(driverName, dataSourceName string) (*DB, error)
  ```

  **Open打开一个dirverName指定的数据库，dataSourceName指定数据源，一般包至少括数据库文件名和（可能的）连接信息。**

  Open函数可能只是验证其参数，而不创建与数据库的连接。如果要检查数据源的名称是否合法，应调用返回值的Ping方法。

  * **参数 dataSourceName 的格式：**

    **`数据库用户名:数据库密码@[tcp(localhost:3306)]/数据库名`**

  ```go
  import (
  	"database/sql"
  	_ "github.com/go-sql-driver/mysql"
  )
  
  func main() {
     // DSN:Data Source Name
  	dsn := "user:password@tcp(127.0.0.1:3306)/dbname"
  	db, err := sql.Open("mysql", dsn)
  	if err != nil {
  		panic(err)
  	}
      // 做完错误检查之后，确保db不为nil
  	defer db.Close()  // 注意这行代码要写在上面err判断的下面
  }
  ```

* **DB结构体说明**

  ```go
  type DB struct {
      // 内含隐藏或非导出字段
  }
  ```

  * **DB是一个数据库（操作）句柄，代表一个具有零到多个底层连接的连接池。它可以安全的被多个go程同时使用。**

  * **sql包会自动创建和释放连接；它也会维护一个闲置连接的连接池。如果数据库具有单连接状态的概念，该状态只有在事务中被观察时才可信。一旦调用了BD.Begin，返回的Tx会绑定到单个连接。当调用事务Tx的Commit或Rollback后，该事务使用的连接会归还到DB的闲置连接池中。连接池的大小可以用SetMaxIdleConns方法控制。**

* **Ping()方法测试连接是否有效**

  ```go
  func (db *DB) Ping() error
  ```

  Ping检查与数据库的连接是否仍有效，如果需要会创建连接。



**初始化连接实例：**可以将数据库对象的初始化放到`init()`函数中

```go
import (
	"database/sql"
	_ "github.com/go-sql-driver/mysql"
)
// 定义一个全局对象db
var db *sql.DB

// 定义一个初始化数据库的函数
func initDB() (err error) {
	// DSN:Data Source Name
	dsn := "user:password@tcp(127.0.0.1:3306)/test?charset=utf8mb4&parseTime=True"
	// 不会校验账号密码是否正确
	// 注意！！！这里不要使用:=，我们是给全局变量赋值，然后在main函数中使用全局变量db
	db, err = sql.Open("mysql", dsn)
	if err != nil {
		return err
	}
	// 尝试与数据库建立连接（校验dsn是否正确）
	err = db.Ping()
	if err != nil {
		return err
	}
	return nil
}

func main() {
	err := initDB() // 调用输出化数据库的函数
	if err != nil {
		fmt.Printf("init db failed,err:%v\n", err)
		return
	}
}
```

执行结果：

![image-20220321142656948](https://gitee.com/jobim/blogimage/raw/master/img/20220321142657.png)



<font color='blue' size='4'>**数据库的设置：**</font>

* **`func (*DB) SetMaxOpenConns`，设置与数据库建立连接的最大数目**

  ```go
  func (db *DB) SetMaxOpenConns(n int)
  ```

  * 如果n大于0且小于最大闲置连接数，会将最大闲置连接数减小到匹配最大开启连接数的限制。
  * 如果n<=0，不会限制最大开启连接数，默认为0（无限制）。

* **`func (*DB) SetMaxIdleConns`，设置连接池中的最大闲置连接数**

  ```go
  func (db *DB) SetMaxIdleConns(n int)
  ```

  * 如果n大于最大开启连接数，则新的最大闲置连接数会减小到匹配最大开启连接数的限制。
  * 如果n<=0，不会保留闲置连接。

## 5.2 增删改查操作

* **在连接的 test 数据库中创建一个 `users` 表**

  ```sql
  CREATE TABLE users(
      id INT PRIMARY KEY AUTO_INCREMENT,
      username VARCHAR(100) UNIQUE NOT NULL,
      PASSWORD VARCHAR(100) NOT NULL,
      email VARCHAR(100)
  )
  ```

* **创建连接数据库的工具包**

  ```go
  package utils
  
  import (
  	"database/sql"
  	_ "github.com/go-sql-driver/mysql"
  )
  
  var (
  	Db *sql.DB
  	err error
  )
  
  func init() {
  	Db, err = sql.Open("mysql", "root:root@tcp(localhost:3306)/test")
  	if err != nil {
  		panic(err.Error())
  	}
  	// 尝试与数据库建立连接（校验dsn是否正确）
  	err = Db.Ping()
  	if err != nil {
  		panic(err.Error())
  	}
  }
  ```

### 5.2.1 增删改操作

* **func (db *DB) Exec执行一次命令，插入、更新和删除操作该方法。**

  ```go
  func (db *DB) Exec(query string, args ...interface{}) (Result, error)
  ```

  Exec执行一次命令（包括查询、删除、更新、插入等），返回的Result是对已执行的SQL命令的总结。参数args表示query中的占位参数。

<font color='blue' size='4'>**插入数据：**</font>

```go
//添加User
func (user *User) AddUser2() error {
	//1.写sql语句
	sqlStr := "insert into users(username,password,email) values(?,?,?)"
	//2.执行
	result, err := utils.Db.Exec(sqlStr, "admin4", "666666", "admin2@sina.com")
	if err != nil {
		fmt.Println("执行出现异常：", err)
		return err
	}
	theID, err := result.LastInsertId() // 新插入数据的id
	if err != nil {
		fmt.Printf("get lastinsert ID failed, err:%v\n", err)
		return err
	}
	fmt.Printf("insert success, the id is %d.\n", theID)	//insert success, the id is 5.
	return nil
}
```

<font color='blue' size='4'>**更新数据：**</font>

```go
// 更新数据
func updateUser() {
	sqlStr := "update users set password=? where id = ?"
	ret, err := utils.Db.Exec(sqlStr, "123456", 4)
	if err != nil {
		fmt.Printf("update failed, err:%v\n", err)
		return
	}
	n, err := ret.RowsAffected() // 操作影响的行数
	if err != nil {
		fmt.Printf("get RowsAffected failed, err:%v\n", err)
		return
	}
	fmt.Printf("update success, affected rows:%d\n", n)
}
```

<font color='blue' size='4'>**删除数据：**</font>

```go
// 删除数据
func deleteUser() {
	sqlStr := "delete from users where id = ?"
	ret, err := utils.Db.Exec(sqlStr, 4)
	if err != nil {
		fmt.Printf("delete failed, err:%v\n", err)
		return
	}
	n, err := ret.RowsAffected() // 操作影响的行数
	if err != nil {
		fmt.Printf("get RowsAffected failed, err:%v\n", err)
		return
	}
	fmt.Printf("delete success, affected rows:%d\n", n)
}
```



### 5.2.2 查询操作

* **`func (*DB) Query`，用于查询多行结果**

  ```go
  func (db *DB) Query(query string, args ...interface{}) (*Rows, error)
  ```

  Query执行一次查询，返回多行结果（即Rows），一般用于执行select命令。参数args表示query中的占位参数。

* **`func (*DB) QueryRow`，用于期望返回最多一行结果的查询**

  ```go
  func (db *DB) QueryRow(query string, args ...interface{}) *Row
  ```

  QueryRow执行一次查询，并期望返回最多一行结果（即Row）。QueryRow总是返回非nil的值，直到返回值的Scan方法被调用时，才会返回被延迟的错误

* **Row 结构体及它的方法的说明：**

  ![image-20220321150900096](https://gitee.com/jobim/blogimage/raw/master/img/20220321150900.png)

* **Rows 结构体及它的方法的说明：**![image-20220321151008612](https://gitee.com/jobim/blogimage/raw/master/img/20220321151008.png)

<font color='blue' size='4'>**单行查询：**</font>

```go
// 查询单条数据示例
func queryRowDemo() {
	sqlStr := "select id, username, password, email from users where id = ?"
	var u User

	row := db.QueryRow(sqlStr, 1)
	// 非常重要：确保QueryRow之后调用Scan方法，否则持有的数据库链接不会被释放
	err := row.Scan(&u.ID, &u.Username, &u.Password, &u.Email)
	if err != nil {
		fmt.Printf("scan failed, err:%v\n", err)
		return
	}
	fmt.Println("u的数据：", u)
}
```

* **注意：确保QueryRow之后调用Scan方法，否则持有的数据库链接不会被释放，一直占用着**

  ![image-20220331160520217](https://gitee.com/jobim/blogimage/raw/master/img/202203311605446.png)



<font color='blue' size='4'>**多行查询：**</font>

```go
// 查询多条数据示例
func queryMultiRowDemo() {
	//写sql语句
	sqlStr := "select id, username, password, email from users"
	//执行
	rows, err := db.Query(sqlStr)
	if err != nil {
		fmt.Printf("query failed, err:%v\n", err)
		return
	}
	// 非常重要：关闭rows释放持有的数据库链接
	defer rows.Close()

	// 循环读取结果集中的数据
	for rows.Next() {
		var u User
		err := rows.Scan(&u.ID, &u.Username, &u.Password, &u.Email)
		if err != nil {
			fmt.Printf("scan failed, err:%v\n", err)
			return
		}
		fmt.Println("u的数据：", u)
	}
}
```

**注意：使用Query()方法查询时，最好使用defer关闭rows释放持有的数据库链接。rows.Next()当返回false的时候会自动关闭，所以如果不能循环读取所有数据连接将不能释放**



### 5.2.3 预处理操作

**普通SQL语句执行过程：**

* 客户端对SQL语句进行占位符替换得到完整的SQL语句。
* 客户端发送完整SQL语句到MySQL服务端
* MySQL服务端执行完整的SQL语句并将结果返回给客户端。

**预处理执行过程：**

* 把SQL语句分成两部分，命令部分与数据部分。
* 先把命令部分发送给MySQL服务端，MySQL服务端进行SQL预处理。
* 然后把数据部分发送给MySQL服务端，MySQL服务端对SQL语句进行占位符替换。
* MySQL服务端执行完整的SQL语句并将结果返回给客户端。

**为什么要预处理？**

* 优化MySQL服务器重复执行SQL的方法，可以提升服务器性能，提前让服务器编译，一次编译多次执行，节省后续编译的成本。
* 避免SQL注入问题。

<font color='blue' size='4'>**预处理方法：**</font>

* **`Prepare`方法**

  ```go
  func (db *DB) Prepare(query string) (*Stmt, error)
  ```

  **Prepare创建一个准备好的状态用于之后的查询和命令。返回值可以同时执行多个查询和命令**

* **Stmt 结构体及它的方法的说明**

  ![image-20220321150009167](https://gitee.com/jobim/blogimage/raw/master/img/20220321150009.png)

**预编译的方法添加User：**

```go
func (user *User) AddUser() error {
	//1.写sql语句
	sqlStr := "insert into users(username,password,email) values(?,?,?)"
	//2.预编译
	inStmt, err := utils.Db.Prepare(sqlStr)
	if err != nil {
		fmt.Println("预编译出现异常：", err)
		return err
	}
	_, err2 := inStmt.Exec("admin", "123456", "admin@qq.com")
	if err2 != nil {
		fmt.Println("执行出现异常：", err2)
		return err
	}
	return nil
}
```

**查询操作的预处理示例代码如下：**

```go
// 预处理查询示例
func prepareQueryDemo() {
	sqlStr := "select id, username, password, email from users where id > ?"
	stmt, err := db.Prepare(sqlStr)
	if err != nil {
		fmt.Printf("prepare failed, err:%v\n", err)
		return
	}
	defer stmt.Close()
	rows, err := stmt.Query(0) // 预处理的时候传入参数
	if err != nil {
		fmt.Printf("query failed, err:%v\n", err)
		return
	}
	defer rows.Close()
	// 循环读取结果集中的数据
	for rows.Next() {
		var u User
		err := rows.Scan(&u.ID, &u.Username, &u.Password, &u.Email)
		if err != nil {
			fmt.Printf("scan failed, err:%v\n", err)
			return
		}
		fmt.Println("读取到的数据：", u)
	}
}
```



## 5.3 Go实现MySQL事务

Go语言中使用以下三个方法实现MySQL中的事务操作

* **开始事务**

  ```go
  func (db *DB) Begin() (*Tx, error)
  ```

* **提交事务**

  ```go
  func (tx *Tx) Commit() error
  ```

* **回滚事务**

  ```go
  func (tx *Tx) Rollback() error
  ```

**事务示例：**该事物操作能够确保两次更新操作要么同时成功要么同时失败，不会存在中间状态。

```go
// 事务操作示例
func transactionDemo() {
	tx, err := db.Begin() // 开启事务
	if err != nil {
		if tx != nil {
			tx.Rollback() // 回滚
		}
		fmt.Printf("begin trans failed, err:%v\n", err)
		return
	}
	sqlStr1 := "Update user set age=30 where id=?"
	_, err = tx.Exec(sqlStr1, 2)
	if err != nil {
		tx.Rollback() // 回滚
		fmt.Printf("exec sql1 failed, err:%v\n", err)
		return
	}
	sqlStr2 := "Update user set age=40 where id=?"
	_, err = tx.Exec(sqlStr2, 4)
	if err != nil {
		tx.Rollback() // 回滚
		fmt.Printf("exec sql2 failed, err:%v\n", err)
		return
	}
    err = tx.Commit() // 提交事务，如果前面的sql语句出现问题发生了回滚，则Commit()会出现异常
	if err != nil {
		tx.Rollback() // 回滚
		fmt.Printf("commit failed, err:%v\n", err)
		return
	}
	fmt.Println("exec trans success!")
}
```



## 5.4 sqlx使用

第三方库sqlx能够简化操作，提高开发效率。

* **安装：**

  ```go
  go get github.com/jmoiron/sqlx
  ```

### 连接数据库

```go
import (
	"database/sql/driver"
	"errors"
	"fmt"
	_ "github.com/go-sql-driver/mysql"
	"github.com/jmoiron/sqlx"
)

var db *sqlx.DB

func initDB() (err error) {
	dsn := "user:password@tcp(127.0.0.1:3306)/test?charset=utf8mb4&parseTime=True"
	// 也可以使用MustConnect连接不成功就panic
	db, err = sqlx.Connect("mysql", dsn)
	if err != nil {
		fmt.Printf("connect DB failed, err:%v\n", err)
		return
	}
	db.SetMaxOpenConns(20)
	db.SetMaxIdleConns(10)
	return
}
```

### 查询

* **查询单行数据示例代码如下：**

  可以使用结构体tag来指定数据库中对应的那一列

  ```go
  type User struct {
  	ID   int    `db:"id"` //可以使用结构体tag来指定数据库中对应的那一列
  	Name string `db:"name"`
  	Age  int    `db:"age"`
  }
  
  // 查询单条数据示例
  func queryRowDemo() {
  	sqlStr := "select id, name, age from user where id=?"
  	var u user
  	err := db.Get(&u, sqlStr, 1)
  	if err != nil {
  		fmt.Printf("get failed, err:%v\n", err)
  		return
  	}
  	fmt.Printf("id:%d name:%s age:%d\n", u.ID, u.Name, u.Age)
  }
  ```

* **查询多行数据示例代码如下：**

  ```go
  // 查询多条数据示例
  func queryMultiRowDemo() {
  	sqlStr := "select id, name, age from user where id > ?"
  	var users []user
  	err := db.Select(&users, sqlStr, 0)
  	if err != nil {
  		fmt.Printf("query failed, err:%v\n", err)
  		return
  	}
  	fmt.Printf("users:%#v\n", users)
  }
  ```

### 插入、更新和删除

sqlx中的exec方法与原生sql中的exec使用基本一致：

```go
// 插入数据
func insertRowDemo() {
	sqlStr := "insert into user(name, age) values (?,?)"
	ret, err := db.Exec(sqlStr, "沙河小王子", 19)
	if err != nil {
		fmt.Printf("insert failed, err:%v\n", err)
		return
	}
	theID, err := ret.LastInsertId() // 新插入数据的id
	if err != nil {
		fmt.Printf("get lastinsert ID failed, err:%v\n", err)
		return
	}
	fmt.Printf("insert success, the id is %d.\n", theID)
}

// 更新数据
func updateRowDemo() {
	sqlStr := "update user set age=? where id = ?"
	ret, err := db.Exec(sqlStr, 39, 6)
	if err != nil {
		fmt.Printf("update failed, err:%v\n", err)
		return
	}
	n, err := ret.RowsAffected() // 操作影响的行数
	if err != nil {
		fmt.Printf("get RowsAffected failed, err:%v\n", err)
		return
	}
	fmt.Printf("update success, affected rows:%d\n", n)
}

// 删除数据
func deleteRowDemo() {
	sqlStr := "delete from user where id = ?"
	ret, err := db.Exec(sqlStr, 6)
	if err != nil {
		fmt.Printf("delete failed, err:%v\n", err)
		return
	}
	n, err := ret.RowsAffected() // 操作影响的行数
	if err != nil {
		fmt.Printf("get RowsAffected failed, err:%v\n", err)
		return
	}
	fmt.Printf("delete success, affected rows:%d\n", n)
}
```

### 事务操作

对于事务操作，我们可以使用sqlx中提供的db.Beginx()和tx.MustExec()方法来简化错误处理过程。示例代码如下：

```go
func transactionDemo() {
	tx, err := db.Beginx() // 开启事务
	if err != nil {
		if tx != nil {
			tx.Rollback()
		}
		fmt.Printf("begin trans failed, err:%v\n", err)
		return
	}
	sqlStr1 := "Update user set age=40 where id=?"
	tx.MustExec(sqlStr1, 2)
	sqlStr2 := "Update user set age=50 where id=?"
	tx.MustExec(sqlStr2, 4)
	err = tx.Commit() // 提交事务
	if err != nil {
		tx.Rollback() // 回滚
		fmt.Printf("commit failed, err:%v\n", err)
		return
	}
	fmt.Println("exec trans success!")
}
```



`sqlx.In`是`sqlx`提供的一个非常方便的函数。

### sqlx.In

####  批量插入示例：

* **自己拼接语句实现批量插入**：比较笨，但是很好理解。就是有多少个User就拼接多少个`(?, ?)`。

  ```go
  type User struct {
  	Name string `db:"name"`
  	Age  int    `db:"age"`
  }
  
  // BatchInsertUsers 自行构造批量插入的语句
  func BatchInsertUsers(users []*User) error {
  	// 存放 (?, ?) 的slice
  	valueStrings := make([]string, 0, len(users))
  	// 存放values的slice
  	valueArgs := make([]interface{}, 0, len(users) * 2)
  	// 遍历users准备相关数据
  	for _, u := range users {
  		// 此处占位符要与插入值的个数对应
  		valueStrings = append(valueStrings, "(?, ?)")
  		valueArgs = append(valueArgs, u.Name)
  		valueArgs = append(valueArgs, u.Age)
  	}
  	// 自行拼接要执行的具体语句
  	stmt := fmt.Sprintf("INSERT INTO user (name, age) VALUES %s",
  		strings.Join(valueStrings, ","))
  	_, err := DB.Exec(stmt, valueArgs...)
  	return err
  }
  ```

* **使用sqlx.In实现批量插入：**

  * **前提是需要我们的结构体实现`driver.Valuer`接口：**

    ```go
    type User struct {
    	Name string `db:"name"`
    	Age  int    `db:"age"`
    }
    
    func (u User) Value() (driver.Value, error) {
    	return []interface{}{u.Name, u.Age}, nil
    }
    ```

  * **使用`sqlx.In`实现批量插入代码如下**

    ```go
    // BatchInsertUsers 使用sqlx.In帮我们拼接语句和参数, 注意传入的参数是[]interface{}
    func BatchInsertUsers(users []interface{}) error {
    	query, args, _ := sqlx.In(
    		"INSERT INTO user (name, age) VALUES (?), (?), (?)",
    		users..., // 如果arg实现了 driver.Valuer, sqlx.In 会通过调用 Value()来展开它
    	)
    	fmt.Println("新生成的sql语句：", query) // 查看生成的querystring
    	fmt.Println("新生成的参数：", args)     // 查看生成的args
    	_, err := db.Exec(query, args...)
    	return err
    }
    
    func main() {
    	initDB()
        
    	u1 := User{Name: "七米", Age: 18}
    	u2 := User{Name: "q1mi", Age: 28}
    	u3 := User{Name: "小王子", Age: 38}
    	users2 := []interface{}{u1, u2, u3}
    	err := BatchInsertUsers(users2)
    	if err != nil {
    		fmt.Printf("BatchInsertUsers2 failed, err:%v\n", err)
    	}
    }
    ```

  * **执行结果：**

    ![image-20220401112428091](https://gitee.com/jobim/blogimage/raw/master/img/202204011124152.png)

#### sqlx.In的查询示例：

* **查询id在给定id集合中的数据：**

  ```go
  // QueryByIDs 根据给定ID查询
  func QueryByIDs(ids []int) (users []User, err error) {
  	// 动态填充id
  	query, args, err := sqlx.In("SELECT name, age FROM user WHERE id IN (?)", ids)
  	if err != nil {
  		return
  	}
  	fmt.Println("新生成的sql语句：", query) // 查看生成的querystring
  	fmt.Println("新生成的参数：", args)     // 查看生成的args
  	// sqlx.In 返回带 `?` bindvar的查询语句, 我们使用Rebind()重新绑定它，可以不写
  	query = db.Rebind(query)
  	fmt.Println("Rebind()后的sql语句：", query) // 查看生成的querystring
  	err = db.Select(&users, query, args...)
  	return
  }
  
  func main() {
  	initDB()
  	us, _ := QueryByIDs([]int{1, 2, 3, 4})
  	fmt.Println(us)
  }
  ```

  * **sqlx.DB.Rebind(string) string的作用： 函数利用? 语法来得到一个合适在当前数据库上执行的query语句。即：可以生成在其他数据库执行的query语句**

  * **执行结果：**

    ![image-20220401113300560](https://gitee.com/jobim/blogimage/raw/master/img/202204011133622.png)

  



* **查询id在给定id集合的数据并维持给定id集合的顺序**

  **使用`in查询和FIND_IN_SET函数`**
  ```go
  // QueryAndOrderByIDs 按照指定id查询并维护顺序
  func QueryAndOrderByIDs(ids []int)(users []User, err error){
  	// 动态填充id
  	strIDs := make([]string, 0, len(ids))
  	for _, id := range ids {
  		strIDs = append(strIDs, fmt.Sprintf("%d", id))
  	}
  	query, args, err := sqlx.In("SELECT name, age FROM user WHERE id IN (?) ORDER BY FIND_IN_SET(id, ?)", ids, strings.Join(strIDs, ","))
  	if err != nil {
  		return
  	}
  
  	// sqlx.In 返回带 `?` bindvar的查询语句, 我们使用Rebind()重新绑定它
  	query = DB.Rebind(query)
  
  	err = DB.Select(&users, query, args...)
  	return
  }
  ```

  也可以先使用`IN`查询，然后通过代码按给定的ids对查询结果进行排序







# 部署go程序到服务器



[Go项目如何部署到阿里云 - 简书 (jianshu.com)](https://www.jianshu.com/p/64363dff9721)





