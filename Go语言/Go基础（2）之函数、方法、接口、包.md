# 一、函数

**函数的基本语法：**

```go
func 函数名 (形参列表) (返回值列表) {
    执行语句...
    return 返回值列表
}
1、形参列表:表示函数的输入
2、函数中的语句:表示为了实现某一功能代码块
3、函数可以有返回值,也可以没有;
```

* **函数的调用**

  ```go
  func main() {
      
  	/* 调用函数并返回最大值 */
  	ret = max(100, 200)
  	 
  	fmt.Printf( "最大值是 : %d\n", ret )
   }
   
   /* 函数返回两个数的最大值，一个返回值可以不加括号 */
   func max(num1, num2 int) int {
  	/* 定义局部变量 */
  	var result int
   
  	if (num1 > num2) {
  	   result = num1
  	} else {
  	   result = num2
  	}
  	return result
   }
  ```

* **函数首字母`大写`该函数可以被本包文件和其它包文件使用，类似public；首字母`小写`，只能被本包文件使用，其它包文件不能使用，类似private**

* **Go函数不支持函数重载**

  ![image-20220310101319079](https://gitee.com/jobim/blogimage/raw/master/img/20220310101319.png)

* **在Go中`函数也是一种数据类型`，可以赋值给一个变量，则该变量就是一个函数类型的变量了。通过该变量可以对函数调用**

  ```go
  func getSum(n1 int, n2 int) int {
  	return n1 + n2
  }
  
  func main() {
  	
  	a := getSum //将函数赋值给一个变量，此时变量a是函数类型
  	fmt.Printf("a的类型%T, getSum类型是%T\n", a, getSum)
      // a的类型func(int, int) int, getSum类型是func(int, int) int
  
  	res := a(10, 40) // 等价  res := getSum(10, 40)
  	fmt.Println("res=", res) //res= 50
      
  }
  ```

  



## 1.1 函数参数

<font color='blue' size='4'>**参数的传递：**</font>

* **基本数据类型和数组默认都是值传递的，即进行值拷贝。在函数内修改，不会影响到原来的值。**

  ```go
  func test01(n1 int) {
  	
  	n1 = n1 + 10
  	fmt.Println("test01() n1= ", n1) //test01() n1=  30
  }
  
  func main() {
  	num := 20
  	test01(num)
  	fmt.Println("main() num= ", num) //main() num=  20
  }
  ```

* **如果希望函数内的变量能修改函数外的变量(指的是默认以值传递的方式的数据类型)，可以传入变量的地址&，函数内以指针的方式操作变量。**从效果上看类似引用。

  ```go
  // n1 就是 *int 类型
  func test02(n1 *int) {
  	fmt.Printf("n1的值=%v\n", n1) //n1的值=0xc04200e0b0
  	*n1 = *n1 + 10
  	fmt.Println("test02() n1= ", *n1) // test02() n1=  30
  }
  
  func main() {
  	num = 20
  	fmt.Printf("num的地址=%v\n", &num) //num的地址=0xc04200e0b0
  	test02(&num)
  	fmt.Println("main() num= ", num) // main() num=  30
  }
  ```

* 值类型和引用类型
  * 值类型:基本数据类型int系列, float 系列, boo1, string 、数组和结构体struct
  * 引用类型:指针、slice切片、map、管道chan、interface 等都是引用类型
* 不管是指针、引用类型，还是其他类型参数，都是**值拷贝传递**。区别只是拷贝目标对象还是拷贝指针而已。



<font color='blue' size='4'>**可变参数：**</font>

**基本语法：**

```go
//支持o到多个参数
func sum(args... int) sum int {}
//支持1到多个参数
func sum(n1 int, args... int) sum int {
}
```

* **可变参数本质就是一个切片，args[index]可以访间到各个值，只能接收一个到多个同类型参数，且必须放在列表尾部**

  ```go
  func sum(n1 int, args... int) int {
  	sum := n1 
  	//遍历args 
  	for i := 0; i < len(args); i++ {
  		sum += args[i]  //args[0] 表示取出args切片的第一个元素值，其它依次类推
  	}
  	return sum
  }
  
  func main() {
  	res4 := sum(10, 90, 10,100)
  	fmt.Println("res4=", res4) //res4= 210
  }
  ```

* **将切片作为变参时，需进行展开操作，如果是数组，先将其转换为切片**

  ```go
  func test(a ...int) {
  	fmt.Println(a) //[10 20 30]
  }
  
  func main() {
  	
  	a := []int{10, 20, 30} //先将数组转成slice
  	test(a...) //将slice展开
  }
  ```

  



<font color='blue'>**函数作为另外一个函数的参数：**</font>

* 函数既然是一种数据类型，因此在Go中，函数可以作为形参，并且调用

  ```go
  func getSum(n1 int, n2 int) int {
  	return n1 + n2
  }
  
  //函数既然是一种数据类型，因此在Go中，函数可以作为形参，并且调用
  func myFun(funvar func(int, int) int, num1 int, num2 int ) int {
  	return funvar(num1, num2)
  }
  
  func main() {
  	//看案例
  	res2 := myFun(getSum, 50, 60) //将getSum函数作为myFun函数的参数
  	fmt.Println("res2=", res2)
  }
  ```





## 1.2 返回值

* **返回值列表也可以是多个**

  ```go
  func swap(x, y string) (string, string) {
     return y, x
  }
  
  func main() {
     a, b := swap("Google", "Runoob")
     fmt.Println(a, b)
  }
  ```

* **使用`_`标识符，忽略返回值**

  ```go
  func cal(n1 int, n2 int) (int, int) {
  	sum := n1 + n2
  	sub := n1 - n2
  	return sum, sub
  }
  
  func main() {
  	res1, _ := cal(10, 20) //忽略第二个返回值
  	fmt.Printf("res1=%d\n", res1) //res1=30
  }
  ```

  

**<font color='blue'>命名返回值</font>：支持对函数返回值命名**（优缺点共存）

* **命名返回值和参数一样，可当作函数局部变量使用，最后由return隐式返回**

  ```go
  //支持对函数返回值命名
  func getSumAndSub(n1 int, n2 int) (sum int, sub int){
  	sub = n1 - n2
  	sum = n1 + n2
  	return
  }
  func main() {
  	a1, b1 := getSumAndSub(1, 2)
  	fmt.Printf("a=%v b=%v\n", a1, b1) //a=3 b=-1
  }
  ```

## 1.3 匿名函数

**Go支持匿名函数，匿名函数就是没有名字的函数，如果我们某个函数只是希望使用一次，可以考虑使用匿名函数，匿名函数也可以实现多次调用。**



* **在定义匿名函数时就直接调用，这种方式匿名函数只能调用一次。**

  ```go
  func main() {
  	//在定义匿名函数时就直接调用，这种方式匿名函数只能调用一次
  	res1 := func (n1 int, n2 int) int {
  		return n1 + n2
  	}(10, 20)
  
  	fmt.Println("res1=", res1) //res1= 30
  }
  ```

* **将匿名函数赋给一个变量(函数变量)，再通过该变量来调用匿名函数**

  ```go
  func main() {
  	//将匿名函数func (n1 int, n2 int) int赋给 a变量
  	//则a 的数据类型就是函数类型 ，此时,我们可以通过a完成调用
  	a := func (n1 int, n2 int) int {
  		return n1 - n2
  	}
  
  	res2 := a(10, 30)
  	fmt.Println("res2=", res2) //res2= -20
  	res3 := a(90, 30)
  	fmt.Println("res3=", res3) //res3= 60
  }
  ```

  



<font color='blue' size='4'>**闭包：**</font>

**闭包就是一个函数和与其相关的引用环境组合的一个整体(实体)**

```go
//累加器
func AddUpper() func (int) int {
	var n int = 10 
	return func (x int) int {
		n = n + x
		return n
	}
}

func main() {
	
	//使用前面的代码
	f := AddUpper()
	fmt.Println(f(1))// 11 
	fmt.Println(f(2))// 13
	fmt.Println(f(3))// 16
}
```

* 返回的是一个匿名函数，但是这个匿名函数引用到函数外的n ,因此这个匿名函数就和n形成一个整体，构成闭包。

## 1.4 延迟处理defer

**在函数中，程序员经常需要创建资源(比如:数据库连接、文件句柄、锁等)，为了在函数执行完毕后，及时的释放资源，Go的设计者提供defer (延时机制)。**

* **当go执行到一个defer时，不会立即执行defer后的语句，而是将defer后的语句压入到一个栈中，然后继续执行函数下一个语句。**
* **当函数执行完毕后，在从defer栈中，依次从栈顶取出语句执行(注:遵守栈先入后出的机制)。**
* **在defer将语句放入到栈时，也会将相关的值拷贝同时入栈**

```go
func sum(n1 int, n2 int) int {
	
	//当执行到defer时，暂时不执行，会将defer后面的语句压入到独立的栈(defer栈)
	//当函数执行完毕后，再从defer栈，按照先入后出的方式出栈，执行
	defer fmt.Println("ok1 n1=", n1) //defer 3. ok1 n1 = 10
	defer fmt.Println("ok2 n2=", n2) //defer 2. ok2 n2= 20
	//增加一句话
	n1++ // n1 = 11
	n2++ // n2 = 21
	res := n1 + n2 // res = 32
	fmt.Println("ok3 res=", res) // 1. ok3 res= 32
	return res
}

func main() {
	res := sum(10, 20)
	fmt.Println("res=", res)  // 4. res= 32
}
```

**执行结果：**

![image-20220311104038112](https://gitee.com/jobim/blogimage/raw/master/img/20220311110912.png)

* **最佳实践：当函数执行完毕后，可以及时的释放函数创建的资源**

  ```go
  func test() {
      //关闭文件资源
      file = openfile(文件名)
      defer file.close()
      //其他代码
  }
  ```



## 1.5 错误处理

**在Go语言中，错误被认为是一种可以预期的结果；而异常则是一种非预期的结果，发生异常可能表示程序中存在BUG或发生了其它不可控的问题。**

<font color='blue' size='4'>**错误：**</font>

**Go中的错误类型：`error`**

```go
type error interface {
    Error() string
}
```

**函数通常可在最后一个返回值中返回错误信息**，自定义错误：errors.New("错误说明")，会返回一个error类型的值，表示一个错误

```go
func myF(f float64) (float64, error) {
	if f < 0 {
		return 0, errors.New("Not legal input ")
	}
	// 实现
	return 0.0, nil
}

func main() {
	_, e := myF(-1)
	_, e2 := myF(2)
	fmt.Println(e) // Not legal input
	fmt.Println(e2) // <nil>
}
```



<font color='blue' size='4'>**异常处理：**</font>

**Go语言追求简洁优雅，所以，Go语言不支持传统的 try…catch…finally 这种处理**

**Go中引入的处理方式为: defer, panic, recover。Go中可以抛出一个panic 的异常，然后在defer中通过recover捕获这个异常，然后正常处理**

* `defer`是Go提供的一种延迟执行机制，每次执行 defer，都会将对应的函数压入栈中。在函数返回或者 panic 异常结束时，Go 会依次从栈中取出延迟函数执行。

* `panic`用于主动抛出程序执行的异常，会终止其后将要执行的代码，并依次逆序执行 panic 所在函数可能存在的 defer 函数列表。panic 内置函数 ,接收一个interface{}类型的值（也就是任何值了）作为参数。可以接收error类型的变量，输出错误信息，并退出程序.

* `recover` 关键字主要用于捕获异常，将程序状态从严重的错误中恢复到正常状态。 必须在 defer 函数中才能生效。

**defer+panic+recover的代码样例：**

```go
func my(i int) int {
	defer func() {
		if err := recover(); err != nil {
			fmt.Println("发生了异常", err)
		}
	}()
	if i != 5 {
		return i
	} else {
		panic("panic")
	}
	return -1
}
func main() {
	for i := 0; i < 10; i++ {
		a := my(i)
		fmt.Println(a)
	}
}
```

代码输出结果：

```
0
1
2
发生了异常 panic
0
4
```



**使用defer+recover来处理错误：**

```go
func test() {
	//使用defer + recover 来捕获和处理异常
	defer func() {
		err := recover()  // recover()内置函数，可以捕获到异常
		if err != nil {  // 说明捕获到错误
			fmt.Println("err=", err)
			//这里就可以将错误信息发送给管理员....
			fmt.Println("发送邮件给admin@sohu.com~")
		}
	}()
	num1 := 10
	num2 := 0
	res := num1 / num2
	fmt.Println("res=", res)
}
func main() {
	test()
}
```

**执行结果：**

![image-20220311150532130](https://gitee.com/jobim/blogimage/raw/master/img/20220311150532.png)



## 1.6 内置函数

**Go 语言拥有一些不需要进行导入操作就可以使用的内置函数。它们有时可以针对不同的类型进行操作，例如：len、cap 和 append，或必须用于系统级的操作**

```
append          -- 用来追加元素到数组、slice中,返回修改后的数组、slice
close           -- 主要用来关闭channel
delete            -- 从map中删除key对应的value
panic            -- 停止常规的goroutine  （panic和recover：用来做错误处理）
recover         -- 允许程序定义goroutine的panic动作
real            -- 返回complex的实部   （complex、real imag：用于创建和操作复数）
imag            -- 返回complex的虚部
make            -- 用来分配内存，返回Type本身(只能应用于slice, map, channel)
new                -- 用来分配内存，主要用来分配值类型，比如int、struct。返回指向Type的指针
cap                -- capacity是容量的意思，用于返回某个类型的最大容量（只能用于切片和 map）
copy            -- 用于复制和连接slice，返回复制的数目
len                -- 来求长度，比如string、array、slice、map、channel ，返回长度
print、println     -- 底层打印函数，在部署环境中建议使用 fmt 包
```

**new的使用：**

```go
func main() {

	num1 := 100
	fmt.Printf("num1的类型%T , num1的值=%v , num1的地址%v\n", num1, num1, &num1)

	num2 := new(int) // *int
	//num2的类型%T => *int
	//num2的值 = 地址 0xc04204c098 （这个地址是系统分配）
	//num2的地址%v = 地址 0xc04206a020  (这个地址是系统分配)
	//num2指向的值 = 100
	*num2  = 100
	fmt.Printf("num2的类型%T , num2的值=%v , num2的地址%v\n num2这个指针，指向的值=%v", 
		num2, num2, &num2, *num2)
}
```

**执行结果：**

![image-20220311145053979](https://gitee.com/jobim/blogimage/raw/master/img/20220311145054.png)





## 1.7 常用的相关函数

<font color='blue' size='4'>**字符串常用的系统函数：** </font>

* **统计字符串的长度,按字节 len(str)**

  ```go
  str := "hello北" 
  fmt.Println("str len=", len(str)) // 8
  ```

* **字符串遍历，同时处理有中文的问题 。r := []rune(str)**

  ```go
  str2 := "hello北京"
  //字符串遍历，同时处理有中文的问题 r := []rune(str)
  r := []rune(str2)
  for i := 0; i < len(r); i++ {
      fmt.Printf("字符=%c\n", r[i])
  }
  ```

* **字符串转整数： `n , err := strconv.Atoi(“12”)`**

  ```go
  //字符串转整数:	 n, err := strconv.Atoi("12")
  n, err := strconv.Atoi("123")
  if err != nil {
  	fmt.Println("转换错误", err)
  }else {
  	fmt.Println("转成的结果是", n)
  }
  ```

* **整数转字符串: `str = strconv.Itoa(12345)`**

  ```go
  str = strconv.Itoa(12345)
  fmt.Printf("str=%v, str=%T\n", str, str) //str=12345, str=string
  ```

* **字符串转[]byte: `var bytes= []byte(“hello go”)`**

  ```go
  var bytes = []byte("hello go")
  fmt.Printf("bytes=%v\n", bytes) //bytes=[104 101 108 108 111 32 103 111]
  ```

* **[]byte转字符串: `str = string([]byte{97,98,99})`**

  ```go
  str = string([]byte{97, 98, 99}) 
  fmt.Printf("str=%v\n", str) //str=abc
  ```

* **10进制转2,8,16进制: `str = strconv.FormatInt(123,2)`**

  ```go
  str = strconv.FormatInt(123, 2)
  fmt.Printf("123对应的二进制是=%v\n", str) //123对应的二进制是=1111011
  str = strconv.FormatInt(123, 16)
  fmt.Printf("123对应的16进制是=%v\n", str) //123对应的16进制是=7b
  ```

* **查找子串是否在指定的字符串中: `strings.Contains(“seafood”, “foo”) //true`**

  ```go
  b := strings.Contains("seafood", "mary")
  fmt.Printf("b=%v\n", b)  //b=false
  ```

* **统计一个字符串有几个指定的子串：`strings.Count(“ceheese”, “e”) //4`**

  ```go
  num := strings.Count("ceheese", "e")
  fmt.Printf("num=%v\n", num) //num=4
  ```

* **不区分大小写的字符串比较(== 是区分字母大小写的): `fmt.PrintIn(strings.EqualFold(“abc”, “Abc”) // true`**

  ```go
  b = strings.EqualFold("abc", "Abc")
  fmt.Printf("b=%v\n", b) //b=true
  
  fmt.Println("结果","abc" == "Abc") // 结果 false //区分字母大小写
  ```

* **返回子串在字符串第一次出现的index值，如果没有返回-1 : `strings.Index(“NLT_abc”,”abc”) //4`**

  ```go
  index := strings.Index("NLT_abcabcabc", "abc") // 4
  fmt.Printf("index=%v\n",index) //index=4
  ```

* **返回子串在字符串最后一次出现的index，如没有返回-1 : `strings.LastIndex(“go golang” , “go”)`**

  ```go
  index = strings.LastIndex("go golang", "go") //3
  fmt.Printf("index=%v\n",index) //index=3
  ```

* **将指定的子串替换成 另外一个子串: `strings.Replace(“go go hello” , “go”, “go语言”, n)` n 可以指定你希望替换几个,如果n = -1表示全部替换**

  ```go
  str2 = "go go hello"
  str = strings.Replace(str2, "go", "北京", -1)
  fmt.Printf("str=%v str2=%v\n", str, str2) //str=北京 北京 hello str2=go go hello
  ```

* **按照指定的某个字符，为分割标识，将一个字符串拆分成字符串数组:**
  **`strings.Split(“hello,wrold,ok” , “,”)`**

  ```go
  strArr := strings.Split("hello,wrold,ok", ",")
  for i := 0; i < len(strArr); i++ {
  	fmt.Printf("str[%v]=%v\n", i, strArr[i])
  } 
  fmt.Printf("strArr=%v\n", strArr) //strArr=[hello wrold ok]
  ```

* **将字符串的字母进行大小写的转换: `strings.ToLower(“Go”) // go` `strings.ToUpper(“Go”) //GO`**

  ```go
  str = "goLang Hello"
  str = strings.ToLower(str) 
  str = strings.ToUpper(str) 
  fmt.Printf("str=%v\n", str) //str=GOLANG HELLO
  ```

* **将字符串左右两边的空格去掉 : `strings.TrimSpace(“ tn a lone gopher ntrn “)`**

  ```go
  str = strings.TrimSpace(" tn a lone gopher ntrn   ")
  fmt.Printf("str=%q\n", str) //str="tn a lone gopher ntrn"
  ```

* **将字符串左右两边指定的字符去掉: `strings.Trim(“! hello! “, “ !”) // [“hello”]`//将左右两边!和””去掉**

  ```go
  str = strings.Trim("! he!llo! ", " !")
  fmt.Printf("str=%q\n", str) //str="he!llo"
  ```

* **将字符串左边指定的字符去掉: `strings.TrimLeft(“! hello! “,” !”) // [“hello”]`//将左边!和”“去掉**

* **将字符串右边指定的字符去掉: `strings.TrimRight(“! hello! “,” !”) // [“hello”]`//将右边!和””去掉**

* **判断字符串是否以指定的字符串开头: `strings.HasPrefix("ftp://192.168.10.1" ,”ftp”) // true`**

* **判断字符串是否以指定的字符串结束: `strings.HasSuffix(“‘NLT_abc.jpg”,”abc”) //false`**

  ```go
  //strings.HasPrefix("ftp://192.168.10.1", "ftp") // true
  b = strings.HasPrefix("ftp://192.168.10.1", "hsp") //true
  fmt.Printf("b=%v\n", b) //b=false
  ```



<font color='blue' size='4'>**时间和日期相关函数：**</font>

* **获取当前时间**。time.Time类型，用于表示时间

  ```go
  //获取当前时间
  now := time.Now()
  fmt.Printf("now=%v \nnow type=%T\n", now, now)
  ```

  **执行结果：**

  ![通过now可以获取到年月日，时分秒](https://gitee.com/jobim/blogimage/raw/master/img/20220311113606.png)

* **通过now可以获取到年月日，时分秒**

  ```go
  func main() {
      now := time.Now()
      fmt.Printf("年=%v\n", now.Year())
      fmt.Printf("月=%v\n", now.Month())
      fmt.Printf("月=%v\n", int(now.Month()))
      fmt.Printf("日=%v\n", now.Day())
      fmt.Printf("时=%v\n", now.Hour())
      fmt.Printf("分=%v\n", now.Minute())
      fmt.Printf("秒=%v\n", now.Second())
  }
  ```

  **执行结果：**

  ![image-20220311113946294](https://gitee.com/jobim/blogimage/raw/master/img/20220311113946.png)

* **格式化日期时间**

  ```go
  func main() {
  	now := time.Now()
  	//格式化日期时间
  	fmt.Printf("\n当前年月日 %d-%d-%d %d:%d:%d \n", now.Year(), 
  	now.Month(), now.Day(), now.Hour(), now.Minute(), now.Second())
  
  	dateStr := fmt.Sprintf("当前年月日 %d-%d-%d %d:%d:%d \n", now.Year(), 
  	now.Month(), now.Day(), now.Hour(), now.Minute(), now.Second())
  
  	fmt.Printf("dateStr=%v\n", dateStr)
  
  	//格式化日期时间的第二种方式
  	fmt.Printf(now.Format("2006-01-02 15:04:05"))
  	fmt.Println()
  	fmt.Printf(now.Format("2006-01-02"))
  	fmt.Println()
  	fmt.Printf(now.Format("15:04:05"))
  	fmt.Println()
  
  	fmt.Printf(now.Format("2006"))
  	fmt.Println()
  }
  ```

  **执行结果：**

  ![image-20220311114700338](https://gitee.com/jobim/blogimage/raw/master/img/20220311114700.png)

  * "2006/01/0215:04:05”这个字符串的各个数字是固定的，必须是这样写。
  * "2006/01/02 15:04:05”这个字符串各个数字可以自由的组合，这样可以按程序需求来返回时间和日期

* **时间的常量**

  ```go
  const (
      Nanosecond  Duration = 1
      Microsecond          = 1000 * Nanosecond
      Millisecond          = 1000 * Microsecond
      Second               = 1000 * Millisecond
      Minute               = 60 * Second
      Hour                 = 60 * Minute
  )
  ```

  **100毫秒：100 *time.Millisecond**

* **time的Unix 和UnixNano的方法**

  ![image-20220311143405036](https://gitee.com/jobim/blogimage/raw/master/img/20220311143405.png)

  ```go
  func main() {
  	//看看日期和时间相关函数和方法使用
  	//1. 获取当前时间
  	now := time.Now()
  	fmt.Printf("now=%v\n", now)
  	//Unix和UnixNano的使用
  	fmt.Printf("unix时间戳=%v unixnano时间戳=%v\n", now.Unix(), now.UnixNano())
  }
  ```

  ![image-20220311143615296](https://gitee.com/jobim/blogimage/raw/master/img/20220311143615.png)





# 二、方法

## 2.1 方法简介

**方法是与指定的数据类型绑定的特殊函数**

* **go方法的声明：**

  ```go
  func (t type) methodName (参数列表) (返回值列表){
  	方法体
  	return 返回值
  }
  // t type 表示这个方法和type这个类型进行绑定，t为type的一个实例
  ```

* **func (p Person) methodName (参数列表) (返回值列表){}**，t表示哪个Person变量调用，这个p就是它的副本。

* **举例说明：**

  ```go
  type Person struct {
      Name string
      Age int 
      Hometown string 
      score map[string]int
  }
  func (p Person) test() {
      p.Age += 1
      p.score["China"] += 1 //对于引用数据类型会修改其值
  }
   
  func main()  {
      m0 := make(map[string]int)
      m0["China"] = 80
      person0 := Person{"szc", 23, "Henan Anyang", m0}
   
      person2 := new (Person)
      (*person2).Name = "Jason"
      (*person2).Age = 24
      m2 := make(map[string]int)
      m2["Math"] = 90
      (*person2).score = m2
   
      person0.test()
      fmt.Println(person0)
      (*person2).test()
      fmt.Println(*person2)
  }
  ```

  **执行结果：**

  ![image-20220314113038894](https://gitee.com/jobim/blogimage/raw/master/img/20220314113038.png)



**方法的调用和传参机制：**

* **方法的调用和传参机制和函数基本一样。不一样的地方时，变量调用方法时，该变量本身也会作为一个参数传递到方法(如果变量是值类型，则进行值拷贝，如果变量是引用类型，则进行地质拷贝)**





**方法的注意事项：**

* **如果希望修改结构体变量的值，可以通过`结构体指针`的方式来处理**

  ```go
  type Circle struct {
  	radius float64
  }
  
  //为了提高效率，通常我们方法和结构体的指针类型绑定
  func (c *Circle) area2() float64 {
  	//因为 c是指针，因此我们标准的访问其字段的方式是 (*c).radius
  	//return 3.14 * (*c).radius * (*c).radius
  	// (*c).radius 等价  c.radius 
  	fmt.Printf("c 是  *Circle 指向的地址=%p\n", c)
  	c.radius = 10
  	return 3.14 * c.radius * c.radius
  }
  
  func main() {
  
  	//创建一个Circle 变量
  	var c Circle 
  	fmt.Printf("main c 结构体变量地址 =%p\n", &c)
  	c.radius = 7.0
  	//res2 := (&c).area2()
  	//编译器底层做了优化  (&c).area2() 等价 c.area()
  	//因为编译器会自动的给加上 &c
  	res2 := c.area2()
  	fmt.Println("面积=", res2)
  	fmt.Println("c.radius = ", c.radius) //10
  }
  ```

  **执行结果：**

  ![image-20220314111414574](https://gitee.com/jobim/blogimage/raw/master/img/20220314111414.png)

* **Golang中的方法作用在指定的数据类型上的(即:和指定的数据类型绑定)，因此自定义类型，都可以有方法，而不仅仅是struct，比如int , float32等都可以有方法。**

  ```go
  func (i integer) print() {
  	fmt.Println("i=", i)
  }
  //编写一个方法，可以改变i的值
  func (i *integer) change() {
  	*i = *i + 1
  }
  
  func main() {
  	var i integer = 10
  	i.print()
  	i.change()
  	fmt.Println("i=", i)
  } 
  ```

* **如果一个类型实现了`String()`这个方法，那么fmt.Println默认会调用这个变量的String()进行输出**

  如果String()方法绑定的是结构体指针，那么输出时要传入地址，否则会按照原来的方式输出

  ```go
  type Student struct {
  	Name string
  	Age int
  }
  
  //给*Student实现方法String()
  func (stu *Student) String() string {
  	str := fmt.Sprintf("Name=[%v] Age=[%v]", stu.Name, stu.Age)
  	return str
  }
  
  func main() {
  	//定义一个Student变量
  	stu := Student{
  		Name : "tom",
  		Age : 20,
  	}
  	//如果你实现了 *Student 类型的 String方法，就会自动调用
  	fmt.Println(&stu) 
  }
  ```

* **对于方法（如 struct的方法)，接收者为值类型时，可以直接用指针类型的变量调用方法，反过来同样也可以**

  对于普通函数，接收者为值类型时，不能将指针类型的数据直接传递，反之亦然

  ```go
  type Person struct {
  	Name string
  } 
  
  //函数
  //对于普通函数，接收者为值类型时，不能将指针类型的数据直接传递，反之亦然
  
  func test01(p Person) {
  	fmt.Println(p.Name)
  }
  
  func test02(p *Person) {
  	fmt.Println(p.Name)
  }
  
  //对于方法（如struct的方法），
  //接收者为值类型时，可以直接用指针类型的变量调用方法，反过来同样也可以
  
  func (p Person) test03() {
  	p.Name = "jack"
  	fmt.Println("test03() =", p.Name) // jack
  }
  
  func (p *Person) test04() {
  	p.Name = "mary"
  	fmt.Println("test03() =", p.Name) // mary
  }
  
  func main() {
  
  	p := Person{"tom"}
  	test01(p)
  	test02(&p)
  
  	p.test03()
  	fmt.Println("main() p.name=", p.Name) // tom
  	
  	(&p).test03() // 从形式上是传入地址，但是本质仍然是值拷贝
  
  	fmt.Println("main() p.name=", p.Name) // tom
  
  
  	(&p).test04()
  	fmt.Println("main() p.name=", p.Name) // mary
  	p.test04() // 等价 (&p).test04 , 从形式上是传入值类型，但是本质仍然是地址拷贝
  
  }
  ```

  **不管调用形式如何，真正决定是值拷贝还是地址拷贝，看这个方法是和哪个类型绑定**。如果是值类型，比如(p Person)，则是值拷贝，如果和指针类型，比如是( *Person)则是地址拷贝。

  **执行结果：**

  ![image-20220314112859801](https://gitee.com/jobim/blogimage/raw/master/img/20220314112859.png)





## 2.2 通过方法封装

**封装的实现步骤：**

- **将结构体、字段的首字母小写；**
- **给结构体所在的包提供一个工厂模式的函数，首字母大写，类似一个构造函数；**
- **提供一个首字母大写的 Set 方法（类似其它语言的 public），用于对属性判断并赋值；**
- **提供一个首字母大写的 Get 方法（类似其它语言的 public），用于获取属性的值。**



```go
type person struct {
    Name string
    age int   //其它包不能直接访问..
    sal float64
}

//写一个工厂模式的函数，相当于构造函数
func NewPerson(name string) *person {
    return &person{
        Name : name,
    }
}

//为了访问age 和 sal 我们编写一对SetXxx的方法和GetXxx的方法
func (p *person) SetAge(age int) {
    if age >0 && age <150 {
        p.age = age
    } else {
        fmt.Println("年龄范围不正确..")
        //给程序员给一个默认值
    }
}
func (p *person) GetAge() int {
    return p.age
}

func (p *person) SetSal(sal float64) {
    if sal >= 3000 && sal <= 30000 {
        p.sal = sal
    } else {
        fmt.Println("薪水范围不正确..")
    }
}

func (p *person) GetSal() float64 {
    return p.sal
}

func main() {
    var p *person = NewPerson("smith")
    p.SetAge(18)
    p.SetSal(5000)
    fmt.Println(*p)
    fmt.Println(p.Name, " age =", p.GetAge(), " sal = ", p.GetSal())
}
```





# 三、接口

## 3.1 接口简介

**interface 类型可以定义一组方法，但是这些不需要实现，并且 interface不能包含任何变量。**



* **Go接口实现机制很简洁，`只要目标类型方法集内包含接口声明的全部方法，就被视为实现了该接口，无须做显式声明`**

* **接口可以嵌入其他接口类型**

* **接口只能声明方法，不能实现**

* **一个自定义类型可以实现多个接口**

* **接口通常以 er 作为名称后缀**

* **只要是自定义数据类型，就可以实现接口，不仅仅是结构体类型。**

  ![image-20220314153159700](https://gitee.com/jobim/blogimage/raw/master/img/20220314153159.png)

**代码示例：**

```go
//声明/定义一个接口
type Usber interface {
	//声明了两个没有实现的方法
	Start() 
	Stop()
}

type Phone struct {

}  

//让Phone 实现 Usb接口的方法，就实现了Usb接口
func (p Phone) Start() {
	fmt.Println("手机开始工作。。。")
}
func (p Phone) Stop() {
	fmt.Println("手机停止工作。。。")
}

//计算机
type Computer struct {

}

//编写一个方法Working 方法，接收一个Usb接口类型变量
//只要是实现了 Usb接口 （所谓实现Usb接口，就是指实现了 Usb接口声明所有方法）
func (c Computer) Working(usb Usber) {

	//通过usb接口变量来调用Start和Stop方法
	usb.Start()
	usb.Stop()
}

func main() {

	//测试
	//先创建结构体变量
	computer := Computer{}
	phone := Phone{}

	//关键点
	computer.Working(phone)
}
```

**执行结果：**

![image-20220314153759623](https://gitee.com/jobim/blogimage/raw/master/img/20220314153759.png)





## 3.2 类型转换

* **类型转换可将接口变量还原为原始类型，或用来判断是否实现了某个更具体的接口类型**

  ```go
  //类型断言的其它案例
  var x interface{}
  var b2 float32 = 1.1
  x = b2  //空接口，可以接收任意类型
  // x=>float32 [使用类型断言]
  y := x.(float32)
  fmt.Printf("y 的类型是 %T 值是=%v", y, y)	//y 的类型是 float32 值是=1.1
  ```

* **带检测机制的类型断言：**

  ```go
  type Point struct {
  	x int
  	y int
  }
  
  func main() {
  
  	var a interface{}
  	var point Point = Point{1, 2}
  	a = point //ok
  	var b Point
  	//类型断言(带检测的)，如果成功返回true
  	b, ok := a.(Point)
  	if ok {
  		fmt.Println("convert success")
  		fmt.Printf("y 的类型是 %T 值是=%v", b, b)
  	} else {
  		fmt.Println("convert fail")
  	}
  }
  ```

  

# 四、包的基本概念

**go的每一个文件都是属于一个包的，也就是说go是以包的形式来管理文件和项目目录结构的**

* **包的本质：**

  **包的本质实际上就是创建不同的文件夹，来存放程序文件**

  ![image-20220316160819291](https://gitee.com/jobim/blogimage/raw/master/img/20220316160819.png)



* **声明其所属的包：package 包名**
* **引入包的基本语法：import "包的路径"**



**引用其他包的代码示例：**

* 文件结构：

  ![image-20220316161949853](https://gitee.com/jobim/blogimage/raw/master/img/20220316161949.png)

* utils.go文件：

  ```go
  package utils
  import (
  	"fmt"
  )
  
  //定义一个函数
  func Cal(n1 float64, n2 float64, operator byte) float64 {
  	var res float64
  	switch operator {
  	case '+':
  		res = n1 + n2
  	case '-':
  		res = n1 - n2
  	case '*':
  		res = n1 * n2
  	case '/':
  		res = n1 / n2
  	default:
  		fmt.Println("操作符号错误...")
  	}
  	return res
  }
  ```

* main.go文件：

  ```go
  package main
  
  import (
  	"fmt"
  	//在import包时，路径从$GOPATH的src下开始，不用带src，编译器会自动从src下开始引入
  	"go_code/project02/util"
  )
  
  func main() {
  	var n1 float64 = 1.2
  	var n2 float64 = 2.3
  	var operator byte = '+'
  	//包名.函数名()
  	result := utils.Cal(n1, n2, operator)
  	fmt.Println("result=", result)
  }
  ```

  

**包使用的注意事项：**

* **在 import 包时，路径从$GOPATH 的 src下开始，不用带src，编译器会自动从src下开始引入**

* **为了让其它包的文件，可以访问到本包的函数，则该函数名的首字母需要大写，类似其它语言的 public ,这样才能跨包访问。**

* **如果包名较长，Go支持给包取别名，注意细节:取别名后，原来的包名就不能使用了**

  ![image-20220316162557520](https://gitee.com/jobim/blogimage/raw/master/img/20220316162557.png)

* **如果你要编译成一个可执行程序文件，就需要将这个包声明为main，即 package main .这个就是一个语法规范，如果你是写一个库﹐包名可以自定义**



