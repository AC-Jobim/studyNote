

# 用户表结构设计



```sql
CREATE TABLE `user` (
	`id` bigint(20) NOT NULL AUTO_INCREMENT,
	`user_id` bigint(20) NOT NULL,
	`username` varchar(64) COLLATE utf8mb4_general_ci NOT NULL,
	`password` varchar(64) COLLATE utf8mb4_general_ci NOT NULL,
	`email` varchar(64) COLLATE utf8mb4_general_ci,
	`gender` tinyint(4) NOT NULL DEFAULT '0',
	`create_time` timestamp NULL DEFAULT CURRENT_TIMESTAMP,
	`update_time` timestamp NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
	PRIMARY KEY (`id`),
	UNIQUE KEY `idx_username` (`username`) USING BTREE,
	UNIQUE KEY `idx_user_id` (`user_id`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;

```





# 雪花算法生成用户id

开源地址

https://github.com/bwmarrin/snowflake

```go
package main

import (
	"fmt"
	"github.com/bwmarrin/snowflake"
	"time"
)

var node *snowflake.Node

func Init(startTime string, machineID int64) (err error) {
	var st time.Time
	// 格式化 1月2号下午3时4分5秒  2006年
	st, err = time.Parse("2006-01-02", startTime)
	if err != nil {
		fmt.Println(err)
		return
	}
	snowflake.Epoch = st.UnixNano() / 1e6
	node, err = snowflake.NewNode(machineID)
	if err != nil {
		fmt.Println(err)
		return
	}
	return
}

// 生成 64 位的 雪花 ID
func GenID() int64 {
	return node.Generate().Int64()
}

func main() {
	if err := Init("2022-01-01", 1); err != nil {
		fmt.Println("Init() failed, err = ", err)
		return
	}
	id := GenID()
	fmt.Println(id)
}
```







https://github.com/sony/sonyflake



```go
package main

import (
	"fmt"
	"github.com/sony/sonyflake"
	"time"
)

var (
	sonyFlake     *sonyflake.Sonyflake
	sonyMachineID uint16
)

func getMachineID() (uint16, error) {
	return sonyMachineID, nil
}

// 需传入当前的机器ID
func Init(startTime string, machineId uint16) (err error) {
	sonyMachineID = machineId
	var st time.Time
	st, err = time.Parse("2006-01-02", startTime)
	if err != nil {
		return err
	}
	settings := sonyflake.Settings{
		StartTime: st,
		MachineID: getMachineID,
	}
	sonyFlake = sonyflake.NewSonyflake(settings)
	return
}

// GetID 返回生成的id值
func GetID() (id uint64, err error) {
	if sonyFlake == nil {
		err = fmt.Errorf("snoy flake not inited")
		return
	}
	id, err = sonyFlake.NextID()
	return
}

func main() {
	if err := Init("2022-01-01", 1); err != nil {
		fmt.Printf("Init failed, err:%v\n", err)
		return
	}
	id, _ := GetID()
	fmt.Println(id)
}
```







# 使用validator库进行参数校验





## viper







## 插入用户使用md5加密









## 定义错误码并封装响应方法







# JWT认证





## 使用refresh token刷新access token

之前的代码：

```go
package jwt

import (
	"errors"
	"github.com/dgrijalva/jwt-go"
	"time"
)

const TokenExpireDuration = time.Hour * 2

var mySecret = []byte("夏天夏天悄悄过去")

// MyClaims 自定义声明结构体并内嵌jwt.StandardClaims
// jwt包自带的jwt.StandardClaims只包含了官方字段
// 我们这里需要额外记录一个username字段，所以要自定义结构体
// 如果想要保存更多信息，都可以添加到这个结构体中
type MyClaims struct {
	UserID   int64  `json:"user_id"`
	Username string `json:"username"`
	jwt.StandardClaims
}

// GenToken 生成JWT
func GenToken(userID int64, username string) (string, error) {
	// 创建一个我们自己的声明
	c := MyClaims{
		userID,
		username, // 自定义字段
		jwt.StandardClaims{
			ExpiresAt: time.Now().Add(TokenExpireDuration).Unix(), // 过期时间
			Issuer:    "my-project",                               // 签发人
		},
	}
	// 使用指定的签名方法创建签名对象
	token := jwt.NewWithClaims(jwt.SigningMethodHS256, c)
	// 使用指定的secret签名并获得完整的编码后的字符串token
	return token.SignedString(mySecret)
}

//// ParseToken 解析JWT
//func ParseToken(tokenString string) (*MyClaims, error) {
//	// 解析token
//	token, err := jwt.ParseWithClaims(tokenString, &MyClaims{}, func(token *jwt.Token) (i interface{}, err error) {
//		return MySecret, nil
//	})
//	fmt.Println("解析之后的token：", token)
//	if err != nil {
//		return nil, err
//	}
//	if claims, ok := token.Claims.(*MyClaims); ok && token.Valid { // 校验token
//		return claims, nil
//	}
//	return nil, errors.New("invalid token")
//}

// ParseToken 解析JWT的另外一种写法
func ParseToken(tokenString string) (*MyClaims, error) {
	// 解析token
	var mc = new(MyClaims)
	//解析token
	token, err := jwt.ParseWithClaims(tokenString, mc, func(token *jwt.Token) (i interface{}, err error) {
		return mySecret, nil
	})
	if err != nil {
		return nil, err
	}
	if token.Valid { // 校验token
		return mc, nil
	}
	return nil, errors.New("invalid token")
}

```







