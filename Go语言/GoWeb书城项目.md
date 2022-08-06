**项目结构：**

![image-20220323180203977](https://gitee.com/jobim/blogimage/raw/master/img/20220323180204.png)

# 数据库相关工具类

**utils/db.go：**

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
	Db, err = sql.Open("mysql", "root:root@tcp(localhost:3306)/bookstore")
	if err != nil {
		panic(err.Error())
	}
}
```





# 图书分页查询

![image-20220323175646452](https://gitee.com/jobim/blogimage/raw/master/img/20220323175646.png)

**book_manager.html相关代码：**

```html
<div id="main">
    <table>
        <tr>
            <td>名称</td>
            <td>价格</td>
            <td>作者</td>
            <td>销量</td>
            <td>库存</td>
            <td colspan="2">操作</td>
        </tr>
        {{range .Books}}
        <tr>
            <td>{{.Title}}</td>
            <td>{{.Price}}</td>
            <td>{{.Author}}</td>
            <td>{{.Sales}}</td>
            <td>{{.Stock}}</td>
            <td><a href="/toUpdateOrAddBookPage?bookId={{.ID}}">修改</a></td>
            <td><a id="{{.Title}}" class="deleteBook" href="/deleteBook?bookId={{.ID}}">删除</a></td>
        </tr>	
        {{end}}	
        <tr>
            <td></td>
            <td></td>
            <td></td>
            <td></td>
            <td></td>
            <td></td>
            <td><a href="/toUpdateOrAddBookPage">添加图书</a></td>
        </tr>	
    </table>
    <div id="page_nav">
        {{if .IsHasPrev}}
        <a href="/getPageBooks">首页</a>
        <a href="/getPageBooks?pageNo={{.GetPrevPageNo}}">上一页</a>
        {{end}}	
        当前是第{{.PageNo}}页，共{{.TotalPageNo}}页，共{{.TotalRecord}}条记录
        {{if .IsHasNext}}	
        <a href="/getPageBooks?pageNo={{.GetNextPageNo}}">下一页</a>
        <a href="/getPageBooks?pageNo={{.TotalPageNo}}">末页</a>
        {{end}}	
        到第<input value="{{.PageNo}}" name="pn" id="pn_input"/>页
        <input type="button" value="确定" id="sub">
        <script>
            //给确定按钮绑定单击事件
            $("#sub").click(function(){
                //获取输入的页码
                var pageNo = $("#pn_input").val();
                location = "/getPageBooks?pageNo="+pageNo
            });
        </script>
    </div>
</div>
```

**page结构体：用于存放分页数据**

```go
package model

//Page 结构
type Page struct {
	Books       []*Book //每页查询出来的图书存放的切片
	PageNo      int64   //当前页
	PageSize    int64   //每页显示的条数
	TotalPageNo int64   //总页数，通过计算得到
	TotalRecord int64   //总记录数，通过查询数据库得到
	MinPrice    string
	MaxPrice    string
	IsLogin     bool
	Username    string
}

//IsHasPrev 判断是否有上一页
func (p *Page) IsHasPrev() bool {
	return p.PageNo > 1
}

//IsHasNext 判断是否有下一页
func (p *Page) IsHasNext() bool {
	return p.PageNo < p.TotalPageNo
}

//GetPrevPageNo 获取上一页
func (p *Page) GetPrevPageNo() int64 {
	if p.IsHasPrev() {
		return p.PageNo - 1
	} else {
		return 1
	}
}

