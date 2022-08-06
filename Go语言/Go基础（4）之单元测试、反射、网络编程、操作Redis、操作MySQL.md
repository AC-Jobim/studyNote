# 一、单元测试

**Go语言中自带有一个轻量级的测试框架testing和自带的go test命令来实现单元测试和性能测试。**

* **`我们执行go test`命令时，它会遍历该go包中所有以`_test.go结尾的测试`文件， 然后调用并执行测试文件中符合go test 规则的函数帮助我们实现自动化测试。**

* **其过程为生成1个临时的main包用于调用相应的测试函数，然后构建并运行测试文件中的函数、报告测试结果，最后清理测试中生成的临时文件。**



**单元测试的注意：**

* **在包目录内所有测试文件必须以_test.go结尾**，`go build不会把这些测试文件`编译到最终的可执行文件中。比如 cal_test.go , cal不是固定的

* **测试用例函数必须以Test 开头**，一般来说就是Test+被测试的函数名。即测试函数格式：

  ```go
  func TestXxxx(*testing.T){...}
  ```

* **每个单元测试函数的参数必须为*testing.T**，参数`t`用于报告测试是否失败以及日志信息。

* 运行测试用例指令
  * `cmd>go test`[如果运行正确，无日志，错误时，会输出日志]
  * `cmd>go test -v`[运行正确或是错误，都输出日志]
  
* 当出现错误时，可以使用t.Fatalf来格式化输出错误信息，并退出程序

* t.Logf方法可以输出相应的日志

* PASS表示测试用例运行成功，FAIL表示测试用例运行失败

* 测试单个文件，一定要带上被测试的原文件
  **go test -v cal_test.go cal.go**
  
* 测试单个方法
  **go test -v -test.run TestAddUpper**



**测试案例：**

