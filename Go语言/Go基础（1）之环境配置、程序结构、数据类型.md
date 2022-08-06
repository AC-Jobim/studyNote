# 一、简介

## 1.1 Go语言的介绍

* Go语言是编程语言设计的又一次尝试，是对类C语言的重大改进，它不但能让你访问底层操作系统，还提供了强大的网络编程和并发编程支持。Go语言的用途众多，可以进行网络编程、系统编程、并发编程、分布式编程。
* Go语言的推出，旨在不损失应用程序性能的情况下降低代码的复杂性，具有“部署简单、并发性好、语言设计良好、执行性能好”等优势，目前国内诸多 IT 公司均已采用Go语言开发项目。
* Go语言有时候被描述为“C 类似语言”，或者是“21 世纪的C语言”。Go 从C语言继承了相似的表达式语法、控制流结构、基础数据类型、调用参数传值、指针等很多思想，还有C语言一直所看中的编译后机器码的运行效率以及和现有操作系统的无缝适配。
* 因为Go语言没有类和继承的概念，所以它和 Java 或 C++ 看起来并不相同。但是它通过接口（interface）的概念来实现多态性。Go语言有一个清晰易懂的轻量级类型系统，在类型之间也没有层级之说。因此可以说Go语言是一门混合型的语言。
* 此外，很多重要的开源项目都是使用Go语言开发的，其中包括 Docker、Go-Ethereum、Thrraform 和 Kubernetes。



更多请参考go的官方文档：https://studygolang.com/pkgdoc



## 1.2 环境配置

<font color='blue' size='4'>**Go安装包下载：**</font>

Go官网下载地址：https://golang.org/dl/

Go官方镜像站（推荐）：https://golang.google.cn/dl/