//GetNextPageNo 获取下一页
func (p *Page) GetNextPageNo() int64 {
	if p.IsHasNext() {
		return p.PageNo + 1
	} else {
		return p.TotalPageNo
	}
}
```

**bookhandler.go相关代码：**

```go
//GetPageBooks 获取带分页的图书
func GetPageBooks(w http.ResponseWriter, r *http.Request) {
	//获取页码
	pageNo := r.FormValue("pageNo")
	if pageNo == "" {
		pageNo = "1"
	}
	//调用bookdao中获取带分页的图书的函数
	page, _ := dao.GetPageBooks(pageNo)
	//解析模板文件
	t := template.Must(template.ParseFiles("views/pages/manager/book_manager.html"))
	//执行
	t.Execute(w, page)
}
```



**bookdao.go相关代码：**

```go
//GetPageBooks 获取带分页的图书信息
func GetPageBooks(pageNo string) (*model.Page, error) {
	//将页码转换为int64类型
	iPageNo, _ := strconv.ParseInt(pageNo, 10, 64)
	//获取数据库中图书的总记录数
	sqlStr := "select count(*) from books"
	//设置一个变量接收总记录数
	var totalRecord int64
	//执行
	row := utils.Db.QueryRow(sqlStr)
	row.Scan(&totalRecord)
	//设置每页只显示4条记录
	var pageSize int64 = 4
	//设置一个变量接收总页数
	var totalPageNo int64
	if totalRecord%pageSize == 0 {
		totalPageNo = totalRecord / pageSize
	} else {
		totalPageNo = totalRecord/pageSize + 1
	}
	//获取当前页中的图书
	sqlStr2 := "select id,title,author,price,sales,stock,img_path from books limit ?,?"
	//执行
	rows, err := utils.Db.Query(sqlStr2, (iPageNo-1)*pageSize, pageSize)
	if err != nil {
		return nil, err
	}
	var books []*model.Book
	for rows.Next() {
		book := &model.Book{}
		rows.Scan(&book.ID, &book.Title, &book.Author, &book.Price, &book.Sales, &book.Stock, &book.ImgPath)
		//将book添加到books中
		books = append(books, book)
	}
	//创建page
	page := &model.Page{
		Books:       books,
		PageNo:      iPageNo,
		PageSize:    pageSize,
		TotalPageNo: totalPageNo,
		TotalRecord: totalRecord,
	}
	return page, nil
}
```







# 合并添加图书和更新图书的页面和方法

![image-20220323172851311](https://gitee.com/jobim/blogimage/raw/master/img/20220323172851.png)

**bookhandler.go相关代码：**

```go
//ToUpdateBookPage 去更新或者添加图书的页面
func ToUpdateOrAddBookPage(w http.ResponseWriter, r *http.Request) {
	//获取要更新的图书的id
	bookID := r.FormValue("bookId")
	//调用bookdao中获取图书的函数
	book, _ := dao.GetBookByID(bookID)
	if book.ID > 0 {
		//在更新图书
		//解析模板
		t := template.Must(template.ParseFiles("views/pages/manager/book_edit.html"))
		//执行
		t.Execute(w, book)
	} else {
		//在添加图书
		//解析模板
		t := template.Must(template.ParseFiles("views/pages/manager/book_edit.html"))
		//执行
		t.Execute(w, "")
	}
}

//UpdateOrAddBook 更新或添加图书
func UpdateOrAddBook(w http.ResponseWriter, r *http.Request) {
	//获取图书信息
	bookID := r.PostFormValue("bookId")
	title := r.PostFormValue("title")
	author := r.PostFormValue("author")
	price := r.PostFormValue("price")
	sales := r.PostFormValue("sales")
	stock := r.PostFormValue("stock")
	//将价格、销量和库存进行转换
	fPrice, _ := strconv.ParseFloat(price, 64)
	iSales, _ := strconv.ParseInt(sales, 10, 0)
	iStock, _ := strconv.ParseInt(stock, 10, 0)
	ibookID, _ := strconv.ParseInt(bookID, 10, 0)
	//创建Book
	book := &model.Book{
		ID:      int(ibookID),
		Title:   title,
		Author:  author,
		Price:   fPrice,
		Sales:   int(iSales),
		Stock:   int(iStock),
		ImgPath: "/static/img/default.jpg",
	}
	if book.ID > 0 {
		//在更新图书
		//调用bookdao中更新图书的函数
		dao.UpdateBook(book)
	} else {
		//在添加图书
		//调用bookdao中添加图书的函数
		dao.AddBook(book)
	}
	//调用GetBooks处理器函数再次查询一次数据库
	GetPageBooks(w, r)
}
```

**book_edit.html核心代码：通过模板引擎的条件动作判断是添加图书还是更新图书**

```html
<div id="main">
    <form action="/updateOraddBook" method="POST">
        <table>
            <tr>
                <td>名称</td>
                <td>价格</td>
                <td>作者</td>
                <td>销量</td>
                <td>库存</td>
                <td colspan="2">操作</td>
            </tr>
            <tr>
                {{if .}}
                <input type="hidden" name="bookId" value="{{.ID}}" />
                <td><input name="title" type="text" value="{{.Title}}"/></td>
                <td><input name="price" type="text" value="{{.Price}}"/></td>
                <td><input name="author" type="text" value="{{.Author}}"/></td>
                <td><input name="sales" type="text" value="{{.Sales}}"/></td>
                <td><input name="stock" type="text" value="{{.Stock}}"/></td>
                {{else}}
                <td><input name="title" type="text" value="时间简史"/></td>
                <td><input name="price" type="text" value="30.00"/></td>
                <td><input name="author" type="text" value="霍金"/></td>
                <td><input name="sales" type="text" value="200"/></td>
                <td><input name="stock" type="text" value="300"/></td>
                {{end}}	
                <td><input type="submit" value="提交"/></td>
            </tr>
        </table>
    </form>