# 解决前端JSON转换时长整型（整数）的精度问题





解决：通常转换成字符串





### 优雅处理字符串格式的数字

有时候，前端在传递来的json数据中可能会使用字符串类型的数字，这个时候可以在结构体tag中添加`string`来告诉json包从字符串中解析相应字段的数据：

```go
type Card struct {
	ID    int64   `json:"id,string"`    // 添加string tag
	Score float64 `json:"score,string"` // 添加string tag
}

func intAndStringDemo() {
	jsonStr1 := `{"id": "1234567","score": "88.50"}`
	var c1 Card
	if err := json.Unmarshal([]byte(jsonStr1), &c1); err != nil {
		fmt.Printf("json.Unmarsha jsonStr1 failed, err:%v\n", err)
		return
	}
	fmt.Printf("c1:%#v\n", c1) // c1:main.Card{ID:1234567, Score:88.5}
}
```



## 投票功能分析

```
docker run -itd -p 26379:6379 -v /mydocker/myredis/data:/data -v /mydocker/myredis/conf/redis.conf:/etc/redis/redis.conf  redis redis-server /etc/redis/redis.conf

```

redis.conf的配置文件可以在 http://download.redis.io/redis-stable/redis.conf 上下载

```
1. $ docker run -itd --name redis-test -p 6379:6379 redis

docker run -itd -p 192.168.220.129:6379:6379 --name redis -v /usr/local/docker/redis.conf:/etc/redis/redis.conf -v /usr/local/docker/data:/data redis redis-server /etc/redis/redis.conf 

-d 以守护线程的方式运行（后台运行）
-i 以交互模式运行容器
-t 为容器重新分配一个伪输入终端 
-p 映射容器服务的 6379 端口到宿主机的 6379 端口。外部可以直接通过宿主机ip:6379 访问到 Redis 的服务。

 //未加-it可能会运行不起来因为，Docker容器后台运行,就必须有一个前台进程，容器运行的命令不是那些一直挂起的命令（比如运行top，tail），会自动退出

-v /usr/local/docker/redis.conf:/etc/redis/redis.conf //把宿主机配置好的redis.conf挂载到容器内的指定位置

-v /usr/local/docker/data:/data  //把redis持久化的数据挂载到宿主机内，做数据备份

redis-server /etc/redis/redis.conf  //使redis按照redis.conf的配置启动

–appendonly yes //redis启动后数据持久化
```





用户投票的相关算法

http://www.ruanyifeng.com/blog/algorithm/



投票算法基于Redis实战中的简单算法





## 实现按照帖子的时间或者分数排序







## 按照社区查询



在redis中添加一个set类型的数据，用来记录该社区下所发布的帖子

![image-20220412172038696](https://blog.zhaobincode.cn/blogimages/202204121720767.png)

使用 zinterstore 把分区的帖子set与帖子分数的 zset 生成一个新的zset（可以设置过期时间60来减少zinterstore 的执行次数）







核心代码：





### Redis Zinterstore 命令







# swagger生成接口文档





添加单元测试，主要完成参数的校验





```
go-wrk -t=8 -c=100 -n=10000 -H="Authorization :Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjozMzg5Njc3NTAxNDE1ODMzNiwidXNlcm5hbWUiOiJ6aGFvYmluIiwiZXhwIjoxNjgwOTQ3NjEwLCJpc3MiOiJteS1wcm9qZWN0In0.-chpxmZJ5f9EpwNAKmOZHgC4PiQTB7AwlAZ3OnTb24U" http://127.0.0.1:8081/api/v1/posts
```



```bash
go-wrk -t=8 -c=100 -n=10000 http://localhost:8081/api/v1/post
```





# 项目的部署



## TODO：

对于7天之后不能投票的数据，可以设置一个定时任务，7天后的数据从redis中删除，然后保存到数据库中



# 前端功能

* 登陆注册
* token相关
* 创建帖子功能
* 查看帖子详情
* 分页展示帖子
* 给帖子投票功能
* 根据时间排序或者得分排序获取帖子列表
* 查询某社区的帖子列表







