

> 参考博客：
>
> [在Go语言项目中使用Zap日志库 | 李文周的博客](https://www.liwenzhou.com/posts/Go/zap/)
>
> [使用zap接收gin框架默认的日志并配置日志归档 | 李文周的博客 ](https://www.liwenzhou.com/posts/Go/use_zap_in_gin/)



# 默认的Go Logger

* **Go语言提供的基本日志功能。Go语言提供的默认日志包是https://golang.org/pkg/log/**



<font color='blue' size='4'>**使用Log：**</font>

* 代码：

  ```go
  import (
  	"log"
  	"net/http"
  )
  func simpleHttpGet(url string) {
  	resp, err := http.Get(url)
  	if err != nil {
  		log.Printf("Error fetching url %s : %s", url, err.Error())
  	} else {
  		log.Printf("Status Code for %s : %s", url, resp.Status)
  		resp.Body.Close()
  	}
  }
  
  func main() {
  	//SetupLogger()
  	simpleHttpGet("www.baidu.com")
  	simpleHttpGet("https://www.baidun.com")
  }
  ```

* 输出结果：

  ![image-20220402125634747](https://gitee.com/jobim/blogimage/raw/master/img/202204021256801.png)

<font color='blue' size='4'>**设置日志的输出位置：**</font>

* **设置初始化到E盘的test.log文件中**

  ```go
  // 可以设置日志的输出位置，在使用log之前调用即可
  func SetupLogger() {
  	logFileLocation, _ := os.OpenFile("E:/test.log", os.O_CREATE|os.O_APPEND|os.O_RDWR, 0744)
  	log.SetOutput(logFileLocation)
  }
  ```



**Go Log的优势和劣势：**

* 优势：它最大的优点是使用非常简单。我们可以设置任何`io.Writer`作为日志记录输出并向其发送要写入的日志。

* 劣势：
  * 仅限基本的日志级别

    - 只有一个`Print`选项。不支持`INFO`/`DEBUG`等多个级别。

  * 对于错误日志，它有`Fatal`和`Panic`

    - Fatal日志通过调用`os.Exit(1)`来结束程序
    - Panic日志在写入日志消息之后抛出一个panic
    - 但是它缺少一个ERROR日志级别，这个级别可以在不抛出panic或退出程序的情况下记录错误

  * 缺乏日志格式化的能力——例如记录调用者的函数名和行号，格式化日期和时间格式。等等。

  * 不提供日志切割的能力。



# zap的使用

`zap`是`Uber`开发的非常快的、结构化的，分日志级别的Go日志库。根据Uber-go Zap的文档，它的性能比类似的结构化日志包更好，也比标准库更快。

github地址：`https://github.com/uber-go/zap`

**安装：**

```bash
go get -u go.uber.org/zap
```

## 配置Zap Logger

**Zap提供了两种类型的日志记录器：`Sugared Logger`和`Logger`**

* 在性能很好但不是很关键的上下文中，使用`SugaredLogger`。它比其他结构化日志记录包快4-10倍，并且支持结构化和printf风格的日志记录。

* 在每一微秒和每一次内存分配都很重要的上下文中，使用`Logger`。它甚至比`SugaredLogger`更快，内存分配次数也更少，但它只支持强类型的结构化日志记录。

<font color='blue' size='4'>**Logger：**</font>

- **通过调用`zap.NewProduction()`/`zap.NewDevelopment()`或者`zap.Example()`创建一个Logger**
- 三种创建方式对比：
  - `Example`和`Production`使用的是`json`格式输出，`Development`使用行的形式输出
  - `Development`
    - 从警告级别向上打印到堆栈中来跟踪
    - 始终打印包/文件/行（方法）
    - 在行尾添加任何额外字段作为json字符串
    - 以大写形式打印级别名称
    - 以毫秒为单位打印ISO8601格式的时间戳
  - `Production`
    - 调试级别消息不记录
    - Error,Dpanic级别的记录，会在堆栈中跟踪文件，Warn不会
    - 始终将调用者添加到文件中
    - 以时间戳格式打印日期
    - 以小写形式打印级别名称
- 通过Logger调用Info/Error等。
- 默认情况下日志都会打印到应用程序的console界面。

```go
import (
	"go.uber.org/zap"
)

func main() {

	logger = zap.NewExample()
	//logger, _ := zap.NewDevelopment()
	//logger, _ := zap.NewProduction()
    defer logger.Sync()	// 将 buffer 中的日志写到文件中
	logger.Debug("DEBUG message", zap.String("status", "成功"), zap.String("key", "val"))
	logger.Info("INFO message", zap.String("status", "成功"), zap.String("key", "val"))
    
}

//***********执行结果*************
//Example 输出
{"level":"debug","msg":"DEBUG message","status":"成功","key":"val"}
{"level":"info","msg":"INFO message","status":"成功","key":"val"}

//Development 输出
2022-04-02T14:52:08.447+0800    DEBUG   custom_log/main.go:55   DEBUG message   {"status": "成功", "key": "val"}
2022-04-02T14:52:08.460+0800    INFO    custom_log/main.go:56   INFO message    {"status": "成功", "key": "val"}


//Production 输出
{"level":"info","ts":1648882199.5008874,"caller":"custom_log/main.go:56","msg":"INFO message","status":"成功","key":"val"}

```

* **日志记录器方法的语法是这样的：**

  ```go
  func (log *Logger) Info(msg string, fields ...Field) 
  ```

  其中`MethodXXX`是一个可变参数函数，可以是Info / Error/ Debug / Panic等。每个方法都接受一个消息字符串和任意数量的`zapcore.Field`场参数。

  每个`zapcore.Field`其实就是一组键值对参数。



<font color='blue' size='4'>**配置Sugared Logger：**</font>

大部分的实现基本都相同，不同的是通过调用主logger的`. Sugar()`方法来获取一个`SugaredLogger`，然后使用`SugaredLogger`以`printf`格式记录语句，例如：

```go
func main() {

	//logger := zap.NewExample()
	//logger, _ := zap.NewDevelopment()
	logger, _ := zap.NewProduction()
	sugarLogger := logger.Sugar()
	defer sugarLogger.Sync() // 将 buffer 中的日志写到文件中
	sugarLogger.Infof("状态%s，key的数据%s", "200", "value")
    
}
```

执行结果：

```json
{"level":"info","ts":1648883129.7547314,"caller":"custom_log/main.go:57","msg":"状态200，key的数据value"}
```



## 定制logger

### 将日志写入文件

我们将使用`zap.New(…)`方法来手动传递所有配置，而不是使用像`zap.NewProduction()`这样的预置方法来创建logger。

```go
func New(core zapcore.Core, options ...Option) *Logger
```

`zapcore.Core`需要三个配置——`Encoder`，`WriteSyncer`，`LogLevel`。

* **Encoder**:编码器(如何写入日志)。我们将使用开箱即用的`NewJSONEncoder()`，并使用预先设置的`ProductionEncoderConfig()`。

  ```go
  zapcore.NewJSONEncoder(zap.NewProductionEncoderConfig())
  ```

* **WriterSyncer** ：指定日志将写到哪里去。我们使用`zapcore.AddSync()`函数并且将打开的文件句柄传进去。

  ```go
  file, _ := os.Create("./test.log")
  writeSyncer := zapcore.AddSync(file)
  ```

* **Log Level**：哪种级别的日志将被写入。

**代码示例：**

```go
import (
	"go.uber.org/zap"
	"go.uber.org/zap/zapcore"
	"os"
)

var logger *zap.Logger

//初始化logger
func InitLogger() {
	writeSyncer := getLogWriter()
	encoder := getEncoder()
	core := zapcore.NewCore(encoder, writeSyncer, zapcore.DebugLevel)
	logger = zap.New(core)
}

func getEncoder() zapcore.Encoder {
    //return zapcore.NewConsoleEncoder(zap.NewProductionEncoderConfig())	//按照普通的Log Encoder格式输出
	return zapcore.NewJSONEncoder(zap.NewProductionEncoderConfig())
}

func getLogWriter() zapcore.WriteSyncer {
	file, _ := os.Create("./test.log")	//创建一个test.log文件写入
	return zapcore.AddSync(file)
}

func main() {
	InitLogger()
	defer logger.Sync() // 将 buffer 中的日志写到文件中
	logger.Info("INFO message", zap.String("status", "成功"), zap.String("key", "val"))

}
```

输出将打印在文件——`test.log`中。

```go
{"level":"info","ts":1648884052.6416597,"msg":"INFO message","status":"成功","key":"val"}

//使用NewJSONEncoder数据结果
//1.6488843211482797e+09	info	INFO message	{"status": "成功", "key": "val"}
```



### 其他设置配置

```go
var logger *zap.Logger

func InitLogger() {
	writeSyncer := getLogWriter()
	encoder := getEncoder()
	core := zapcore.NewCore(encoder, writeSyncer, zapcore.DebugLevel)
	//logger = zap.New(core)
    //AddCaller()添加将调用函数信息记录到日志中的功能,显示文件名和行号
	logger = zap.New(core, zap.AddCaller())
}

func getEncoder() zapcore.Encoder {
	encoderConfig := zap.NewProductionEncoderConfig()
	encoderConfig.EncodeTime = zapcore.ISO8601TimeEncoder        //指定时间格式
	encoderConfig.EncodeLevel = zapcore.CapitalColorLevelEncoder //按级别显示不同颜色，不需要的话取值zapcore.CapitalLevelEncoder就可以了
	encoderConfig.EncodeCaller = zapcore.FullCallerEncoder       //显示完整文件路径
	return zapcore.NewConsoleEncoder(encoderConfig)
    //return zapcore.NewConsoleEncoder(zap.NewProductionEncoderConfig())
}

func getLogWriter() zapcore.WriteSyncer {
	file, _ := os.Create("./test.log")
	//设置同时输出控制台和文件
	return zapcore.NewMultiWriteSyncer(zapcore.AddSync(file), zapcore.AddSync(os.Stdout))
    //return zapcore.AddSync(file)
}

func main() {
	InitLogger()
	defer logger.Sync() // 将 buffer 中的日志写到文件中
	logger.Info("INFO message", zap.String("status", "成功"), zap.String("key", "val"))

}
```



![image-20220402165631211](https://gitee.com/jobim/blogimage/raw/master/img/202204021656278.png)

# 使用Lumberjack进行日志切割

**zap包本身不提供文件切割的功能，但是可以用zap官方推荐的第三方库`lumberjack`处理**

* **安装：**

  ```go
  go get -u github.com/natefinch/lumberjack
  ```

**使用：**

要在zap中加入Lumberjack支持，我们需要修改`WriteSyncer`代码。我们将按照下面的代码修改`getLogWriter()`函数：

```go
func getLogWriter() zapcore.WriteSyncer {
	lumberJackLogger := &lumberjack.Logger{
		Filename:   "./info.log", //日志文件存放目录，如果文件夹不存在会自动创建
		MaxSize:    1,            //文件大小限制,单位MB
		MaxBackups: 5,            //最大保留日志文件数量
		MaxAge:     30,           //日志文件保留天数
		Compress:   false,        //是否压缩处理
	}
	return zapcore.AddSync(lumberJackLogger)
}
```





# gin框架添加zap

### gin默认的中间件

* Gin 框架的日志 `Logger` 是在我们调用 `gin.Default()` 的时候, 会加载两个中间件 `engine.Use(Logger(), Recovery())`
  * `Logger()` - 中间件是将 Gin 框架中日志输出到标准输出.
  * `Recovery()` - 中间件是 程序出现 `panic` 的时候尝试恢复程序,然后将错误代码写入到日志中.

首先我们来看一个最简单的gin项目：

```go
func main() {
	r := gin.Default()
	r.GET("/hello", func(c *gin.Context) {
		c.JSON(http.StatusOK, gin.H{
			"message": "success",
		})
	})
	r.Run()
}
```

接下来我们看一下`gin.Default()`的源码：

```go
func Default() *Engine {
	debugPrintWARNINGDefault()
	engine := New()
	engine.Use(Logger(), Recovery())
	return engine
}
```

### 基于zap的中间件

我们可以模仿`Logger()`和`Recovery()`的实现，使用我们的日志库来接收gin框架默认输出的日志。

这里以zap为例，我们实现两个中间件如下：

```go
// GinLogger 接收gin框架默认的日志
func GinLogger(logger *zap.Logger) gin.HandlerFunc {
	return func(c *gin.Context) {
		start := time.Now()
		path := c.Request.URL.Path
		query := c.Request.URL.RawQuery
		c.Next()

		cost := time.Since(start)
		logger.Info(path,
			zap.Int("status", c.Writer.Status()),
			zap.String("method", c.Request.Method),
			zap.String("path", path),
			zap.String("query", query),
			zap.String("ip", c.ClientIP()),
			zap.String("user-agent", c.Request.UserAgent()),
			zap.String("errors", c.Errors.ByType(gin.ErrorTypePrivate).String()),
			zap.Duration("cost", cost),
		)
	}
}

// GinRecovery recover掉项目可能出现的panic
func GinRecovery(logger *zap.Logger, stack bool) gin.HandlerFunc {
	return func(c *gin.Context) {
		defer func() {
			if err := recover(); err != nil {
				// Check for a broken connection, as it is not really a
				// condition that warrants a panic stack trace.
				var brokenPipe bool
				if ne, ok := err.(*net.OpError); ok {
					if se, ok := ne.Err.(*os.SyscallError); ok {
						if strings.Contains(strings.ToLower(se.Error()), "broken pipe") || strings.Contains(strings.ToLower(se.Error()), "connection reset by peer") {
							brokenPipe = true
						}
					}
				}

				httpRequest, _ := httputil.DumpRequest(c.Request, false)
				if brokenPipe {
					logger.Error(c.Request.URL.Path,
						zap.Any("error", err),
						zap.String("request", string(httpRequest)),
					)
					// If the connection is dead, we can't write a status to it.
					c.Error(err.(error)) // nolint: errcheck
					c.Abort()
					return
				}

				if stack {
					logger.Error("[Recovery from panic]",
						zap.Any("error", err),
						zap.String("request", string(httpRequest)),
						zap.String("stack", string(debug.Stack())),
					)
				} else {
					logger.Error("[Recovery from panic]",
						zap.Any("error", err),
						zap.String("request", string(httpRequest)),
					)
				}
				c.AbortWithStatus(http.StatusInternalServerError)
			}
		}()
		c.Next()
	}
}
```

*如果不想自己实现，可以使用github上有别人封装好的https://github.com/gin-contrib/zap。*

这样我们就可以在gin框架中使用我们上面定义好的两个中间件来代替gin框架默认的`Logger()`和`Recovery()`了。

```go
r := gin.New()
r.Use(GinLogger(), GinRecovery())
```



**完整使用：**

```go
package main

import (
	"github.com/gin-gonic/gin"
	"github.com/natefinch/lumberjack"
	"go.uber.org/zap"
	"go.uber.org/zap/zapcore"
	"net"
	"net/http"
	"net/http/httputil"
	"os"
	"runtime/debug"
	"strings"
	"time"
)

var logger *zap.Logger

// GinLogger 接收gin框架默认的日志
func GinLogger(logger *zap.Logger) gin.HandlerFunc {
	return func(c *gin.Context) {
		start := time.Now()
		path := c.Request.URL.Path
		query := c.Request.URL.RawQuery
		c.Next()

		cost := time.Since(start)
		logger.Info(path,
			zap.Int("status", c.Writer.Status()),
			zap.String("method", c.Request.Method),
			zap.String("path", path),
			zap.String("query", query),
			zap.String("ip", c.ClientIP()),
			zap.String("user-agent", c.Request.UserAgent()),
			zap.String("errors", c.Errors.ByType(gin.ErrorTypePrivate).String()),
			zap.Duration("cost", cost),
		)
	}
}

// GinRecovery recover掉项目可能出现的panic
func GinRecovery(logger *zap.Logger, stack bool) gin.HandlerFunc {
	return func(c *gin.Context) {
		defer func() {
			if err := recover(); err != nil {
				// Check for a broken connection, as it is not really a
				// condition that warrants a panic stack trace.
				var brokenPipe bool
				if ne, ok := err.(*net.OpError); ok {
					if se, ok := ne.Err.(*os.SyscallError); ok {
						if strings.Contains(strings.ToLower(se.Error()), "broken pipe") || strings.Contains(strings.ToLower(se.Error()), "connection reset by peer") {
							brokenPipe = true
						}
					}
				}

				httpRequest, _ := httputil.DumpRequest(c.Request, false)
				if brokenPipe {
					logger.Error(c.Request.URL.Path,
						zap.Any("error", err),
						zap.String("request", string(httpRequest)),
					)
					// If the connection is dead, we can't write a status to it.
					c.Error(err.(error)) // nolint: errcheck
					c.Abort()
					return
				}

				if stack {
					logger.Error("[Recovery from panic]",
						zap.Any("error", err),
						zap.String("request", string(httpRequest)),
						zap.String("stack", string(debug.Stack())),
					)
				} else {
					logger.Error("[Recovery from panic]",
						zap.Any("error", err),
						zap.String("request", string(httpRequest)),
					)
				}
				c.AbortWithStatus(http.StatusInternalServerError)
			}
		}()
		c.Next()
	}
}

func InitLogger() {
	writeSyncer := getLogWriter()
	encoder := getEncoder()
	core := zapcore.NewCore(encoder, writeSyncer, zapcore.DebugLevel)
	//AddCaller()添加将调用函数信息记录到日志中的功能,显示文件名和行号
	logger = zap.New(core, zap.AddCaller())
}

func getEncoder() zapcore.Encoder {
	encoderConfig := zap.NewProductionEncoderConfig()
	encoderConfig.EncodeTime = zapcore.ISO8601TimeEncoder   //指定时间格式
	encoderConfig.EncodeLevel = zapcore.CapitalLevelEncoder //在日志文件中使用大写字母记录日志级别
	return zapcore.NewConsoleEncoder(encoderConfig)
}

func getLogWriter() zapcore.WriteSyncer {
	lumberJackLogger := &lumberjack.Logger{
		Filename:   "./test.log",
		MaxSize:    10,    // 单位M
		MaxBackups: 5,     // 最大备份数量
		MaxAge:     30,    // 最大备份天数
		Compress:   false, // 是否压缩
	}
	return zapcore.AddSync(lumberJackLogger)
}

func main() {
	InitLogger()
	r := gin.New()
	r.Use(GinLogger(logger), GinRecovery(logger, true))
	r.GET("/hello", func(c *gin.Context) {
		logger.Info("请求了/hello路径")
		c.JSON(http.StatusOK, gin.H{
			"message": "success",
		})
	})
	r.Run()
}

```

访问http://localhost:8080/hello日志打印结果：

```
2022-04-02T17:22:29.464+0800	INFO	gin_zap/main.go:121	请求了/hello路径
2022-04-02T17:22:29.480+0800	INFO	gin_zap/main.go:30	/hello	{"status": 200, "method": "GET", "path": "/hello", "query": "", "ip": "::1", "user-agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/99.0.4844.74 Safari/537.36 Edg/99.0.1150.52", "errors": "", "cost": 0.0160032}
```