</div>
```



# 登陆时使用session存储用户信息

**创建session的数据库表：**

```sql
-- 创建sessions表
    CREATE TABLE sessions(
    session_id VARCHAR(100) PRIMARY KEY,
    username VARCHAR(100) NOT NULL,
    user_id INT NOT NULL,
    FOREIGN KEY(user_id) REFERENCES users(id)
)
```

**session的结构体：**

```go
//Session 结构
type Session struct {
	SessionID string
	UserName  string
	UserID    int
}
```

**登陆成功的时候创建session并写入数据库，同时创建cookie发送给浏览器：**

**userhandler.go：**

```go
//Login 处理用户登录的函数
func Login(w http.ResponseWriter, r *http.Request) {
	//获取用户名和密码
	username := r.PostFormValue("username")
	password := r.PostFormValue("password")
	//调用userdao中验证用户名和密码的方法
	user, _ := dao.CheckUserNameAndPassword(username, password)
	if user.ID > 0 {
		//用户名和密码正确
		//生成UUID作为Session的id
		uuid := utils.CreateUUID()
		//创建一个Session
		sess := &model.Session{
			SessionID: uuid,
			UserName:  user.Username,
			UserID:    user.ID,
		}
		//将Session保存到数据库中
		dao.AddSession(sess)
		//创建一个Cookie，让它与Session相关联
		cookie := http.Cookie{
			Name:    "user",
			Value:    uuid,
			HttpOnly: true,
		}
		//将cookie发送给浏览器
		http.SetCookie(w, &cookie)
		t := template.Must(template.ParseFiles("views/pages/user/login_success.html"))
		t.Execute(w, user)
	} else {
		//用户名或密码不正确
		t := template.Must(template.ParseFiles("views/pages/user/login.html"))
		t.Execute(w, "用户名或密码不正确！")
	}
}
```

**浏览器中保存了cookie：**![image-20220319162015983](https://gitee.com/jobim/blogimage/raw/master/img/20220319162016.png)

**数据库中添加了数据：**

![image-20220319161926688](https://gitee.com/jobim/blogimage/raw/master/img/20220319161926.png)





# 处理重复登陆问题

**登陆成功后刷新页面会出现重复登陆的问题：**

![image-20220319172106669](https://gitee.com/jobim/blogimage/raw/master/img/20220319172106.png)



**在登陆函数中判断是否登陆：**

![image-20220319173057642](https://gitee.com/jobim/blogimage/raw/master/img/20220319173057.png)



 # 购物车的设计

**购物车表：**

```sql
CREATE TABLE carts(
    id VARCHAR(100) PRIMARY KEY,
    total_count INT NOT NULL,
    total_amount DOUBLE(11,2) NOT NULL,
    user_id INT NOT NULL,
    FOREIGN KEY(user_id) REFERENCES users(id)
)
```

**购物项表：**

```sql
CREATE TABLE cart_itmes(
    id INT PRIMARY KEY AUTO_INCREMENT,
    COUNT INT NOT NULL,
    amount DOUBLE(11,2) NOT NULL,
    book_id INT NOT NULL,
    cart_id VARCHAR(100) NOT NULL,
    FOREIGN KEY(book_id) REFERENCES books(id),
    FOREIGN KEY(cart_id) REFERENCES carts(id)
)
```

**添加图书到购物车的方法：**

```go
//AddBook2Cart 添加图书到购物车
func AddBook2Cart(w http.ResponseWriter, r *http.Request) {
	//判断是否登陆
	flag, session := dao.IsLogin(r)
	if flag {
		//已经登录
		//获取要添加的图书的id
		bookID := r.FormValue("bookId")
		//根据图书的id获取图书信息
		book, _ := dao.GetBookByID(bookID)

		//获取用户的id
		userID := session.UserID
		//判断数据库中是否有当前用户的购物车
		cart, _ := dao.GetCartByUserID(userID)
		if cart != nil {
			//当前用户已经有购物车，此时需要判断购物车中是否有当前这本图书
			carItem, _ := dao.GetCartItemByBookIDAndCartID(bookID, cart.CartID)
			if carItem != nil {
				//购物车的购物项中已经有该图书，只需要将该图书所对应的购物项中的数量加1即可
				//1.获取购物车切片中的所有的购物项
				cts := cart.CartItems
				//2.遍历得到每一个购物项
				for _, v := range cts {
					fmt.Println("当前购物项中是否有Book：", v)
					fmt.Println("查询到的Book是：", carItem.Book)
					//3.找到当前的购物项
					if v.Book.ID == carItem.Book.ID {
						//将购物项中的图书的数量加1
						v.Count = v.Count + 1
						//更新数据库中该购物项的图书的数量
						dao.UpdateBookCount(v)
					}
				}
			} else {
				//购物车的购物项中还没有该图书，此时需要创建一个购物项并添加到数据库中
				//创建购物车中的购物项
				cartItem := &model.CartItem{
					Book:   book,
					Count:  1,
					CartID: cart.CartID,
				}
				//将购物项添加到当前cart的切片中
				cart.CartItems = append(cart.CartItems, cartItem)
				//将新创建的购物项添加到数据库中
				dao.AddCartItem(cartItem)
			}
			//不管之前购物车中是否有当前图书对应的购物项，都需要更新购物车中的图书的总数量和总金额
			fmt.Println("=====购物车中的图书信息：", cart)
			for _, v := range cart.CartItems {
				fmt.Println(v)
			}
			dao.UpdateCart(cart)
		} else {
			//证明当前用户还没有购物车，需要创建一个购物车并添加到数据库中
			//1.创建购物车
			//生成购物车的id
			cartID := utils.CreateUUID()
			cart := &model.Cart{
				CartID: cartID,
				UserID: userID,
			}
			//2.创建购物车中的购物项
			//声明一个CartItem类型的切片
			var cartItems []*model.CartItem
			cartItem := &model.CartItem{
				Book:   book,
				Count:  1,
				CartID: cartID,
			}
			//将购物项添加到切片中
			cartItems = append(cartItems, cartItem)
			//3将切片设置到cart中
			cart.CartItems = cartItems
			//4.将购物车cart保存到数据库中
			dao.AddCart(cart)
		}
		w.Write([]byte("您刚刚将" + book.Title + "添加到了购物车！"))
	} else {
		//没有登陆
		w.Write([]byte("请先登陆！"))
	}
}
```

**点击加入购物车：**

![image-20220320144454527](https://gitee.com/jobim/blogimage/raw/master/img/20220320144454.png)

查看数据库添加了数据：

![image-20220320144716112](https://gitee.com/jobim/blogimage/raw/master/img/20220320144716.png)



# ajax更新购物车中商品的数量



![image-20220323175042398](https://gitee.com/jobim/blogimage/raw/master/img/20220323175042.png)

**cart.html相关代码：**

```html
<script>
	$(function(){
		//给输入购物项数量的input绑定change事件
		$(".updateCartItem").change(function(){
			//获取购物项的id
			var cartItemId = $(this).attr("id");
			//获取用户输入的图书的数量
			var bookCount = $(this).val();
			//发送请求
			// location = "/updateCartItem?cartItemId="+cartItemId+"&bookCount="+bookCount;
			//设置请求的url
			var url = "/updateCartItem";
			//设置请求参数
			var params = {"cartItemId":cartItemId,"bookCount":bookCount};
			//获取显示购物项中的金额小计的td元素
			var $tdEle = $(this).parent().next().next();
			//发送Ajax请求
			$.post(url,params,function(res){
				//设置总数量
				$("#totalCount").text(res.TotalCount);
				//设置总金额
				$("#totalAmount").text(res.TotalAmount);
				//设置金额小计
				$tdEle.text(res.Amount);
			},"json");
		});
	});