![image-20220216111347929](https://gitee.com/jobim/blogimage/raw/master/img/20220216111347.png)

**解压之后的一些文件：**

![image-20220216113416818](https://gitee.com/jobim/blogimage/raw/master/img/20220216113416.png)

各文件的含义：

| 目录名 | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| api    | 每个版本的 api 变更差异                                      |
| bin    | go 源码包编译出的编译器（go）、文档工具（godoc）、格式化工具（gofmt） |
| doc    | 英文版的 Go 文档                                             |
| lib    | 引用的一些库文件                                             |
| misc   | 杂项用途的文件，例如 Android 平台的编译、git 的提交钩子等    |
| pkg    | Windows 平台编译好的中间文件                                 |
| src    | 标准库的源码                                                 |
| test   | 测试用例                                                     |



<font color='blue' size='4'>**环境变量配置**</font>

**需要配置的环境变量：**

![image-20220216111858051](https://gitee.com/jobim/blogimage/raw/master/img/20220216111858.png)



* **新建系统变量`GOROOT`**，这个是配合go编译器安装的目录

  ![image-20220215174334690](https://gitee.com/jobim/blogimage/raw/master/img/20220215174334.png)

* **新建系统变量`GOPATH`**，自己写的代码要放到这个变量中配置的目录中，go编译器才会找到并编译

  ![image-20220215175013493](https://gitee.com/jobim/blogimage/raw/master/img/20220215175013.png)

* **修改path变量**，把你的go安装的bin目录添加到里面去

  ![image-20220216112601633](https://gitee.com/jobim/blogimage/raw/master/img/20220216112601.png)



环境变量设置好后，可以通过`go env `命令来进行测试。

![image-20220216113445543](https://gitee.com/jobim/blogimage/raw/master/img/20220216113445.png)



![image-20220215180548549](https://gitee.com/jobim/blogimage/raw/master/img/20220215180548.png)



## 1.3 快速入门

> 开发一个hello.go程序，可以输出"hello,world”

**hello.go代码：**

![image-20220308162107544](https://gitee.com/jobim/blogimage/raw/master/img/20220308162107.png)

对上图的说明：

* **go文件的后缀是.go**

* **`package main`**：表示该hello.go文件所在的包是main，在 go 中，每个文件都必须归属于一个包。

* **`import "fmt"`**：表示引入一个包，包名fmt，引入该包后，就可以使用fmt包的函数，比如: fmt.Println()

* ```go
  func main(){
  }
  ```

  * func是一个关键字，表示一个函数。
  * **main是函数名，是一个主函数，即我们程序的入口**。

* fmt.Println("hello")：表示调用fmt包的函数Println输出“hello,world”

**代码的执行：**

* 通过**`go build`**命令对该go文件进行编译，生成.exe文件

  ![image-20220308170815658](https://gitee.com/jobim/blogimage/raw/master/img/20220308170815.png)

* 运行hello.exe文件即可

  ![image-20220308170904470](https://gitee.com/jobim/blogimage/raw/master/img/20220308170904.png)

* 可以通过**`go run`**命令可以直接运行hello.go程序

  ![image-20220308170913537](https://gitee.com/jobim/blogimage/raw/master/img/20220308170913.png)

* **编译时可以指定生成的可执行文件名**

  ![image-20220215191037230](https://gitee.com/jobim/blogimage/raw/master/img/20220215191037.png)

## 1.4 Go开发的注意事项

* Go源文件以"go”为扩展名。

* Go应用程序的执行入口是main()函数，main函数保存在名为main的包里。**如果 main 函数不在main包里，构建工具就不会生成可执行的文件**
* Go语言严格区分大小写。
* Go方法由一条条语句构成，**每个语句后不需要分号(编译器会主动把特定符号后的换行符转换为分号)**。比如：函数的左括号`{`必须和`func`函数声明在同一行上，且位于末尾，不能独占一行，而在表达式`x + y`中，可在`+`后换行，不能在`+`前换行（译注：以+结尾的话不会被插入分号分隔符，但是以x结尾的话则会被分号分隔符，从而导致编译错误）
* **go语言定义的变量或者import的包如果没有使用到，代码不能编译通过**



# 二、程序结构

## 2.1 标识符

* **Golang 对各种变量、方法、函数等命名时使用的字符序列称为标识符**

**表示符命名规则：**

* 以字母或下画线开始，由多个字母、数字和下画线组合而成。
* 区分大小写。
* 使用驼峰( camel case）拼写格式。
* 局部变量优先使用短名。
* 不要使用保留关键字。
* 不建议使用与预定义常量、类型、内置函数相同的名字。
* 专有名词通常会全部大写,例如escapeHTML。

**注意：符号名字首字母大小写决定了其作用域。首字母大写的为导出成员，可被包外引用，而小写则仅能在包内使用。**



<font color='blue' size='4'>**空标识符：**</font>

**和Python类似，Go也有个名为“`_`”的特殊成员 ( blank identifier )。通常作为忽略占位符使用，可作表达式左值，无法读取内容。**

```go
import "strconv"
func main({
    x, _ := strconv.Atoi("12")  //忽略Atoi的err返回值
    println(x)
}
```

**空标识符可用来临时规避编译器对未使用变量和导入包的错误检查。**



<font color='blue' size='4'>**系统保留关键字：**</font>

* 关键字即是被Go语言赋予了特殊含义的单词，也可以称为保留字。Go语言中的关键字一共有 25 个

  ![image-20220309175254696](https://gitee.com/jobim/blogimage/raw/master/img/20220309175254.png)



<font color='blue' size='4'>**系统的预定义标识符：**</font>

* **在Go语言中还存在着一些特殊的标识符，叫做预定义标识符**

  ![image-20220309175244281](https://gitee.com/jobim/blogimage/raw/master/img/20220309175244.png)

## 2.2 变量

<font color='blue' size='4'>**变量的定义：**</font>

* **`var`声明语句可以创建一个特定类型的变量，然后给变量附加一个名字，并且设置变量的初始值**。变量声明的一般语法如下：

  ```Go
  var 变量名字 类型 = 表达式
  ```

* 其中**`类型`**或 **`= 表达式`**两个部分可以省略其中的一个。

  * 如果省略的是类型信息，那么将根据初始化表达式来推导变量的类型信息。
  * 如果初始化表达式被省略，那么将用零值初始化该变量。

  ```go
  var x int		// 自动初始化为0
  var y = false	// 自动推断为bool类型
  ```

  > * 数值类型变量对应的零值是0
  >
  > * 布尔类型变量对应的零值是false
  > * 字符串类型对应的零值是空字符串
  > * 接口或引用类型（包括slice、指针、map、chan和函数）变量对应的零值是nil。
  > * 数组或结构体等聚合类型对应的零值是每个元素或字段都是对应该类型的零值

* 可一次定义多个变量，也可以初始化定义不同类型

  ```go
  var i, j, k int                 // int, int, int
  var b, f, s = true, 2.3, "four" // bool, float64, string
  ```

* **可以以组方式整理多行变量定义**

  ```go
  var (
  	x, y int
      a, s = 100, "abc"
  )
  ```

<font color='blue' size='4'>**简短模式：**</font>

* **使用更加简短的变量定义和初始化语法：**

  ```go
  func main() {
      x := 100
      a, s := 1, "abc"
  }
  ```

  简短模式的使用限制：

  * **定义变量，同时显示初始化**
  * 不能提供数据类型，根据初始化进行类型推断
  * **只能用在函数内部**

**简短模式使用注意：**

* **`:=`是一个变量声明语句，而`=`是一个变量赋值操作**

  ```go
  i, j = j, i // 交换 i 和 j 的值
  ```

* 简短模式并不总是重新定义变量，也可能是**部分退化的赋值操作**。退化的前提条件：**最少有一个新变量被定义**，且必须是同一作用域

  ```go
  func main() {
  	x :=  100
  	x, y := 200, "abc" // 注意：x退化为赋值操作，仅有y是变量定义
  	println(x) //200
  	println(y) //abc
  }
  ```

<font color='blue' size='4'>**变量的使用细节：**</font>

* **未使用错误：编译器将未使用局部变量当作错误**。不要觉得麻烦，这有助于培养良好的编码习惯

  ```go
  var x int
  func main() {
      y := 10	//报错：y declared and not used
  }
  ```

* **查看变量的字节大小和数据类型**

  ```go
  //查看某个变量的字节大小和数据类型
  var n2 int64 = 10
  fmt.Printf("n2的类型 %T n2占用的字节数是 %d ",n2,unsafe.Sizeof(n2)) //n2的类型 int64 n2占用的字节数是 8
  ```

  

## 2.3 常量

* **常量值必须是`编译期可确定`的字符、字符串、数字或布尔值。可指定常量类型，或由译器通过初始化值推断。**
* **常量使用const修饰**

* **可在函数代码块中定义常量,不曾使用的常量不会引发编译错误**

  ```go
  const x,y int =123, 0x22
  const s = "hello, world! "
  const c = '我'
  
  //以组的方式定义多行变量
  const(
  	i, f= 1, 0.123	//int, float64(默认)
  	b = false
  )
  
  //可在函数代码块中定义常量,不曾使用的常量不会引发编译错误
  func main() {
  	const x = 123
  	println(x)
  	const y = 1.23 //未使用,不会引发编译错误
  }
  ```

* **在常量组中如果不指定类型和初始化值，则与上一行非空常量初始化表达式值相同**

  ```go
  func main() {
  	const (
  		x uint16 = 123
  		y				// 与上一行x类型、右值相同
  		s = "abc"
  		z 				// 与s类型、右值相同
  	)
  	fmt.Printf("%T, %v\n", y, y) //uint16, 123
  	fmt.Printf("%T, %v\n", s, s) //string, abc
  }
  ```



<font color='blue' size='4'>**iota 常量生成器：**</font>

* 在一个const声明语句中，在第一个声明的常量所在的行，**iota将会被置为0，然后在每一个有常量声明的行加一。**

  ```go
  type Weekday int
  
  const (
      Sunday Weekday = iota
      Monday
      Tuesday
      Wednesday
      Thursday
      Friday
      Saturday
  )
  ```

* **可在多常量定义中使用多个iota，它们各自单独计数，只须确保组中每行常量的列数量相同即可。**

  ```go
  const(
  	_, _ = iota, iota * 10	//0, 0 * 10
  	a, b					//1, 1 * 10
  	c, d					//2, 2 * 10
  )
  ```

  

## 2.4 运算符

<font color='blue' size='4'>**算数运算符：**</font>

![image-20220309170304132](https://gitee.com/jobim/blogimage/raw/master/img/20220309170304.png)

**注意：**

* Go语言明确不支持三元运算符

* **Golang 的自增自减只能当做一个独立语言使用**

* **Golang 的++和 --只能写在变量的后面，不能写在变量的前面**，即:只有 a++ a--没有++a --a

  ![image-20220309170941657](https://gitee.com/jobim/blogimage/raw/master/img/20220309170941.png)

<font color='blue' size='4'>**关系运算符：**</font>

![image-20220309171009705](https://gitee.com/jobim/blogimage/raw/master/img/20220309171009.png)

<font color='blue' size='4'>**逻辑运算符：**</font>

下表列出了所有Go语言的逻辑运算符。假定 A 值为 True，B 值为 False。

| 运算符 | 描述                                                         | 实例               |
| :----- | :----------------------------------------------------------- | :----------------- |
| &&     | 逻辑 AND 运算符。 如果两边的操作数都是 True，则条件 True，否则为 False。 | (A && B) 为 False  |
| \|\|   | 逻辑 OR 运算符。 如果两边的操作数有一个 True，则条件 True，否则为 False。 | (A \|\| B) 为 True |
| !      | 逻辑 NOT 运算符。 如果条件为 True，则逻辑 NOT 条件 False，否则为 True。 | !(A && B) 为 True  |



<font color='blue' size='4'>**赋值运算符：**</font>

下表列出了所有Go语言的赋值运算符。

| 运算符 | 描述                                           | 实例                                  |
| :----- | :--------------------------------------------- | :------------------------------------ |
| =      | 简单的赋值运算符，将一个表达式的值赋给一个左值 | C = A + B 将 A + B 表达式结果赋值给 C |
| +=     | 相加后再赋值                                   | C += A 等于 C = C + A                 |
| -=     | 相减后再赋值                                   | C -= A 等于 C = C - A                 |
| *=     | 相乘后再赋值                                   | C *= A 等于 C = C * A                 |
| /=     | 相除后再赋值                                   | C /= A 等于 C = C / A                 |
| %=     | 求余后再赋值                                   | C %= A 等于 C = C % A                 |
| <<=    | 左移后赋值                                     | C <<= 2 等于 C = C << 2               |
| >>=    | 右移后赋值                                     | C >>= 2 等于 C = C >> 2               |
| &=     | 按位与后赋值                                   | C &= 2 等于 C = C & 2                 |
| ^=     | 按位异或后赋值                                 | C ^= 2 等于 C = C ^ 2                 |
| \|=    | 按位或后赋值                                   | C \|= 2 等于 C = C \| 2               |



<font color='blue' size='4'>**位运算符：**</font>

位运算符对整数在内存中的二进制位进行操作。假定 A 为60，B 为13

| 运算符 | 描述                                                         | 实例                                   |
| :----- | :----------------------------------------------------------- | :------------------------------------- |
| &      | 按位与运算符”&”是双目运算符。 其功能是参与运算的两数各对应的二进位相与。 | (A & B) 结果为 12, 二进制为 0000 1100  |
| \|     | 按位或运算符”\|”是双目运算符。 其功能是参与运算的两数各对应的二进位相或 | (A \| B) 结果为 61, 二进制为 0011 1101 |
| ^      | 按位异或运算符”^”是双目运算符。 其功能是参与运算的两数各对应的二进位相异或，当两对应的二进位相异时，结果为1。 | (A ^ B) 结果为 49, 二进制为 0011 0001  |
| <<     | 左移运算符”<<”是双目运算符。左移n位就是乘以2的n次方。 其功能把”<<”左边的运算数的各二进位全部左移若干位，由”<<”右边的数指定移动的位数，高位丢弃，低位补0。 | A << 2 结果为 240 ，二进制为 1111 0000 |
| >>     | 右移运算符”>>”是双目运算符。右移n位就是除以2的n次方。 其功能是把”>>”左边的运算数的各二进位全部右移若干位，”>>”右边的数指定移动的位数。 | A >> 2 结果为 15 ，二进制为 0000 1111  |



<font color='blue' size='4'>**其他运算符：**</font>

| 运算符 | 描述             | 实例                       |
| :----- | :--------------- | :------------------------- |
| &      | 返回变量存储地址 | &a; 将给出变量的实际地址。 |
| *      | 指针变量。       | *a; 是一个指针变量         |

```go
func main() {
	a := 100
	fmt.Println("a 的地址=", &a)

	var ptr *int = &a
	fmt.Println("ptr 指向的值是=", *ptr) //ptr 指向的值是= 100
}
```



<font color='blue' size='4'>**运算符优先级：**</font>

![image-20220309172749630](https://gitee.com/jobim/blogimage/raw/master/img/20220309172749.png)



## 2.5 流程控制

<font color='blue' size='4'>**if-else 流程控制：**</font>

```go
func main() {
	var age int
	fmt.Println("请输入年龄:")
	fmt.Scanln(&age)

	if age > 18 {
		fmt.Println("你年龄大于18~....")
	} else {
		fmt.Println("你的年龄不大这次放过你了")
	}

}
```

<font color='blue'>**switch分支结构：**</font>

**基本语法：**

```go
switch 表达式 {
 case 表达式1，表达式2，…… ：
 // 语句块1
 case 表达式3，表达式4，…… ：
 // 语句块2
 // 多个case，结构同上
 default ：
 // 语句块3
}
```

**注意：**

* **匹配项后面也不需要再加 break**
* **多个表达式使用逗号间隔**
* **switch 后也可以不带表达式，类似if --else分支来使用**
* **如果在case 语句块后增加fallthrough ,则会继续执行下一个case，也叫 switch穿透**

```go
func main() {

	var n1 int32 = 5
	var n2 int32 = 20
	switch n1 {
		case n2, 10, 5 :  // case 后面可以有多个表达式
			fmt.Println("ok1~")
		case 90 : 
			fmt.Println("ok2~")
		default :
			fmt.Println("ok2~")
	}

	//switch 后也可以不带表达式，类似 if --else分支来使用。【案例演示】
	var score int = 90
	switch {
		case score > 90 :
			fmt.Println("成绩优秀..")
		case score >=70 && score <= 90 :
			fmt.Println("成绩优良...")
		case score >= 60 && score < 70 :
			fmt.Println("成绩及格...")
		default :
			fmt.Println("不及格")
	}

	//switch 的穿透 fallthrought
	var num int = 10
	switch num {
		case 10:
			fmt.Println("ok1")
			fallthrough //默认只能穿透一层
		case 20:
			fmt.Println("ok2")
			fallthrough
		case 30:
			fmt.Println("ok3")	
		default:
			fmt.Println("没有匹配到..")
	}

}
```

* **Type Switch: switch 语句还可以被用于type-switch 来判断某个interface变量中实际指向的变量类型**

  ```go
  func main() {
  	var x interface{}
  	var y = 10.0
  	x = y
  	switch i := x.(type) {
  	case nil:
  		fmt.Printf("× 的类型~:%T",i)
  	case int:
  		fmt.Printf("x 是 int 型")
  	case float64:
  		fmt.Printf("x 是 f1oat64 型")
  	case func(int) float64:
  		fmt.Printf("x 是 func(int) 型")
  	case bool, string:
  		fmt.Printf("x 是 bool 或 string 型")
  	default:
  		fmt.Printf("未知型")
  	}
  }
  ```

  

<font color='blue' size='4'>**for循环：**</font>

**基本语法：**

```go
for 循环变量初始化 ；循环条件 ；循环变量迭代 {
	//循环操作
}
```

**注意：**

* **Go中没有while，do…while循环，但可以通过for循环和break实现其功能**
* **break 出现在多层嵌套循环中可以使用标签（label）表明要终止哪个循环**

```go
func main() {

	//for循环的第一种写法
	//指定标签的形式来使用 break
	for i := 0; i < 4; i++ {
		lable1: // 设置一个标签
		for j := 0; j < 10; j++ {
			if j == 2 {
				break lable1
			}
			fmt.Println("j=", j) 
		}
	}
	//for循环的第二种写法
	j := 1 //循环变量初始化
	for j <= 10 { //循环条件
		fmt.Println("hello,world~", j)
		j++ //循环变量迭代
	}

	//for循环的第三种写法, 这种写法通常会配合break使用
	k := 1
	for {  // 这里也等价 for ; ; { 
		if k <= 10 {
			fmt.Println("ok~~", k)
		} else {
			break //break就是跳出这个for循环
		}
		k++
	}

}
```

**for-range遍历：这是一种同时获取`索引`和`值或键值`的遍历方式**

```go
func main() {
	fmt.Println()
	//字符串遍历方式2-for-range
	str := "abc~ok上海"
	for index, val := range str {
		fmt.Printf("index=%d, val=%c \n", index, val)
	}
}
```



<font color='blue' size='4'>**跳转控制语句-goto：**</font>

* **goto语句可以无条件地转移到程序中指定的行。**

```go
func main() {
	var n int = 30
	// 演示goto的使用
	fmt.Println("ok1")
	if n > 20 {
		goto label1
	}
	fmt.Println("ok2")
	label1:
	fmt.Println("ok3")
}
```



## 2.7 init函数

* **每一个源文件都可以包含一个`init函数`，该函数会在main函数执行前，被Go运行框架调用，也就是说`init会在main函数前被调用`。**

* **如果一个文件同时包含全局变量定义, init 函数和 main函数，则执行的流程：全局变量定义->init函数->main函数**

  ```go
  var age int = test()
  
  //初始化全局变量的时候执行该函数
  func test() int {
  	fmt.Println("test()...")//1
  	return 18
  }
  
  //init函数，通常可以在init函数中完成初始化工作
  func init() {
  	fmt.Println("init()...")//2
  }
  
  func main() {
  	fmt.Println("main()...")//3
  }
  ```

  执行结果：

  ![image-20220223151527267](https://gitee.com/jobim/blogimage/raw/master/img/20220223151527.png)

  

* 如果 main.go 文件中导入了utils.go而且都含有变量定义，init 函数时，执行的流程又是怎么样的呢?

  ![image-20220223160517922](https://gitee.com/jobim/blogimage/raw/master/img/20220223160518.png)







# 三、数据类型



<font color='blue' size='4'>**数据类型介绍：**</font>![image-20220309113944898](https://gitee.com/jobim/blogimage/raw/master/img/20220309113945.png)







<font color='blue' size='4'>**值类型与引用类型：**</font>

* **值类型：基本数据类型int系列、float系列、bool、string、数组和结构体struct。变量直接存储值，内存通常在栈中分配**

* **引用类型：指针、slice切片、map、管道chan、interface 等都是引用类型。变量存储的是一个地址，这个地址对应的空间才真正存储数据(值)，内存通常在堆上分配**。当没有任何变量引用这个地址时，该地址对应的数据空间就成为一个垃圾，由GC来回收





## 3.1 基本数据类型

### 3.1.1 整型

**整数的各个类型：**

| 类型       | 有无符号 | 占用存储空间                        | 范围                                  |
| :--------- | :------- | ----------------------------------- | ------------------------------------- |
| **int8**   | 有符号   | 1字节                               | -128 ~ 127                            |
| **int16**  | 有符号   | 2字节                               | -2^15^ ~ 2^15^-1                      |
| **int32**  | 有符号   | 4字节                               | -2^31^ ~ 2^31^-1                      |
| **int64**  | 有符号   | 8字节                               | -2^63^ ~ 2^63^-1                      |
| **uint8**  | 无符号   | 1字节                               | 0 ~ 255                               |
| **uint16** | 无符号   | 2字节                               | 0 ~ 2^16^-1                           |
| **uint32** | 无符号   | 4字节                               | 0 ~ 2^32^-1                           |
| **uint64** | 无符号   | 8字节                               | 0 ~ 2^64^-1                           |
| **int**    | 有符号   | 32位系统4个字节<br/>64位系统8个字节 | -2^31^ ~ 2^31^-1<br/>-2^63^ ~ 2^63^-1 |
| **uint**   | 无符号   | 32位系统4个字节<br/>64位系统8个字节 | 0 ~ 2^32^-1<br/>0 ~ 2^64^-1           |
| **rune**   | 有符号   | 等价int32<br/>表示一个Unicode码     | -2^31^ ~ 2^31^-1                      |
| **byte**   | 无符号   | 当要存储字符时选用byte              | 2 ~ 255                               |



**整型的使用注意：**

* **int 和 unit 的大小和系统有关**

* **Golang 中没有专门的字符类型，如果要存储单个字符(字母)，一般使用byte来保存**

* **Golang的整型默认声明为int型**

* **unsafe.Sizeof()函数可以查看内存大小**

  ```go
  func main() {
  	//查看变量的数据类型
  	var n1 = 100
  	fmt.Printf("n1的类型 %T \n",n1) //n1的类型 int
      
      var a int32 = 10
  	fmt.Printf("a占用内存大小：%d", unsafe.Sizeof(a)) //a占用内存大小：4
  }
  ```

* **就算在64位平台上 int 和 int64结构完全一致，也分属不同类型，须显式转换。**

  ```go
  func main() {
  	var x int = 100
  	var y int64 = x //错误:cannot use x (type int) as type int64 in assignment
  }
  ```

### 3.1.2 浮点型

**浮点类型分类：**

| 类型        | 有无符号 | 占用存储空间 | 范围                    |
| :---------- | :------- | ------------ | ----------------------- |
| **float32** | 有符号   | 4字节        | -3.403E38 ~ 3.403E38    |
| **float64** | 有符号   | 8字节        | -1.798E308 ~ -1.798E308 |

**浮点型的使用注意：**

* **Golang 的浮点型默认声明为 `float64` 类型。**
* 浮点型常量有两种表示形式
  * 十进制数形式：如：5.12       .512   (必须有小数点）
  * 科学计数法形式：如：5.1234e2 = 5.1234 * 10的2次方

```go
func main() {
	//Golang 的浮点型默认声明为float64 类型
	var num5 = 1.1
	fmt.Printf("num5的数据类型是 %T \n", num5)


	//十进制数形式：如：5.12       .512   (必须有小数点）
	num6 := 5.12
	num7 := .123 //=> 0.123
	fmt.Println("num6=", num6, "num7=", num7)

	//科学计数法形式
	num8 := 5.1234e2 // ? 5.1234 * 10的2次方
	num9 := 5.1234E2 // ? 5.1234 * 10的2次方 shift+alt+向下的箭头
	num10 := 5.1234E-2 // ? 5.1234 / 10的2次方 0.051234
	
	fmt.Println("num8=", num8, "num9=", num9, "num10=", num10)
}
```



### 3.1.3 布尔型

* 布尔类型也叫 bool类型，bool类型数据只允许取值 true和 false
* **bool类型占1个字节**

```go
func main() {
	var b = false
	fmt.Println("b=", b)
	//注意事项
	//1. bool类型占用存储空间是1个字节
	fmt.Println("b的占用空间 =", unsafe.Sizeof(b) ) //b的占用空间 = 1
	//2. bool类型只能取true或者false
}
```

### 3.1.4 字符串

<font color='blue' size='4'>**字符类型：**</font>

* **Golang 中没有专门的字符类型，如果要存储单个字符(字母)，一般使用byte来保存。**

* **如果我们保存的字符在ASCI表的,比如[O-1, a-z,A-Z..]直接可以保存到byte**

* **如果我们保存的字符对应码值大于255,这时我们可以考虑使用int类型保存**

* **如果我们需要按照字符的方式输出，这时我们需要格式化输出，即fmt.Printf("%c",c1)**

  ```go
  //演示golang中字符类型使用
  func main() {
  	
  	var c1 byte = 'a'
  	var c2 byte = '0' //字符的0
  
  	//当我们直接输出byte值，就是输出了的对应的字符的码值
  	// 'a' ==> 
  	fmt.Println("c1=", c1)	//c1= 97
  	fmt.Println("c2=", c2)	//c2= 48
  	//如果我们希望输出对应字符，需要使用格式化输出
  	fmt.Printf("c1=%c c2=%c\n", c1, c2)	//c1=a c2=0
  
  	//var c3 byte = '北' //overflow溢出
  	var c3 int = '北'
  	fmt.Printf("c3=%c c3对应码值=%d\n", c3, c3)	//c3=北 c3对应码值=21271
      
  }
  ```

* 字符常量是用单引号(")括起来的单个字符。例如: 

  ```go
  var c1 byte = 'a'
  var c2 int = '中'
  var c3 byte = '9'
  ```

* Go 语言的字符使用UTF-8编码，如果想查询字符对应的utf8码值http://www.mytju.com/classcode/tools/encode_utf8.asp，英文字母-1个字节汉字-3个字节

* **在Go中，字符的本质是一个整数，直接输出时，是该字符对应的UTF-8编码的码值。**

* **可以直接给某个变量赋一个数字，然后按格式化输出时%c，会输出该数字对应的unicode字符**

  ```go
  //可以直接给某个变量赋一个数字，然后按格式化输出时%c，会输出该数字对应的unicode 字符
  var c4 int = 22269 // 22269 -> '国' 120->'x'
  fmt.Printf("c4=%c\n", c4) //c4=国
  ```

* 字符类型是可以进行运算的，相当于一个整数，因为它都对应有Unicode码

  ```go
  //字符类型是可以进行运算的，相当于一个整数,运输时是按照码值运行
  var n1 = 10 + 'a' //  10 + 97 = 107
  fmt.Println("n1=", n1)	//n1= 107
  ```

  

<font color='blue' size='4'>**String类型：**</font>

**字符串就是一串固定长度的字符连接起来的字符序列。Go的字符串是由单个字节连接起来的。GO语言的字符串的字节使用UTF-8编码标识Unicode文本**

```go
func main() {
	//string的基本使用
	var address string = "北京长城 110 hello world!"
	fmt.Println(address)
}
```

* **Go中字符串是不可变的：字符串一旦赋值了，字符串就不能修改了**

  ![image-20220309155346060](https://gitee.com/jobim/blogimage/raw/master/img/20220309155346.png)

* **如果需要修改字符串，可以先将string ->[]byte/或者[]rune->修改→重写转成string**

  ```go
  func main() {
  
  	str := "hello@atguigu"
  	//string是不可变的，也就说不能通过 str[0] = 'z' 方式来修改字符串 
  	//如果需要修改字符串，可以先将string -> []byte / 或者 []rune -> 修改 -> 重写转成string
  	// "hello@atguigu" =>改成 "zello@atguigu"
  	arr1 := []byte(str) 
  	arr1[0] = 'z'
  	str = string(arr1)
  	fmt.Println("str=", str) //str= zello@atguigu
  
  	// 细节，我们转成[]byte后，可以处理英文和数字，但是不能处理中文
  	// 原因是 []byte 字节来处理 ，而一个汉字，是3个字节，因此就会出现乱码
  	// 解决方法是 将  string 转成 []rune 即可， 因为 []rune是按字符处理，兼容汉字
  
  	arr2 := []rune(str) 
  	arr2[0] = '北'
  	str = string(arr2)
  	fmt.Println("str=", str) //str= 北ello@atguigu
  }
  ```

* **字符串的两种表示形式**

  * **双引号**，会识别转义字符
  * **反引号**，以字符串的原生形式输出，包括换行和特殊字符，可以实现防止攻击、输出源代码等效果

  ![image-20220309155650078](https://gitee.com/jobim/blogimage/raw/master/img/20220309155650.png)

* 当一行字符串太长时，需要使用到多行字符串，可以如下处理：

  ```go
  //当一个拼接的操作很长时，怎么办，可以分行写,但是注意，需要将+保留在上一行.
  str4 := "hello " + "world" + "hello " + "world" + "hello " + 
  "world" + "hello " + "world" + "hello " + "world" + 
  "hello " + "world"
  fmt.Println(str4)
  ```



**字符串的遍历：**

> **如果我们的字符串含有中文，那么传统的遍历字符串方式，就是错误，会出现乱码。原因是传统的对字符串的遍历是按照字节来遍历，而一个汉字在utf8编码是对应3个字节**。

* **方法一：需要要将str转成[]rune切片进行遍历**

  ```go
  var str string = "hello,world!北京"
  str2 := []rune(str) // 就是把 str 转成 []rune
  for i := 0; i < len(str2); i++ {
      fmt.Printf("%c \n", str2[i]) //使用到下标...
  }
  ```

* **方法二：使用for-range进行遍历**

  ```go
  //字符串遍历方式2-for-range
  str := "hello北京"
  for index, val := range str {
      fmt.Printf("index=%d, val=%c \n", index, val)
  }
  ```

  ![image-20220222154612853](https://gitee.com/jobim/blogimage/raw/master/img/20220222154612.png)

* **如果需要修改字符串，可以先将string ->[]byte/或者[]rune->修改→重写转成string**

  ```go
  func main() {
  
  	str := "hello@atguigu"
  	//string是不可变的，也就说不能通过 str[0] = 'z' 方式来修改字符串 
  	//如果需要修改字符串，可以先将string -> []byte / 或者 []rune -> 修改 -> 重写转成string
  	// "hello@atguigu" =>改成 "zello@atguigu"
  	arr1 := []byte(str) 
  	arr1[0] = 'z'
  	str = string(arr1)
  	fmt.Println("str=", str) //str= zello@atguigu
  
  	// 细节，我们转成[]byte后，可以处理英文和数字，但是不能处理中文
  	// 原因是 []byte 字节来处理 ，而一个汉字，是3个字节，因此就会出现乱码
  	// 解决方法是 将  string 转成 []rune 即可， 因为 []rune是按字符处理，兼容汉字
  
  	arr2 := []rune(str) 
  	arr2[0] = '北'
  	str = string(arr2)
  	fmt.Println("str=", str) //str= 北ello@atguigu
  }
  ```

  

### 3.1.5 基本数据类型的相互转换

**Golang 和 java / c不同，Go在不同类型的变量之间赋值时需要显式转换。也就是说Golang 中数据类型不能自动转换。**

**基本语法：**

* 表达式T(v)将值v转换为类型T，T:就是数据类型，比如 int32，int64，float32等等v:就是需要转换的变量

```go
func main() {

	var i int32 = 100
	//希望将 i => float
	var n1 float32 = float32(i)
	var n2 int8 = int8(i)
	var n3 int64 = int64(i) //低精度->高精度

	fmt.Printf("i=%v n1=%v n2=%v n3=%v \n", i ,n1, n2, n3)
	//被转换的是变量存储的数据(即值)，变量本身的数据类型并没有变化
	fmt.Printf("i type is %T\n", i) // int32
}
```

**转换注意：**

* 在转换中，比如将 int64转成int8【-128---127】，编译时不会报错，只是转换的结果是按溢出处理，和我们希望的结果不一样。因此在转换时，需要考虑范围.

  ```go
  //在转换中，比如将 int64  转成 int8 【-128---127】 ，编译时不会报错，
  //只是转换的结果是按溢出处理，和我们希望的结果不一样
  var num1 int64 = 999999
  var num2 int8 = int8(num1)
  fmt.Println("num2=", num2)
  ```



### 3.1.6 基本类型和string的转换

<font color='blue' size='4'>**基本类型转string类型：**</font>

**使用strconv包的函数：**

![image-20220427093212704](https://blog.zhaobincode.cn/blogimages/202204270932808.png)

![image-20220427093222274](https://blog.zhaobincode.cn/blogimages/202204270932319.png)

```go
func main() {
	var str string //空的str

	//第二种方式 strconv 函数 
	var num3 int = 99
	var num4 float64 = 23.456
	var b2 bool = true

	str = strconv.FormatInt(int64(num3), 10)
	fmt.Printf("str type %T str=%q\n", str, str)
	
	// strconv.FormatFloat(num4, 'f', 10, 64)
	// 说明： 'f' 格式 10：表示小数位保留10位 64 :表示这个小数是float64
	str = strconv.FormatFloat(num4, 'f', 10, 64)
	fmt.Printf("str type %T str=%q\n", str, str)

	str = strconv.FormatBool(b2)
	fmt.Printf("str type %T str=%q\n", str, str)

	//strconv包中有一个函数Itoa
	var num5 int64 = 4567
	str = strconv.Itoa(int(num5))
	fmt.Printf("str type %T str=%q\n", str, str)

}
```



<font color='blue' size='4'>**string类型转基本数据类型：**</font>

![image-20220309163842341](https://gitee.com/jobim/blogimage/raw/master/img/20220309163842.png)

![image-20220427093145940](https://blog.zhaobincode.cn/blogimages/202204270931057.png)

```go
func main() {

	var str string = "true"
	var b bool
	// b, _ = strconv.ParseBool(str)
	// 说明
	// 1. strconv.ParseBool(str) 函数会返回两个值 (value bool, err error)
	// 2. 因为我只想获取到 value bool ,不想获取 err 所以我使用_忽略
	b , _ = strconv.ParseBool(str)
	fmt.Printf("b type %T  b=%v\n", b, b)
	
	var str2 string = "1234590"
	var n1 int64
	var n2 int
	n1, _ = strconv.ParseInt(str2, 10, 64)
	n2 = int(n1)
	fmt.Printf("n1 type %T  n1=%v\n", n1, n1)
	fmt.Printf("n2 type %T n2=%v\n", n2, n2)

	var str3 string = "123.456"
	var f1 float64
	f1, _ = strconv.ParseFloat(str3, 64)
	fmt.Printf("f1 type %T f1=%v\n", f1, f1)


	//注意：
	var str4 string = "hello"
	var n3 int64 = 11
	n3, _ = strconv.ParseInt(str4, 10, 64)
	fmt.Printf("n3 type %T n3=%v\n", n3, n3)

}
```

**注意：因为ParseInt() 和 ParseFloat() 返回的是int64或者float64,如希望要得到int32 ,float32等需要进行显示转换**





## 3.2 派生数据类型

### 3.2.1 指针

* **Go 语言的取地址符是 `&`，放到一个变量前使用就会返回相应变量的内存地址**

  ```go
  package main
  
  import "fmt"
  
  func main() {
     var a int = 10  
  
     fmt.Println("变量的地址: ", &a)
  }
  ```

* **指针类型，指针变量存的是一个地址，这个地址指向的空间存的才是值**

* **在指针类型前面加上 `*` 号（前缀）来获取指针所指向的内容。**

  ```go
  func main() {
  
  	//基本数据类型在内存布局
  	var i int = 10
  	// i 的地址是什么,&i
  	fmt.Println("i的地址=", &i)
  	
  	//下面的 var ptr *int = &i
  	//1. ptr 是一个指针变量
  	//2. ptr 的类型 *int
  	//3. ptr 本身的值&i
  	var ptr *int = &i 
  	fmt.Printf("ptr=%v\n", ptr)
  	fmt.Printf("ptr 的地址=%v\n", &ptr) 
  	fmt.Printf("ptr 指向的值=%v\n", *ptr)
  
  }
  ```

* **值类型，都有对应的指针类型**，形式为`*数据类型`，比如 int的对应的指针就是`*int `, float32对应的指针类型就是`*float32`，依次类推。



### 3.2.2 数组

**初始化数组：**元素值默认为0

```go
var nums [4]int = [4]int{1, 2, 3, 4}
 
var nums1 = [4]int{1, 2, 3, 4}
 
var nums3 = [...]int{1, 2, 3, 4} // 自行判断长度，中括号里...一个不能少
 
var num4 = [...]int{1:3, 0:4, 2:5} // 指定索引和值
//类型推断
var strArr := [...]stirng{1:"tom", 0:"jack", 2:"mary"}
```

**注意：数组是多个相同类型数据的组合,一个数组一旦声明/定义了,其长度是固定的,不能动态变化**



**数组指针：**

* **Go的数组属值类型，在默认情况下是值传递，因此会进行值拷贝。数组间不会相互影响**

* **如想在其它函数中，去修改原来的数组，可以使用引用传递(指针方式)**

```go
//函数
func test01(arr [3]int) {
	arr[0] = 88 //不会影响原数组的值
} 

//函数
func test02(arr *[3]int) {
	fmt.Printf("arr指针的地址=%p", &arr)
	(*arr)[0] = 88 //原数组的值发生了变化
}

func main() {

	//Go的数组属值类型， 在默认情况下是值传递， 因此会进行值拷贝。数组间不会相互影响
	arr := [3]int{11, 22, 33}
	test01(arr)
	fmt.Println("main arr=", arr) 

	arr2 := [3]int{11, 22, 33}
	fmt.Printf("arr 的地址=%p", &arr2)
	test02(&arr2)
	fmt.Println("main arr=", arr2)
}

```



<font color='blue'>**二维数组：**</font>

**二维数组在声明/定义时也对应有四种写法：**

```go
var 数组名 [大小][大小]类型 = [大小][大小]类型{{初值..},{初值..}}
var 数组名 [大小][大小]类型 = [...][大小]类型{{初值..}，{初值..}}
var 数组名 = [大小][大小]类型{{初值..},{初值..}}
var 数组名 = [..][大小]类型{{初值..},{初值..}}
```

**二维数组的使用：**

```go
func main() {

	//定义/声明二维数组
	var arr [4][6]int
	//赋初值
	arr[1][2] = 1
	arr[2][1] = 2
	arr[2][3] = 3

	//遍历二维数组，按照要求输出图形
	for i := 0; i < 4; i++ {
		for j := 0; j < 6; j++ {
			fmt.Print(arr[i][j], " ")
		}
		fmt.Println()
	}
	
	fmt.Println()
	
	var arr2 [2][3]int //以这个为例来分析arr2在内存的布局!!
	arr2[1][1] = 10
	fmt.Println(arr2) //[[0 0 0] [0 10 0]]

	fmt.Printf("arr2[0]的地址%p\n", &arr2[0]) //arr2[0]的地址0xc04200a270
	fmt.Printf("arr2[1]的地址%p\n", &arr2[1]) //arr2[1]的地址0xc04200a288

	fmt.Printf("arr2[0][0]的地址%p\n", &arr2[0][0]) //arr2[0][0]的地址0xc04200a270
	fmt.Printf("arr2[1][0]的地址%p\n", &arr2[1][0]) //arr2[1][0]的地址0xc04200a288

	fmt.Println()
	//直接初始化
	arr3  := [2][3]int{{1,2,3}, {4,5,6}}
	fmt.Println("arr3=", arr3)

}
```



### 3.2.3 切片（Slice）

**切片就是可以动态变化的数组，是数组的一个引用因此切片是引用类型。遍历，访问切片元素，获取切片长度和数组一样。**

**切片内存结构相当于一个结构体，由三部分构成：引用数组部分的首地址(ptr*)、切片长度(len)和切片容量(cap)**

**基本语法：**

```go
 var 变量名 [] 类型 
//例如： var a [] int
```



**引用切片的三种方式：**

* **方式一：定义一个切片，让切片去引用一个已经创建好的数组，此时切片表示引用到该数组**

  ```go
  func main() {
  	var intArr [5]int = [...]int{1, 22, 33, 66, 99}
  	slice := intArr[1:3]
  	// 切片 array0[1:4] 表示slice引用array0数组起始下标为1，最后下标为4(不包含4)
  	fmt.Println(slice)	//[22 33]
  	fmt.Println(intArr)	//[1 22 33 66 99]
  	slice[0] = 1
  	fmt.Println(slice)	//[1 33]
  	//修改切片的数据，数组的数据也发生了更改
  	fmt.Println(intArr)	//[1 1 33 66 99]
  }
  ```

  **切片的内存布局：**

  ![image-20220313110256456](https://gitee.com/jobim/blogimage/raw/master/img/20220313110256.png)

* **方式二：通过make来创建切片** ：

  **var 切片名 [] type = make([],len,[cap])** 

  **参数说明: `type`就是数据类型 、`len`大小、`cap` 指定切片容量（可选）、如果你分配了cap,则要求cap>=len。**

  ```go
  func main() {
  	var slice []float64 = make([]float64, 5, 10)
  	slice[1] = 10
  	slice[3] = 20
  	//对于切片，必须make使用
  	fmt.Println(slice) //[0 10 0 20 0]
  	fmt.Println("slice的size=", len(slice)) //slice的size= 5
  	fmt.Println("slice的cap=", cap(slice)) //slice的cap= 10
  }
  ```

* **方式三：定义一个切片，直接指定具体数组，使用原理类似make的方式**

  ```go
  func main() {
  	var intArr []int = [...]int{1, 22, 33, 66, 99}
  	fmt.Println(intArr) //[1 22 33 66 99]
  	fmt.Println("intArr的size=", len(intArr)) //intArr的size= 5
  	fmt.Println("intArr的cap=", cap(intArr)) //intArr的size= 5
  }
  ```

**面试问：方式一和方式二的区别？**

* **方式1是直接引用数组，这个数组是事先存在的，程序员是可见的。**

* **方式2是通过make来创建切片，make也会创建一个数组，是由切片在底层进行维护，程序员是看不见的。make创建切片的示意图:**

  ![image-20220313111918654](https://gitee.com/jobim/blogimage/raw/master/img/20220313111918.png)



<font color='blue' size='4'>**append和copy函数：**</font>

* **用append 内置函数，可以对切片进行动态追加**

  ![image-20220313112838247](C:\Users\ZB\AppData\Roaming\Typora\typora-user-images\image-20220313112838247.png)

  ```go
  func main() {
  	//用append内置函数，可以对切片进行动态追加
  	var slice3 []int = []int{100, 200, 300}
  	//通过append直接给slice3追加具体的元素
  	slice3 = append(slice3, 400, 500, 600)
  	fmt.Println("slice3", slice3) //100, 200, 300,400, 500, 600
  
  	//通过append将切片slice3追加给slice3
  	slice3 = append(slice3, slice3...) // 100, 200, 300,400, 500, 600 100, 200, 300,400, 500, 600
  	fmt.Println("slice3", slice3)
  }
  ```

  **切片append操作的底层原理分析：**

  * 切片 append操作的本质就是对数组扩容。**go底层会创建一下新的数组newArr(安装扩容后大小)将slice原来包含的元素拷贝到新的数组newArr，slice重新引用到newArr。**注意：newArr是在底层来维护的，程序员不可见。

    ![image-20220313113044289](https://gitee.com/jobim/blogimage/raw/master/img/20220313113044.png)



* **使用copy内置函数完成拷贝**

  ![image-20220313112910252](https://gitee.com/jobim/blogimage/raw/master/img/20220313112910.png)

  ```go
  func main() {
  	//切片的拷贝操作
  	//切片使用copy内置函数完成拷贝，举例说明
  	var slice4 []int = []int{1, 2, 3, 4, 5}
  	var slice5 = make([]int, 10)
  	//slice4和slice5的数据空间是独立，相互不影响
  	copy(slice5, slice4)
  	fmt.Println("slice4=", slice4)	//slice4= [1 2 3 4 5]
  	fmt.Println("slice5=", slice5)	//slice5= [1 2 3 4 5 0 0 0 0 0]
  }
  ```

**切片使用的注意：**

* **切片初始化时`var slice = arr[startIndex:endIndex]`**
  **说明:从arr数组下标为startIndex，取到下标为endIndex的元素(不含arr[endIndex])。**

* **切片初始化的一些简写：**

  ```go
  var slice = arr[0:end] 			//可以简写:var slice = arr[ :end]
  var slice = arr[start:len(arr)]	//可以简写:var slice = arr[start:]
  var slice = arr[0:len(arr)]		//可以简写:var slice= arr[:]
  ```

* **cap是一个内置函数，用于统计切片的容量，即最大可以存放多少个元素**

* **切片可以继续切片**

  ```go
  func main() {
  	var arr [5]int = [...]int{10, 20, 30, 40, 50}
  	//slice := arr[1:4] // 20, 30, 40
  	slice := arr[1:4]
  	//切片继续引用切片
  	slice2 := slice[1:2]
  	fmt.Println(slice)		//slice [ 20, 30, 40]
  	fmt.Println(slice2)		//[30]
  }	
  ```



### 3.2.4 映射(map)

**map是key-value数据结构，又称为字段或者关联数组。类似其它编程语言的集合**



**基本语法：`var 变量名 map[keytype] valuetype`**

* **key的类型可以为：bool，数字，string，指针，channel，接口，结构体，数组 （slice和function不可以）**

* **map声明后无法直接使用，要先make申请内存，再使用**

  ![image-20220313121807980](https://gitee.com/jobim/blogimage/raw/master/img/20220313121808.png)

* **map的key-value是无序**

```go
func main() {
	//第一种使用方式
	var a map[string]string
	//在使用map前，需要先make , make的作用就是给map分配数据空间
	a = make(map[string]string, 10)
	a["no1"] = "宋江" //ok?
	a["no2"] = "吴用" //ok?
	a["no1"] = "武松" //ok?
	a["no3"] = "吴用" //ok?
	fmt.Println(a) //map[no1:武松 no3:吴用 no2:吴用]

	//第二种方式
	cities := make(map[string]string)
	cities["no1"] = "北京"
	cities["no2"] = "天津"
	cities["no3"] = "上海"
	fmt.Println(cities) //map[no1:北京 no2:天津 no3:上海]

	//第三种方式
	heroes := map[string]string{
		"hero1" : "宋江",
		"hero2" : "卢俊义",
		"hero3" : "吴用",
	}
	heroes["hero4"] = "林冲"
	fmt.Println("heroes=", heroes) //heroes= map[hero1:宋江 hero2:卢俊义 hero3:吴用 hero4:林冲]
}
```

* **map是引用类型，遵守引用类型传递的机制，在一个函数接收map，修改后，会直接修改原来的map**

  ```go
  func modify(map1 map[int]int) {
  	map1[10] = 900
  }
  
  
  func main() {
  
  	//map是引用类型，遵守引用类型传递的机制，在一个函数接收map，
  	//修改后，会直接修改原来的map
  
  	map1 := make(map[int]int, 2)
  	map1[1] = 90
  	map1[2] = 88
  	map1[10] = 1
  	map1[20] = 2
  	modify(map1)
  	// 看看结果， map1[10] = 900 ,说明map是引用类型
  	fmt.Println(map1) //map[1:90 2:88 10:900 20:2]
  
  }
  ```

  

**map的增删改查和遍历操作：**

* **删除：delete(map，"key") , delete是一个内置函数，如果key存在，就删除该key-value,如果key不存在，不操作，但是也不会报错**

  ![image-20220313122905006](https://gitee.com/jobim/blogimage/raw/master/img/20220313122905.png)

* **如果我们要删除map的所有key ,没有一个专门的方法一次删除，可以遍历一下key，逐个删除或者map = make(...)，make一个新的，让原来的成为垃圾，被gc回收**

```go
func main() {
	cities := make(map[string]string)
	cities["no1"] = "北京"
	cities["no2"] = "天津"
	cities["no3"] = "上海"
	fmt.Println(cities)
    
    //map的遍历
	for k, v := range cities {
		fmt.Printf("k=%v v=%v\n", k, v)
	}
    
	//因为 no3这个key已经存在，因此下面的这句话就是修改
	cities["no3"] = "上海~" 
	fmt.Println(cities)

	//演示删除
	delete(cities, "no1")
	fmt.Println(cities)
	//当delete指定的key不存在时，删除不会操作，也不会报错
	delete(cities, "no4")
	fmt.Println(cities)


	//演示map的查找，如果存在ok就为true，否则返回false
	val, ok := cities["no2"]
	if ok {
		fmt.Printf("有no1 key 值为%v\n", val)
	} else {
		fmt.Printf("没有no1 key\n")
	}

	//如果希望一次性删除所有的key
	//1. 遍历所有的key,如何逐一删除 [遍历]
	//2. 直接make一个新的空间
	cities = make(map[string]string)
	fmt.Println(cities)

}
```



<font color='blue' size='4'>**map切片：**</font>

* 切片的数据类型如果是map，则我们称为 slice of map，map切片，这样使用则map个数就可以动态变化了。

  ```go
  func main() {
  	//演示map切片的使用
  	/*
  	要求：使用一个map来记录monster的信息 name 和 age, 也就是说一个
  	monster对应一个map,并且妖怪的个数可以动态的增加=>map切片
  	*/
  	//1. 声明一个map切片
  	var monsters []map[string]string
  	monsters = make([]map[string]string, 2) //准备放入两个妖怪
  	//2. 增加第一个妖怪的信息
  	if monsters[0] == nil {
  		monsters[0] = make(map[string]string, 2)
  		monsters[0]["name"] = "牛魔王"
  		monsters[0]["age"] = "500"
  	}
  
  	if monsters[1] == nil {
  		monsters[1] = make(map[string]string, 2)
  		monsters[1]["name"] = "玉兔精"
  		monsters[1]["age"] = "400"
  	}
  
  	//这里我们需要使用到切片的append函数，可以动态的增加monster
  	//1. 先定义个monster信息
  	newMonster := map[string]string{
  		"name" : "新的妖怪~火云邪神",
  		"age" : "200",
  	}
  	monsters = append(monsters, newMonster)
  
  	fmt.Println(monsters) //[map[name:牛魔王 age:500] map[name:玉兔精 age:400] map[name:新的妖怪~火云邪神 age:200]]
  }
  ```



<font color='blue' size='4'>**map排序：**</font>

* **golang中没有一个专门的方法针对map的key进行排序**
* **golang 中的map默认是无序的，注意也不是按照添加的顺序存放的，你每次遍历，得到的输出可能不一样。**
* **golang中map的排序，是先将key进行排序，然后根据key值遍历输出即可a**

```go
func main() {

	//map的排序
	map1 := make(map[int]int, 10)
	map1[10] = 100
	map1[1] = 13
	map1[4] = 56
	map1[8] = 90
	//输出无序的map
	fmt.Println(map1) //map[4:56 8:90 10:100 1:13]

	//如果按照map的key的顺序进行排序输出
	//1. 先将map的key 放入到 切片中
	//2. 对切片排序 
	//3. 遍历切片，然后按照key来输出map的值

	var keys []int
	for k, _ := range map1 {
		keys = append(keys, k)
	}
	//排序好的切片
	sort.Ints(keys) //[1 4 8 10]
	fmt.Println(keys)

	for _, k := range keys{
		fmt.Printf("map1[%v]=%v \n", k, map1[k])
	}
	
}
```



### 3.2.5 结构体

**结构体的声明和使用：**

```go
type Person struct{
	Name string
	Age int
}
func main() {
	//方式1
	var p1 Person
	p1.Name = "jack"
	p1.Age = 15
	fmt.Println(p1) //{jack 15}
	//方式2
	p2 := Person{"mary", 20}
	// p2.Name = "tom"
	// p2.Age = 18
	fmt.Println(p2) //{mary 20}
	//方式3：创建结构体变量时指定字段值
	//把字段名和字段值写在一起，就不依赖字段的定义顺序
	p3 := Person{
		Age : 59,
		Name : "小李",
	}
	fmt.Println(p3)	//{小李 59}
}
```

* **结构体是值类型，因此函数传参是值传递，而且拷贝也是浅拷贝**
* **如果结构体有切片、映射等属性，也要先分配内存再使用**
* 结构体地址为首字段地址，且**内部字段在内存中的地址连续分配**

<font color='blue' size='4'>**结构体指针：**</font>

* **结构体指针访问字段的标准方式为：`(*结构体指针).字段名` go底层做了优化支持 ： `结构体指针.字段名` 的访问方式**

* **`new(Type) *Type` 函数**

  ![image-20220314113808387](https://gitee.com/jobim/blogimage/raw/master/img/20220314113808.png)

```go
type Person struct{
	Name string
	Age int
}
func main() {

	//方式3-&
	//案例: var person *Person = new (Person)

	var p3 *Person= new(Person)
	//因为p3是一个指针，因此标准的给字段赋值方式
	//(*p3).Name = "smith" 也可以这样写 p3.Name = "smith"

	//原因: go的设计者 为了程序员使用方便，底层会对 p3.Name = "smith" 进行处理
	//会给 p3 加上 取值运算 (*p3).Name = "smith"
	(*p3).Name = "smith" 
	p3.Name = "john" 

	(*p3).Age = 30
	p3.Age = 100
	fmt.Println(*p3)//{john 100}
	fmt.Println(p3)	//&{john 100}

	//方式4-{}
	//案例: var person *Person = &Person{}

	//下面的语句，也可以直接给字符赋值
	//var person *Person = &Person{"mary", 60} 
	var person *Person = &Person{}

	//因为person 是一个指针，因此标准的访问字段的方法
	// (*person).Name = "scott"
	// go的设计者为了程序员使用方便，也可以 person.Name = "scott"
	// 原因和上面一样，底层会对 person.Name = "scott" 进行处理， 会加上 (*person)
	(*person).Name = "scott"
	person.Name = "scott~~"

	(*person).Age = 88
	person.Age = 10
	fmt.Println(*person)//{scott~~ 10}

}
```



**结构体使用注意：**

* **结构体是用户单独定义的类型，和其它类型进行转换时需要有完全相同的字段(名字、个数和类型**)

  ```go
  type A struct {
  	Num int
  }
  type B struct {
  	Num int
  }
  
  func main() {
  	var a A
  	var b B
  	a = A(b) // ? 可以转换，但是有要求，就是结构体的的字段要完全一样(包括:名字、个数和类型！)
  	fmt.Println(a, b)	//{0} {0}
  }
  ```

* **结构体进行 type重新定义(相当于取别名)，Golang认为是新的数据类型，但是相互间可以强转**




<font color='blue' size='4'>**字段标签：** </font>

**struct的每个字段上，可以写上一个 tag，用来对字段进行描述的元数据。**

**在运行期，可用反射获取标签信息。它长被用作格式校验，数据库关系映射等**

```go
type Monster struct{
	Name string `json:"name"` // `json:"name"` 就是 struct tag
	Age int `json:"age"`
	Skill string `json:"skill"`
}

func main() {

	//1. 创建一个Monster变量
	monster := Monster{"牛魔王", 500, "芭蕉扇~"}

	//2. 将monster变量序列化为 json格式字串
	//   json.Marshal 函数中使用反射，这个讲解反射时，我会详细介绍
	jsonStr, err := json.Marshal(monster)
	if err != nil {
		fmt.Println("json 处理错误 ", err)
	}
	fmt.Println("jsonStr", string(jsonStr))	//jsonStr {"name":"牛魔王","age":500,"skill":"芭蕉扇~"}
}
```



<font color='blue' size='4'>**匿名字段：**</font>

* **结构体可以包含一个或多个匿名（或内嵌）字段，即这些字段没有显式的名字，只有字段的类型是必须的，此时类型也就是字段的名字。匿名字段本身可以是一个结构体类型，即结构体可以包含内嵌结构体。**

* **在一个结构体中对于每一种数据类型只能有一个匿名字段**

```go
type outerS struct {
    c float32
    int // 匿名字段
}
func main() {
    outer := new(outerS)
    outer.c = 7.5
    outer.int = 60
    fmt.Printf("outer.c is: %f\n", outer.c)		//outer.c is: 7.500000
    fmt.Printf("outer.int is: %d\n", outer.int)	//outer.int is: 60
}
```



<font color='blue' size='4'>**内嵌结构体：**</font>

* **在Golang中，如果一个struct嵌套了另一个匿名结构体，那么这个结构体可以直接访问匿名结构体的字段和方法，从而实现了继承特性。**

**基本语法：**

```go
type Goods struct {
	Name string
	Price int
}
type Book struct {
	Goods //这里就是嵌套匿名结构体Goods
	Writer string
}
```



* **匿名结构体字段访问可以简化**

  ```go
  type Student struct {
  	Name string
  	//首字母大写或者小写的字段、方法，都可以使用
  	score int 
  }
  
  func (stu *Student) ShowInfo() {
  	fmt.Printf("学生名=%v 成绩=%v\n", stu.Name, stu.score)
  }
  
  //大学生
  type Graduate struct {
  	Student //嵌入了Student匿名结构体
  }
  
  //显示他的成绩
  //这是Graduate结构体特有的方法，保留
  func (p *Graduate) testing() {
  	fmt.Println("大学生正在考试中.....")
  }
  
  
  func main() {
  	//当我们对结构体嵌入了匿名结构体使用方法会发生变化
  	graduate := &Graduate{}
  	graduate.Student.Name = "mary~"
  	//graduate.Student.score 可以简写成 graduate.score
  	graduate.Student.score = 90
  	graduate.testing() 
  	//graduate.Student.ShowInfo()
  	//匿名结构体字段访问可以简化
  	graduate.ShowInfo()
  	fmt.Println("res=", graduate.score)
  }
  ```

  

* **当结构体和匿名结构体有相同的字段或者方法时，编译器采用就近访问原则访问，如希望访问匿名结构体的字段和方法，可以通过匿名结构体名来区分**

  ![image-20220314150648600](https://gitee.com/jobim/blogimage/raw/master/img/20220314150648.png)

* **如果一个struct 嵌套了一个有名结构体，这种模式就是组合，如果是组合关系，那么在访问组合的结构体的字段或方法时，必须带上结构体的名字**

  ```go
  type A struct {
  	Name string
  	age int
  }
  type D struct {
  	a A //有名结构体，组合关系
  }
  func main() {
      //如果D 中是一个有名结构体，则访问有名结构体的字段时，就必须带上有名结构体的名字
  	//比如 d.a.Name 
  	var d D 
  	d.a.Name = "jack"
  }
  ```

