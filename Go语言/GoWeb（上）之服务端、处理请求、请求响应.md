> **好的博客：**[Go HTTP编程](https://www.cnblogs.com/itbsl/p/12175645.html)

# 一、服务端

**Go语言标准库内建提供了net/http包，涵盖了HTTP客户端和服务端的具体实现。**



## 1.1 构建服务器

首先，我们编写一个最简单的Web服务器。编写这个Web服务只需要两步:

* **注册一个处理器函数(注册到DefaultServeMux)；**
* **设置监听的TCP地址并启动服务；**

**代码示例：**

```go
package main

import (
	"fmt"
	"net/http"
)

func handler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintln(w, "正在通过处理器函数处理请求!", r.URL.Path)
}

func main() {
	//1.注册一个给定模式的处理器函数到DefaultServeMux
	http.HandleFunc("/", handler)

	//2.设置监听的TCP地址并启动服务
	//参数1：TCP地址(IP+Port)
	//参数2：当设置为nil时表示使用DefaultServeMux
	err := http.ListenAndServe("127.0.0.1:8080", nil)
	fmt.Println(err)

}

```

运行该程序，通过浏览器访问`localhost:8080`

![image-20220321162021737](https://gitee.com/jobim/blogimage/raw/master/img/20220321162021.png)

* **ListenAndServe使用指定的监听地址和处理器启动一个HTTP服务端。**
  * 如果网络地址参数为空字符串，那么服务器默认使用 80 端口进行网络连接当
  * 处理器参数是nil时，这表示采用包变量DefaultServeMux作为处理器。

* **Handle和HandleFunc函数可以向DefaultServeMux添加任意多个处理器函数。**





<font color='blue' size='4'>**还可以通过 Server 结构对服务器进行更详细的配置：**</font>

* **结构体Server的结构：**

  ```go
  type Server struct {
      Addr           string        // 监听的TCP地址，如果为空字符串会使用":http"
      Handler        Handler       // 调用的处理器，如为nil会调用http.DefaultServeMux
      ReadTimeout    time.Duration // 请求的读取操作在超时前的最大持续时间
      WriteTimeout   time.Duration // 回复的写入操作在超时前的最大持续时间
      MaxHeaderBytes int           // 请求的头域最大长度，如为0则用DefaultMaxHeaderBytes
      TLSConfig      *tls.Config   // 可选的TLS配置，用于ListenAndServeTLS方法
      // TLSNextProto（可选地）指定一个函数来在一个NPN型协议升级出现时接管TLS连接的所有权。
      // 映射的键为商谈的协议名；映射的值为函数，该函数的Handler参数应处理HTTP请求，
      // 并且初始化Handler.ServeHTTP的*Request参数的TLS和RemoteAddr字段（如果未设置）。
      // 连接在函数返回时会自动关闭。
      TLSNextProto map[string]func(*Server, *tls.Conn, Handler)
      // ConnState字段指定一个可选的回调函数，该函数会在一个与客户端的连接改变状态时被调用。
      // 参见ConnState类型和相关常数获取细节。
      ConnState func(net.Conn, ConnState)
      // ErrorLog指定一个可选的日志记录器，用于记录接收连接时的错误和处理器不正常的行为。
      // 如果本字段为nil，日志会通过log包的标准日志记录器写入os.Stderr。
      ErrorLog *log.Logger
      // 内含隐藏或非导出字段
  }
  ```

**自定义Server：**

```go
//定义多个处理器
type h1 struct{}
func (h1 *h1) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "h1")
}

type h2 struct{}
func (h2 *h2) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "h2")
}

func main() {
    h1 := h1{}
    h2 := h2{}
    //nil表明服务器使用默认的多路复用器DefaultServeMux
    server := http.Server{
        Addr : "127.0.0.1:8080",
        Handler : nil,
		ReadTimeout : 2 * time.Second,
    }
    //handle函数实际调用的是多路复用器DefaultServeMux.Handle方法
    http.Handle("/h1", &h1)
    http.Handle("/h2", &h2)

    server.ListenAndServe()
}
```



## 1.2 处理器和处理器函数

### 1.2.1 处理器

* **一个处理器就是实现了Handler这个接口：实现了Handler接口的对象可以注册到HTTP服务端，为特定的路径及其子树提供服务。**

* **那么也就是说，任何接口只要有一个`ServeHTTP`方法，并且该方法带有以下的签名，那么他就是一个处理器**

  ```go
  type Handler interface {
      ServeHTTP(ResponseWriter, *Request)
  }
  ```

* **多路复用器`DefaultServeMux`是一个特殊的处理器，它的任务是根据请求的URL将请求重定向到不同的处理器**

**实现一个处理器：**

```go
package main

import (
	"log"
	"net/http"
)
type MyHandler struct {}

func (m *MyHandler)ServeHTTP(w http.ResponseWriter, r *http.Request) {
	_, _ = w.Write([]byte("Hello World!"))
}
func main() {

	//1.将处理器和URL进行绑定
	http.Handle("/", &MyHandler{})

	//2.设置监听的TCP地址并启动服务
	//参数1：TCP地址(IP+Port)
	//参数2：当设置为nil时表示使用DefaultServeMux
	err := http.ListenAndServe("127.0.0.1:8080", nil)
	log.Fatal(err)
}
```



### 1.2.2 处理器函数

* **处理器函数就是与ServeHTTP方法拥有相同签名的函数。**

* **`HandlerFunc`函数，它可以把一个带有正确签名的函数f转换成一个带有方法f的Handler**

  ```go
  func HandleFunc(pattern string, handler func(ResponseWriter, *Request))
  ```

**使用处理器函数处理请求：**

```go
package main

import (
    "fmt"
    "net/http"
)

func index(w http.ResponseWriter, r *http.Request) {
    hello := "Hello Wrold!"
    fmt.Fprintf(w, "%s\n", hello)
}

func login(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "You are in the login\n")
}

func home(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "You are in the home\n")
}

func news(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "You are in the news\n")
}

func main() {

    server := &http.Server{
        Addr: "0.0.0.0:9090",
    }

    http.HandleFunc("/", index)
    http.HandleFunc("/login", login)
    http.HandleFunc("/home", home)
    http.HandleFunc("/news", news)

    err := server.ListenAndServe()

    if err != nil {
        fmt.Println(err)
    }
}
```

**处理器函数的实现原理：**

通过源码可知，这个函数实际上是调用了默认的DefaultServeMux的HandleFunc方法，即：默认注册到DefaultServeMux中

![image-20220321172919798](https://gitee.com/jobim/blogimage/raw/master/img/20220321172919.png)

## 1.3 多路复用器

<font color='blue' size='4'>**ServeMux和DefaultServeMux：**</font>

* **ServeMux是一个HTTP请求多路复用器，它负责接受HTTP请求并根据请求中的URL将请求重定向到正确的处理器。**

  ![image-20220321174034074](https://gitee.com/jobim/blogimage/raw/master/img/20220321174034.png)

* **ServeMux结构也实现了ServeHTTP方法，所以它也是一个处理器**

* **结构体 ServeMux 的相关方法：**

  ![image-20220321174809593](https://gitee.com/jobim/blogimage/raw/master/img/20220321174809.png)

* **`DefaultServeMux`是ServeMux的一个实例，当用户没有为Server结构指定处理器时，处理器就会使用DefaultServeMux作为ServeMux的默认实例**



**使用NewServeMux创建的多路复用器代码：**

```go
//创建处理器函数
func handler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintln(w, "通过自己创建的多路复用器处理请求!", r.URL.Path)
}

func main() {
	//创建多路复用器
	mux := http.NewServeMux()

	mux.HandleFunc("/", handler)

	//创建路由
	http.ListenAndServe(":8080", mux)
}
```





# 二、处理请求

> 好的博客：[Go Web：处理请求 ](https://www.cnblogs.com/f-ck-need-u/p/10035801.html)

Go 语言的 net/http 包提供了一系列用于表示 HTTP 报文的结构，我们可以使用它处理请求和发送相应，其中 **Request 结构代表了客户端发送的请求报文，下面让我们看一下 Request 结构体**

```go
type Request struct {
    // Method指定HTTP方法（GET、POST、PUT等）。对客户端，""代表GET。
    Method string
    // URL在服务端表示被请求的URI，在客户端表示要访问的URL。
    //
    // 在服务端，URL字段是解析请求行的URI（保存在RequestURI字段）得到的，
    // 对大多数请求来说，除了Path和RawQuery之外的字段都是空字符串。
    // （参见RFC 2616, Section 5.1.2）
    //
    // 在客户端，URL的Host字段指定了要连接的服务器，
    // 而Request的Host字段（可选地）指定要发送的HTTP请求的Host头的值。
    URL *url.URL
    // 接收到的请求的协议版本。本包生产的Request总是使用HTTP/1.1
    Proto      string // "HTTP/1.0"
    ProtoMajor int    // 1
    ProtoMinor int    // 0
    // Header字段用来表示HTTP请求的头域。如果头域（多行键值对格式）为：
    //	accept-encoding: gzip, deflate
    //	Accept-Language: en-us
    //	Connection: keep-alive
    // 则：
    //	Header = map[string][]string{
    //		"Accept-Encoding": {"gzip, deflate"},
    //		"Accept-Language": {"en-us"},
    //		"Connection": {"keep-alive"},
    //	}
    // HTTP规定头域的键名（头名）是大小写敏感的，请求的解析器通过规范化头域的键名来实现这点。
    // 在客户端的请求，可能会被自动添加或重写Header中的特定的头，参见Request.Write方法。
    Header Header
    // Body是请求的主体。
    //
    // 在客户端，如果Body是nil表示该请求没有主体买入GET请求。
    // Client的Transport字段会负责调用Body的Close方法。
    //
    // 在服务端，Body字段总是非nil的；但在没有主体时，读取Body会立刻返回EOF。
    // Server会关闭请求的主体，ServeHTTP处理器不需要关闭Body字段。
    Body io.ReadCloser
    // ContentLength记录相关内容的长度。
    // 如果为-1，表示长度未知，如果>=0，表示可以从Body字段读取ContentLength字节数据。
    // 在客户端，如果Body非nil而该字段为0，表示不知道Body的长度。
    ContentLength int64
    // TransferEncoding按从最外到最里的顺序列出传输编码，空切片表示"identity"编码。
    // 本字段一般会被忽略。当发送或接受请求时，会自动添加或移除"chunked"传输编码。
    TransferEncoding []string
    // Close在服务端指定是否在回复请求后关闭连接，在客户端指定是否在发送请求后关闭连接。
    Close bool
    // 在服务端，Host指定URL会在其上寻找资源的主机。
    // 根据RFC 2616，该值可以是Host头的值，或者URL自身提供的主机名。
    // Host的格式可以是"host:port"。
    //
    // 在客户端，请求的Host字段（可选地）用来重写请求的Host头。
    // 如过该字段为""，Request.Write方法会使用URL字段的Host。
    Host string
    // Form是解析好的表单数据，包括URL字段的query参数和POST或PUT的表单数据。
    // 本字段只有在调用ParseForm后才有效。在客户端，会忽略请求中的本字段而使用Body替代。
    Form url.Values
    // PostForm是解析好的POST或PUT的表单数据。
    // 本字段只有在调用ParseForm后才有效。在客户端，会忽略请求中的本字段而使用Body替代。
    PostForm url.Values
    // MultipartForm是解析好的多部件表单，包括上传的文件。
    // 本字段只有在调用ParseMultipartForm后才有效。
    // 在客户端，会忽略请求中的本字段而使用Body替代。
    MultipartForm *multipart.Form
    // Trailer指定了会在请求主体之后发送的额外的头域。
    //
    // 在服务端，Trailer字段必须初始化为只有trailer键，所有键都对应nil值。
    // （客户端会声明哪些trailer会发送）
    // 在处理器从Body读取时，不能使用本字段。
    // 在从Body的读取返回EOF后，Trailer字段会被更新完毕并包含非nil的值。
    // （如果客户端发送了这些键值对），此时才可以访问本字段。
    //
    // 在客户端，Trail必须初始化为一个包含将要发送的键值对的映射。（值可以是nil或其终值）
    // ContentLength字段必须是0或-1，以启用"chunked"传输编码发送请求。
    // 在开始发送请求后，Trailer可以在读取请求主体期间被修改，
    // 一旦请求主体返回EOF，调用者就不可再修改Trailer。
    //
    // 很少有HTTP客户端、服务端或代理支持HTTP trailer。
    Trailer Header
    // RemoteAddr允许HTTP服务器和其他软件记录该请求的来源地址，一般用于日志。
    // 本字段不是ReadRequest函数填写的，也没有定义格式。
    // 本包的HTTP服务器会在调用处理器之前设置RemoteAddr为"IP:port"格式的地址。
    // 客户端会忽略请求中的RemoteAddr字段。
    RemoteAddr string
    // RequestURI是被客户端发送到服务端的请求的请求行中未修改的请求URI
    // （参见RFC 2616, Section 5.1）
    // 一般应使用URI字段，在客户端设置请求的本字段会导致错误。
    RequestURI string
    // TLS字段允许HTTP服务器和其他软件记录接收到该请求的TLS连接的信息
    // 本字段不是ReadRequest函数填写的。
    // 对启用了TLS的连接，本包的HTTP服务器会在调用处理器之前设置TLS字段，否则将设TLS为nil。
    // 客户端会忽略请求中的TLS字段。
    TLS *tls.ConnectionState
}
```

**一些比较重要的字段：**

- URL：请求 URL
- Method：请求方法
- Proto：HTTP 协议版本
- Header：请求头（字典类型的键值对集合）
- Body：请求实体（实现了 `io.ReadCloser` 接口的只读类型）
- Form、PostForm、MultipartForm：请求表单相关字段，可用于存储表单请求信息

## 2.1 获取请求URL

**URL也是一个结构体：**

```go
type URL struct {
	Scheme 	string
	Opaque 	string		//编码后的不透明数据User
	User	*Userinfo 	//用户名和密码信息
	Host	string		//host或host:port
	Path	string		//表示 HTTP 请求路径
	RawQuery string 	//编码后的查询字符串，没有"?’
	Fragment string 	//引用的片段（文档位置），没有'#'
}
```

* **Path 字段和RawQuery 字段：**

  对于URL：`http://127.0.0.1:8080/hello?username=admin&password=123456`

  * **通过 r.URL.Path 只能得到 /hello**
  * **通过 r.URL.RawQuery 得到的是 username=admin&password=123456**

**URL解析示例：**

```go
func handler(w http.ResponseWriter, r *http.Request) {
    
	fmt.Printf("URL: %#v\n", r.URL)
	fmt.Printf("URL.Path: %#v\n", r.URL.Path)
	fmt.Printf("URL.RawQuery: %#v\n", r.URL.RawQuery)
    
}

func main() {
	http.HandleFunc("/hello", handler)
	
	http.ListenAndServe(":8080", nil)
}
```

访问`http://127.0.0.1:8080/hello?username=admin&password=123456`，执行结果：

![image-20220321221926473](https://gitee.com/jobim/blogimage/raw/master/img/20220321221926.png)



## 2.2 获取请求头中的信息

**通过 Request 结果中的` Header 字段`用来获取请求头中的所有信息，Header 字段的类型是 Header 类型，而 Header 类型是一个 `map[string][]string`，string 类型的 key，string 切片类型的值。**

**下面是 Header 类型及它的方法：**

![image-20220322101122173](https://gitee.com/jobim/blogimage/raw/master/img/20220322101122.png)

**获取请求头中的某个具体属性的值，如获取 Accept-Encoding 的值：**

* **方式一：`r.Header["Accept-Encoding"]`，得到的是一个字符串切片**
* **方式二：`r.Header.Get("Accept-Encoding")`，得到的是字符串形式的值，多个值使用逗号分隔**

* 代码示例：

  ```go
  func handler(w http.ResponseWriter, r *http.Request) {
  
  	//请求头
  	fmt.Printf("Header: %#v\n", r.Header)
  	fmt.Println("请求头中Accept-Encoding属性的值：",r.Header["Accept-Encoding"])
  	fmt.Println("请求头中Accept-Encoding属性的值：",r.Header.Get("Accept-Encoding"))
  	
  }
  
  func main() {
  	http.HandleFunc("/hello", handler)
      
  	http.ListenAndServe(":8080", nil)
  }
  ```

  访问`http://127.0.0.1:8080/hello?username=admin&password=123456`，执行结果：

  ![image-20220322102004575](https://gitee.com/jobim/blogimage/raw/master/img/20220322102004.png)

  

## 2.3 获取请求体中的信息

**请求和响应的主体都是有 Request 结构中的 Body 字段表示，该字段的类型是`io.ReadCloser `接口。该接口包含了 Reader 接口和 Closer 接口，Reader 接口拥有 Read方法，Closer 接口拥有 Close 方法**

![image-20220322103103553](https://gitee.com/jobim/blogimage/raw/master/img/20220322103103.png)

* 代码示例：从请求中读取Body并输出

  ```go
  func handler(w http.ResponseWriter, r *http.Request) {
  
  	//获取请求体中内容的长度
  	len := r.ContentLength
  	//创建byte切片
  	body := make([]byte, len)
  	//将请求体中的内容读取到body中
  	r.Body.Read(body)
  	//在浏览器中显示请求体中的内容
  	fmt.Fprintln(w, "请求体中的内容有：", string(body))
      
  }
  
  func main() {
  	http.HandleFunc("/getBody", handler)
  	
  	http.ListenAndServe(":8080", nil)
  }
  
  ```

  使用表单发送`post请求`

  ```html
  <form action="http://localhost:8080/getBody" method="POST">
      用户名：<input type="text" name="username" value="zhangsan" /><br/>
      密码：<input type="password" name="password" value="666666" /><br/>
      <input type="submit" />
  </form>
  ```

  访问`http://localhost:8080/getBody`，浏览器显示结果：

  ![image-20220322103721164](https://gitee.com/jobim/blogimage/raw/master/img/20220322103721.png)

## 2.4 获取请求参数

**在Request结构中，有3个和form有关的字段：**

```less
// Form字段包含了解析后的form数据，包括URL的query、POST/PUT提交的form数据
// 该字段只有在调用了ParseForm()之后才有数据
Form url.Values

// PostForm字段不包含URL的query，只包括POST/PATCH/PUT提交的form数据
// 该字段只有在调用了ParseForm()之后才有数据
PostForm url.Values // Go 1.1

// MultipartForm字段包含multipart form的数据
// 该字段只有在调用了ParseMultipartForm()之后才有数据
MultipartForm *multipart.Form
```

**一般获取请求参数的逻辑：**

* **方法一：**
  * **先调用ParseForm()或ParseMultipartForm()解析请求中的数据**
  * **按需访问Request结构中的Form、PostForm或MultipartForm字段**

* **方法二：直接使用Request的方法：**
  * **FormValue(key)**
  * **PostFormValue(key)**

### 2.4.1 Form和PostForm字段

**Form 和 PostForm字段都是 url.Values 类型，都能用来获取表单中的请求参数。**

![image-20220322105756567](https://gitee.com/jobim/blogimage/raw/master/img/20220322105756.png)

<font color='red'>**注意：Form 和 PostForm字段必须调用 Request 的 ParseForm() 方法后才有效**</font>

* **不同的是：**
  * **Form 字段包括 `URL 字段的请求参数`和 `POST 或 PUT 的表单数据`。**
  * **PostForm字段只包括` POST 或 PUT 的表单数据`。**

* **Form 和 PostForm 字段只支持 application/x-www-form-urlencoded 编码，如果 form 表单的 enctype 属性值为 multipart/form-data，那么使用 Form 和 PostForm 字段无法获取表单中的数据，此时需要使用 MultipartForm 字段。（此时Form 字段仍然有 URL 字段的请求参数）**

代码示例：（获取请求参数）

* form表单

  ```html
  <form action="http://localhost:8080/getBody?username=admin&pwd=123456" method="POST">
      用户名：<input type="text" name="username" value="zhangsan" /><br/>
      密码：<input type="password" name="password" value="666666" /><br/>
      <input type="submit" />
  </form>
  ```

* web handler代码：

  ```go
  func handler(w http.ResponseWriter, r *http.Request) {
  
  	//解析表单，在调用r.Form之前必须执行该操作
  	r.ParseForm()
  	//获取请求参数
  	//如果form表单的action属性的URL地址中也有与form表单参数名相同的请求参数，
  	//那么参数值都可以得到，并且form表单中的参数值在URL的参数值的前面
  	fmt.Fprintln(w, "请求参数有：", r.Form)
  	fmt.Fprintln(w, "POST请求的form表单中的请求参数有：", r.PostForm)
  
  }
  
  func main() {
  	http.HandleFunc("/getBody", handler)
  	
  	http.ListenAndServe(":8080", nil)
  }
  
  ```

* 点击提交按钮显示结果：

  ![image-20220322111424767](https://gitee.com/jobim/blogimage/raw/master/img/20220322111424.png)

### 2.4.2 FormValue()和PostFormValue()

这两个方法在需要时**会自动调用ParseForm()或ParseMultipartForm()**，所以使用这两个方法取Form数据的时候，可以不用显式解析Form。

* **FormValue()返回form数据和url 请求参数后的第一个值。要取得完整的值，还是需要访问Request.Form或Request.PostForm字段。但因为FormValue()已经解析过Form了，所以无需再显式调用ParseForm()再访问request中Form相关字段。**

* **PostFormValue()返回form数据的第一个值，因为它只能访问form数据，所以忽略URL的请求参数部分**。



代码示例：

* form表单

  ```html
  <form action="http://localhost:8080/getBody?user=admin&pwd=123456" method="POST">
      用户名：<input type="text" name="username" value="zhangsan" /><br/>
      密码：<input type="password" name="password" value="666666" /><br/>
      <input type="submit" />
  </form>
  ```

* web handler代码：

  ```go
  func handler(w http.ResponseWriter, r *http.Request) {
  
  	//通过直接调用FormValue方法和PostFormValue方法直接获取请求参数的值
  	fmt.Fprintln(w, "URL中的user请求参数的值是：", r.FormValue("user"))
  	fmt.Fprintln(w, "Form表单中的username请求参数的值是：", r.PostFormValue("username"))
      
  }
  
  func main() {
  	http.HandleFunc("/getBody", handler)
      
  	http.ListenAndServe(":8080", nil)
  }
  ```

* 点击提交按钮显示结果：

  ![image-20220322112918889](https://gitee.com/jobim/blogimage/raw/master/img/20220322112918.png)

### 2.4.3 MultipartForm字段

**为了取得 multipart/form-data 编码的表单数据，我们需要用到 Request 结构的ParseMultipartForm() 方法和 MultipartForm 字段，我们通常上传文件时会将 form 表单的enctype 属性值设置为 multipart/form-data**

![image-20220322115215895](https://gitee.com/jobim/blogimage/raw/master/img/20220322115215.png)

* **MultipartForm字段的类型为 `*multipart.Form`，multipart 包下 Form 结构的指针类型，该结构中有两个映射**

  ![image-20220322144529832](https://gitee.com/jobim/blogimage/raw/master/img/20220322144530.png)

代码示例：

* form表单：

  ```html
  <form action="http://localhost:8080/upload" method="POST" enctype="multipart/form-data">
      用 户 名 ： <input type="text" name="username"value="hanzong"><br/>
      文件：<input type="file" name="file" /><br/>
      <input type="submit">
  </form>
  ```

* web handler代码：

  ```go
  func handler(w http.ResponseWriter, r *http.Request) {
      
  	//解析表单
  	r.ParseMultipartForm(1024)
  	//打印表单数据
  	fmt.Fprintln(w, r.MultipartForm)
  
  	//打开上传的文件
  	fileHeader := r.MultipartForm.File["file"][0]
  	file, err := fileHeader.Open()
  	if err == nil {
  		data, err := ioutil.ReadAll(file)
  		if err == nil {
  			fmt.Fprintln(w, string(data))
  		}
  	}
  }
  
  func main() {
  	http.HandleFunc("/upload", handler)
  	
  	http.ListenAndServe(":8080", nil)
  }
  
  ```

* 点击提交按钮显示结果：（上传.txt文件）

  ![image-20220322145521951](https://gitee.com/jobim/blogimage/raw/master/img/20220322145522.png)

### 2.4.4 文件相关（待补）



# 三、请求响应

>  好的博客：[Go 语言通过 ResponseWriter 对象发送 HTTP 响应](https://laravelacademy.org/post/21659)



**客户端请求信息都封装到了 `Request` 对象，而发送给客户端的响应通过`ResponseWriter`对象完成**

* **`ResponseWriter` 是处理器用来创建 HTTP 响应的接口，其源码结构如下所示：**

  ![image-20220322152014573](https://gitee.com/jobim/blogimage/raw/master/img/20220322152014.png)

  

## 3.1 设置响应状态码

**`WriteHeader()` 该方法支持传入一个整型数据用来表示响应状态码，如果不调用该方法的话，默认响应状态码是 `200 OK`。**

代码示例：

* web handler代码：

  ```go
  func testStatusCode(w http.ResponseWriter, r *http.Request) {
  	//设置响应的状态码
  	w.WriteHeader(302)
  	w.Write([]byte("资源临时被移动"))
  }
  
  func main() {
  	http.HandleFunc("/testStatusCode", testStatusCode)
  
  	http.ListenAndServe(":8080", nil)
  }
  
  ```

* 访问`localhost:8080/testStatusCode`可查看状态码已经更改：

  ![image-20220322153138836](https://gitee.com/jobim/blogimage/raw/master/img/20220322153138.png)

## 3.2 设置响应头

**`Header` 方法用于设置响应头信息，我们可以通过 `w.Header().Set` 方法设置响应头（`w.Header()` 方法返回的是 `Header` 响应头对象，它和请求头共用一个结构体，因此请求头上支持的方法这里都支持，比如可以通过 `w.Header().Add` 方法新增响应头）。**



<font color='blue' size='4'>**让客户端重定向：**</font>

* 这里我们设置一个 302 重定向响应，只需要通过 `w.WriteHeader` 方法将响应状态码设置为 301，再通过 `w.Header().Set` 方法将负责重定向的响应头 `Location` 设置为一个可访问域名即可。

* web handler代码：

  ```go
  func testRedire(w http.ResponseWriter, r *http.Request) {
  	//设置响应头中的Location属性
  	w.Header().Set("Location", "https://www.baidu.com")
  	//设置响应的状态码
  	w.WriteHeader(302)))
  }
  ```

## 3.3 写入数据到响应实体

**`Write` 方法用于写入数据到 HTTP 响应实体**

### 3.3.1 返回文本字符串

*  `Write` 方法接受的参数类型是 `[]byte` 切片，所以需要将字符串转换为字节切片类型。

  ```go
  func handler(w http.ResponseWriter, r *http.Request)  {
      w.Write([]byte("你的请求我已经收到"));
  }
  ```

* 启动 HTTP 服务器，就可以在浏览器中看到数据：

  ![image-20220322171951049](https://gitee.com/jobim/blogimage/raw/master/img/20220322171951.png)

### 3.3.2 返回 HTML 文档

* 处理器中的代码：

  ```go
  func handler(w http.ResponseWriter, r *http.Request) {
  	html := `<html>
  		<head>
  			<title>测试响应内容为网页</title>
  			<meta charset="utf-8"/>
  		</head>
  		<body>
  			我是以网页的形式响应过来的！
  		</body>
  	</html>`
  	w.Write([]byte(html))
  }
  ```

* 在浏览器显示的结果：

  ![image-20220322172330718](https://gitee.com/jobim/blogimage/raw/master/img/20220322172330.png)

  可以看到，由于响应数据的内容类型变成了 HTML，在响应头中，也可以看到 `Content-Type` 也自动调整成了 `text/html`，不再是纯文本格式。这里的 Content-Type 就是根据传入的数据自行判断出来的。

### 3.3.3 返回 JSON 格式数据

* 处理器中的代码：

  ```go
  func testJsonRes(w http.ResponseWriter, r *http.Request) {
  	//设置响应内容的类型
  	w.Header().Set("Content-Type", "application/json")
  	//创建User
  	user := model.User{
  		ID:       1,
  		Username: "admin",
  		Password: "123456",
  		Email:    "admin@atguigiu.com",
  	}
  	//将User转换为Json格式
  	json, _ := json.Marshal(user)
  	//将json格式的数据响应给客户端
  	w.Write(json)
  }
  func main() {
  	http.HandleFunc("/testJson", testJsonRes)
  	http.ListenAndServe(":8080", nil)
  }
  ```

* 在浏览器显示的结果：

  ![image-20220322172851836](https://gitee.com/jobim/blogimage/raw/master/img/20220322172851.png)

  **注意：返回JSON类型需要设置响应头中 `Content-Type字段`为 `text/html`，否则内容类型仍然为`text/plain`**