</script>

{{if .Cart}}
<table>
    <tr>
        <td>商品名称</td>
        <td>数量</td>
        <td>单价</td>
        <td>金额</td>
        <td>操作</td>
    </tr>
    {{range .Cart.CartItems}}	
    <tr>
        <td>{{.Book.Title}}</td>
        <td>
            <!-- 更新购物车中商品的数量 -->
            <input id="{{.CartItemID}}" class="updateCartItem" type="number" min="1" value="{{.Count}}" style="text-align:center;width: 50px;"/>
        </td>
        <td>{{.Book.Price}}</td>
        <td>{{.Amount}}</td>
        <td><a id="{{.Book.Title}}" class="deleteCartItem" href="/deleteCartItem?cartItemId={{.CartItemID}}">删除</a></td>
    </tr>
    {{end}}
</table>

<div class="cart_info">
    <span class="cart_span">购物车中共有<span class="b_count" id="totalCount">{{.Cart.TotalCount}}</span>件商品</span>
    <span class="cart_span">总金额<span class="b_price" id="totalAmount">{{.Cart.TotalAmount}}</span>元</span>
    <span class="cart_span"><a href="/main">继续购物</a></span>
    <span class="cart_span"><a href="/deleteCart?cartId={{.Cart.CartID}}" id="emptyCart">清空购物车</a></span>
    <span class="cart_span"><a href="/checkout">去结账</a></span>
