# 一、Gin框架安装与使用

**Go社区中，有许多非常优秀的Web框架，如`Gin`,`Iris`,`Echo`,`Martini`,`Revel`以及国人开发的`Beego`框架。**

**Gin：Go世界里最流行的Web框架，[Github](https://github.com/gin-gonic/gin)上有`32K+`star。 基于[httprouter](https://github.com/julienschmidt/httprouter)开发的Web框架。 [中文文档](https://gin-gonic.com/zh-cn/docs/)齐全，简单易用的轻量级框架。**



<font color='blue' size='4'>**下载并安装`Gin`：**</font>

* 使用Go命令安装：

  ```go
  go get -u github.com/gin-gonic/gin	///使用-u安装最新版本
  ```

* 使用import导入到代码中：

  ```go
  import "github.com/gin-gonic/gin"
  ```



**使用示例：**

```go
package main

import (
	"github.com/gin-gonic/gin"
)

func main() {
	// 创建一个默认的路由引擎
	r := gin.Default()
	// GET：请求方式；/hello：请求的路径
	// 当客户端以GET方法请求/hello路径时，会执行后面的匿名函数
    r.GET("/hello", func(c *gin.Context) {
        // c.JSON：返回JSON格式的数据
        c.JSON(200, gin.H{
            "message": "Hello world!",
        })
    })
	// 启动HTTP服务，默认在0.0.0.0:8080启动服务
	r.Run()
}
```

* **创建路由：使用gin.Default()方法会返回gin.Engine实例，表示默认路由引擎**

  ```go
  r := gin.Default()
  ```

  **通过这种方式创建的gin.Engine，会默认使用`Logger`和`Recovery`两个中间件，可以用`gin.New()`方法创建一个不包含任何中间件的默认路由。**

  ```go
  r := gin.New()
  ```

* 定义处理HTTP的方法：通过默认路由，我们可以创建处理HTTP请求的方法，示例中使用GET方法:

  ```go
  r.GET("/hello", func(c *gin.Context) {
      // c.JSON：返回JSON格式的数据
      c.JSON(200, gin.H{
      	"message": "Hello world!",
      })
  })
  ```

  **Gin支持所有通用的HTTP请求方法：`GET`,`POST`,`PUT`,`PATCH`,`OPTIONS`,`HEAD`,`DELETE`,其使用方式与上面例子相同。**

  **每种方法都只处理对应的HTTP请求，使用Any方法则可以处理任何的HTTP请求。**

  ```go
  r.Any("/test",func(c *gin.Context){
      c.JSON(200,gin.H{"hello":"world"})
  })
  ```

* **监听端口定义好请求之后，使用Run()方法便可监听端口，开始接受HTTP请求,如果Run()方法没有传入参数的话，则默认监听的端口是8080。**

  ```go
  r.Run() //r.Run(":3000")
  ```

# 二、获取请求参数

## 2.1 获取query参数

**query指的是URL ? 后面携带的参数，例如user/info?username=张三&password=123。**

```go
func (c *Context) GetQuery(key string) (string, bool)
func (c *Context) Query(key string) string
func (c *Context) DefaultQuery(key, defaultValue string) string
```

代码示例：

```go
r.GET("/user", func(c *gin.Context) {
    id,_ := c.GetQuery("id")
    //id := c.Query("id")
    //id := c.DefaultQuery("id","10")
    c.JSON(200,id)
})

请求：http://localhost:8080/user?id=11

响应：11
```

## 2.2 获取form表单参数

```go
func (c *Context) PostForm(key string) string
func (c *Context) PostFormArray(key string) []string
func (c *Context) PostFormMap(key string) map[string]string
func (c *Context) DefaultPostForm(key, defaultValue string) string
func (c *Context) GetPostForm(key string) (string, bool)
func (c *Context) GetPostFormArray(key string) ([]string, bool)
func (c *Context) GetPostFormMap(key string) (map[string]string, bool)
func (c *Context) GetRawData() ([]byte, error) s
```

**代码示例：**

- **index.html中表单：**

  ```html
  <!--表单内容，action设置请求到的地址， method设置请求方式-->
  <form action="http://127.0.0.1:8000/user/info" method="post" enctype="application/x-www-form-urlencoded">
      用户名:<input type="text" name="username" value="zhaobin"><br>
      密&nbsp&nbsp码:<input type="text" name="password" value="123456"><br>
      <!--一个复选框-->
      <input type="checkbox" name="info" value="recommend" checked="checked">推荐
      <input type="checkbox" name="info" value="game" checked="checked"> 游戏
      <input type="checkbox" name="info" value="video"> 影视
      提交：<input type="submit" value="Submit">
  </form>

* **后端处理逻辑：**

  ```go
  package main
  
  import (
  	"github.com/gin-gonic/gin"
  	"net/http"
  )
  
  func main()  {
  	// 创建一个默认的路由引擎
  	r := gin.Default()
  	// GET：请求方式； /hello：请求的路径
  	r.POST("/user/info", func(c *gin.Context) {
  		//提交单个表单数据时
                  //如果没有在请求中获取到表单参数，则返回默认值"张三"
  		username := c.DefaultPostForm("username", "张三")
                  //如果没有在请求中获取到表单参宿，则返回""空串
  		password := c.PostForm("password")      
                  //提交复选框多个数据时
  		info := c.PostFormArray("info")
  		// 输出json结果给调用方
  		c.JSON(http.StatusOK, gin.H{
  			"message": "success",
  			"username": username,
  			"password": password,
              "info": info,
  		})
  
  	})
  	// 启动HTTP服务，默认在8080端口启动服务，也可以设置为其他端口如8000
  	r.Run(":8000")
  }
  ```

* **点击提交浏览器返回结果：**

  ![image-20220423124322389](https://blog.zhaobincode.cn/blogimages/202204231243513.png)

## 2.3 获取JSON参数

```go
func (c *Context) GetRawData() ([]byte, error) s	// 从c.从c.Request.Body读取请求数据
```

代码示例：

- 使用postman配置json请求如下：

  ![image-20220423124351198](https://blog.zhaobincode.cn/blogimages/202204231243265.png)

- 后端处理逻辑：

  ```go
  func main()  {
  	// 创建一个默认的路由引擎
  	r := gin.Default()
  	r.POST("/json", func(c *gin.Context) {
  		// 注意：下面为了举例子方便，暂时忽略了错误处理
  		b, _ := c.GetRawData() // 从c.Request.Body读取请求数据
  		// 定义map或结构体
  		var m map[string]interface{}
  		// 反序列化
  		_ = json.Unmarshal(b, &m)
  		c.JSON(http.StatusOK, m)
  	})
  	// 启动HTTP服务，默认在8080端口启动服务，也可以设置为其他端口如8000
  	r.Run(":8000")
  }
  ```

- 返回结果如下所示：

  ![image-20220328011822720](https://blog.zhaobincode.cn/blogimages/202204231244172.png)

## 2.4 获取path参数

**path是指请求的url中域名之后从/开始的部分，如掘地址：`localhost:8080/timeline`，`/timeline`部分便是path，可以使用gin.Context中的`Param()`方法获取这部分参数**

```go
func (c *Context) Param(key string) string
```

- **浏览器输入为：`127.0.0.1:8000/user/info/张三/123456`**

- 后端处理逻辑如下：

  ```go
  func main()  {
  	// 创建一个默认的路由引擎
  	r := gin.Default()
  	r.GET("/user/info/:username/:password", func(c *gin.Context) {
  		// 如果没有获取到相关路径参数，则返回""空串
  		username := c.Param("username")
  		password := c.Param("password")
  		//返回结果给调用方
  		c.JSON(http.StatusOK, gin.H{
  			"message": "success",
  			"username": username,
  			"password": password,
  		})
  	})
  	r.Run(":8080")
  }
  ```

  

## 2.5 数据绑定

为了能够更方便的获取请求相关参数，提高开发效率，我们可以**基于请求的`Content-Type`识别请求数据类型并利用反射机制自动提取请求中`QueryString`、`form表单`、`JSON`、`XML`等参数到结构体中。**



**Gin提供了两类绑定方法：**

* **Must bind：**

  * **Methods：**

    **`Bind, BindJSON, BindXML, BindQuery, BindYAML`**

  * **Behavior：**

    这些方法属于MustBindWith的具体调用. 如果发生绑定错误, 则请求终止，并触发 `c.AbortWithError(400, err).SetType(ErrorTypeBind)`响应状态码被设置为 400 并且Content-Type被设置为`text/plain; charset=utf-8`. 如果您在此之后尝试设置响应状态码, Gin会输出日志`[GIN-debug] [WARNING] Headers were already written. Wanted to override status code 400 with 422`. 如果您希望更好地控制绑定, 考虑使用ShouldBind等效方法.

* **Should bind：**

  - **Methods：**

    **`ShouldBind, ShouldBindJSON, ShouldBindXML, ShouldBindQuery, ShouldBindYAML`**

  - **Behavior：**

    **这些方法属于ShouldBindWith的具体调用. 如果发生绑定错误, Gin 会返回错误并由开发者处理错误和请求。**



### 2.5.1 以ShouldBind为前缀的系列方法

<font color='blue' size='4'>**Path：**</font>

```go
func (c *Context) ShouldBindUri(obj interface{}) error
```

代码示例：

* 服务端代码：

  ```go
  type User struct {
      Uid      int    
      Username string `uri:"username"` //使用tag字段，此时uri中设置的占位符为username
      Password string 
  }
  
  func main() {
  	//Default返回一个默认的路由引擎
  	r := gin.Default()
  	//password和struct中字段大小写不一致，则绑定不成功
  	r.GET("/bind/:Uid/:username/:password", func(c *gin.Context) {
  		var u User
  		error := c.BindUri(&u)
  		if error == nil {
  			c.JSON(200, u)
  		}
  	})
  }
  ```

* 使用Postman测试：

  ![image-20220328104626856](https://blog.zhaobincode.cn/blogimages/202204231244152.png)

<font color='blue' size='4'>**Body：**</font>

```go
//ShouldBind() 方法, 根据请求方式，自动提取 JSON, form表单 和 Query 类型的参数，放入结构体中
func (c *Context) ShouldBind(obj interface{}) error
//ShouldBindJSON, ShouldBindXML, ShouldBindQuery, ShouldBindYAML等函数只绑定对应格式的参数
func (c *Context) ShouldBindQuery(obj interface{}) error
func (c *Context) ShouldBindJSON(obj interface{}) error
func (c *Context) ShouldBindXML(obj interface{}) error
func (c *Context) ShouldBindYAML(obj interface{}) error
//使用显式绑定声明绑定
func (c *Context) ShouldBindBodyWith(obj interface{}, bb binding.BindingBody) (err error)
func (c *Context) ShouldBindWith(obj interface{}, b binding.Binding) error
```



**代码示例：**

* 服务端代码：

  ```go
  type User struct {
  	Name string `form:"name" json:"name"`
  	Age  int    `form:"age" json:"age"`
  	Addr string `form:"addr" json:"addr"`
  }
  
  func main() {
  	//Default返回一个默认的路由引擎
  	r := gin.Default()
  
  	// query绑定的时候使用tag中form标签
  	r.GET("/query", func(c *gin.Context) {
  		var u User
  		//err := c.ShouldBindQuery(&u)
  		err := c.ShouldBind(&u)
  		if err == nil {
  			c.JSON(200, u)
  		}
  	})
  
  	r.POST("/form", func(c *gin.Context) {
  		var u User
  		//err := c.ShouldBind(&u)
  		err := c.ShouldBind(&u)
  		if err == nil {
  			c.JSON(200, u)
  		}
  	})
  
  	r.POST("/json", func(c *gin.Context) {
  		var u User
  		// ShouldBind()会根据请求的Content-Type自行选择绑定器
  		err := c.ShouldBindJSON(&u)
  		//err := c.ShouldBindJSON(&u)	//也可以使用ShouldBindJSON
  		if err == nil {
  			c.JSON(200, u)
  		}
  	})
  	r.Run()
  }
  ```

* 使用Postman测试：

  ![image-20220328111146975](https://blog.zhaobincode.cn/blogimages/202204231244566.png)

  
  

`ShouldBind`会按照下面的顺序解析请求中的数据完成绑定：

1. 如果是 `GET` 请求，只使用 `Form` 绑定引擎（`query`）。
2. 如果是 `POST` 请求，首先检查 `content-type` 是否为 `JSON` 或 `XML`，然后再使用 `Form`（`form-data`）。

**注意：使用`Bind`方法，结构体需要先设置好`tag`才行**

```go
type Login struct {
   // binding:"required" 若接收为空值，则报错
   User    string `form:"username" json:"user" uri:"user" xml:"user" binding:"required"`
   Pssword string `form:"password" json:"password" uri:"password" xml:"password" binding:"required"`
}
```



### 2.5.2 以Bind为前缀的系列方法

以Bind为前缀的相应的方法与以Bind为前缀的方法使用基本相同

```go
//Path
func (c *Context) BindUri(obj interface{}) error
//Query
func (c *Context) BindQuery(obj interface{}) error
//Body
func (c *Context) BindJSON(obj interface{}) error
func (c *Context) BindXML(obj interface{}) error
func (c *Context) BindYAML(obj interface{}) error
func (c *Context) Bind(obj interface{}) error	//Bind()方法会自动根据Content-Type的值选择不同的绑定类型

//也可以使用下面两个方法自行选择绑定类型
func (c *Context) BindWith(obj interface{}, b binding.Binding) error
func (c *Context) MustBindWith(obj interface{}, b binding.Binding) error
```

**部分代码示例：**

```go
r.GET("/bind/:uid/:username", func(c *gin.Context) {
    var u User
    e := c.BindUri(&u)
    if e == nil{
        c.JSON(200,u)
    }
})

//自行选择绑定类型
r.POST("bind",func(c *gin.Context){
	u := User{}
	c.BindWith(&u,binding.JSON)
    c.MustBindWith(&u,binding.JSON)
})
```



# 三、数据响应

**Gin请求的方法，回调用函数的参数为`*gin.Context`,Gin框架在`*gin.Context`实例中封装了所以处理请求并响应客户端的方法，Gin支持多种响应方法，包括我们常见的`String`,`HTML`,`JSON`,`XML`,`YAML`,`JSONP`,也支持直接响应`Reader`和`[]byte`，而且还支持重定向。**

**gin.Context中响应客户端的方法列表：**

```go
// HTML 呈现指定文件名的 HTTP 模板。
// 更新 HTTP 状态代码并将 Content-Type设置为 “text/html”。
// 参见 http://golang.org/doc/articles/wiki/
func (c *Context) HTML(code int, name string, obj interface{})

// IndentedJSON 将给定结构序列化为漂亮的 JSON（缩进+结束行）到响应体中。
// 同时将 Content-Type设置为 “application/json”。
// 警告：建议仅将此用于开发目的，因为打印漂亮的 JSON 会占用更多 CPU 和带宽。
// 请改用 Context.JSON() 。
func (c *Context) IndentedJSON(code int, obj interface{})

// SecureJSON 将给定结构序列化为安全 JSON 到响应主体中。
// 如果给定的结构是数组值，则默认将 “while（1）” 添加到响应主体。
// 将 Content-Type设置为 “application/json” 。
func (c *Context) SecureJSON(code int, obj interface{})

// JSONP 将给定结构序列化为 JSON 到响应体中。
// 它可跨域向服务器请求数据。
// 将 Content-Type 设置为 “application/javascript” 。
func (c *Context) JSONP(code int, obj interface{})

// JSON 将给定结构序列化为 JSON 到响应主中。
// 将 Content-Type 设置为 “application/json” 。
func (c *Context) JSON(code int, obj interface{})

// AsciiJSON 将给定结构作为 JSON 序列化到响应体中，并将 Unicode 序列化为 ASCII 字符串。
// 将 Content-Type 设置为 “application/json” 。
func (c *Context) AsciiJSON(code int, obj interface{})

// PureJSON 将给定结构序列化为 JSON 到响应体中。
// 与 JSON 不同，PureJSON 不会用他们的 Unicode 实体替换特殊的 HTML 字符。
func (c *Context) PureJSON(code int, obj interface{})

// XML 将给定结构序列化为 XML 到响应体中。
// 将 Content-Type 设置为 “application/xml” 。
func (c *Context) XML(code int, obj interface{})

// YAML 将给定结构序列化为 YAML 到响应体中。
func (c *Context) YAML(code int, obj interface{})

// ProtoBuf 将给定结构序列化为 ProtoBuf 到响应体中。
func (c *Context) ProtoBuf(code int, obj interface{})

// String 将给定字符串写入到响应体中。
func (c *Context) String(code int, format string, values ...interface{})

// Redirect 将 HTTP 重定向返回到特定位置。
func (c *Context) Redirect(code int, location string)

// Data 将一些数据写入正文流并更新 HTTP 状态代码。
func (c *Context) Data(code int, contentType string, data []byte)

// DataFromReader 将指定的读取器写入正文流并更新 HTTP 代码。
func (c *Context) DataFromReader(code int, contentLength int64, contentType string, reader io.Reader, extraHeaders map[string]string)

// File 以有效的方式将指定的文件写入正文流。
func (c *Context) File(filepath string)

// FileAttachment 以有效的方式将指定的文件写入正文流中在客户端，
// 通常会使用给定的文件名下载该文件。
func (c *Context) FileAttachment(filepath, filename string)

// SSEvent 将 Server-Sent 事件写入正文流。
func (c *Context) SSEvent(name string, message interface{})

// Stream 发送流响应并返回布尔值，表示“客户端在流中间断开连接”。
func (c *Context) Stream(step func(w io.Writer) bool) bool

```



## 3.1 HTML渲染

Gin也支持传统Web编程中的HTML模板渲染，直接返回HTML代码给客户端，主要步骤：**定义模板文件、解析模板文件和模板渲染。**



**使用gin.Engine中的`LoadHTMLFiles()`或`LoadHTMLGlob()`方法解析模板：**

* LoadHTMLFiles(files ...string)方法可以接收一个或多个参数，用于加载单个或多个模板文件。
* LoadHTMLGlob(pattern string)方法则用于加载整个目录下的模板文件，如果目录不存在或目录下没有模板文件会引发panic错误。

**代码示例：**

* 项目结构：

  ![image-20220326181253700](https://blog.zhaobincode.cn/blogimages/202204231245157.png)

* 我们首先定义一个存放模板文件的`templates`文件夹，然后在其内部按照业务分别定义一个`posts`文件夹和一个`users`文件夹。 

  * `posts/index.html`文件的内容如下：

    ```html
    {{define "posts/index.html"}}
    <!DOCTYPE html>
    <html lang="en">
    
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <meta http-equiv="X-UA-Compatible" content="ie=edge">
        <title>posts/index</title>
    </head>
    <body>
        {{.title}}
    </body>
    </html>
    {{end}}
    ```

  * `users/index.html`文件的内容如下：

    ```html
    {{define "users/index.html"}}
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <meta http-equiv="X-UA-Compatible" content="ie=edge">
        <title>users/index</title>
    </head>
    <body>
        {{.title}}
    </body>
    </html>
    {{end}}
    ```

* Gin框架中使用`LoadHTMLGlob()`或者`LoadHTMLFiles()`方法进行HTML模板渲染。

  ```go
  import (
  	"github.com/gin-gonic/gin"
  	"net/http"
  )
  
  func main() {
  	r := gin.Default()
  	// 模板解析
  	r.LoadHTMLGlob("templates/**/*") //两个星表示目录，一个星表示文件
  	//r.LoadHTMLFiles("templates/posts/index.tmpl", "templates/users/index.tmpl")
  	r.GET("/posts/index", func(c *gin.Context) {
  		//HTTP请求
  		c.HTML(http.StatusOK, "posts/index.html", gin.H{ // 模板渲染
  			"title": "posts/index",
  		})
  	})
  	r.GET("/users/index", func(c *gin.Context) {
  		//HTTP请求
  		c.HTML(http.StatusOK, "users/index.html", gin.H{ // 模板渲染
  			"title": "users/index",
  		})
  	})
  	r.Run(":8080")
  }
  ```

* 访问`http://localhost:8080/posts/index` 和 `http://localhost:8080/users/index`的显示结果：

  ![image-20220326181329042](https://blog.zhaobincode.cn/blogimages/202204231245139.png)

### 3.1 静态文件处理

`Static()` 方法指定某个目录为静态资源目录。

如：

```go
r.Static("/statics", "./statics")
```

**说明：/statics会匹配以/statics开头的路径，当浏览器请求index.html页面中的style.css文件时，/statics前缀会被替换为./statics,然后去 ./statics目录下查找style.css文件**

**代码示例：**

* 项目结构：

  ![image-20220328151336342](https://blog.zhaobincode.cn/blogimages/202204231245709.png)

* posts/inde.tmpl代码：

  ```html
  {{define "posts/index.html"}}
      <!DOCTYPE html>
      <html lang="en">
  
      <head>
          <meta charset="UTF-8">
          <meta name="viewport" content="width=device-width, initial-scale=1.0">
          <meta http-equiv="X-UA-Compatible" content="ie=edge">
          <link rel="stylesheet" href="/statics/index.css">
          <title>posts/index</title>
      </head>
      <body>
      {{.title}}
      </body>
      </html>
  {{end}}
  ```

* 后端代码使用`r.Static("/statics", "./statics")`处理静态资源：

  ```go
  package main
  
  import (
  	"github.com/gin-gonic/gin"
  	"net/http"
  )
  
  // 静态文件：html页面上用到的样式文件: css文件 js文件 图片
  
  func main() {
  	r := gin.Default()
  	//加载静态文件
  	r.Static("/statics", "./statics")
  
  	// 模板解析
  	r.LoadHTMLGlob("templates/**/*") //两个星表示目录，一个星表示文件
  	//r.LoadHTMLFiles("templates/posts/index.tmpl", "templates/users/index.tmpl")
  	r.GET("/posts/index", func(c *gin.Context) {
  		//HTTP请求
  		c.HTML(http.StatusOK, "posts/index.html", gin.H{ // 模板渲染
  			"title": "posts/index",
  		})
  	})
  	r.GET("/users/index", func(c *gin.Context) {
  		//HTTP请求
  		c.HTML(http.StatusOK, "users/index.html", gin.H{ // 模板渲染
  			"title": "users/index",
  		})
  	})
  	r.Run(":8080")
  }
  ```

  



## 3.2 JSON/XML/YAML/String响应

**Gin提供的响应JSON、XML、YAML或String格式的相关函数：**

```go
// JSON 将给定结构序列化为 JSON 到响应主中。
// 将 Content-Type 设置为 “application/json” 。
func (c *Context) JSON(code int, obj interface{})

// XML 将给定结构序列化为 XML 到响应体中。
// 将 Content-Type 设置为 “application/xml” 。
func (c *Context) XML(code int, obj interface{})

// YAML 将给定结构序列化为 YAML 到响应体中。
func (c *Context) YAML(code int, obj interface{})

// String 将给定字符串写入到响应体中。
func (c *Context) String(code int, format string, values ...interface{})
```

**代码示例：**

* 服务端代码：

  ```go
  func main() {
  	router := gin.Default()
  
  	router.GET("/someJSON", func(c *gin.Context) {
  		// 也可使用一个结构体
  		var msg struct {
  			Name     string `json:"user"`
  			UserInfo string
  			Number   int
  		}
  		msg.Name = "Lena"
  		msg.UserInfo = "Mo"
  		msg.Number = 123
  		// 注意 msg.Name 在 JSON 中变成了 "user"
  		c.JSON(http.StatusOK, msg)
  	})
  
  	router.GET("/someXML", func(c *gin.Context) {
  		c.XML(http.StatusOK, gin.H{"userinfo": "Mo", "status": http.StatusOK})
  	})
  
  	router.GET("/someYAML", func(c *gin.Context) {
  		c.YAML(http.StatusOK, gin.H{"userinfo": "Mo", "status": http.StatusOK})
  	})
  
  	router.GET("/someString", func(c *gin.Context) {
  		c.String(200, "Hello World")
  	})
  
  	// 监听并启动服务
  	router.Run(":8080")
  }
  ```

* 使用Postman测试：

  ![image-20220328115202472](https://blog.zhaobincode.cn/blogimages/202204231245644.png)

## 3.3 ProtoBuf响应

Protobuf是一种与平台无关和语言无关，且可扩展且轻便高效的序列化数据结构的协议，可以用于网络通信和数据存储，其实序列化的速度要比JSON和XML快，但其易用性和可阅读性远不如JSON和XML，因此并没有广泛使用。

不过Gin也为Protobuf数据响应提供了支持，以下例子来自Gin官方文档：

```go
r.GET("/someProtoBuf", func(c *gin.Context) {
    reps := []int64{int64(1), int64(2)}
    label := "test"
    // protobuf 的具体定义写在 testdata/protoexample 文件中。
    data := &protoexample.Test{
        Label: &label,
        Reps:  reps,
    }
    // 请注意，数据在响应中变为二进制数据
    // 将输出被 protoexample.Test protobuf 序列化了的数据
    c.ProtoBuf(http.StatusOK, data)
})
```



## 3.4 重定向

**Gin框架也支持重定向操作，重定向分为外部和内部重定向。**

<font color='blue' size='4'>**外部重定向：**</font>

* 服务器无法处理浏览器发送过来的请求（request），服务器告诉浏览器跳转到可以处理请求的url上。

* 常见的 301 重定向，用于跳转其他外部的链接

```go
r.GET("/test", func(c *gin.Context) {
	c.Redirect(http.StatusMovedPermanently, "https://www.baidu.com")
})
```

<font color='blue' size='4'>**内部重定向：**</font>

也叫路由重定向（类似于请求转发），使用`HandleContext`方法来重新调用其他的 `Handler` ：

```go
r.GET("/test", func(c *gin.Context) {
    // 指定重定向的URL
    c.Request.URL.Path = "/test2"
    r.HandleContext(c)
})
r.GET("/test2", func(c *gin.Context) {
    c.JSON(http.StatusOK, gin.H{"hello": "world"})
})
```

访问：`localhost:8080/test`浏览器地址栏地址不变

![image-20220328153000196](https://blog.zhaobincode.cn/blogimages/202204231246043.png)

# 四、文件上传

> 好的博客：[一文搞懂gin各种上传文件](https://blog.csdn.net/kuangshp128/article/details/109598786)

## 上传单个文件



项目结构：

![image-20220328161241481](https://blog.zhaobincode.cn/blogimages/202204231246901.png)

**1、模板文件index.html：**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<form action="/upload" method="post" enctype="multipart/form-data">
    <div>
        用户名:<input type="text" name="name" />
    </div>
    <div>
        <input type="file" name="file" />
    </div>
    <div>
        <input type="submit" value="提交"/>
    </div>
</form>
</body>
</html>
```

**2、`gin`后端实现文件上传：使用FormFile(name)返回form表达对应名称的文件**

```go
package main

import (
	"fmt"
	"github.com/gin-gonic/gin"
	"net/http"
	"path"
	"path/filepath"
)

func main() {
	r := gin.Default()
	// 加载模板文件
	r.LoadHTMLFiles("index.html")
	r.GET("/index", func(c *gin.Context) {
		//渲染模板
		c.HTML(http.StatusOK, "index.html", nil)
	})
	r.POST("/upload", func(c *gin.Context) {
		//获取普通文本
		name := c.PostForm("name")
		//获取文件
		file, err := c.FormFile("file")
		if err != nil {
			c.JSON(http.StatusOK, gin.H{
				"message": "获取数据失败",
			})
		} else {
			fmt.Println("接收的name数据：", name)
			//获取文件名称
			fmt.Println("文件名：", file.Filename)
			//文件大小
			fmt.Println("文件大小：", file.Size)
			//获取文件的后缀名
			extstring := path.Ext(file.Filename)
			fmt.Println("文件的后缀名：", extstring)
			//进行文件路径的拼接
			filePath := filepath.Join("./images/", file.Filename)
			//保存上传的文件
			c.SaveUploadedFile(file, filePath)
			c.JSON(http.StatusOK, gin.H{"message": "保存文件成功"})
		}
	})

	r.Run(":8080")
}
```

**3、结果：**

![image-20220328161446516](https://blog.zhaobincode.cn/blogimages/202204231246799.png)

![image-20220328161545465](https://blog.zhaobincode.cn/blogimages/202204231247093.png)



## 上传多个文件



**1、上传多个文件首先在`html`中加上`multiple`属性(前提是后端支持)**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<form action="/uploads" method="post" enctype="multipart/form-data">
    用户名:<input type="text" name="name" /><br/>
    <input type="file" name="file" multiple/>
    <input type="submit" value="提交"/>
</form>
</body>
</html>
```

**2、后端代码**

* `MultipartForm()`方法返回一个form结构体。

  ```go
  type Form struct {
  	Value map[string][]string
  	File  map[string][]*FileHeader
  }
  //包含两个属性，一个存放文件，一个存放value。
  ```

```go
package main

import (
	"fmt"
	"github.com/gin-gonic/gin"
	"net/http"
	"path/filepath"
)

func main() {
	r := gin.Default()
	// 加载模板文件
	r.LoadHTMLFiles("index.html")
	r.GET("/index", func(c *gin.Context) {
		//渲染模板
		c.HTML(http.StatusOK, "index.html", nil)
	})

	r.POST("/uploads", func(c *gin.Context) {
		form, err := c.MultipartForm()
		if err != nil {
			c.JSON(http.StatusOK, gin.H{
				"message": "获取数据失败",
			})
		} else {
			name := form.Value["name"]
			fmt.Println("接收的name数据：", name)
			//获取文件
			files := form.File["file"]
			//循环全部的文件
			for _, file := range files {
				//进行文件路径的拼接
				filePath := filepath.Join("./images/", file.Filename)
				//保存上传的文件
				c.SaveUploadedFile(file, filePath)
			}
			c.JSON(http.StatusOK, gin.H{"message": "保存文件成功"})
		}
	})
	r.Run(":8080")
}
```

**3、结果**

![image-20220328163258653](https://blog.zhaobincode.cn/blogimages/202204231247868.png)

![image-20220328163244743](https://blog.zhaobincode.cn/blogimages/202204231247033.png)





# 五、Gin中间件

> 参考博客：
>
> [Go Web轻量级框架Gin学习系列：中间件使用详解](https://juejin.cn/post/6844903833164857358#heading-9)
>
> [golang学习之gin(七):中间件](https://blog.csdn.net/qq_35709559/article/details/109609174)
>
> [6. Gin中间件](https://zhuanlan.zhihu.com/p/410400470)

## 中间件介绍

* **Gin框架允许开发者在处理请求的过程中，加入用户自己的钩子（Hook）函数。这个钩子函数就叫中间件，中间件适合处理一些公共的业务逻辑，比如登录认证、权限校验、数据分页、记录日志、耗时统计等。**

<font color='blue' size='4'>**Gin中间件的作用：**</font>

* **Web请求到到达我们定义的HTTP请求处理方法之前，拦截请求并进行相应处理(比如：权限验证，数据过滤等)，这个可以类比为`前置拦截器`或`前置过滤器`，**
* **在我们处理完成请求并响应客户端时，拦截响应并进行相应的处理(比如：添加统一响应部头或数据格式等)，这可以类型为`后置拦截器`或`后置过滤器`。**



## 中间件的使用

### 全局使用中间件

**直拉使用`gin.Engine`结构体的`Use()`方法便可以在所有请求应用中间件，这样做，中间件便会在全局起作用。**

```go
router := gin.New()
router.Use(gin.Recovery())//在全局使用内置中间件
```

### 自定义中间件

```go
// 自定义中间件第1种定义方式
func MiddleWare1(ctx *gin.Context)  {
    fmt.Println("这是自定义中间件1")
}

// 自定义中间件第2种定义方式
func MiddleWare2() gin.HandlerFunc  {
    return func(ctx *gin.Context) {
        fmt.Println("这是自定义中间件2")
    }
}

func main() {
	router := gin.Default()
    //全局注册中间件函数
    router.Use(MiddleWare1)      // 需要加括号
    router.Use(MiddleWare2())    // 不需要加括号，当成参数
	r.GET("/index", xxxx)
	r.Run(":8080")
}
```



### Gin内置中间件

**在使用Gin框架开发Web应用时，常常需要自定义中间件，不过，Gin也内置一些中间件，我们可以直接使用，下面是内置中间件列表：**

```go
// gin自带默认有这些中间件
func BasicAuth(accounts Accounts) HandlerFunc
func BasicAuthForRealm(accounts Accounts, realm string) HandlerFunc
func Bind(val interface{}) HandlerFunc //拦截请求参数并进行绑定
func ErrorLogger() HandlerFunc       //错误日志处理
func ErrorLoggerT(typ ErrorType) HandlerFunc //自定义类型的错误日志处理
func Logger() HandlerFunc //日志记录
func LoggerWithConfig(conf LoggerConfig) HandlerFunc
func LoggerWithFormatter(f LogFormatter) HandlerFunc
func LoggerWithWriter(out io.Writer, notlogged ...string) HandlerFunc
func Recovery() HandlerFunc
func RecoveryWithWriter(out io.Writer) HandlerFunc
func WrapF(f http.HandlerFunc) HandlerFunc //将http.HandlerFunc包装成中间件
func WrapH(h http.Handler) HandlerFunc //将http.Handler包装成中间件
```

### 不使用默认中间件

**使用`gin.Default()`返回的`gin.Engine`时，已经默认使用了`Recovery`和`Logger`中间件，从下面`gin.Default()`方法的源码可以看出：**

```go
func Default() *Engine {
    debugPrintWARNINGDefault()
    engine := New()
    engine.Use(Logger(), Recovery())//使用Recovery和Logger中间件
    return engine
}
```

**当我们不想使用这两个中间件时，可以使用gin.New()方法返回一个不带中间件的gin.Engine对象：**

```go
//去除默认全局中间件
router := gin.New()//不带中间件
```

### 某个路由使用中间件

**单个请求路由，也可以应用中间件，如下：**

```go
router := gin.New()
router.GET("/test",gin.Recovery(),func(c *gin.Context){
    c.JSON(200,"test")
})
```

**也可以在单个路由中使用多个中间件，如下：**

```go
router := gin.New()
router.GET("/test",gin.Recovery(),gin.Logger(),func(c *gin.Context){
    c.JSON(200,"test")
})
```

### 路由分组使用中间件

**根据业务不同划分不同`路由分组(RouterGroup )`,不同的路由分组再应用不同的中间件，这样就达到了不同的请求由不同的中间件进行拦截处理。**

```go
router := gin.New()
user := router.Group("user", gin.Logger(),gin.Recovery())
{
    user.GET("info", func(context *gin.Context) {

    })
    user.GET("article", func(context *gin.Context) {

    })
}
```




## 中间件中的Next()和Abort()

<font color='blue' size='4'>**Next()：**</font>

在中间件调用`Next()`方法，`Next()`方法之前的代码会在到达请求方法前执行，`Next()`方法之后的代码则在请求方法处理后执行：

```go
func MyMiddleware(c *gin.Context){
    //请求前
    c.Next()
    //请求后
}
```



<font color='blue' size='4'>**Abort()：终止调用整个链条**</font>

**中间件的作用包括拦截过滤请求，比如我们有些请求需要用户登录或者需要特定权限才能访问，这时候便可以中间件中做过滤拦截，当用户请求不合法时，可以使用下面列出的`gin.Context`的几个方法中断用户请求：**

* 下面三个方法中断请求后，直接返回200，但响应的body中不会有数据。

  ```go
  func (c *Context) Abort()
  func (c *Context) AbortWithError(code int, err error) *Error
  func (c *Context) AbortWithStatus(code int)
  ```

* 使用AbortWithStatusJSON()方法，中断用户请求后，则可以返回json格式的数据。

  ```go
  func (c *Context) AbortWithStatusJSON(code int, jsonObj interface{})
  ```



<font color='blue' size='4'>**中间件执行顺序：**</font>

处理请求代码：

```go
func MiddleWare1(ctx *gin.Context) {

	fmt.Println("这是自定义中间件1--开始")
	ctx.Next()
	fmt.Println("这是自定义中间件1--结束")
}

func MiddleWare2() gin.HandlerFunc {

	return func(ctx *gin.Context) {
		fmt.Println("这是自定义中间件2--开始")
		age, _ := strconv.Atoi(ctx.Query("age"))
		fmt.Println("年龄是：", age)
		if age <= 18 { // 满足条件
			ctx.JSON(http.StatusOK, gin.H{
				"message": "不好意思您未成年",
			})
			ctx.Abort()
		}
		ctx.Next()
		fmt.Println("这是自定义中间件2--结束")
	}
}

func MiddleWare3(ctx *gin.Context) {
	fmt.Println("这是自定义中间件3--开始")
	ctx.Next()
	fmt.Println("这是自定义中间件3--结束")
}

func main() {
	router := gin.Default()
	router.Use(MiddleWare1, MiddleWare2(), MiddleWare3)
	router.GET("/index", func(ctx *gin.Context) {
		ctx.JSON(http.StatusOK, gin.H{
			"message": "已经成年",
		})
	})
	router.Run(":8080")
}
```

访问`ttp://localhost:8080/index?age=12`结果：





![image-20220328180126105](https://blog.zhaobincode.cn/blogimages/202204231248231.png)

访问`http://localhost:8080/index?age=12`结果：

![image-20220328180217837](https://blog.zhaobincode.cn/blogimages/202204231248904.png)

## 数据传递

`gin.Context`中的`Set()`通过一个key来存储作何类型的数据，实现了跨中间件取值。

```go
func (c *Context) Set(key string, value interface{})
```

可以使用下面列出的方法通过key获取对应数据。

* 其中，gin.Context的Get方法返回`interface{}`，通过返回exists可以判断key是否存在。

  ```go
  func (c *Context) Get(key string) (value interface{}, exists bool)
  ```

* 当我们确定通过Set方法设置对应数据类型的值时，可以使用下面方法获取应数据类型的值。

  ```go
  func (c *Context) GetBool(key string) (b bool)
  func (c *Context) GetDuration(key string) (d time.Duration)
  func (c *Context) GetFloat64(key string) (f64 float64)
  func (c *Context) GetInt(key string) (i int)
  func (c *Context) GetInt64(key string) (i64 int64)
  func (c *Context) GetString(key string) (s string)
  func (c *Context) GetStringMap(key string) (sm map[string]interface{})
  func (c *Context) GetStringMapString(key string) (sms map[string]string)
  func (c *Context) GetStringMapStringSlice(key string) (smss map[string][]string)
  func (c *Context) GetStringSlice(key string) (ss []string)
  func (c *Context) GetTime(key string) (t time.Time)
  ```



**示例代码：**

* 后端代码：

  ```go
  // 自定义中间件
  func MyMiddleware(c *gin.Context) {
  	c.Set("mykey", 10)
  }
  
  func main() {
  	router := gin.Default()
  	router.GET("test", MyMiddleware, func(c *gin.Context) {
  		value := c.GetInt("mykey") //我们知道设置进行的是整型，所以使用GetInt方法来获取
  		c.JSON(http.StatusOK, gin.H{"value": value})
  	})
  	router.Run()
  }
  ```

* 访问`http://localhost:8080/test`显示结果

  ![image-20220328213120794](https://blog.zhaobincode.cn/blogimages/202204231250055.png)



**gin中间件中使用goroutine：**

* **当在中间件或 handler 中启动新的 goroutine 时，不能使用原始的上下文（c *gin.Context）， 必须使用其只读副本（c.Copy()）**

  ```go
  r.GET("/", func(c *gin.Context) {
  	cCp := c.Copy()
  	go func() {
  		// simulate a long task with time.Sleep(). 5 seconds
  		time.Sleep(5 * time.Second)
  		// 这里使用你创建的副本
  		fmt.Println("Done! in path " + cCp.Request.URL.Path)
  	}()
  	c.String(200, "首页")
  })
  ```

  



# 六、路由分组

## 基本路由

**<font color='blue' size='4'>支持HTTP的方法：</font>**

**定义路由是为了处理HTTP请求，而HTTP请求包含不同方法，包括`GET`,`POST`,`PUT`,`PATCH`,`OPTIONS`,`HEAD`,`DELETE`等七种方法，Gin框架中都有对应的方法来定义路由。**

```go
router := gin.New()

router.GET("/testGet",func(c *gin.Context){
    //处理逻辑
})

router.POST("/testPost",func(c *gin.Context){
    //处理逻辑
})

router.PUT("/testPut",func(c *gin.Context){
    //处理逻辑
})

router.DELETE("/testDelete",func(c *gin.Context){
    //处理逻辑
})

router.PATCH("/testPatch",func(c *gin.Context){
    //处理逻辑
})

router.OPTIONS("/testOptions",func(c *gin.Context){
    //处理逻辑
})

router.OPTIONS("/testHead",func(c *gin.Context){
    //处理逻辑
})
```

**上面通过对应方法创建的路由，只能处理对应请求方法，如果GET定义的路由无法处理POST请求，我们可以通过`Any()`方法定义可以处理任何请求的路由。**

```go
//可以处理GET,POST等各种请求
router.Any("/testAny",func(c *gin.Context){
    //处理逻辑
})
```

除了上面几种简便的方法，也可以使用`Handle()`创建路由，通过指定`Handle()`函数的第一个参数来确定处理何种请求：

```go
//定义处理POST请求的方法
router.Handle("POST","/testHandlePost",func(c *gin.Context){
    
})
//定义处理GET请求的方法
router.Handle("GET","/testHandleGet",func(c *gin.Context){
    
})
```



<font color='blue' size='4'>**RESTful风格 API：**</font>

- `GET`用来获取资源
- `POST`用来新建资源
- `PUT`用来更新资源
- `DELETE`用来删除资源。

**之前的设计模式：**

| 请求方法 |     URL      |     含义     |
| :------: | :----------: | :----------: |
|   GET    |    /book     | 查询书籍信息 |
|   POST   | /create_book | 创建书籍记录 |
|   POST   | /update_book | 更新书籍信息 |
|   POST   | /delete_book | 删除书籍信息 |

**同样的需求我们按照RESTful API设计如下：**

| 请求方法 |  URL  |     含义     |
| :------: | :---: | :----------: |
|   GET    | /book | 查询书籍信息 |
|   POST   | /book | 创建书籍记录 |
|   PUT    | /book | 更新书籍信息 |
|  DELETE  | /book | 删除书籍信息 |



## 路由请求路径

除了直接匹配路径，Gin框架还支持使用通配符冒号(:)和星号(*)来匹配请求路径，如：

**使用冒号(:)定义路由路径：**

```go
router.GET("user/:name",func(c *gin.Context){
    
})
```

上面的请求路径，请求结果对应如下：

```
/user/gordon              匹配
/user/you                 匹配
/user/gordon/profile      不匹配
/user/                    不匹配
```

**使用星号(*)定义路由请求路径：**

```
router.GET("user/*name",func(c *gin.Context){
    
})
```

上面的请求路径，请求结果对应如下：

```
/user/gordon              匹配
/user/you                 匹配
/user/gordon/profile      匹配
/user/                    匹配
```



## 路由分组

**路由分组用于将多个路由进行统一的处理，例如统一的前缀，统一的中间件等。例如，在做需要认证的业务逻辑时，大量的路由需要经过认证校验后才能交由请求处理器来完成业务逻辑，此时就可以将大量的路由定义在一个组中，集中的设置这个用于认证校验的中间件。**

- **创建分组，使用函数 `router.Group()` 完成分组的创建。创建时可以提供路径前缀和公用中间件，`router.Group()` 函数签名如下：**

  ```go
  func (group *RouterGroup) Group(relativePath string, handlers ...HandlerFunc) *RouterGroup
  ```

代码示例：

```go
package main

import (
	"github.com/gin-gonic/gin"
)

func main() {
	// 创建一个路由引擎
	r := gin.Default()
	// 处理请求路由，处理此路径下的请求
	// 简单的路由组: v1
	v1 := r.Group("/v1")
	{
		v1.POST("/login", loginEndpoint)
		v1.POST("/submit", submitEndpoint)
		v1.POST("/read", readEndpoint)
	}

	// 简单的路由组: v2
	v2 := r.Group("/v2")
	{
		v2.POST("/login", loginEndpoint)
		v2.POST("/submit", submitEndpoint)
		v2.POST("/read", readEndpoint)
                v1 := r.Group("/v1")  // 路由嵌套
	}
	// 启动HTTP服务，默认在8080端口启动服务，也可以设置为其他端口如8000
	r.Run(":8000")
}
```





# Cookie和Session

## Cookie

* **gin 框架获取 cookie 键值的方法：**

  ```go
  func (c *Context) Cookie(key string) (value string, err error) 
  ```

  其中 key 为 cookie 键，value 为返回的对应值。

* **gin 框架写入 cookie 键值的方法：**

  ```go
  func (c *Context) SetCookie(key, value string, maxAge int, path, domain string, secure, httpOnly bool)
  ```

  其中 key 为 cookie 键，value 为设置的对应值。

**代码示例：**

```go
func main() {
    engine := gin.Default()

    // 读取 cookie
    engine.GET("/read_cookie", func(context *gin.Context) {
    val, _ := context.Cookie("name")
      context.String(200, "Cookie:%s", val)
    })

    // 写入 cookie
    engine.GET("/write_cookie", func(context *gin.Context) {
       context.SetCookie("name", "Shimin Li", 24*60*60, "/", "localhost", false, true)
    })

    // 清理 cookie
    engine.GET("/clear_cookie", func(context *gin.Context) {
        //删除cookie则设置过期时间为-1
      context.SetCookie("name", "Shimin Li", -1, "/", "localhost", false, true)
    })

    engine.Run()
}
```



## Session

**go 语言 和 gin 框架都没有单独提供 session 对象或者操作方法。通常我们使用 gorilla/sessions包，它是由第三方提供的 session 操作包**

**官方网址：http://www.gorillatoolkit.org/pkg/sessions**

**github：https://github.com/gin-gonic/gin**

### 基本的session用法

```go
func main() {
	// 创建一个默认的路由引擎
	r := gin.Default()
	// 创建基于 cookie 的存储引擎，secret11111 参数是用于加密的密钥
	store := cookie.NewStore([]byte("secret111"))
	// 设置 session 中间件，参数 mysession，指的是 session 的名字，也是 cookie 的名字
	// store 是前面创建的存储引擎，我们可以替换成其他存储引擎
	r.Use(sessions.Sessions("mysession", store))
	r.GET("/", func(c *gin.Context) {
		//初始化 session 对象
		session := sessions.Default(c)
		//设置过期时间
		session.Options(sessions.Options{
			MaxAge: 3600 * 6, // 6hrs
		})
		//设置 Session
		session.Set("username", "张三 111")
		// 必须调用Save()方法才能设置成功
		session.Save()
		// 通过 session.Get 读取 session 值
		c.JSON(200, gin.H{"msg": session.Get("username")})
	})
    r.Run(":8080")
}
```

![image-20220423200556178](https://blog.zhaobincode.cn/blogimages/202204232005231.png)

### 基于redis存储引擎的session

```go
func main() {
	// 创建一个默认的路由引擎
	r := gin.Default()
	
	// 初始化基于redis的存储引擎
	// 参数说明：
	//    第1个参数 - redis最大的空闲连接数
	//    第2个参数 - 数通信协议tcp或者udp
	//    第3个参数 - redis地址, 格式，host:port
	//    第4个参数 - redis密码
	//    第5个参数 - session加密密钥
	store, _ := redis.NewStore(10, "tcp", "101.200.241.73:16379", "", []byte("secret"))
	// 设置 session 中间件，参数 mysession，指的是 session 的名字，也是 cookie 的名字
	// store 是前面创建的存储引擎，我们可以替换成其他存储引擎
	r.Use(sessions.Sessions("mysession", store))
	r.GET("/", func(c *gin.Context) {
		//初始化 session 对象
		session := sessions.Default(c)
		//设置过期时间
		session.Options(sessions.Options{
			MaxAge: 3600 * 6, // 6hrs
		})
		//设置 Session
		session.Set("username", "张三 222")
		// 必须调用Save()方法才能设置成功
		session.Save()

		c.JSON(200, gin.H{"msg": session.Get("username")})
	})
    r.Run(":8080")
}
```

![image-20220423200645930](https://blog.zhaobincode.cn/blogimages/202204232006979.png)