* 文件结构：

  ![image-20220302175540118](https://gitee.com/jobim/blogimage/raw/master/img/20220302175540.png)

* 测试文件

  ```go
  package cal
  import (
  	"fmt"
  	"testing" //引入go 的testing框架包
  )
  
  //编写要给测试用例，去测试addUpper是否正确
  func TestAddUpper(t *testing.T) {
  
  	//调用
  	res := addUpper(10)
  	if res != 55 {
  		//fmt.Printf("AddUpper(10) 执行错误，期望值=%v 实际值=%v\n", 55, res)
  		t.Fatalf("AddUpper(10) 执行错误，期望值=%v 实际值=%v\n", 55, res)
  	}
  
  	//如果正确，输出日志
  	t.Logf("AddUpper(10) 执行正确...")
  
  }
  
  func TestHello(t *testing.T) {
  
  	fmt.Println("TestHello被调用..")
  
  }
  ```

* 待测试文件：

  ```go
  package cal
  
  //一个被测试函数
  func addUpper(n int) int {
  	res := 0
  	for i := 1; i <= n; i++ {
  		res += i
  	}
  	return res
  }
  
  ```

* 执行结果：

  ![image-20220302175641782](https://gitee.com/jobim/blogimage/raw/master/img/20220302175641.png)



<font color='blue' size='4'>**子测试：**</font>

* 如果一个测试函数的函数名的不是以 Test 开头，那么在使用 go test 命令时默认不会执行，**不过我们可以设置该函数时一个子测试函数，可以在其他测试函数里通过 t.Run 方法来执行子测试函数**

  ![image-20220321204209776](https://gitee.com/jobim/blogimage/raw/master/img/20220321204209.png)



<font color='blue' size='4'>**TestMain(m *testing.M)：**</font>

* **测试文件中有 `TestMain(m *testing.M)`函数时，执行 go test 命令将直接运行 TestMain函数，不直接运行测试函数，只有在 TestMain 函数中执行 `m.Run()`时才会执行测试函数**

  ![image-20220321204747633](https://gitee.com/jobim/blogimage/raw/master/img/20220321204747.png)



# 二、反射

> 好的博客：
>
> [Go语言反射reflect](https://www.cnblogs.com/itbsl/p/10551880.html)
>
> [Go语言反射（reflect）](http://c.biancheng.net/golang/reflect/)

* **反射可以在运行时动态获取变量的各种信息，比如变量的类型(type)，类别(kind)**
* **如果是结构体变量，还可以获取到结构体本身的信息(包括结构体的字段、方法)**
* 通过反射，可以修改变量的值，可以调用关联的方法。
* 使用反射，需要import (“reflect”)

## 2.1 两个重要函数和类型

**两个重要的函数：**

* **`func TypeOf(i interface{}) Type`**

  **TypeOf返回接口中保存的值的类型，TypeOf(nil)会返回nil。**

* **`Copyfunc ValueOf(i interface{}) Value`**

  **ValueOf返回一个初始化为i接口保管的具体值的Value，ValueOf(nil)返回Value零值。**



**两个类型是 `reflect.Type `和 `reflect.Value`**



**变量、interface和 reflect.Value的相互转换：**

![image-20220315211740674](https://gitee.com/jobim/blogimage/raw/master/img/20220315211740.png)





反射快速入门案例：

```go
package main

import (
	"reflect"
	"fmt"
)

//专门演示反射
func reflectTest01(b interface{}) {
	//通过反射获取的传入的变量的 type ,kind, 值
	//1. 先获取到 reflect.Type
	rType := reflect.TypeOf(b)
	fmt.Println("rType=", rType) //rType= int
	
	//2. 获取到reflect.Value
	rVal := reflect.ValueOf(b)
	fmt.Printf("rVal=%v,rVal的type=%T\n", rVal, rVal) //rVal=100,rVal的type=reflect.Value

	n2 := 2 + rVal.Int() //不能写成 2 + rVal
	fmt.Println("n2=", n2) //n2= 102
	
	//下面我们将 rVal 转成 interface{}
	iV := rVal.Interface()
	//将 interface{} 通过断言转成需要的类型
	num2 := iV.(int)
	fmt.Printf("num2=%v,num2的type=%T\n", num2, num2) //num2=100,num2的type=int

	
}

//专门演示反射[对结构体的反射]
func reflectTest02(b interface{}) {

	//通过反射获取的传入的变量的 type , kind, 值
	//1. 先获取到 reflect.Type
	rType := reflect.TypeOf(b)
	fmt.Println("rType=", rType) //rType= main.Student

	//2. 获取到reflect.Value
	rVal := reflect.ValueOf(b)

	//3. 获取变量对应的Kind，下面两个得到的值一样
	kind2 := rType.Kind()
	kind1 := rVal.Kind()
	
	fmt.Printf("kind =%v kind=%v\n", kind1, kind2) //kind =struct kind=struct


	//下面我们将 rVal 转成 interface{}
	iV := rVal.Interface()
	fmt.Printf("iv=%v,iv的type=%T \n", iV, iV) //iv={tom 20},iv的type=main.Student
	//将 interface{} 通过断言转成需要的类型
	//这里，我们就简单使用了一带检测的类型断言.
	//同学们可以使用 swtich 的断言形式来做的更加的灵活
	stu, ok := iV.(Student)
	if ok {
		fmt.Printf("stu.Name=%v\n", stu.Name) //stu.Name=tom
	}


}

type Student struct {
	Name string
	Age int
}

func main() {

	//1. 先定义一个int
	var num int = 100
	reflectTest01(num)
	
	fmt.Println("-----------")

	//2. 定义一个Student的实例
	stu := Student{
		Name : "tom",
		Age : 20,
	}
	reflectTest02(stu)

}
```

执行结果：

![image-20220315204805258](https://gitee.com/jobim/blogimage/raw/master/img/20220315204805.png)

注意：使用反射的方式来获取变量的值(并返回对应的类型)，要求数据类型匹配，比如x是int那么就应该使用reflect.Value(x).Int()，而不能使用其它的，否则报panic





## 2.2 类型（Type）与种类（Kind）

* **Go 程序中的类型（Type）指的是系统原生数据类型，如 int、string、bool、float32 等类型，以及使用 type 关键字定义的类型，这些类型的名称就是其类型本身的名称。例如使用 type A struct{} 定义结构体时，A 就是 struct{} 的类型。**

* **种类（Kind）指的是对象归属的品种，在 reflect 包中有如下定义：**

  ```go
  const (
      Invalid Kind = iota
      Bool
      Int
      Int8
      Int16
      Int32
      Int64
      Uint
      Uint8
      Uint16
      Uint32
      Uint64
      Uintptr
      Float32
      Float64
      Complex64
      Complex128
      Array
      Chan
      Func
      Interface
      Map
      Ptr
      Slice
      String
      Struct
      UnsafePointer
  )
  ```

比如:`var num int` = 10，`num` 的 `Type` 是 `int`, `Kind` 也是 `int`；

比如: `var stu Student stu` 的`Type` 是 `packageXXX.Student` , `Kind`是 `struct`。



## 2.3 通过反射获取值信息

### 2.3.1 从反射值对象获取值

* **反射值获取原始值的方法:**

  | 方法名                  | 说 明                                                        |
  | ----------------------- | ------------------------------------------------------------ |
  | Interface() interface{} | 将值以 interface{} 类型返回，可以通过类型断言转换为指定类型  |
  | Int() int64             | 将值以 int 类型返回，所有有符号整型均可以此方式返回          |
  | Uint() uint64           | 将值以 uint 类型返回，所有无符号整型均可以此方式返回         |
  | Float() float64         | 将值以双精度（float64）类型返回，所有浮点数（float32、float64）均可以此方式返回 |
  | Bool() bool             | 将值以 bool 类型返回                                         |
  | Bytes() []bytes         | 将值以字节数组 []bytes 类型返回                              |
  | String() string         | 将值以字符串类型返回                                         |

* 代码示例：

  ```go
  package main
  
  import (
  	"fmt"
  	"reflect"
  )
  
  func main() {
  
  	//声明整型变量a并赋初值
  	var a int = 1024
  
  	//获取变量a的反射值对象
  	valueOfA := reflect.ValueOf(a)
  
  	//获取interface{}类型的值，通过类型断言转换
  	var getA int = valueOfA.Interface().(int)
  
  	//获取64位的值，强制类型转换为int类型
  	var getB int = int(valueOfA.Int())
  
  	fmt.Println(getA, getB)	//1024 1024
  }
  ```

### 2.3.2 通过反射访问结构体成员的值

* **反射值对象（reflect.Value）提供对结构体访问的方法，通过这些方法可以完成对结构体任意值的访问，如下表所示：**

  | 方 法                                          | 备 注                                                        |
  | ---------------------------------------------- | ------------------------------------------------------------ |
  | Field(i int) Value                             | 根据索引，返回索引对应的结构体成员字段的反射值对象。当值不是结构体或索引超界时发生宕机 |
  | NumField() int                                 | 返回结构体成员字段数量。当值不是结构体或索引超界时发生宕机   |
  | FieldByName(name string) Value                 | 根据给定字符串返回字符串对应的结构体字段。没有找到时返回零值，当值不是结构体或索引超界时发生宕机 |
  | FieldByIndex(index []int) Value                | 多层成员访问时，根据 []int 提供的每个结构体的字段索引，返回字段的值。 没有找到时返回零值，当值不是结构体或索引超界时发生宕机 |
  | FieldByNameFunc(match func(string) bool) Value | 根据匹配函数匹配需要的字段。找到时返回零值，当值不是结构体或索引超界时发生宕机 |

* 代码示例：

  ```go
  package main
  import (
  	"fmt"
  	"reflect"
  )
  //定义了一个Monster结构体
  type Monster struct {
  	Name  string `json:"name"`
  	Age   int `json:"monster_age"`
  	Score float32 `json:"成绩"`
  	Sex   string
  }
  
  //方法， 接收四个值，给s赋值
  func (s Monster) Set(name string, age int, score float32, sex string) {
  	s.Name = name
  	s.Age = age
  	s.Score = score
  	s.Sex = sex
  }
  
  func TestStruct(a interface{}) {
  	//获取reflect.Type 类型
  	typ := reflect.TypeOf(a)
  	//获取reflect.Value 类型
  	val := reflect.ValueOf(a)
  	//获取到a对应的类别
  	kd := val.Kind()
  	//如果传入的不是struct，就退出
  	if kd !=  reflect.Struct {
  		fmt.Println("expect struct")
  		return
  	}
  
  	//获取到该结构体有几个字段
  	num := val.NumField()
  
  	fmt.Printf("struct has %d fields\n", num) //4
  	//变量结构体的所有字段
  	for i := 0; i < num; i++ {
  		fmt.Printf("Field %d: 值为=%v\n", i, val.Field(i))
  		//获取到struct标签, 注意需要通过reflect.Type来获取tag标签的值
  		tagVal := typ.Field(i).Tag.Get("json")
  		//如果该字段于tag标签就显示，否则就不显示
  		if tagVal != "" {
  			fmt.Printf("Field %d: tag为=%v\n", i, tagVal)
  		}
  	}
  
  }
  func main() {
  	//创建了一个Monster实例
  	var a Monster = Monster{
  		Name:  "黄鼠狼精",
  		Age:   400,
  		Score: 30.8,
  	}
  	//将Monster实例传递给TestStruct函数
  	TestStruct(a)	
  }
  ```

  执行结果：

  ![image-20220316091219191](https://gitee.com/jobim/blogimage/raw/master/img/20220316091219.png)

### 2.3.3 判断反射值的空和有效性

**反射值对象的零值和有效性判断方法：**

| 方 法          | 说 明                                                        |
| -------------- | ------------------------------------------------------------ |
| IsNil() bool   | 返回值是否为 nil。如果值类型不是通道（channel）、函数、接口、map、指针或 切片时发生 panic，类似于语言层的`v== nil`操作 |
| IsValid() bool | 判断值是否有效。 当值本身非法时，返回 false，例如 reflect Value不包含任何值，值为 nil 等。 |

代码示例：

```go
func main() {

	//*int的空指针
	var a *int
	fmt.Println("var a *int:", reflect.ValueOf(a).IsNil())

	//nil值
	fmt.Println("nil:", reflect.ValueOf(nil).IsValid())

	//*int类型的空指针
	fmt.Println("(*int)(nil):", reflect.ValueOf((*int)(nil)).Elem().IsValid())

	//实例化一个结构体
	s := struct {}{}

	//尝试从结构体中查找一个不存在的字段
	fmt.Println("不存在的结构体成员:", reflect.ValueOf(s).FieldByName("").IsValid())

	//尝试从结构体中查找一个不存在的方法
	fmt.Println("不存在的方法:", reflect.ValueOf(s).MethodByName("").IsValid())

	//实例化一个map
	m := map[int]int{}

	//尝试从map中查找一个不存在的键
	fmt.Println("不存在的键:", reflect.ValueOf(m).MapIndex(reflect.ValueOf(3)).IsValid())
}
```

输出结果：

```shell
var a *int: true
nil: false
(*int)(nil): false
不存在的结构体成员: false
不存在的方法: false
不存在的键: false
```

**IsNil() 常被用于判断指针是否为空；IsValid() 常被用于判定返回值是否有效。**

### 2.3.4 通过反射修改变量的值

**反射值对象的判定及获取元素的方法：**

| 方法名         | 备 注                                                        |
| -------------- | ------------------------------------------------------------ |
| Elem() Value   | 取值指向的元素值，类似于语言层`*`操作。当值类型不是指针或接口时发生宕机，空指针时返回 nil 的 Value |
| Addr() Value   | 对可寻址的值返回其地址，类似于语言层`&`操作。当值不可寻址时发生宕机 |
| CanAddr() bool | 表示值是否可寻址                                             |
| CanSet() bool  | 返回值能否被修改。要求值可寻址且是导出的字段                 |

**值修改相关方法：**

| 方法名              | 备注                                                         |
| ------------------- | ------------------------------------------------------------ |
| Set(x Value)        | 将值设置为传入的反射值对象的值                               |
| Setlnt(x int64)     | 使用 int64 设置值。当值的类型不是 int、int8、int16、 int32、int64 时会发生宕机 |
| SetUint(x uint64)   | 使用 uint64 设置值。当值的类型不是 uint、uint8、uint16、uint32、uint64 时会发生宕机 |
| SetFloat(x float64) | 使用 float64 设置值。当值的类型不是 float32、float64 时会发生宕机 |
| SetBool(x bool)     | 使用 bool 设置值。当值的类型不是 bod 时会发生宕机            |
| SetBytes(x []byte)  | 设置字节数组 []bytes值。当值的类型不是 []byte 时会发生宕机   |
| SetString(x string) | 设置字符串值。当值的类型不是 string 时会发生宕机             |

**值可被修改的两个条件：**

* **可被寻址，简单地说就是这个变量必须能被修改，需要传入对应的指针类型**
  * `v := reflect.ValueOf(&a)`，将`a`的地址传递给了`ValueOf`，值传递复制的就是`a`的地址。
  * `v = v.Elem()`，这部分很关键，因为传递的是`a`的地址，那么对应`ValueOf函数`的入参的值就是一个地址，地址是禁止修改的。`v.Elem()`就是解引用，返回的`v`就是变量`a`真正的`reflection Value`。
* **结构体成员中，字段可以被导出，即字段需要大写**



**代码示例：**

```go
func main() {

	type dog struct {
		// 如果希望通过反射能修改该值，则必须保证首字母大写
		LegCount int
	}

	//获取dog实例的反射值对象
	//如果希望反射能修改值，则reflect.ValueOf必须传入指针类型
	valueOfDog := reflect.ValueOf(&dog{})

  	// 取出dog实例地址的元素
	valueOfDog = valueOfDog.Elem()

	//获取legCount字段的值
	vLegCount := valueOfDog.FieldByName("LegCount")

	//尝试设置legCount的值
	vLegCount.SetInt(4)

	fmt.Println(vLegCount.Int())	//4
}
```



### 2.3.5 通过类型信息创建实例

**`func New(typ Type) Value`**

* **New返回一个Value类型值，该值持有一个指向类型为typ的新申请的零值的指针，返回值的Type为PtrTo(type)**

当已知 reflect.Type 时，可以动态地创建这个类型的实例，实例的类型为指针。例如 reflect.Type 的类型为 int 时，创建 int 的指针，即`*int`，代码如下：

```go
func main() {

	var a int

	//取变量a的反射类型对象
	typeOfA := reflect.TypeOf(a)

	//根据反射类型对象创建类型实例
	aIns := reflect.New(typeOfA)

	//输出Value的类型和种类
	fmt.Println(aIns.Type(), aIns.Kind())	//*int ptr
}
```



### 2.3.6 通过反射调用函数

**如果反射值对象（reflect.Value）中值的类型为函数时，可以通过 reflect.Value 调用该函数。使用反射调用函数时，需要将参数使用反射值对象的切片 []reflect.Value 构造后传入 Call() 方法中，调用完成时，函数的返回值通过 []reflect.Value 返回。**

反射调用函数:

```go
//普通函数
func add(a, b int) int {
	return a + b
}

func main() {

	//将函数包装为反射值对象
	funcValue := reflect.ValueOf(add)

	//构造函数参数，传入两个整形值
	paramList := []reflect.Value{reflect.ValueOf(2), reflect.ValueOf(3)}

	//反射调用函数
	retList := funcValue.Call(paramList)

	fmt.Println(retList[0].Int())
}
```



### 2.3.7 通过反射调用方法

**调用方法和调用函数是一样的，只不过结构体需要先通过rValue.Method()先获取方法再调用**

```go
package main
import (
	"fmt"
	"reflect"
)
//定义了一个Monster结构体
type Monster struct {
	Name  string `json:"name"`
	Age   int `json:"monster_age"`
	Score float32 `json:"成绩"`
	Sex   string
}

//方法，返回两个数的和
func (s Monster) GetSum(n1, n2 int) int {
	return n1 + n2
}
//方法， 接收四个值，给s赋值
func (s Monster) Set(name string, age int, score float32, sex string) {
	s.Name = name
	s.Age = age
	s.Score = score
	s.Sex = sex
}

//方法，显示s的值
func (s Monster) Print() {
	fmt.Println("---start~----")
	fmt.Println(s)
	fmt.Println("---end~----")
}
func TestStruct(a interface{}) {
	//获取reflect.Value 类型
	val := reflect.ValueOf(a)

	//获取到该结构体有多少个方法
	numOfMethod := val.NumMethod()
	fmt.Printf("struct has %d methods\n", numOfMethod)
	
	//var params []reflect.Value
	//方法的排序默认是按照 函数名的排序（ASCII码）
	val.Method(1).Call(nil) //获取到第二个方法。调用它

	
	//调用结构体的第1个方法Method(0)
	var params []reflect.Value  //声明了 []reflect.Value
	params = append(params, reflect.ValueOf(10))
	params = append(params, reflect.ValueOf(40))
	res := val.Method(0).Call(params) //传入的参数是 []reflect.Value, 返回[]reflect.Value
	fmt.Println("res=", res[0].Int()) //返回结果, 返回的结果是 []reflect.Value*/

}
func main() {
	//创建了一个Monster实例
	var a Monster = Monster{
		Name:  "黄鼠狼精",
		Age:   400,
		Score: 30.8,
	}
	//将Monster实例传递给TestStruct函数
	TestStruct(a)	
}

```

执行结果：

![image-20220316091750438](https://gitee.com/jobim/blogimage/raw/master/img/20220316091750.png)







# 三、网络编程

## 3.1 TCP通信

<font color='blue' size='4'>**TCP通信常用API：**</font>

**Server端：**

* **`Listen函数`**

  ![img](https://img2018.cnblogs.com/blog/720430/202001/720430-20200106162640116-1166530080.png)
  * network: 选用的协议:TCP、UDP 如: "tcp"或"udp"
  * address：IP地址+端口号 如: "127.0.0.1:8000"或":8000"

* **`Listener接口`**

  ![img](https://img2018.cnblogs.com/blog/720430/202001/720430-20200106162651462-1412042685.png)

* **`Conn接口`**

  ![img](https://img2018.cnblogs.com/blog/720430/202001/720430-20200106162701452-1483515998.png)

**Client端：**

* **`Dial函数：用于创建一个链接（如tcp或udp）`**

  ![img](https://img2018.cnblogs.com/blog/720430/202001/720430-20200106162906263-623283986.png)

  * network: 选用的协议:TCP、UDP 如: "tcp"或"udp"
  * address：IP地址+端口号 如: "127.0.0.1:8000"或":8000"



<font color='blue' size='4'>**服务端程序的处理流程：**</font>

1. 监听端口
2. 接收客户端请求建立链接
3. 创建goroutine处理链接。

服务端代码：

```go
package main
import (
	"fmt"
	"net" //做网络socket开发时,net包含有我们需要所有的方法和函数
)

func process(conn net.Conn) {

	//这里我们循环的接收客户端发送的数据
	defer conn.Close() //关闭conn

	for {
		//创建一个新的切片
		buf := make([]byte, 1024)
		//conn.Read(buf)
		//1. 等待客户端通过conn发送信息
		//2. 如果客户端没有wrtie[发送]，那么协程就阻塞在这里
		//fmt.Printf("服务器在等待客户端%s 发送信息\n", conn.RemoteAddr().String())
		n , err := conn.Read(buf) //从conn读取
		if err != nil {
			fmt.Printf("客户端退出 err=%v", err)
			return //!!!
		}
		//3. 显示客户端发送的内容到服务器的终端
		fmt.Print(string(buf[:n]))
		//回写数据给客户端
		conn.Write([]byte("数据已收到（来自服务端的消息）"))
	}

}

func main() {

	fmt.Println("服务器开始监听....")
	//net.Listen("tcp", "0.0.0.0:8888")
	//1. tcp 表示使用网络协议是tcp
	//2. 0.0.0.0:8888 表示在本地监听 8888端口
	listen, err := net.Listen("tcp", "0.0.0.0:8888")
	if err != nil {
		fmt.Println("listen err=", err)
		return 
	}
	defer listen.Close() //延时关闭listen

	//循环等待客户端来链接我
	for {
		//等待客户端链接
		fmt.Println("等待客户端来链接....")
		conn, err := listen.Accept()
		if err != nil {
			fmt.Println("Accept() err=", err)
			
		} else {
			fmt.Printf("Accept() suc con=%v 客户端ip=%v\n", conn, conn.RemoteAddr().String())
		}
		//这里准备其一个协程，为客户端服务
		go process(conn)
	}
	
	//fmt.Printf("listen suc=%v\n", listen)
}
```



<font color='blue' size='4'>**TCP客户端进行TCP通信的流程：**</font>

1. 建立与服务端的链接
2. 进行数据收发
3. 关闭链接

客户端代码：

```go
func main() {

	conn, err := net.Dial("tcp", "127.0.0.1:8888")
	if err != nil {
		fmt.Println("client dial err=", err)
		return 
	}
	defer conn.Close() //关闭连接
	//功能一：客户端可以发送单行数据，然后就退出
	reader := bufio.NewReader(os.Stdin) //os.Stdin 代表标准输入[终端]

	for {

		//从终端读取一行用户输入，并准备发送给服务器
		line, err := reader.ReadString('\n')
		if err != nil {
			fmt.Println("readString err=", err)
		}
		//如果用户输入的是 exit就退出
		line = strings.Trim(line, " \r\n")
		if line == "exit" {
			fmt.Println("客户端退出..")
			break
		}

		//再将line 发送给 服务器
		_, err = conn.Write([]byte(line + "\n"))
		if err != nil {
			fmt.Println("conn.Write err=", err)	
		}
		//接收服务端发送的数据
		buf := [512]byte{}
        n, err := conn.Read(buf[:])
        if err != nil {
            fmt.Println("recv failed, err:", err)
            return
        }
        fmt.Println(string(buf[:n]))
	}
	
}
```

执行结果：

![image-20220316105914856](https://gitee.com/jobim/blogimage/raw/master/img/20220316105914.png)



## 3.2 UDP通信

<font color='blue' size='4'>**UDP通信常用API：**</font>

* **创建监听地址**

  ```go
  func ResolveUDPAddr(network, address string) (*UDPAddr, error)
  ```

  ResolveUDPAddr将addr作为UDP地址解析并返回。参数addr格式为"host:port"或"[ipv6-host%zone]:port"，解析得到网络名和端口名；net必须是"udp"、"udp4"或"udp6"。
  IPv6地址字面值/名称必须用方括号包起来，如"[::1]:80"、"[ipv6-host]:http"或"[ipv6-host%zone]:80"。

* **创建用于通信的socket**

  ```go
  func ListenUDP(network string, laddr *UDPAddr) (*UDPConn, error)
  ```

  ListenUDP创建一个接收目的地是本地地址laddr的UDP数据包的网络连接。net必须是"udp"、"udp4"、"udp6"；如果laddr端口为0，函数将选择一个当前可用的端口，可以用Listener的Addr方法获得该端口。返回的*UDPConn的ReadFrom和WriteTo方法可以用来发送和接收UDP数据包（每个包都可获得来源地址或设置目标地址）。

* **接受UDP数据**

  ```go
  func (c *UDPConn) ReadFromUDP(b []byte) (n int, addr *UDPAddr, err error)
  ```

  ReadFromUDP从c读取一个UDP数据包，将有效负载拷贝到b，返回拷贝字节数和数据包来源地址。

  ReadFromUDP方法会在超过一个固定的时间点之后超时，并返回一个错误。

* **写出数据到UDP**

  ```go
  func (c *UDPConn) WriteToUDP(b []byte, addr *UDPAddr) (int, error)
  ```

  WriteToUDP通过c向地址addr发送一个数据包，b为包的有效负载，返回写入的字节。

  WriteToUDP方法会在超过一个固定的时间点之后超时，并返回一个错误。在面向数据包的连接上，写入超时是十分罕见的。



服务端代码：

```go
package main

import (
    "fmt"
    "net"
)

func main() {

    //0.本应从步骤1开始,但是在写步骤1的时候发现,步骤1还需要*UDPAddr类型的参数,所以需要先创建一个*DUPAddr
    //组织一个udp地址结构,指定服务器的IP+port
    udpAddr, err := net.ResolveUDPAddr("udp", "127.0.0.1:8000")
    if err != nil {
        fmt.Printf("net.ResolveUDPAddr()函数执行出错,错误为:%v\n", err)
        return
    }
    fmt.Printf("UDP服务器地址结构创建完成!!!\n")

    //1.创建用户通信的socket
    //由于ListenUDP需要一个*UDPAddr类型的参数,所以我们还需要先创建一个监听地址
    udpConn, err := net.ListenUDP("udp", udpAddr)
    if err != nil {
        fmt.Printf("net.ListenUDP()函数执行出错,错误为:%v\n", err)
        return
    }
    defer udpConn.Close()
    fmt.Printf("UDP服务器通信socket创建完成!!!\n")

    for {
        //2.读取客户端发送的数据(阻塞发生在ReadFromUDP()方法中)
        buf := make([]byte, 4096)
        //ReadFromUDP()方法返回三个值,分别是读取到的字节数,客户端的地址,error
        n, clientUDPAddr, err := udpConn.ReadFromUDP(buf)
        if err != nil {
            fmt.Printf("*UDPAddr.ReadFromUDP()方法执行出错,错误为:%v\n", err)
            continue
        }
        //3.模拟处理数据
        fmt.Printf("服务器读到%v的数据:%s\n",clientUDPAddr, buf[:n])

        //4.回写数据给客户端
        _, err = udpConn.WriteToUDP([]byte("I am OK!"), clientUDPAddr)
        if err != nil {
            fmt.Printf("*UDPAddr.WriteToUDP()方法执行出错,错误为:%v\n", err)
            continue
        }
    }
}
```

客户端代码：

```go
package main

import (
    "fmt"
    "net"
    "os"
)

func main() {
    conn, err := net.Dial("udp", "127.0.0.1:8000")
    if err != nil {
        fmt.Printf("net.Dial()函数执行出错,错误为:%v\n", err)
        return
    }
    defer conn.Close()

    go func() {
        buf := make([]byte, 4096)
        for {
            //从键盘读取内容,放入buf
            n, err := os.Stdin.Read(buf)
            if err != nil {
                fmt.Printf("os.Stdin.Read()执行出错,错误为:%v\n", err)
                return
            }
            //给服务器发送
            conn.Write(buf[:n])
        }
    }()
    for {
        buf := make([]byte, 4096)
        n, err := conn.Read(buf)
        if err != nil {
            fmt.Printf("Conn.Read()方法执行出错,错误为:%v\n", err)
            return
        }
        fmt.Printf("服务器发来数据:%s\n", buf[:n])
    }
}
```

执行结果：

![image-20220316112001045](https://gitee.com/jobim/blogimage/raw/master/img/20220316112001.png)





