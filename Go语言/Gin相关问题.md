# Gin解决请求跨域问题



**简单请求和非简单请求：**

* **简单请求**：浏览器先发送（执行）请求然后再判断是否跨域。

  请求方法为 GET、POST、HEAD，请求头header中无自定义的请求头信息，请求类型Content-Type 为 text/plain、multipart/form-data、application/x-www-form-urlencoded 的请求都是简单请求。

* **非简单请求**：浏览器先发送预检命令（OPTIONS方法），检查通过后才发送真正的数据请求。

## 简单请求

当浏览器发现发起的ajax请求是简单请求时，会在请求头中携带一个字段：

```text
Origin: http://localhost:8080
```

Origin中会指出当前请求属于哪个域（协议+域名+端口）。服务器会根据这个值决定是否允许其跨域。

**对于简单请求，如果服务器允许跨域，需要在返回的响应头中携带下面信息：**

```
Access-Control-Allow-Origin: http://localhost:8080
Access-Control-Allow-Credentials: true    //(可选)
```

* Access-Control-Allow-Origin：允许哪个域名进行跨域，是一个具体域名或者*（代表任意域名）
* Access-Control-Allow-Credentials：是否允许携带cookie，默认情况下，cors不会携带cookie，除非这个值是true

**要想操作cookie，需要满足3个条件：**

* 服务的响应头中需要携带Access-Control-Allow-Credentials并且为true。
* 浏览器发起ajax需要指定 withCredentials 为true
* 响应头中的Access-Control-Allow-Origin一定不能为*，必须是指定的域名



## 非简单请求

非简单请求会在正式通信之前，增加一次HTTP查询请求，称为**预检请求(preflight)**。浏览器先询问服务器，当前网页所在的域名是否在服务器的许可名单之中，以及可以使用哪些HTTP动词和头信息字段。



**请求头中除了Origin以外，多了两个头：**

* Access-Control-Request-Method：接下来会用到的请求方式，比如PUT
* Access-Control-Request-Headers：会额外用到的头信息

**响应头通常包含下面信息：**

* Access-Control-Allow-Origin：允许哪个域名进行跨域，是一个具体域名或者*（代表任意域名）
* Access-Control-Allow-Credentials：是否允许携带cookie，默认情况下，cors不会携带cookie，除非这个值是true（可选）
* Access-Control-Allow-Methods：允许访问的方式
* Access-Control-Allow-Headers：允许携带的头
* Access-Control-Max-Age：本次许可的有效时长，单位是秒，过期之前的ajax请求就无需再次进行预检了（可选）

> 如果浏览器得到上述响应，则认定为可以跨域，后续就跟简单请求的处理是一样的了。



## Gin解决跨域代码

**Gin解决跨域代码的中间件代码：**

```go
func main() {
	// 创建一个默认的路由引擎
	r := gin.Default()
	r.Use(Cors())
	r.GET("/api/hello", func(c *gin.Context) {
		// c.JSON：返回JSON格式的数据
		c.JSON(200, gin.H{
			"message": "跨域请求成功",
		})
	})
	// 启动HTTP服务，默认在0.0.0.0:8080启动服务
	r.Run()
}

// Cors 处理跨域请求,支持options访问
func Cors() gin.HandlerFunc {
	return func(c *gin.Context) {
		method := c.Request.Method
		// 允许哪个域名进行跨域
		c.Header("Access-Control-Allow-Origin", c.Request.Header.Get("Origin"))
		// 会额外用到的头信息
		c.Header("Access-Control-Allow-Headers", "Content-Type,AccessToken,X-CSRF-Token, Authorization, Token")
		// 会用到的请求方式
		c.Header("Access-Control-Allow-Methods", "POST, GET, OPTIONS, PUT, DELETE")
		// 允许浏览器（客户端）可以解析的头部
		c.Header("Access-Control-Expose-Headers", "Content-Length, Access-Control-Allow-Origin, Access-Control-Allow-Headers, Content-Type")
		// 允许客户端传递校验信息比如 cookie
		c.Header("Access-Control-Allow-Credentials", "true")
		//设置缓存时间，过期之前的请求就无需再次进行预检了
		c.Header("Access-Control-Max-Age", "17280")
		// 放行所有OPTIONS方法，返回前端options方法响应成功
		if method == "OPTIONS" {
			c.AbortWithStatus(http.StatusNoContent)
		}
		// 处理请求
		c.Next()
	}
}
```

**如果前端需要使用到cookie需要指定请求中 withCredentials 为true**

* **以vue中axios为例：**

  ```js
  axios.defaults.withCredentials = true; //允许跨域携带cookie信息
  ```