</div>
{{else}}
<br/><br/><br/><br/><br/><br/><br/><br/><br/>
<h1 style="text-align: center">您的购物车饥渴难耐，快去<a href="/main" style="color:red">购物</a>吧！</h1>
{{end}}
```

**carthandler.go相关代码：**

```go
//UpdateCartItem 更新购物项
func UpdateCartItem(w http.ResponseWriter, r *http.Request) {
	//获取要更新的购物项的id
	cartItemID := r.FormValue("cartItemId")
	//将购物项的id转换为int64
	iCartItemID, _ := strconv.ParseInt(cartItemID, 10, 64)
	//获取用户输入的图书的数量
	bookCount := r.FormValue("bookCount")
	iBookCount, _ := strconv.ParseInt(bookCount, 10, 64)
	//获取session
	_, session := dao.IsLogin(r)
	//获取用户的id
	userID := session.UserID
	//获取该用户的购物车
	cart, _ := dao.GetCartByUserID(userID)
	//获取购物车中的所有的购物项
	cartItems := cart.CartItems
	//遍历得到每一个购物项
	for _, v := range cartItems {
		//寻找要更新的购物项
		if v.CartItemID == iCartItemID {
			//这个就是我们要更新的购物项
			//将当前购物项中的图书的数量设置为用户输入的值
			v.Count = iBookCount
			//更新数据库中该购物项的图书的数量和金额小计
			dao.UpdateBookCount(v)
		}
	}
	//更新购物车中的图书的总数量和总金额
	dao.UpdateCart(cart)
	//调用获取购物项信息的函数再次查询购物车信息
	cart, _ = dao.GetCartByUserID(userID)
	// GetCartInfo(w, r)
	//获取购物车中图书的总数量
	totalCount := cart.TotalCount
	//获取购物车中图书的总金额
	totalAmount := cart.TotalAmount
	var amount float64
	//获取购物车中更新的购物项中的金额小计
	cIs := cart.CartItems
	for _, v := range cIs {
		if iCartItemID == v.CartItemID {
			//这个就是我们寻找的购物项，此时获取当前购物项中的金额小计
			amount = v.Amount
		}
	}
	//创建Data结构
	data := model.Data{
		Amount:      amount,
		TotalAmount: totalAmount,
		TotalCount:  totalCount,
	}
	//将data转换为json字符串
	json, _ := json.Marshal(data)
	//响应到浏览器
	w.Write(json)
}
```



# 生成订单

**点击结账生成订单信息：**

![image-20220323173829985](https://gitee.com/jobim/blogimage/raw/master/img/20220323173830.png)

* **查看我的订单：**

  ![image-20220323174253610](https://gitee.com/jobim/blogimage/raw/master/img/20220323174253.png)



* **查看订单详情：**

  ![image-20220323174314713](https://gitee.com/jobim/blogimage/raw/master/img/20220323174314.png)

**数据库：**

* **订单表：**

  ```sql
  -- 创建订单表
  CREATE TABLE orders(
      id VARCHAR(100) PRIMARY KEY,
      create_time DATETIME NOT NULL,
      total_count INT NOT NULL,
      total_amount DOUBLE(11,2) NOT NULL,
      state INT NOT NULL,
      user_id INT,
      FOREIGN KEY(user_id) REFERENCES users(id)
  )
  ```

* **订单项表：**

  ```sql
  -- 创建订单项表
  CREATE TABLE order_items(
      id INT PRIMARY KEY AUTO_INCREMENT,
      COUNT INT NOT NULL,
      amount DOUBLE(11,2) NOT NULL,
      title VARCHAR(100) NOT NULL,
      author VARCHAR(100) NOT NULL,
      price DOUBLE(11,2) NOT NULL,
      img_path VARCHAR(100) NOT NULL,
      order_id VARCHAR(100) NOT NULL,
      FOREIGN KEY(order_id) REFERENCES orders(id)
  )
  ```

**orderhandler.go中生成订单函数：**

```go
//Checkout //去结账
func Checkout(w http.ResponseWriter, r *http.Request) {
	//获取session
	_, session := dao.IsLogin(r)
	//获取用户的id
	userID := session.UserID
	//获取购物车
	cart, _ := dao.GetCartByUserID(userID)
	//生成订单号
	orderID := utils.CreateUUID()
	//创建生成订单的时间
	timeStr := time.Now().Format("2006-01-02 15:04:05")
	//创建Order
	order := &model.Order{
		OrderID:     orderID,
		CreateTime:  timeStr,
		TotalCount:  cart.TotalCount,
		TotalAmount: cart.TotalAmount,
		State:       0,
		UserID:      int64(userID),
	}
	//将订单保存到数据库中
	dao.AddOrder(order)
	//保存订单项
	//获取购物车中的购物项
	cartItems := cart.CartItems
	//遍历得到每一个购物项
	for _, v := range cartItems {
		//创建OrderItem
		orderItem := &model.OrderItem{
			Count:   v.Count,
			Amount:  v.Amount,
			Title:   v.Book.Title,
			Author:  v.Book.Author,
			Price:   v.Book.Price,
			ImgPath: v.Book.ImgPath,
			OrderID: orderID,
		}
		//将购物项保存到数据库中
		dao.AddOrderItem(orderItem)
		//更新当前购物项中图书的库存和销量
		book := v.Book
		book.Sales = book.Sales + int(v.Count)
		book.Stock = book.Stock - int(v.Count)
		//更新图书的信息
		dao.UpdateBook(book)
	}
	//清空购物车
	dao.DeleteCartByCartID(cart.CartID)
	//将订单号设置到session中
	session.OrderID = orderID
	//解析模板
	t := template.Must(template.ParseFiles("views/pages/cart/checkout.html"))
	//执行
	t.Execute(w, session)
}
```

