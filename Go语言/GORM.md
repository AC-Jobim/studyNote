# 一、gorm介绍

**`ORM`，即对象关系映射(`Object Relational Mapping`)，可以简单理解为将`关系型数据库`中的`数据表`映射为编程语言中的具体的数据类型(如`struct`)，而`GORM`库就是一个使用Go语言实现的且功能非常完善易使用的`ORM`框架。**



[Github连接](https://github.com/go-gorm/gorm)

[中文官方网站](https://gorm.io/zh_CN/)：写的很详细



**gorm的安装：**

* 安装`GORM`非常简单，使用`go get -u`就可以在`GOPATH`目录下安装最新`GROM`框架。

  ```go
  go get -u github.com/jinzhu/gorm
  ```

* 使用`import`关键字导入`GORM`库

  ```go
  import "github.com/jinzhu/gorm"
  ```



# 二、连接数据库

* **支持的数据库：`GORM`框架支持`MySQL`,`SQL Server`,`Sqlite3`,`PostgreSQL`四种数据库驱动，如果我们要连接这些数据库，则需要导入不同的驱动包及定义不同格式的`DSN`(`Data Source Name`)。**



<font color='blue' size='4'>**第一步：导入驱动**</font>

* 连接不同的数据库都需要导入对应数据的驱动程序，`GORM`已经为我们包装了一些驱动程序，只需要按如下方式导入需要的数据库驱动即可：

  ```go
  import _ "github.com/jinzhu/gorm/dialects/mysql"
  // import _ "github.com/jinzhu/gorm/dialects/postgres"
  // import _ "github.com/jinzhu/gorm/dialects/sqlite"
  // import _ "github.com/jinzhu/gorm/dialects/mssql"
  ```

  

<font color='blue' size='4'>**第二步：连接数据库**</font>

* **连接MySQL：**

  ```go
  import (
    "github.com/jinzhu/gorm"
    _ "github.com/jinzhu/gorm/dialects/mysql"
  )
  
  func main() {
    db, err := gorm.Open("mysql", "user:password@(localhost)/dbname?charset=utf8mb4&parseTime=True&loc=Local")
    defer db.Close()
  }
  ```

* **连接PostgreSQL：**

  ```go
  import (
    "github.com/jinzhu/gorm"
    _ "github.com/jinzhu/gorm/dialects/postgres"
  )
  
  func main() {
    db, err := gorm.Open("postgres", "host=myhost port=myport user=gorm dbname=gorm password=mypassword")
    defer db.Close()
  }
  ```

* **连接Sqlite3：**

  ```go
  import (
    "github.com/jinzhu/gorm"
    _ "github.com/jinzhu/gorm/dialects/sqlite"
  )
  
  func main() {
    db, err := gorm.Open("sqlite3", "/tmp/gorm.db")
    defer db.Close()
  }
  ```

* **连接SQL Server：**

  ```go
  import (
    "github.com/jinzhu/gorm"
    _ "github.com/jinzhu/gorm/dialects/mssql"
  )
  
  func main() {
    db, err := gorm.Open("mssql", "sqlserver://username:password@localhost:1433?database=dbname")
    defer db.Close()
  }
  ```

# 三、基本示例

**使用GORM进行创建、查询、更新、删除操作：**

```go
package main

import (
	"fmt"
	"github.com/jinzhu/gorm"
	_ "github.com/jinzhu/gorm/dialects/mysql"
)
 
//UserInfo --> 数据表
type UserInfo struct {
	ID     uint
	Name   string
	Gender string
	Hobby  string
}

func main() {
    
	db, err := gorm.Open("mysql", "root:root@(127.0.0.1:3306)/db1?charset=utf8mb4&parseTime=True&loc=Local")
	if err != nil {
		panic(err)
	}
	defer db.Close()
	fmt.Println("连接成功")
	// 创建表，自动迁移（把结构体和数据表进行对应）
	db.AutoMigrate(&UserInfo{})

	// 创建数据行
	u1 := UserInfo{1, "Tom", "男", "蛙泳"}
	db.Create(&u1)

	// 查询
	var u UserInfo
	db.First(&u) //查询表中第一条数据保存到u中
	fmt.Println("查询到的数据是：", u)

	// 更新
	db.Model(&u).Update("hobby", "双色球")
	fmt.Println("更新后的数据：", u)

	// 删除
	//db.Delete(&u)
}
```

**查看数据库：**

![image-20220329110916024](E:\个人\学习\笔记\Go语言\GORM.assets\image-20220329110916024.png)



* **`db.AutoMigrate(&UserInfo{})`：自动迁移**

  **自动迁移仅仅会创建表，添加缺少列和索引，并且不会改变现有列的类型或删除未使用的列以保护数据。默认使用结构体名的`蛇形复数`作为表名**



# 四、模型定义

**一般来说，我们说`GROM`的模型定义，是指定义代表一个数据表的结构体(`struct`)，然后我们可以使用`GROM`框架可以将结构体映射为相对应的关系数据库的数据表，或者查询数据表中的数据来填充结构体。**



## 结构体标记（tags）

`Go`语言的结构体支持使用`tags`为结构体的每个字段扩展额外的信息。`GROM`框架有自己的一个`tags`约定，如下所示:

| 结构体标记（Tag） |                           描述                           |
| :---------------: | :------------------------------------------------------: |
|      Column       |                         指定列名                         |
|       Type        |                      指定列数据类型                      |
|       Size        |                  指定列大小, 默认值255                   |
|    PRIMARY_KEY    |                      将列指定为主键                      |
|      UNIQUE       |                      将列指定为唯一                      |
|      DEFAULT      |                       指定列默认值                       |
|     PRECISION     |                        指定列精度                        |
|     NOT NULL      |                    将列指定为非 NULL                     |
|  AUTO_INCREMENT   |                   指定列是否为自增类型                   |
|       INDEX       | 创建具有或不带名称的索引, 如果多个索引同名则创建复合索引 |
|   UNIQUE_INDEX    |         和 `INDEX` 类似，只不过创建的是唯一索引          |
|     EMBEDDED      |                     将结构设置为嵌入                     |
|  EMBEDDED_PREFIX  |                    设置嵌入结构的前缀                    |
|         -         |                        忽略此字段                        |

**GROM还支持一些关联数据表的tags约定：**

|        结构体标记（Tag）         |                描述                |
| :------------------------------: | :--------------------------------: |
|            MANY2MANY             |             指定连接表             |
|            FOREIGNKEY            |              设置外键              |
|      ASSOCIATION_FOREIGNKEY      |            设置关联外键            |
|           POLYMORPHIC            |            指定多态类型            |
|        POLYMORPHIC_VALUE         |             指定多态值             |
|       JOINTABLE_FOREIGNKEY       |          指定连接表的外键          |
| ASSOCIATION_JOINTABLE_FOREIGNKEY |        指定连接表的关联外键        |
|        SAVE_ASSOCIATIONS         |    是否自动完成 save 的相关操作    |
|      ASSOCIATION_AUTOUPDATE      |   是否自动完成 update 的相关操作   |
|      ASSOCIATION_AUTOCREATE      |   是否自动完成 create 的相关操作   |
|    ASSOCIATION_SAVE_REFERENCE    | 是否自动完成引用的 save 的相关操作 |
|             PRELOAD              |    是否自动完成预加载的相关操作    |

示例：

```go
type Post struct {
    PostId    int    `gorm:"primary_key;auto_increment"`
    Uid       int    `gorm:"type:int;not null"`
    Title     string `gorm:"type:varchar(255);not null"`
    Content   string `gorm:"type:text;not null"`
    Type      uint8  `gorm:"type:tinyint;default 1;not null"`
    CreatedAt time.Time
    UpdatedAt time.Time
    DeletedAt time.Time
}
```



## 约定

* **除了上面讲的`tags`定义了字段之间的映射规则外，Go将结构体映射为关系型数据表时，还有自己的一套惯例，或称为约定**

### 主键

**GROM的约定中，一般将数据模型中的`ID`字段映射为数据表的主键，如下面定义的`TestModel`,`ID`为主键，如果`ID`的数据类型为`int`，则GROM还会为该设置`AUTO_INCREMENT`，使用`ID`成为自增主键。**

```go
type TestModel struct{
    ID   int
    Name string
}
```

**当然，我们也可以自定义主键字段的名称，如上面的`Post`结构体，我们设置了`PostId`字段为主键，如果我们定义了其他字段为主键，那么，就算结构体中仍有`ID`字段，`GROM`框架也不会把`ID`字段当作主键了。**

```go
type Post struct {
    ID        int
    PostId    int    `gorm:"primary_key;auto_increment"`
    Uid       int    `gorm:"type:int;not null"`
    Title     string `gorm:"type:varchar(255);not null"`
    Content   string `gorm:"type:text;not null"`
    Type      uint8  `gorm:"type:tinyint;default: 1;not null"`
    CreatedAt time.Time
    UpdatedAt time.Time
    DeletedAt time.Time
}
```

所以，我们在`Post`结构体中加一个`ID`字段，`PostId`字段仍是主键，下面是在数据中使用`desc posts`语句打印出来的结果：

![image-20220329154636404](E:\个人\学习\笔记\Go语言\GORM.assets\image-20220329154636404.png)

### 数据表映射规则

**当我们使用结构体创建数据表时，数据表的名称默认为结构体的小写复数形式，如结构体`Post`对应的数据表名称为`posts`**



<font color='blue' size='4'>**更改默认表名**</font>

**可以实现 `Tabler` 接口来更改默认表名，例如：**

```go
type Tabler interface {
    TableName() string
}

type User struct {} // 默认表名是 `users`


// TableName 会将 User 的表名重写为 `profiles`
func (User) TableName() string {
  return "profiles"
}
```



<font color='blue' size='4'>**临时指定表名：**</font>

**可以使用 `Table` 方法临时指定表名，例如：**

```go
// 根据 User 的字段创建 `deleted_users` 表
db.Table("deleted_users").AutoMigrate(&User{})

// 从另一张表查询数据
var deletedUsers []User
db.Table("deleted_users").Find(&deletedUsers)
// SELECT * FROM deleted_users;

db.Table("deleted_users").Where("name = ?", "jinzhu").Delete(&User{})
// DELETE FROM deleted_users WHERE name = 'jinzhu';
```

<font color='blue' size='4'>**数据表前缀：**</font>

可以重写`gorm.DefaultTableNameHandler`这个变量，这样可以为所有数据表指定统一的数据表前缀，如下：

```go
gorm.DefaultTableNameHandler = func (db *gorm.DB, defaultTableName string) string  {
    return "tb_" + defaultTableName;
}
```

这样的话，通过结构体`Post`创建的数据表名称则为`tb_posts`。

### 字段映射规则

**结构体的字段到数据表字段的默认映射规则是用下划线分隔每个大写字母开头的单词，如下：**

```go
type Prize struct {
	ID int
	PrizeName string
}
```

上面的结构体`Prize`中的`PrizeName`字段对应的数据表为`prize_name`。

解决：可以为结构体的某个字段定义`tags`扩展信息,这样结构体字段到数据表字段的映规则就在`tags`中定义。



### 时间点追踪

**如果结构体中有`CreatedAt`,`UpdatedAt`,`DeletedAt`这几个字段的话，则`GROM`框架也会作一些特殊处理**

<font color='blue' size='4'>**CreatedAt：**</font>

如果模型有 `CreatedAt`字段，该字段的值将会是初次创建记录的时间。

```go
db.Create(&user) // `CreatedAt`将会是当前时间

// 可以使用`Update`方法来改变`CreateAt`的值
db.Model(&user).Update("CreatedAt", time.Now())
```

<font color='blue' size='4'>**UpdatedAt：**</font>

如果模型有`UpdatedAt`字段，该字段的值将会是每次更新记录的时间。

```go
db.Save(&user) // `UpdatedAt`将会是当前时间

db.Model(&user).Update("name", "jinzhu") // `UpdatedAt`将会是当前时间
```

<font color='blue' size='4'>**DeletedAt：**</font>

如果模型有`DeletedAt`字段，调用`Delete`删除该记录时，将会设置`DeletedAt`字段为当前时间，而不是直接将记录从数据库中删除。



代码示例：

```go
type User struct {
	ID        uint      // 列名是 `id`
	Name      string    // 列名是 `name`
	Hobby     string    // 列名是 `hobby`
	CreatedAt time.Time // 列名是 `created_at`
	UpdatedAt time.Time // 列名是 `updated_at`
	DeletedAt time.Time // 列名是 `deleted_at`
}

func main() {
	db, err := gorm.Open("mysql", "root:root@(127.0.0.1:3306)/db1?charset=utf8mb4&parseTime=True&loc=Local")
	if err != nil {
		panic(err)
	}
	defer db.Close()
	fmt.Println("连接成功")
	// 创建表，自动迁移（把结构体和数据表进行对应）
	db.AutoMigrate(&User{})

	// 创建数据行
	u := User{Name: "Tom", Hobby: "足球"}
	db.Debug().Create(&u)

	// 查询
	db.Debug().First(&u) //查询表中第一条数据保存到u中
	fmt.Println("查询到的数据是：", u)

	// 更新
	db.Debug().Model(&u).Update("hobby", "双色球")
	fmt.Println("更新后的数据：", u)

	// 删除，模型有DeletedAt字段，调用Delete删除该记录时，将会设置DeletedAt字段为当前时间
	db.Debug().Delete(&u)
}
```

执行结果：

![image-20220329180426317](E:\个人\学习\笔记\Go语言\GORM.assets\image-20220329180426317.png)



## gorm.Model定义

**为了方便模型定义，GORM内置了一个`gorm.Model`结构体。`gorm.Model`是一个包含了`ID`, `CreatedAt`, `UpdatedAt`, `DeletedAt`四个字段的Golang结构体。**

```go
// gorm.Model 定义
type Model struct {
    ID        uint `gorm:"primary_key"`
    CreatedAt time.Time
    UpdatedAt time.Time
    DeletedAt *time.Time `sql:"index"`
}
```

## 嵌入结构体

* **对于匿名字段，GORM 会将其字段包含在父结构体中，例如：**

  ```go
  type User struct {
    gorm.Model
    Name string
  }
  // 等效于
  type User struct {
    ID        uint           `gorm:"primaryKey"`
    CreatedAt time.Time
    UpdatedAt time.Time
    DeletedAt gorm.DeletedAt `gorm:"index"`
    Name string
  }
  ```

* **对于正常的结构体字段，你也可以通过标签 `embedded` 将其嵌入，例如：**

  ```go
  type Author struct {
      Name  string
      Email string
  }
  
  type Blog struct {
    ID      int
    Author  Author `gorm:"embedded"`
    Upvotes int32
  }
  // 等效于
  type Blog struct {
    ID    int64
    Name  string
    Email string
    Upvotes  int32
  }
  ```

  > **注意：如果不使用`gorm:"embedded"`字段则，生成数据库的时候没有Author中的字段**

* 并且，您可以使用标签 `embeddedPrefix` 来为 db 中的字段名添加前缀，例如：

  ```go
  type Blog struct {
    ID      int
    Author  Author `gorm:"embedded;embeddedPrefix:author_"`
    Upvotes int32
  }
  // 等效于
  type Blog struct {
    ID          int64
      AuthorName  string
      AuthorEmail string
    Upvotes     int32
  }
  ```

# 数据库迁移

**我们这里所说的数据库迁移，即通过使用`GROM`提供的一系列方法，根据数据模型定义好的规则，进行创建、删除数据表等操作，也就是数据库的DDL操作。**

<font color='blue' size='4'>**数据表操作：**</font>

```go
//根据模型自动创建数据表
func (s *DB) AutoMigrate(values ...interface{}) *DB 

//根据模型创建数据表
func (s *DB) CreateTable(models ...interface{}) *DB

//删除数据表，相当于drop table语句
func (s *DB) DropTable(values ...interface{}) *DB 

//相当于drop table if exsist 语句
func (s *DB) DropTableIfExists(values ...interface{}) *DB 

//根据模型判断数据表是否存在
func (s *DB) HasTable(value interface{}) bool 
```

**注意： AutoMigrate 会创建表、缺失的外键、约束、列和索引。 如果大小、精度、是否为空可以更改，则 AutoMigrate 会改变列的类型。 出于保护您数据的目的，它 不会 删除未使用的列**

<font color='blue' size='4'>**列操作：**</font>

```go
//删除数据表字段
func (s *DB) DropColumn(column string) *DB

//修改数据表字段的数据类型
func (s *DB) ModifyColumn(column string, typ string) *DB
```

<font color='blue' size='4'>**索引操作：**</font>

```go
//添加外键
func (s *DB) AddForeignKey(field string, dest string, onDelete string, onUpdate string) *DB

//给数据表字段添加索引
func (s *DB) AddIndex(indexName string, columns ...string) *DB

//给数据表字段添加唯一索引
func (s *DB) AddUniqueIndex(indexName string, columns ...string) *DB
```

**数据迁移简单代码示例：**

```go
type User struct {
    Id       int   //对应数据表的自增id
    Username string
    Password string
    Email    string
    Phone    string
}

func main(){
    db.AutoMigrate(&Post{},&User{})//创建posts和users数据表

    db.CreateTable(&Post{})//创建posts数据表
    
    db.Set("gorm:table_options", "ENGINE=InnoDB").CreateTable(&Post{})//创建posts表时指存在引擎
    
    db.DropTable(&Post{},"users")//删除posts和users表数据表
    
    db.DropTableIfExists(&Post{},"users")//删除前会判断posts和users表是否存在
    
    //先判断users表是否存在，再删除users表
    if db.HasTable("users") {
        db.DropTable("users")
    }
    
    //删除数据表字段
    db.Model(&Post{}).DropColumn("id")
    
    //修改字段数据类型
    db.Model(&Post{}).ModifyColumn("id","varchar(255)")
    
    //建立posts与users表之间的外键关联
    db.Model(&Post{}).AddForeignKey("uid", "users(id)", "RESTRICT", "RESTRICT")
    
    //给posts表的title字段添加索引
    db.Model(&Post{}).AddIndex("index_title","title")
    
    //给users表的phone字段添加唯一索引
    db.Model(&User{}).AddUniqueIndex("index_phone","phone")
}
```

# 基本操作

## 链式操作

Gorm 实现了链式操作接口，所以你可以把代码写成这样：

```go
// 创建一个查询
tx := db.Where("name = ?", "jinzhu")

// 添加更多条件
if someCondition {
  tx = tx.Where("age = ?", 20)
} else {
  tx = tx.Where("age = ?", 30)
}

if yetAnotherCondition {
  tx = tx.Where("active = ?", 1)
}
tx.First(&user)
```

在调用立即执行方法前不会生成`Query`语句，借助这个特性你可以创建一个函数来处理一些通用逻辑。

<font color='blue' size='4'>**立即执行方法：**</font>

立即执行方法是指那些会立即生成`SQL`语句并发送到数据库的方法, 他们一般是`CRUD`方法，比如：

`Create`, `First`, `Find`, `Take`, `Save`, `UpdateXXX`, `Delete`, `Scan`, `Row`, `Rows`…

* 基于上面链式方法代码的立即执行方法的例子：

  ```go
  tx.Find(&user)
  ```

* 生成的SQL语句如下：

  ```sql
  SELECT * FROM users where name = 'jinzhu' AND age = 30 AND active = 1;
  ```



<font color='blue' size='4'>**`Scopes`，Scope是建立在链式操作的基础之上的：**</font>

基于它，你可以抽取一些通用逻辑，写出更多可重用的函数库。

```go
func AmountGreaterThan1000(db *gorm.DB) *gorm.DB {
  return db.Where("amount > ?", 1000)
}

func PaidWithCreditCard(db *gorm.DB) *gorm.DB {
  return db.Where("pay_mode_sign = ?", "C")
}

func PaidWithCod(db *gorm.DB) *gorm.DB {
  return db.Where("pay_mode_sign = ?", "C")
}

func OrderStatus(status []string) func (db *gorm.DB) *gorm.DB {
  return func (db *gorm.DB) *gorm.DB {
    return db.Scopes(AmountGreaterThan1000).Where("status IN (?)", status)
  }
}

db.Scopes(AmountGreaterThan1000, PaidWithCreditCard).Find(&orders)
// 查找所有金额大于 1000 的信用卡订单

db.Scopes(AmountGreaterThan1000, PaidWithCod).Find(&orders)
// 查找所有金额大于 1000 的 COD 订单

db.Scopes(AmountGreaterThan1000, OrderStatus([]string{"paid", "shipped"})).Find(&orders)
// 查找所有金额大于 1000 且已付款或者已发货的订单
```

<font color='blue' size='4'>**多个立即执行方法：**</font>

**在 GORM 中使用多个立即执行方法时，后一个立即执行方法会复用前一个立即执行方法的条件 (不包括内联条件) 。**

```go
db.Where("name LIKE ?", "jinzhu%").Find(&users, "id IN (?)", []int{1, 2, 3}).Count(&count)
```

生成的 Sql

```sql
SELECT * FROM users WHERE name LIKE 'jinzhu%' AND id IN (1, 2, 3)

SELECT count(*) FROM users WHERE name LIKE 'jinzhu%'
```

## 创建

**使用`gorm.DB`中的`Create()`方法，`GORM`会根据传给`Create()`方法的模型，向数据表插入一行。**

```go
func (s *DB) Create(value interface{}) *DB  //创建一行
func (s *DB) NewRecord(value interface{}) bool //查询主键是否存在，主键为空返回true
```

代码示例：

```go
type User struct {
	ID        int64
	Name      string `gorm:"default:'小王子'"`
	Age       int64
	CreatedAt time.Time
}

func test01() {
	db, err := gorm.Open("mysql", "root:root@(127.0.0.1:3306)/db1?charset=utf8mb4&parseTime=True&loc=Local")
	if err != nil {
		panic(err)
	}
	defer db.Close()
	// 创建表，自动迁移（把结构体和数据表进行对应）
	db.AutoMigrate(&User{})

	// 创建数据行
	user := User{Name: "Tom", Age: 18}
	// Debug()可以查看查询信息
	db.NewRecord(user)                 // 主键为空返回`true`
	result := db.Debug().Create(&user) // 通过数据的指针来创建
	fmt.Println("插入记录的条数：", result.RowsAffected)
	fmt.Println("返回error：", result.Error)
	db.NewRecord(user) // 创建`user`后返回`false`
}
```

执行结果：

![image-20220330105349036](E:\个人\学习\笔记\Go语言\GORM.assets\image-20220330105349036.png)

<font color='blue' size='4'>**用指定的字段创建记录：**</font>

* **创建记录并更新给出的字段：**

  ```go
  db.Select("Name", "Age", "CreatedAt").Create(&user)
  // INSERT INTO `users` (`name`,`age`,`created_at`) VALUES ("jinzhu", 18, "2020-07-04 11:05:21.775")
  ```

* **创建一个记录且一同忽略传递给略去的字段值：**

  ```go
  db.Omit("Name", "Age", "CreatedAt").Create(&user)
  // INSERT INTO `users` (`birthday`,`updated_at`) VALUES ("2020-01-01 00:00:00.000", "2020-07-04 11:05:21.775")
  ```



<font color='blue' size='4'>**创建时的默认值：**</font>

可以通过 tag 定义字段的默认值，比如：

```go
type User struct {
  ID   int64
  Name string `gorm:"default:'小王子'"`
  Age  int64
}
```

**注意：通过tag定义字段的默认值，在创建记录时候生成的 SQL 语句会排除没有值或值为 零值 的字段。 在将记录插入到数据库后，Gorm会从数据库加载那些字段的默认值。**

举个例子：

```go
var user = User{Name: "", Age: 99}
db.Create(&user)
```

上面代码实际执行的SQL语句是`INSERT INTO users("age") values('99');`，排除了零值字段`Name`，而在数据库中这一条数据会使用设置的默认值`小王子`作为Name字段的值。

**注意：**所有字段的零值, 比如`0`, `""`,`false`或者其它`零值`，都不会保存到数据库内，但会使用他们的默认值。 如果你想避免这种情况，可以考虑使用指针或实现 `Scanner/Valuer`接口，比如：

**使用指针方式实现零值存入数据库：**

```go
// 使用指针
type User struct {
  ID   int64
  Name *string `gorm:"default:'小王子'"`
  Age  int64
}
user := User{Name: new(string), Age: 18))}
db.Create(&user)  // 此时数据库中该条记录name字段的值就是''
```

**使用Scanner/Valuer接口方式实现零值存入数据库：**

```go
// 使用 Scanner/Valuer
type User struct {
	ID int64
	Name sql.NullString `gorm:"default:'小王子'"` // sql.NullString 实现了Scanner/Valuer接口
	Age  int64
}
user := User{Name: sql.NullString{"", true}, Age:18}
db.Create(&user)  // 此时数据库中该条记录name字段的值就是''
```



## 查询

### 一般查询

```go
user := User{ID: 2, Name: "Tom", Age: 18}

// 根据主键查询第一条记录，如果user没有主键则没有查询条件
db.First(&user)
// SELECT * FROM `users`  WHERE `users`.`id` = 2 ORDER BY `users`.`id` ASC LIMIT 1

// 根据主键查询最后一条记录
db.Last(&user)
// SELECT * FROM `users`  WHERE `users`.`id` = 2 ORDER BY `users`.`id` DESC LIMIT 1

// 根据ID随机获取一条记录
db.Take(&user)
// SELECT * FROM `users`  WHERE `users`.`id` = 2 LIMIT 1

// 查询所有的记录
db.Find(&users)
// SELECT * FROM `users`  WHERE `users`.`id` = 2

// 查询指定的某条记录(仅当主键为整型时可用)
db.First(&user, 10)
// SELECT * FROM `users`  WHERE `users`.`id` = 2 AND ((`users`.`id` = 10)) ORDER BY `users`.`id` ASC LIMIT 1
```

**查询是添加额外查询选项：**

```go
// 为查询 SQL 添加额外的 SQL 操作
db.Set("gorm:query_option", "FOR UPDATE").First(&user, 10)
// SELECT * FROM users WHERE id = 10 FOR UPDATE;
```



### String条件查询

```go
// Get first matched record
db.Where("name = ?", "jinzhu").First(&user)
//// SELECT * FROM users WHERE name = 'jinzhu' limit 1;

// Get all matched records
db.Where("name = ?", "jinzhu").Find(&users)
//// SELECT * FROM users WHERE name = 'jinzhu';

// <>
db.Where("name <> ?", "jinzhu").Find(&users)
//// SELECT * FROM users WHERE name <> 'jinzhu';

// IN
db.Where("name IN (?)", []string{"jinzhu", "jinzhu 2"}).Find(&users)
//// SELECT * FROM users WHERE name in ('jinzhu','jinzhu 2');

// LIKE
db.Where("name LIKE ?", "%jin%").Find(&users)
//// SELECT * FROM users WHERE name LIKE '%jin%';

// AND
db.Where("name = ? AND age >= ?", "jinzhu", "22").Find(&users)
//// SELECT * FROM users WHERE name = 'jinzhu' AND age >= 22;

// Time
db.Where("updated_at > ?", lastWeek).Find(&users)
//// SELECT * FROM users WHERE updated_at > '2000-01-01 00:00:00';

// BETWEEN
db.Where("created_at BETWEEN ? AND ?", lastWeek, today).Find(&users)
//// SELECT * FROM users WHERE created_at BETWEEN '2000-01-01 00:00:00' AND '2000-01-08 00:00:00';
```

### Struct & Map查询

```go
// Struct
db.Where(&User{Name: "jinzhu", Age: 20}).First(&user)
//// SELECT * FROM users WHERE name = "jinzhu" AND age = 20 LIMIT 1;

// Map
db.Where(map[string]interface{}{"name": "jinzhu", "age": 20}).Find(&users)
//// SELECT * FROM users WHERE name = "jinzhu" AND age = 20;

// 主键的切片
db.Where([]int64{20, 21, 22}).Find(&users)
//// SELECT * FROM users WHERE id IN (20, 21, 22);
```

**提示：**当通过结构体进行查询时，GORM将会只通过非零值字段查询，这意味着如果你的字段值为`0`，`''`，`false`或者其他`零值`时，将不会被用于构建查询条件，例如：

```go
db.Where(&User{Name: "jinzhu", Age: 0}).Find(&users)
//// SELECT * FROM users WHERE name = "jinzhu";
```

你可以使用指针或实现 Scanner/Valuer 接口来避免这个问题.

```go
// 使用指针
type User struct {
  gorm.Model
  Name string
  Age  *int
}

// 使用 Scanner/Valuer
type User struct {
  gorm.Model
  Name string
  Age  sql.NullInt64  // sql.NullInt64 实现了 Scanner/Valuer 接口
```

<font color='blue' size='4'>**指定结构体查询字段：**</font>

当使用 struct 进行查询时，你可以通过向 `Where()` 传入 struct 来指定查询条件的字段、值、表名，例如：

```go
db.Where(&User{Name: "jinzhu"}, "name", "Age").Find(&users)
// SELECT * FROM users WHERE name = "jinzhu" AND age = 0;

db.Where(&User{Name: "jinzhu"}, "Age").Find(&users)
// SELECT * FROM users WHERE age = 0;
```

### Not 条件

**作用与 Where 类似的情形如下：**

```go
db.Not("name", "jinzhu").First(&user)
//// SELECT * FROM users WHERE name <> "jinzhu" LIMIT 1;

// Not In
db.Not("name", []string{"jinzhu", "jinzhu 2"}).Find(&users)
//// SELECT * FROM users WHERE name NOT IN ("jinzhu", "jinzhu 2");

// Not In slice of primary keys
db.Not([]int64{1,2,3}).First(&user)
//// SELECT * FROM users WHERE id NOT IN (1,2,3);

db.Not([]int64{}).First(&user)
//// SELECT * FROM users;

// Plain SQL
db.Not("name = ?", "jinzhu").First(&user)
//// SELECT * FROM users WHERE NOT(name = "jinzhu");

// Struct
db.Not(User{Name: "jinzhu"}).First(&user)
//// SELECT * FROM users WHERE name <> "jinzhu";
```

### Or条件

```go
db.Where("role = ?", "admin").Or("role = ?", "super_admin").Find(&users)
//// SELECT * FROM users WHERE role = 'admin' OR role = 'super_admin';

// Struct
db.Where("name = 'jinzhu'").Or(User{Name: "jinzhu 2"}).Find(&users)
//// SELECT * FROM users WHERE name = 'jinzhu' OR name = 'jinzhu 2';

// Map
db.Where("name = 'jinzhu'").Or(map[string]interface{}{"name": "jinzhu 2"}).Find(&users)
//// SELECT * FROM users WHERE name = 'jinzhu' OR name = 'jinzhu 2';
```

### 内联条件

**查询条件也可以被内联到 `First` 和 `Find` 之类的方法中，作用与`Where`查询类似，当内联条件与多个`立即执行方法`一起使用时, 内联条件不会传递给后面的立即执行方法。**

```go
// 根据主键获取记录 (只适用于整形主键)
db.First(&user, 23)
//// SELECT * FROM users WHERE id = 23 LIMIT 1;

// 根据主键获取记录, 如果它是一个非整形主键
db.First(&user, "id = ?", "string_primary_key")
//// SELECT * FROM users WHERE id = 'string_primary_key' LIMIT 1;

// Plain SQL
db.Find(&user, "name = ?", "jinzhu")
//// SELECT * FROM users WHERE name = "jinzhu";

db.Find(&users, "name <> ? AND age > ?", "jinzhu", 20)
//// SELECT * FROM users WHERE name <> "jinzhu" AND age > 20;

// Struct
db.Find(&users, User{Age: 20})
//// SELECT * FROM users WHERE age = 20;

// Map
db.Find(&users, map[string]interface{}{"age": 20})
//// SELECT * FROM users WHERE age = 20;
```



### 选择特定字段

`Select` 允许您指定从数据库中检索哪些字段

```go
db.Select("name", "age").Find(&users)
// SELECT name, age FROM users;

db.Select([]string{"name", "age"}).Find(&users)
// SELECT name, age FROM users;

db.Table("users").Select("COALESCE(age,?)", 42).Rows()
// SELECT COALESCE(age,'42') FROM users;
```



### Order

指定从数据库检索记录时的排序方式

```go
db.Order("age desc, name").Find(&users)
//// SELECT * FROM users ORDER BY age desc, name;

// 多字段排序
db.Order("age desc").Order("name").Find(&users)
//// SELECT * FROM users ORDER BY age desc, name;

// 覆盖排序
db.Order("age desc").Find(&users1).Order("age", true).Find(&users2)
//// SELECT * FROM users ORDER BY age desc; (users1)
//// SELECT * FROM users ORDER BY age; (users2)
```

### Limit & Offset

* `Limit` 指定获取记录的最大数量
*  `Offset` 指定在开始返回记录之前要跳过的记录数量

```go
db.Limit(3).Find(&users)
// SELECT * FROM users LIMIT 3;

// 通过 -1 消除 Limit 条件
db.Limit(10).Find(&users1).Limit(-1).Find(&users2)
// SELECT * FROM users LIMIT 10; (users1)
// SELECT * FROM users; (users2)

db.Offset(3).Find(&users)
// SELECT * FROM users OFFSET 3;

db.Limit(10).Offset(5).Find(&users)
// SELECT * FROM users OFFSET 5 LIMIT 10;

// 通过 -1 消除 Offset 条件
db.Offset(10).Find(&users1).Offset(-1).Find(&users2)
// SELECT * FROM users OFFSET 10; (users1)
// SELECT * FROM users; (users2)
```

### 总数

Count，该 model 能获取的记录总数。

```go
db.Where("name = ?", "jinzhu").Or("name = ?", "jinzhu 2").Find(&users).Count(&count)
//// SELECT * from USERS WHERE name = 'jinzhu' OR name = 'jinzhu 2'; (users)
//// SELECT count(*) FROM users WHERE name = 'jinzhu' OR name = 'jinzhu 2'; (count)

db.Model(&User{}).Where("name = ?", "jinzhu").Count(&count)
//// SELECT count(*) FROM users WHERE name = 'jinzhu'; (count)

db.Table("deleted_users").Count(&count)
//// SELECT count(*) FROM deleted_users;

db.Table("deleted_users").Select("count(distinct(name))").Count(&count)
//// SELECT count( distinct(name) ) FROM deleted_users; (count)
```

**注意** `Count` 必须是链式查询的最后一个操作 ，因为它会覆盖前面的 `SELECT`，但如果里面使用了 `count` 时不会覆盖

### Group By & Having

```go
type result struct {
  Date  time.Time
  Total int
}

db.Model(&User{}).Select("name, sum(age) as total").Where("name LIKE ?", "group%").Group("name").First(&result)
// SELECT name, sum(age) as total FROM `users` WHERE name LIKE "group%" GROUP BY `name` LIMIT 1



db.Model(&User{}).Select("name, sum(age) as total").Group("name").Having("name = ?", "group").Find(&result)
// SELECT name, sum(age) as total FROM `users` GROUP BY `name` HAVING name = "group"

rows, err := db.Table("orders").Select("date(created_at) as date, sum(amount) as total").Group("date(created_at)").Rows()
defer rows.Close()
for rows.Next() {
  ...
}

rows, err := db.Table("orders").Select("date(created_at) as date, sum(amount) as total").Group("date(created_at)").Having("sum(amount) > ?", 100).Rows()
defer rows.Close()
for rows.Next() {
  ...
}

type Result struct {
  Date  time.Time
  Total int64
}
db.Table("orders").Select("date(created_at) as date, sum(amount) as total").Group("date(created_at)").Having("sum(amount) > ?", 100).Scan(&results)
```

### Joins

Joins，指定连接条件

```go
type result struct {
  Name  string
  Email string
}

db.Model(&User{}).Select("users.name, emails.email").Joins("left join emails on emails.user_id = users.id").Scan(&result{})
// SELECT users.name, emails.email FROM `users` left join emails on emails.user_id = users.id

rows, err := db.Table("users").Select("users.name, emails.email").Joins("left join emails on emails.user_id = users.id").Rows()
for rows.Next() {
  ...
}

db.Table("users").Select("users.name, emails.email").Joins("left join emails on emails.user_id = users.id").Scan(&results)

// 带参数的多表连接
db.Joins("JOIN emails ON emails.user_id = users.id AND emails.email = ?", "jinzhu@example.org").Joins("JOIN credit_cards ON credit_cards.user_id = users.id").Where("credit_cards.number = ?", "411111111111").Find(&user)
```



### 子查询

**子查询可以嵌套在查询中，GORM 允许在使用 `*gorm.DB` 对象作为参数时生成子查询：**

```go
db.Where("amount > (?)", db.Table("orders").Select("AVG(amount)")).Find(&orders)
// SELECT * FROM "orders" WHERE amount > (SELECT AVG(amount) FROM "orders");

subQuery := db.Select("AVG(age)").Where("name LIKE ?", "name%").Table("users")
db.Select("AVG(age) as avgage").Group("name").Having("AVG(age) > (?)", subQuery).Find(&results)
// SELECT AVG(age) as avgage FROM `users` GROUP BY `name` HAVING AVG(age) > (SELECT AVG(age) FROM `users` WHERE name LIKE "name%")
```

**GORM 允许您在 `Table` 方法中通过 FROM 子句使用子查询，例如：**

```go
db.Table("(?) as u", db.Model(&User{}).Select("name", "age")).Where("age = ?", 18}).Find(&User{})
// SELECT * FROM (SELECT `name`,`age` FROM `users`) as u WHERE `age` = 18

subQuery1 := db.Model(&User{}).Select("name")
subQuery2 := db.Model(&Pet{}).Select("name")
db.Table("(?) as u, (?) as p", subQuery1, subQuery2).Find(&User{})
// SELECT * FROM (SELECT `name` FROM `users`) as u, (SELECT `name` FROM `pets`) as p
```





### Scan语句

**将结果扫描到另一个结构中：**

```go
type Result struct {
    Name string
    Age  int
}

var result Result
db.Table("users").Select("name, age").Where("name = ?", 3).Scan(&result)

// Raw SQL
db.Raw("SELECT name, age FROM users WHERE name = ?", 3).Scan(&result)
```





## 更新

**更新数据可以使用`gorm.DB`的`Save()`或`Update()`,`UpdateColumn()`,`UpdateColumns()`,`Updates()`等方法，后面这四个方法需要与`Model()`方法一起使用。**

```go
//更新所有字段
func (s *DB) Save(value interface{}) *DB

func (s *DB) Model(value interface{}) *DB
//下面的方法需要与Model方法一起使用，通过Model方法指定更新数据的条件
//更新修改字段
func (s *DB) Update(attrs ...interface{}) *DB
func (s *DB) Updates(values interface{}, ignoreProtectedAttrs ...bool) *DB
//无Hooks更新
func (s *DB) UpdateColumn(attrs ...interface{}) *DB
func (s *DB) UpdateColumns(values interface{}) *DB
```

### 更新所有字段

`Save()`默认会更新该对象的所有字段，即使你没有赋值。

```go
db.First(&user)

user.Name = "七米"
user.Age = 99
db.Save(&user)

//  UPDATE `users` SET `created_at` = '2020-02-16 12:52:20', `updated_at` = '2020-02-16 12:54:55', `deleted_at` = NULL, `name` = '七米', `age` = 99, `active` = true  WHERE `users`.`deleted_at` IS NULL AND `users`.`id` = 1
```

### 更新修改字段

如果你只希望更新指定字段，可以使用`Update`或者`Updates`

```go
// 更新单个属性，如果它有变化
db.Model(&user).Update("name", "hello")
//// UPDATE users SET name='hello', updated_at='2013-11-17 21:34:10' WHERE id=111;

// 根据给定的条件更新单个属性
db.Model(&user).Where("active = ?", true).Update("name", "hello")
//// UPDATE users SET name='hello', updated_at='2013-11-17 21:34:10' WHERE id=111 AND active=true;

// 使用 map 更新多个属性，只会更新其中有变化的属性
db.Model(&user).Updates(map[string]interface{}{"name": "hello", "age": 18, "active": false})
//// UPDATE users SET name='hello', age=18, active=false, updated_at='2013-11-17 21:34:10' WHERE id=111;

// 使用 struct 更新多个属性，只会更新其中有变化且为非零值的字段
db.Model(&user).Updates(User{Name: "hello", Age: 18})
//// UPDATE users SET name='hello', age=18, updated_at = '2013-11-17 21:34:10' WHERE id = 111;

// 警告：当使用 struct 更新时，GORM只会更新那些非零值的字段
// 对于下面的操作，不会发生任何更新，"", 0, false 都是其类型的零值
db.Model(&user).Updates(User{Name: "", Age: 0, Active: false})
```

### 更新选定字段

如果你想更新或忽略某些字段，你可以使用 `Select`，`Omit`

```go
db.Model(&user).Select("name").Updates(map[string]interface{}{"name": "hello", "age": 18, "active": false})
//// UPDATE users SET name='hello', updated_at='2013-11-17 21:34:10' WHERE id=111;

db.Model(&user).Omit("name").Updates(map[string]interface{}{"name": "hello", "age": 18, "active": false})
//// UPDATE users SET age=18, active=false, updated_at='2013-11-17 21:34:10' WHERE id=111;
```

### 无Hooks更新

上面的更新操作会自动运行 model 的 `BeforeUpdate`, `AfterUpdate` 方法，更新 `UpdatedAt` 时间戳, 在更新时保存其 `Associations`, 如果你不想调用这些方法，你可以使用 `UpdateColumn`， `UpdateColumns`

```go
// 更新单个属性，类似于 `Update`
db.Model(&user).UpdateColumn("name", "hello")
//// UPDATE users SET name='hello' WHERE id = 111;

// 更新多个属性，类似于 `Updates`
db.Model(&user).UpdateColumns(User{Name: "hello", Age: 18})
//// UPDATE users SET name='hello', age=18 WHERE id = 111;
```

### 批量更新

批量更新时`Hooks（钩子函数）`不会运行。

```go
db.Table("users").Where("id IN (?)", []int{10, 11}).Updates(map[string]interface{}{"name": "hello", "age": 18})
//// UPDATE users SET name='hello', age=18 WHERE id IN (10, 11);

// 使用 struct 更新时，只会更新非零值字段，若想更新所有字段，请使用map[string]interface{}
db.Model(User{}).Updates(User{Name: "hello", Age: 18})
//// UPDATE users SET name='hello', age=18;

// 使用 `RowsAffected` 获取更新记录总数
db.Model(User{}).Updates(User{Name: "hello", Age: 18}).RowsAffected
```

### 使用SQL表达式更新

先查询表中的第一条数据保存至user变量。

```go
var user User
db.First(&user)
db.Model(&user).Update("age", gorm.Expr("age * ? + ?", 2, 100))
//// UPDATE `users` SET `age` = age * 2 + 100, `updated_at` = '2020-02-16 13:10:20'  WHERE `users`.`id` = 1;

db.Model(&user).Updates(map[string]interface{}{"age": gorm.Expr("age * ? + ?", 2, 100)})
//// UPDATE "users" SET "age" = age * '2' + '100', "updated_at" = '2020-02-16 13:05:51' WHERE `users`.`id` = 1;

db.Model(&user).UpdateColumn("age", gorm.Expr("age - ?", 1))
//// UPDATE "users" SET "age" = age - 1 WHERE "id" = '1';

db.Model(&user).Where("age > 10").UpdateColumn("age", gorm.Expr("age - ?", 1))
//// UPDATE "users" SET "age" = age - 1 WHERE "id" = '1' AND quantity > 10;
```

### 修改Hooks中的值

如果你想修改 `BeforeUpdate`, `BeforeSave` 等 Hooks 中更新的值，你可以使用 `scope.SetColumn`, 例如：

```go
func (user *User) BeforeSave(scope *gorm.Scope) (err error) {
  if pw, err := bcrypt.GenerateFromPassword(user.Password, 0); err == nil {
    scope.SetColumn("EncryptedPassword", pw)
  }
}
```

## 删除

**使用`gorm.DB`的`Delete()`方法可以很简单地删除满足条件的记录，下面是`Delete()`方法的定义：**

```go
//value如果有主键id，则包含在判断条件内，通过where可以指定其他条件
func (s *DB) Delete(value interface{}, where ...interface{}) *DB
```

**警告：删除记录时，请确保主键字段有值，GORM 会通过主键去删除记录，如果主键为空，GORM 会删除该 model 的所有记录。**

```go
// 删除现有记录
db.Delete(&email)
// DELETE from emails where id=10;
```

<font color='blue' size='4'>**批量删除：**</font>

* 删除全部匹配的记录

  ```go
  db.Where("email LIKE ?", "%jinzhu%").Delete(Email{})
  //// DELETE from emails where email LIKE "%jinzhu%";
  
  db.Delete(Email{}, "email LIKE ?", "%jinzhu%")
  //// DELETE from emails where email LIKE "%jinzhu%";
  ```

<font color='blue' size='4'>**软删除：**</font>

* **如果一个 model 有 `DeletedAt` 字段，他将自动获得软删除的功能！ 当调用 `Delete` 方法时， 记录不会真正的从数据库中被删除， 只会将`DeletedAt` 字段的值会被设置为当前时间**

  ```go
  db.Delete(&user)
  //// UPDATE users SET deleted_at="2013-10-29 10:23" WHERE id = 111;
  
  // 批量删除
  db.Where("age = ?", 20).Delete(&User{})
  //// UPDATE users SET deleted_at="2013-10-29 10:23" WHERE age = 20;
  
  // 查询记录时会忽略被软删除的记录
  db.Where("age = 20").Find(&user)
  //// SELECT * FROM users WHERE age = 20 AND deleted_at IS NULL;
  
  // Unscoped 方法可以查询被软删除的记录
  db.Unscoped().Where("age = 20").Find(&users)
  //// SELECT * FROM users WHERE age = 20;
  ```

<font color='blue'>**物理删除：**</font>

```go
// Unscoped 方法可以物理删除记录
db.Unscoped().Delete(&order)
//// DELETE FROM orders WHERE id=10;
```



## 原生SQL

原生查询 SQL 和 `Scan`

```go
type Result struct {
  ID   int
  Name string
  Age  int
}

var result Result
db.Raw("SELECT id, name, age FROM users WHERE name = ?", 3).Scan(&result)

db.Raw("SELECT id, name, age FROM users WHERE name = ?", 3).Scan(&result)

var age int
db.Raw("SELECT SUM(age) FROM users WHERE role = ?", "admin").Scan(&age)

var users []User
db.Raw("UPDATE users SET name = ? WHERE age = ? RETURNING id, name", "jinzhu", 20).Scan(&users)
```

`Exec` 原生 SQL

```go
db.Exec("DROP TABLE users")
db.Exec("UPDATE orders SET shipped_at = ? WHERE id IN ?", time.Now(), []int64{1, 2, 3})

// Exec with SQL Expression
db.Exec("UPDATE users SET money = ? WHERE name = ?", gorm.Expr("money * ? + ?", 10000, 1), "jinzhu")
```





## 关联查询

[GORM 关联查询 - Go语言中文网 - Golang中文社区 (studygolang.com)](https://studygolang.com/articles/19720)

### Belongs to

**`belongs to` 会与另一个模型建立了一对一的连接。 这种模型的每一个实例都“属于”另一个模型的一个实例。**

* **默认情况下，外键的名字，使用拥有者的类型名称加上表的主键的字段名字。也可以使用`foreignKey`自定义外键名字**

```go
// ArticleCate foreignKey外键  如果是表名称加上Id的话默认也可以不配置，如果不是，需要通过foreignKey配置外键
type Article struct {
	Id    int
	Title string
	// 默认使用ArticleCateId作为外键
	CateId      int //外键
	State       int
	ArticleCate ArticleCate `gorm:"foreignKey:CateId"`
}

// 文章分类
type ArticleCate struct {
	Id    int //主键
	Title string
	State int
}
```

**Belongs to的查询：**

```go
func main() {
	// 初始化配置
	initGorm()
	getArticleList()
}

// getArticleList 查询所有文章以及文章对应的分类信息
func getArticleList() {
	var articleList []Article
	db.Debug().Preload("ArticleCate").Limit(3).Find(&articleList)
	for i := 0; i < len(articleList); i++ {
		fmt.Println(articleList[i])
	}
}
```

**执行结果：**

![image-20220428133932794](https://blog.zhaobincode.cn/blogimages/202204281339963.png)

### has one

**`has one` 与另一个模型建立一对一的关联，但它和一对一关系有些许不同。 这种关联表明一个模型的每个实例都包含或拥有另一个模型的一个实例。**

**`foreignkey `指定当前表的外键、`references `指定关联表中和外键关联的字段**

例如，您的应用包含 user 和 credit card 模型，且每个 user 只能有一张 credit card。

```go
type User struct {
  gorm.Model
  Name       string     `gorm:"index"`
  CreditCard CreditCard `gorm:"foreignkey:UserName;references:name"`
}

type CreditCard struct {
  gorm.Model
  Number   string
  UserName string
}
```



### Has Many

**`has many` 与另一个模型建立了一对多的连接。 不同于 `has one`，拥有者可以有零或多个关联模型，使用基本和`has one`相同。**

* **默认情况下，外键的名字，使用拥有者的类型名称加上表的主键的字段名字。也可以使用`foreignKey`自定义外键名字**
* **`references `指定关联表中和外键关联的字段，默认就是Id**

```go
type Article struct {
	Id    int
	Title string
	// 默认使用ArticleCateId作为外键
	CateId int //外键
	State  int
}

//ArticleCate foreignKey外键  如果是表名称加上Id的话默认也可以不配置，如果不是我们需要通过foreignKey配置外键
//references表示的是主键    默认就是Id   如果是Id的话可以不配置
type ArticleCate struct {
	Id      int //主键
	Title   string
	State   int
	Article []Article `gorm:"foreignKey:CateId;references:Id"`
}
```

**Has Many的查询：**

```go
func main() {
	// 初始化配置
	initGorm()
	getArticleCateList()
}

// 查找所有分类以及分类下面的文章信息
func getArticleCateList() {
	var articleCateList []ArticleCate
	db.Debug().Preload("Article").Limit(2).Find(&articleCateList)
	for i := 0; i < len(articleCateList); i++ {
		fmt.Println(articleCateList[i])
	}
}

```

**执行结果：**

![image-20220429114103944](https://blog.zhaobincode.cn/blogimages/202204291141114.png)



### Many To Many

**Many to Many 会在两个 model 中添加一张连接表**

**重写外键需要使用：`foreignKey`、`references`、`joinforeignKey`、`joinReferences`**

```go
type Lesson struct {
	Id      int       `json:"id"`
	Name    string    `json:"name"`
	Student []Student `gorm:"many2many:lesson_student;"`
}

type Student struct {
	Id       int
	Number   string
	Password string
	ClassId  int
	Name     string
	Lesson   []Lesson `gorm:"many2many:lesson_student;"`
}

// LessonStudent 连接表
type LessonStudent struct {
	LessonId  int `json:"lesson_id"`
	StudentId int `json:"student_id"`
}
```

**Many To Many查询：**

```go
func main() {
	// 初始化配置，其中禁用默认表名的复数形式
	initGorm()

	getStudentList()
	getLessonList()
}

// 查询张三，以及张三选修了哪些课程
func getStudentList() {
	var studentList []Student
	db.Debug().Preload("Lesson").Where("name = ?", "张三").Find(&studentList)
	for i := 0; i < len(studentList); i++ {
		fmt.Println(studentList[i])
	}
}

// 查询计算机网络被哪些学生选修了
func getLessonList() {
	var lessonList []Lesson
	db.Debug().Preload("Student").Where("name=?", "计算机网络").Find(&lessonList)
	for i := 0; i < len(lessonList); i++ {
		fmt.Println(lessonList[i])
	}
}
```

执行结果：



![image-20220429130132093](https://blog.zhaobincode.cn/blogimages/202204291301291.png)

### 预加载

* **带条件的预加载**

  ```go
  func main() {
  	// 初始化配置，其中禁用默认表名的复数形式
  	initGorm()
      查询课程被哪些学生选修的时候去掉张三
  	lessonList := Lesson{}
  	db.Debug().Preload("Student", "id != ?", 1).Find(&lessonList) // student id为1表示张三
      
  	fmt.Println(lessonList)
  }
  ```

  执行结果：

  ![image-20220429131002121](https://blog.zhaobincode.cn/blogimages/202204291310232.png)

* **自定义预加载SQL**

  ```go
  func main() {
  	// 初始化配置，其中禁用默认表名的复数形式
  	initGorm()
      
  	// 查看计算机网络课程被哪些学生选修 要求：学生 id 倒叙输出(自定义预加载 SQL)
  	var lessonList []Lesson
  	db.Debug().Preload("Student", func(db *gorm.DB) *gorm.DB {
  		return db.Where("id>2").Order("student.id DESC")
  	}).Where("name=?", "计算机网络").Find(&lessonList)
      
  	fmt.Println(lessonList)
  }
  ```

  执行结果：

  ![image-20220429130706989](https://blog.zhaobincode.cn/blogimages/202204291307102.png)





# 事务

## 自动事务处理

通过 db.Transaction 函数实现事务，如果闭包函数返回错误，则回滚事务。

```go
db.Transaction(func(tx *gorm.DB) error {
  // 在事务中执行一些 db 操作（从这里开始，您应该使用 'tx' 而不是 'db'）
  if err := tx.Create(&Animal{Name: "Giraffe"}).Error; err != nil {
    // 返回任何错误都会回滚事务
    return err
  }

  if err := tx.Create(&Animal{Name: "Lion"}).Error; err != nil {
    return err
  }

  // 返回 nil 提交事务
  return nil
})
```

## 手动事务处理

在开发中经常需要数据库事务来保证多个数据库写操作的原子性。例如电商系统中的扣减库存和保存订单。

```go
// 开启事务
tx := db.Begin()

// 在事务中执行数据库操作，使用的是tx变量，不是db。

// 库存减一
// 等价于: UPDATE `foods` SET `stock` = stock - 1  WHERE `foods`.`id` = '2' and stock > 0
// RowsAffected用于返回sql执行后影响的行数
rowsAffected := tx.Model(&food).Where("stock > 0").Update("stock", gorm.Expr("stock - 1")).RowsAffected
if rowsAffected == 0 {
    // 如果更新库存操作，返回影响行数为0，说明没有库存了，结束下单流程
    // 这里回滚作用不大，因为前面没成功执行什么数据库更新操作，也没什么数据需要回滚。
    // 这里就是举个例子，事务中可以执行多个sql语句，错误了可以回滚事务
    tx.Rollback()
    return
}
err := tx.Create(保存订单).Error

// 保存订单失败，则回滚事务
if err != nil {
    tx.Rollback()
} else {
    tx.Commit()
}
```



## 禁用默认事务

为了确保数据一致性，GORM 会在事务里执行写入操作（创建、更新、删除）。如果没有这方面的要求，您可以在初始化时禁用它，这将获得大约 30%+ 性能提升。

```go
// 全局禁用
db, err := gorm.Open(sqlite.Open("gorm.db"), &gorm.Config{
  SkipDefaultTransaction: true,
})

// 持续会话模式
tx := db.Session(&Session{SkipDefaultTransaction: true})
tx.First(&user, 1)
tx.Find(&users)
tx.Model(&user).Update("Age", 18)
```





> 参考博客：
>
> [Golang数据库编程之GORM模型定义与数据库迁移](https://juejin.cn/post/6844903854006337543#heading-3)
>
> [GORM官方中文文档](https://gorm.io/zh_CN/docs/index.html)
>
> [GORM CRUD指南](https://www.liwenzhou.com/posts/Go/gorm_crud/#autoid-1-5-2)

